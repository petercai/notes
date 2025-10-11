<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# kubectl delete a node

To delete a node from a Kubernetes cluster using `kubectl`, follow these steps for a safe and clean removal:

1. **List the nodes to identify the node name:**

```
kubectl get nodes
```

2. **Drain the node to safely evict all pods:**
This marks the node as unschedulable and evicts pods (except DaemonSets and mirror pods) so workloads can be gracefully terminated or moved.

```
kubectl drain <node-name> --ignore-daemonsets
```

The `--ignore-daemonsets` flag ensures DaemonSet-managed pods are not evicted, as they are expected to run on all nodes.
3. **Delete the node from the cluster:**
This removes the node object from the Kubernetes API server.

```
kubectl delete node <node-name>
```

4. **(Optional) If you manage the node with kubeadm, reset the node itself:**
Run this command on the node machine to clean up Kubernetes components.

```
kubeadm reset
```


This sequence ensures workloads are safely evacuated before the node is removed, preventing disruption and resource leaks[^1][^3][^5].

In summary, the typical command flow is:

```bash
kubectl drain <node-name> --ignore-daemonsets
kubectl delete node <node-name>
```

Followed by any node-level cleanup if necessary.

<div style="text-align: center">⁂</div>

[^1]: https://www.ibm.com/docs/en/zoscp/1.1.0?topic=clusters-removing-worker-node-from-cluster

[^2]: https://kubernetes.io/docs/reference/kubectl/generated/kubectl_delete/

[^3]: https://stackoverflow.com/questions/35757620/how-to-gracefully-remove-a-node-from-kubernetes

[^4]: https://docs.mirantis.com/mcp/q4-18/mcp-operations-guide/kubernetes-operations/k8s-node-ops/k8s-node-remove.html

[^5]: https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/

[^6]: https://kubernetes.io/docs/reference/kubectl/

[^7]: https://kubesphere.io/docs/v3.4/installing-on-linux/cluster-operation/remove-nodes/

[^8]: https://www.groundcover.com/blog/kubectl-drain-node

