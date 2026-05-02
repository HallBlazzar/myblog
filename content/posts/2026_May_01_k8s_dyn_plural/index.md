+++
title = "Kubernetes Dynamic Client Requires Plural Resource Type"
date = 2026-05-01
draft = false
categories = ["Kubernetes", "GoLang"]
+++

In GoLang, in addition to invoking `kubectl`, an alternative approach to access CRDs on a Kubernetes cluster via [Dynamic client](https://pkg.go.dev/k8s.io/client-go/dynamic#ResourceInterface). It allows users to access CRDs programmatically without `exec` module and `kubectl`, which requires handling shells, exit code and STDIN/STDOU. However, it seems the module is not well document. An general error type users encounter often is `resource type not found` related error, which could be triggered by the code:

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
	// ... snipped
}
```

The issue here is the resource type. I didn't find documents mention that directly, but value of`Resource` member in the `GroupVersionResource` structure should be plural. To make code above work correctly, `ciliumclusterwidenetworkpolicy` should be `ciliumclusterwidenetworkpolicies`. This is different from writing K8s objects in yaml format which singular is used. By contrary, if the problem is target objects not found, then users should get errors like `ciliumclusterwidenetworkpolicy not found`.
