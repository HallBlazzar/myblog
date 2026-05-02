+++
title = "Kubernetes Dynamic Client 需要指定複數形式的 Resource Type "
date = 2026-05-01
draft = false
categories = ["Kubernetes", "GoLang"]
+++

在 GoLang 中，除了調用 kubectl 之外，另一種存取 Kubernetes 叢集上 CRD 的方法是 Dynamic client。它允許使用者以程式化方式存取 CRD，而無需依賴 exec 以及 kubectl，後者需要處理 shell、exit code以及 STDIN/STDOUT。然而，這個 module 的文件似乎不夠完善。使用者經常遇到的一種常見錯誤類型是與 resource type not found 相關的錯誤，這可能會由以下程式碼觸發：

```go
func GetNetworkPolicy(ctx context.Context, client dynamic.Interface, policy string) error {
	gvr := schema.GroupVersionResource{
        Group:    "cilium.io",
        Version:  "v2",
        Resource: "ciliumclusterwidenetworkpolicy",
    }

    policy, err := client.Resource(gvr).Get(ctx, policy, metav1.GetOptions{})
    if err != nil {
        return fmt.Errof("Error getting policy: %v", err)
    }
	// ... strip
}
```

這裡的問題在於資源類型。雖然我沒有發現有文件直接提到這一點，但 `GroupVersionResource` Structure 中， `Resource` 的值應該要是**複數**。因此，若要使上述程式碼正常運作，`ciliumclusterwidenetworkpolicy` 應改為 `ciliumclusterwidenetworkpolicies`。這與以 YAML 格式撰寫 K8s Object 時使用單數的情況不同。相反地，如果問題是找不到目標 Object，使用者應該會收到類似 `ciliumclusterwidenetworkpolicy not found` 的錯誤。
