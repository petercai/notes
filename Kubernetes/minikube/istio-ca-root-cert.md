
# istio-ca-root-cert not found

Events:
Type     Reason       Age                 From               Message
-------------------------
Normal   Scheduled    10h                 default-scheduler  Successfully assigned istio-system/istio-egressgateway-77f8749c7c-75vgk to minikube
Warning  FailedMount  10h (x12 over 10h)  kubelet            MountVolume.SetUp failed for volume "istiod-ca-cert" : configmap "istio-ca-root-cert" not found

The error `MountVolume.SetUp failed for volume "istiod-ca-cert" : configmap "istio-ca-root-cert" not found` indicates that the ConfigMap `istio-ca-root-cert` is missing in the namespace where the Istio pod is trying to mount it. This ConfigMap contains the Istio CA root certificate and is normally created and managed by Istiod to enable mutual TLS.

### How to Solve This Issue

1. **Restart the Istiod Pod**
    - Several reports indicate that a simple rollout restart of the `istiod` deployment can fix this issue by triggering re-creation of the missing ConfigMap in the affected namespaces:

```
kubectl -n istio-system rollout restart deployment istiod
```

    - This forces Istiod to reconcile and recreate the `istio-ca-root-cert` ConfigMap in the namespaces where it is missing[^2].
2. **Check Istiod Leader Election and Health**
    - The ConfigMap creation depends on Istiod’s namespace controller leader election. If the leader is missing or unhealthy, ConfigMaps may not be created.
    - Check the `istio-status-leader` ConfigMap in `istio-system` namespace for leader status:

```
kubectl -n istio-system get configmap istio-status-leader -o yaml
```

    - If the leader info is empty or outdated, restarting Istiod can help restore leadership and fix ConfigMap creation[^2].
3. **Verify Namespace Injection and ConfigMap Presence**
    - The `istio-ca-root-cert` ConfigMap should be present in every namespace where Istio sidecar injection is enabled.
    - You can manually check:

```
kubectl get configmap istio-ca-root-cert -n <namespace>
```

    - If missing, verify that automatic sidecar injection is enabled for that namespace and that Istiod has permissions to create ConfigMaps there.
4. **Resource Constraints**
    - Insufficient CPU or memory resources in the cluster can cause Istiod to malfunction and fail to create ConfigMaps.
    - Ensure your Minikube cluster has enough allocated resources (e.g., at least 4 CPUs and 8GB RAM) to run Istio components smoothly[^1].
5. **Reinstall or Upgrade Istio**
    - If the problem persists, consider uninstalling and reinstalling Istio or upgrading to a newer stable version, as some versions have known issues with ConfigMap creation on new namespaces[^2].
6. **Manual Workaround: Create ConfigMap Manually**
    - As a last resort, you can manually create the `istio-ca-root-cert` ConfigMap in the affected namespace by copying it from `istio-system`:

```
kubectl -n istio-system get configmap istio-ca-root-cert -o yaml | \
  sed "s/namespace: istio-system/namespace: <target-namespace>/" | \
  kubectl apply -f -
```

    - Replace `<target-namespace>` with the namespace where the pod is failing.

### Summary

The missing `istio-ca-root-cert` ConfigMap error is typically resolved by restarting the `istiod` deployment to restore leader election and ConfigMap creation. Verify resource availability and sidecar injection settings, and consider reinstalling or upgrading Istio if needed. Manual ConfigMap creation can be a temporary workaround.

This approach is based on multiple user reports and Istio GitHub issue discussions addressing the same error[^1][^2].

<div style="text-align: center">⁂</div>

[^1]: https://github.com/istio/istio/issues/22463

[^2]: https://github.com/istio/istio/issues/43824

[^3]: https://stackoverflow.com/questions/66527165/installing-istio-on-wsl2-fails-with-failedmount-for-pods

[^4]: https://istio.io/v1.14/docs/ops/common-problems/injection/

[^5]: https://smallstep.com/blog/istio-with-private-ca/

[^6]: https://blog.nashtechglobal.com/how-to-manage-certification-using-istio-for-kubernetes-cluster-1/

[^7]: https://istio.io/latest/docs/tasks/security/cert-management/plugin-ca-cert/

[^8]: https://discuss.kubernetes.io/t/mountvolume-setup-failed-for-volume-istiod-ca-cert-and-default-token-w8nqb/18973

