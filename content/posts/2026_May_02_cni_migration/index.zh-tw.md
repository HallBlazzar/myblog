+++
title = "遷移 Kubernetes CNI Plugin 的一些經驗談"
date = 2026-05-02
draft = false
categories = ["Kubernetes", "Cilium"]
+++

{{<notice tip>}}
TL;DR：如果你正在為你的 Kubernetes 尋找替代的 CNI Plugin 與對應的遷移計劃，Cilium 是一個很好的選擇。它支援從幾乎任何 CNI 進行 Live-Migration ，並提供 Network Policy 與完整的的可觀測性生態系。
{{</notice>}}

對於正在運行的 Kubernetes 叢集（特別是在 Production 上）而言，遷移 CNI Plugin 並不是簡單的任務。 一個簡易的策略是 [藍綠部署（Blue/Green deployment）](https://www.redhat.com/en/topics/devops/what-is-blue-green-deployment)，即設定一個帶有目標 CNI 的新叢集，並將工作負載從舊叢集轉移到新叢集上。 這種方法能確保停機時間最少（仍需要額外設定，例如 Load Balancer 以確保可用性），且能忽略不同 CNI 之間的差異。 然而，如果必須進行「原地遷移」呢？ 雖然這種場景不常見且具挑戰性，但對於運行有 Stateful 或受託管的工作負載的叢集通常是必要的。 例如，平台透過 Kubernetes 提供運算資源，但營運人員無法控制部署在其中的服務。 如果營運人員希望在不讓使用者察覺或參與的情況下遷移 CNI，則仍需要原地遷移。

原地遷移可以分為兩種類型：替換與 Live-Migration 。 替換意味著移除舊的 CNI 並安裝新的，這代表在移除與安裝之間會發生網路的連接中斷，且會令遷移結果不可預測；相較之下， Live-Migration 在遷移過程能夠中保持網路連接，以將對工作負載的影響降至最低。 大多數 CNI Plugin 完全不或僅有限度地支援 Live-Migration ，例如 [OVN-Kubernetes](https://ovn-kubernetes.io/)、[Kube-Router](https://www.kube-router.io/) 和 [Flannel](https://github.com/flannel-io/flannel)。 雖然有些 CNI 支援 Live-Migration ，但僅在特定條件下運作，例如 [Calico](https://docs.tigera.io/calico/latest/about/) 支援[從 Flannel 和 Canal 遷移](https://docs.tigera.io/calico/latest/getting-started/kubernetes/flannel/migration-from-flannel)，但不支援從其他 CNI Plugin 遷移。 缺乏 Live-Migration 支援源於 CNI Plugin 的設計。 [CNI Specification](https://www.cni.dev/docs/spec/) 定義了 Container Runtime 如何選擇 CNI Plugin 及其應支援的操作，但並未提供跨不同 CNI Plugin 運行時， Pod 之間能夠保持連通性的 Interface。 有些使用者可能會考慮使用 [Multus](https://github.com/k8snetworkplumbingwg/multus-cni)。 該 Plugin 可以作為不同 CNI 之間的中介層運行，這使得支援任意 CNI Plugin 間的 Live-Migration 成為可能。 然而，根據我的經驗，混合不同 CNI 的 Network Interface 及相關的 IP table 規則時，Multus 並不總是能正常運作（取決於目標 CNI Plugin ），這使得 Multus 並非一個可靠的解決方案。

目前，[Cilium](https://cilium.io/) 似乎是唯一一個[官方支援](https://docs.cilium.io/en/stable/installation/k8s-install-migration/)（可能可以）從任何 CNI Plugin 進行 Live-Migration 的方案。 在 Cilium 支援的遷移模式下，不同 CNI 下運行的 Pod/Service 在過程中仍能互相通訊。

![cilium bridge](images/cilium-bridge.avif)

在此流程下，使用者可以每次遷移一個 Node 的 CNI Plugin 為 Cilium，且在整個叢集完成遷移前，該節點上的工作負載不會失去連結。 與 Multus 不同之處在於，它不會建立額外的 Network Interface 。 這意味著 Pod 不需要自行選擇 Network Interface，從而避免了 Pod 內部可能需要額外的 Routing Table、IP Table 或 DNS 設定。 更多細節可以在 [這篇文章](https://isovalent.com/blog/post/tutorial-migrating-to-cilium-part-1/) 中找到。

# 我的案例：自 Calico 遷出

我的使用情境是將使用 Calico（準確地說，是 Flannel + Canal）的叢集遷移到其他 CNI Plugin 。原因是 Calico 的 Binary 未符合組織內某些安全性要求，因此我需要尋找既能滿足安全性要求又支援 Live-Migration 的其他解決方案。

### 測試環境設定

在我的測試中，我使用了：
- [Kind](https://kind.sigs.k8s.io/) 搭配 Canal + Flannel 來模擬來源叢集。
- [Goldpinger](https://github.com/bloomberg/goldpinger) 用於監控 Pod 之間的連通性。

### Calico -> Multus -> 任何其他 CNI

這是我測試的第一個方案，但我未能成功讓 Calico 透過 Multus 與任何其他 CNI（例如 Flannel、loopback 或 macvlan）協作。 我也測試了沒有 Calico 的其他組合，如 Flannel + Flannel 和 macvlan + macvlan，但仍無法讓 Pod 透過次要 CNI 的額外 IP 互相通訊。 在這種情況下，可能會導致 DNS 解析出現更多問題，例如 Pod 透過 DNS 連接到 Service。 雖然可能存在一些設定錯誤，但我決定不再投入更多時間，因為結果可能既 Hacky 又不穩定。

### Calico -> Flannel

這是最簡單且可行的解決方案。 我所要做的就是從所有 Canal Pod 中移除 Canal  Container ，使 Flannel 成為唯一的 Container 。 然而，由於 Flannel 不支援 Network Policy 和額外的可觀測性，我只能繼續尋求其他方案。

### Calico -> Cilium

從 Calico 遷移到 Cilium ，只需要遵循官方文件操作就能完成。 遷移過程中不會影響網路連通性，也無需擔心多個 Network Interface 和 DNS 解析問題。 較困難的部分是 Network Policy 。 原本的叢集上使用 [Calico GlobalNetworkPolicy](https://docs.tigera.io/calico/latest/reference/resources/globalnetworkpolicy)。 雖然 Cilium 支援 [CiliumClusterwideNetworkPolicy](https://docs.cilium.io/en/latest/network/kubernetes/policy/#ciliumclusterwidenetworkpolicy)，但由於文件並不多，我依然花了不少時間追蹤 Source Code 以了解支援的 Selectors 以及重寫語法。 AI 工具雖然於過程有所幫助，但由於沒有太多開發者做過這件事，導致結果中包含了一些不存在的語法和 Selectors 需要手動修正。 此外，在 CiliumClusterwideNetworkPolicy 和 GlobalNetworkPolicy 之間還有幾件事需要注意：
1. Cilium 不支援 [Logged Drop](https://docs.tigera.io/calico/latest/network-policy/get-started/calico-policy/calico-network-policy#generate-logs-for-specific-traffic)。為了監控被丟棄的流量，[Cilium 需要額外依賴 Hubble 來進行擷取以及紀錄](https://oneuptime.com/blog/post/2026-03-13-monitor-log-dropped-network-traffic-cilium/view)。
2. 加入驗證以保證流量能按預期般受 Policy 控制。 例如，在 Test Pipeline 或 E2E Test 中添加腳本來建立具有預期 Selectors 的 Pod，以確保封包能夠如時按 Policy 所要求的被丟棄或發送。
3. 預設情況下，CiliumClusterwideNetworkPolicy 中的 Node Selector（`fromNode` 和 `toNode`）並未啟用。 使用者需要將 `cilium-config` ConfigMap 中的 `enable-node-selector-labels` 或 [Helm](https://docs.cilium.io/en/stable/helm-reference/) 中的 `nodeSelectorLabels` 設定為 `true`。 此外，除了重啟 `cilium` Deployment 外，使用者還需要重啟 `cilium-operator` DaemonSet 才能使變更真正生效。 否則，在 `kubectl get ciliumclusterwidenetworkpolicy` 中會一直看到策略被標記為無效，並在 `kubectl describe` 中看到 `fromNode and toNode selectors not enabled` 相關錯誤。這個問題起因為 CiliumClusterwideNetworkPolicy 實際上是由 `cilium-operator` 所管理的 CRD。

# 延伸閱讀

如果你對相關主題感興趣：
- [Cilium 官方部落格文章](https://cilium.io/blog/2020/10/06/skybet-cilium-migration/)
- [Samsung Ads Canada 的技術分享](https://samsungads.ca/engineering-blog/live-migrating-production-clusters-from-calico-to-cilium/)
- [Isovalent 的部落格文章](https://isovalent.com/blog/post/tutorial-migrating-to-cilium-part-1/)