<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# how to uninstall Istio CRDs in Minikube

To uninstall Istio Custom Resource Definitions (CRDs) and all related components from a Minikube cluster, follow these steps:

## 1. Uninstall Istio using istioctl

Run the official uninstall command which removes Istio control plane components and attempts to clean up resources:

```bash
istioctl uninstall -y --purge
```

This deletes Istio system components and attempts to remove CRDs and related resources.

## 2. Delete the Istio namespace

Remove the `istio-system` namespace where Istio control plane components run:

```bash
kubectl delete namespace istio-system
```


## 3. Delete Istio CRDs explicitly

Istio CRDs are often not deleted automatically by `istioctl uninstall`. To remove them manually, run:

```bash
kubectl delete crd $(kubectl get crd | grep 'istio.io' | awk '{print $1}')
```

This command lists all CRDs containing `istio.io` in their name and deletes them.

## 4. Remove Kubernetes Gateway API CRDs (if installed)

If you installed the Gateway API CRDs as part of Istio, remove them as well:

```bash
kubectl delete -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.3.0/standard-install.yaml
```

(Adjust the URL version if needed.)

## 5. Clean up namespace labels and other Istio-related resources

If you labeled namespaces for Istio injection or ambient mesh, remove those labels:

```bash
kubectl label namespace default istio-injection-
kubectl label namespace default istio.io/dataplane-mode-
kubectl label namespace default istio.io/use-waypoint-
```

Also, delete any sample applications or custom resources you installed with Istio, for example:

```bash
kubectl delete -f https://raw.githubusercontent.com/istio/istio/release-1.23/samples/bookinfo/platform/kube/bookinfo.yaml
```


## Summary

```bash
istioctl uninstall -y --purge
kubectl delete namespace istio-system
kubectl delete crd $(kubectl get crd | grep 'istio.io' | awk '{print $1}')
kubectl delete -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.3.0/standard-install.yaml
kubectl label namespace default istio-injection-
kubectl label namespace default istio.io/dataplane-mode-
kubectl label namespace default istio.io/use-waypoint-
```

This fully removes Istio CRDs and related resources from your Minikube cluster.

---

These steps are based on the official Istio cleanup documentation and community best practices[^1][^2][^3].

<div style="text-align: center">⁂</div>

[^1]: https://istio.io/latest/docs/ambient/getting-started/cleanup/

[^2]: https://stackoverflow.com/questions/54261468/uninstall-istio-all-components-completely-from-kubernetes-cluster

[^3]: https://istio.io/v1.23/docs/ambient/getting-started/cleanup/

[^4]: https://github.com/istio/istio/issues/27065

[^5]: https://learn.microsoft.com/en-us/azure/aks/istio-uninstall-addon

[^6]: http://haralduebele.github.io/2019/02/22/install-istio-and-kiali-on-ibm-cloud-or-minikube/

[^7]: https://github.com/drabo/istio

[^8]: https://faun.pub/istio-step-by-step-part-02-getting-started-with-istio-c24ed8137741

