+++
title = "My Experienc About Kubernetes CNI Plugin Migration"
date = 2026-05-02
draft = false
categories = ["Kubernetes", "Cilium"]
+++

{{<notice tip>}}
TL;DR: If you're a developer looking for alternatives and migration plans for CNI plugin on your Kubernetes cluster, Cilium is a good choise which supports live-migration from almost any CNIs, NetworkPolicies and comprehensive observability ecosystems.
{{</notice>}}

CNI plugin migration for a running Kubernetes cluster, especially in production environment, is never a trivial task. One simple strategy is [Blue/Green deployment](https://www.redhat.com/en/topics/devops/what-is-blue-green-deployment), which provisions a new cluster with target CNI and moving workloads from current cluster to the new one. This approach ensures the least downtime(still need extra setup, e.g., loadbalancer, to ensure availability) and ignores discrepancy between CNIs. However, what if in-place migration is necessary? This is not common but challenging senario, and mostly needed when operating a cluster with stateful or unmanaged workloads. For instance, a platform provides computing resource by Kubernetes but operators have no control to services deployed on it. If operators want to migrate CNI without letting users aware/involve, in-place migration will still be required.

In-place can be categorized into 2 different types, replacement and live-migration. Replacement means removing previous CNI and install a new one, which means network connection disruption between removal/installation and unpredictable results; by contrary, live-migration keeps network connectivity during migration to make lease impacts to workloads. Most modern CNI plugins are without or with limited support of live-migration, e.g., [OVN-Kubernetes](https://ovn-kubernetes.io/), [Kube-Router](https://www.kube-router.io/) and [Flannel](https://github.com/flannel-io/flannel). Though some CNIs support live-migration, they work under specific conditions, e.g., [Calico](https://docs.tigera.io/calico/latest/about/) supports [migrating from Flannel and Canal](https://docs.tigera.io/calico/latest/getting-started/kubernetes/flannel/migration-from-flannel) but not from other CNI plugins. The lack of support of live-migration comes from the design of CNI plugin. The [CNI specification](https://www.cni.dev/docs/spec/) defines how CNI plugins chosen and operations should be supported by container runtime, but doesn't provide interfaces to support connectivity for Pods running on different CNI plugins. Some users might consider to use [Multus](https://github.com/k8snetworkplumbingwg/multus-cni). This CNI plugin can run as an extra layer between different CNIs, which makes supporting live-migration become possible. However, in my experience, mixing network interfaces of difference CNIs and related IP table rules don't always work correctly (depends on target CNI plugins), which makes Multus not a reliable solution.

Currently, [Cilium](https://cilium.io/) looks the only one [officially supports]((https://docs.cilium.io/en/stable/installation/k8s-install-migration/)) live-migration from (possibly) any CNI plugins. It supports migration mode which allows Pods/Services running under different CNI can still communicate with each other. 

![cilium bridge](images/cilium-bridge.avif)

Under the workflow, users can migrate one node a time to Cilium and won't lose connectivity with workloads on it until the whole cluster migrated. Unlike Multus, no additional network interfaces will be created. It means Pods don't need to handle network interface selection by themselves, which avoid possible additional route table/IP table/DNS setup in them. More details can be found in [this article](https://isovalent.com/blog/post/tutorial-migrating-to-cilium-part-1/).

# My Scenario: Leaving Calico

My use case is migrating a cluster with Calico (precisely, Flannel + Canal) to other CNI plugins. The reason was Calico's binary didn't meet some security compliance, so I needed to find other solutions satisfy that and support live-migration.

### Setup

In my tests, I used:
- [Kind](https://kind.sigs.k8s.io/) with Canal + Flannel to simulate source cluster
- [Goldpinger](https://github.com/bloomberg/goldpinger) to monitor connectivity between Pods.

### Calico -> Multus -> Any Other CNIs

This was the first solution I tested, which I didn't successfully make Calico work with any other CNIs via Multus successfully. For instance, Flannel, loopback or macvlan. I also tested other combinations without Calico like Flannel + Flannel and macvlan + macvlan, but still unable to make Pods communicate with each other via extra IP from secondary CNI. Under the situation, it could cause more problems for DNS resolution, e.g., connection between Pods to Services via DNS. There could be some mis-configuration, but I decided no to spend more time and effort on it as the result could be hacky and fragile.

### Calico -> Flannel

This the simplest and workable solution. All I needed to do was removing Canal containers from all Canal Pods, which made Flannel as the only containers. However, as Fannel doesn't support Network Policies and advanced observability, I still moved forward to other CNIs.

### Calico -> Cilium

To migrate from Calico to Cilium, I simply followed the official document and things worked like a charm. No network connectivity affected during migration, and I didn't need to worry about multiple network interface and DNS resolution issues. The difficult part was Network Policies. Originally, clusters I worked on used [Calico GlobalNetworkPolicy](https://docs.tigera.io/calico/latest/reference/resources/globalnetworkpolicy). Thought Cilium supports [CiliumClusterwideNetworkPolicy](https://docs.cilium.io/en/latest/network/kubernetes/policy/#ciliumclusterwidenetworkpolicy), as lack of documentation, it took me some time to trace source code about supported selectors and syntax rewrite them. AI tools helped, but as it seemed not too much developers did that, non-existed syntax and selectors were included in results which requires manual fixes. Also, few things need be aware of between CiliumClusterwideNetworkPolicy and GlobalNetworkPolicy:
1. Cilium doesn't support [logged drop](https://docs.tigera.io/calico/latest/network-policy/get-started/calico-policy/calico-network-policy#generate-logs-for-specific-traffic). To monitor dropped traffic, [Cilium relies on Hubble to capture them](https://oneuptime.com/blog/post/2026-03-13-monitor-log-dropped-network-traffic-cilium/view)
2. It still necessary to write validations to ensure traffics are controlled expected. For instance, adding script to create Pods with expected selectors to ensure packets will be dropped/sent as expect in test pipelines/e2e tests.
3. By default, node selectors (`fromNode` and `toNode`) in CiliumClusterwideNetworkPolicy are not enabled. Users need to enable it via setting the `enable-node-selector-labels` in `cilium-config` ConfigMap or `nodeSelectorLabels` in [Helm config](https://docs.cilium.io/en/stable/helm-reference/) to `true`. Besides, in addition to restart the `cilium` Deployment, users also need to restart the `cilium-operator` DaemonSet to make this change take effect. Otherwise, they'll keep seeing CiliumClusterwideNetworkPolicies are marked as invalid in `kubectl get ciliumclusterwidenetworkpolicy` and `fromNode and toNode selectors not enabled` related errors from `kubectl describe ciliumclusterwidenetworkpolicy`. The cause is CiliumClusterwideNetworkPolicy is the CRD managed by `cilium-operator`.

# Further Readings

If you're interested in related topics:
- [Cilium's blog post](https://cilium.io/blog/2020/10/06/skybet-cilium-migration/)
- [Writeup from Samsung Ads Canada](https://samsungads.ca/engineering-blog/live-migrating-production-clusters-from-calico-to-cilium/)
- [Isovalent's log post](https://isovalent.com/blog/post/tutorial-migrating-to-cilium-part-1/)
