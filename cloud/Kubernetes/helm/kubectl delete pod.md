<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# kubectl delete pod

To delete a pod in Kubernetes using `kubectl`, you can use the following commands and options:

- **Delete a specific pod by name:**

```
kubectl delete pod <pod-name>
```

This deletes the pod gracefully, allowing it to terminate properly[^2][^3][^8].
- **Delete a pod immediately (force delete) without waiting for graceful termination:**

```
kubectl delete pod <pod-name> --force --grace-period=0
```

Use this with caution as it forcibly removes the pod and may cause issues if the pod’s processes are still running[^2][^3].
- **Delete pods by label selector:**

```
kubectl delete pods -l <label-selector>
```

This deletes all pods matching the label selector, useful for bulk deletion[^3][^4].
- **Delete all pods in a namespace:**

```
kubectl delete pods --all -n <namespace>
```

Deletes all pods in the specified namespace[^3].
- **Delete completed or failed pods selectively:**

```
kubectl delete pods --field-selector=status.phase=Succeeded,status.phase=Failed
```

This deletes pods that have completed or failed, helping clean up resources[^1].
- **Delete pods from a manifest file:**

```
kubectl delete -f pod.yaml
```

Deletes the pod(s) defined in the YAML or JSON manifest file[^2][^4][^6].

**Important notes:**

- Deleting a pod managed by a controller (like a Deployment) will cause Kubernetes to create a new pod to maintain the desired state. To stop pod recreation, delete or scale down the controller resource (e.g., Deployment)[^3].
- Use `--dry-run=client` with delete commands to preview what will be deleted without actually deleting resources[^1].

In summary, the basic command to delete a pod is:

```bash
kubectl delete pod <pod-name>
```

with options to delete multiple pods, force delete, or delete by manifest as needed.

<div style="text-align: center">⁂</div>

[^1]: https://spacelift.io/blog/kubectl-delete-pod

[^2]: https://kubernetes.io/docs/reference/kubectl/generated/kubectl_delete/

[^3]: https://stackoverflow.com/questions/75816779/delete-a-pod-from-kubernetes-v1-25-6

[^4]: https://kubernetes.io/docs/reference/kubectl/

[^5]: https://www.bluematador.com/blog/safely-removing-pods-from-a-kubernetes-node

[^6]: https://jamesdefabia.github.io/docs/user-guide/kubectl/kubectl_delete/

[^7]: https://phoenixnap.com/kb/kubectl-delete-pod

[^8]: https://www.warp.dev/terminus/kube-ctl-delete-pod

