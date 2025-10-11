<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# helm issue: no matches for kind "ExternalSecret" in version "external-secrets.io/v1beta1"

ensure CRDs are installed first

The error **"no matches for kind 'ExternalSecret' in version 'external-secrets.io/v1beta1'"** means Kubernetes does not recognize the `ExternalSecret` resource because the required CustomResourceDefinition (CRD) is not installed or not yet available in the cluster.

## How to solve this issue

### 1. Install the CRDs before or during Helm install

The `ExternalSecret` kind is defined by a CRD that must be installed in your cluster before you can create resources of this type. According to the official External Secrets Operator Helm chart instructions:

- Use the Helm flag `--set installCRDs=true` when installing the chart to automatically install the required CRDs.

Example:

```bash
helm repo add external-secrets https://charts.external-secrets.io
# /v1beta1
helm install external-secrets external-secrets/external-secrets -n external-secrets --create-namespace --set installCRDs=true --version 0.16.2
# /v1 (0.17.0)
helm install external-secrets external-secrets/external-secrets -n external-secrets --create-namespace --set installCRDs=true

helm install external-secrets-operator external-secrets/external-secrets -n external-secrets --create-namespace --set installCRDs=true

# fix
helm uninstall external-secrets -n external-secrets

helm install external-secrets-crds external-secrets/external-secrets-crds
helm install external-secrets external-secrets/external-secrets -n external-secrets --create-namespace
```

```
This ensures the CRDs, including `ExternalSecret` in the correct API version, are installed first[^1].
~/workspace/atlas/cdb-int-crd-pr-helm (k8s/init*) » helm install external-secrets external-secrets/external-secrets -n external-secrets --create-namespace --set installCRDs=true
NAME: external-secrets
LAST DEPLOYED: Sat May 24 16:18:52 2025
NAMESPACE: external-secrets
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
external-secrets has been deployed successfully in namespace external-secrets!

In order to begin using ExternalSecrets, you will need to set up a SecretStore
or ClusterSecretStore resource (for example, by creating a 'vault' SecretStore).

More information on the different types of SecretStores and how to configure them
can be found in our Github: https://github.com/external-secrets/external-secrets

```
### 2. Verify CRDs are installed

After installation, verify the CRDs exist:

```bash
kubectl get crd | grep externalsecrets
```

You should see CRDs like `externalsecrets.external-secrets.io` or similar.

### 3. Use the correct API version in your manifests

Check the API version your cluster supports for `ExternalSecret`. The example from the official docs uses `external-secrets.io/v1alpha1` rather than `v1beta1`. Using an unsupported or incorrect API version will cause this error.

Example manifest snippet:

```yaml
apiVersion: external-secrets.io/v1alpha1
kind: ExternalSecret
metadata:
  name: example
spec:
  # spec details
```

Make sure your YAML matches the installed CRD version[^1].

### 4. If CRDs are installed but error persists

- Wait a few moments after CRD installation before applying `ExternalSecret` resources, as CRDs may take time to register.
- Confirm you are connected to the correct Kubernetes cluster and namespace.
- If you installed External Secrets Operator via other methods (e.g., OLM or manual manifests), ensure the CRDs are consistent and up to date.

---

## Summary

| Step | Description |
| :-- | :-- |
| Install CRDs | Use `--set installCRDs=true` with Helm install to install CRDs automatically[^1]. |
| Verify CRDs | Run `kubectl get crd | grep externalsecrets` to confirm CRDs exist. |
| Use correct API version | Use `external-secrets.io/v1alpha1` or the version matching your CRDs in your manifests[^1]. |
| Wait and retry | Allow time for CRDs to register before creating `ExternalSecret` resources. |

Following these steps will resolve the "no matches for kind 'ExternalSecret'" error by ensuring the CRDs are installed and your manifests use the correct API version.

[External-secrets getting start](https://external-secrets.io/latest/introduction/getting-started/)
<div style="text-align: center">⁂</div>

[^1]: https://external-secrets.io/v0.4.4/guides-getting-started/

[^2]: https://github.com/external-secrets/kubernetes-external-secrets

[^3]: https://artifacthub.io/packages/helm/trozz/kubernetes-external-secrets

[^4]: https://external-secrets.io/v0.5.7/provider-kubernetes/

[^5]: https://secrets-store-csi-driver.sigs.k8s.io/getting-started/installation

[^6]: https://kubernetes.io/docs/concepts/configuration/secret/

[^7]: https://www.reddit.com/r/kubernetes/comments/1j8x7gb/is_it_possible_to_install_external_secret/

[^8]: https://github.com/external-secrets/external-secrets
