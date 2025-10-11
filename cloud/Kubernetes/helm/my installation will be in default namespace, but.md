<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# my installation will be in default namespace, but external-secrets is in different namespace "external-secrets", will it be a problem?

It is not a problem that the External Secrets Operator (ESO) is installed in the `external-secrets` namespace while your workloads and ExternalSecret resources are in the `default` namespace. This is a common and supported setup.

Here are the key points:

- The ESO controller runs in its own namespace (e.g., `external-secrets`) but watches for `ExternalSecret` resources across namespaces, including `default`. It can manage secrets in any namespace as long as it has the necessary RBAC permissions[^3].
- You can create `ExternalSecret` resources in the `default` namespace, and ESO will reconcile them even though the operator itself is deployed in `external-secrets` namespace[^2][^7].
- If you want to replicate secrets from one namespace to others or manage secrets cluster-wide, ESO provides cluster-scoped resources like `ClusterExternalSecret` and `ClusterSecretStore` to facilitate this. For example, `ClusterExternalSecret` can push secrets into multiple namespaces based on label selectors[^1][^5].
- Namespace isolation is possible but more complex. The open-source ESO typically uses a cluster-wide controller for ease of management, but you can configure RBAC and use cluster-scoped resources to control access and secret replication securely[^3][^5].

In summary, having ESO installed in a dedicated namespace like `external-secrets` while managing secrets in `default` or other namespaces is a standard and recommended practice. Just ensure the operator has the proper permissions to watch and manage secrets in your target namespaces.

<div style="text-align: center">⁂</div>

[^1]: https://external-secrets.io/v0.5.9/api-clusterexternalsecret/

[^2]: https://github.com/external-secrets/kubernetes-external-secrets/issues/422

[^3]: https://www.externalsecrets.com/blog/namespace-isolated-eso

[^4]: https://github.com/external-secrets/external-secrets/issues/4396

[^5]: https://www.externalsecrets.com/blog/secrets-replication

[^6]: https://stackoverflow.com/questions/46297949/is-there-a-way-to-share-secrets-across-namespaces-in-kubernetes

[^7]: https://external-secrets.io/latest/provider/kubernetes/

[^8]: https://kubernetes.io/docs/concepts/configuration/secret/

