<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# I the first error below. but when I run command "helm install external-secrets external-secrets/external-secrets -n external-secrets --create-namespace --set installCRDs=true", I got 2nd error. how to fix it?

1. Error: INSTALLATION FAILED: unable to build kubernetes objects from release manifest: [resource mapping not found for name: "cdb-int-credential-provisioning-ist" namespace: "" from "": no matches for kind "ExternalSecret" in version "external-secrets.io/v1beta1"
ensure CRDs are installed first, resource mapping not found for name: "provisioning-ist-nane2-es" namespace: "" from "": no matches for kind "ExternalSecret" in version "external-secrets.io/v1beta1"
ensure CRDs are installed first]
2. Error: INSTALLATION FAILED: cannot re-use a name that is still in use

The first error you encountered:

```
Error: INSTALLATION FAILED: unable to build kubernetes objects from release manifest: [resource mapping not found for name: "cdb-int-credential-provisioning-ist" namespace: "" from "": no matches for kind "ExternalSecret" in version "external-secrets.io/v1beta1"
ensure CRDs are installed first, resource mapping not found for name: "provisioning-ist-nane2-es" namespace: "" from "": no matches for kind "ExternalSecret" in version "external-secrets.io/v1beta1"
ensure CRDs are installed first]
```

indicates that the Custom Resource Definitions (CRDs) for `ExternalSecret` are not installed or recognized in your Kubernetes cluster before you tried to create resources of that kind. The error message explicitly says "ensure CRDs are installed first," meaning Kubernetes does not yet know about the `ExternalSecret` kind from the `external-secrets.io/v1beta1` API group[^1][^3].

You attempted to fix this by running:

```
helm install external-secrets external-secrets/external-secrets -n external-secrets --create-namespace --set installCRDs=true
```

which should install the CRDs as part of the Helm release. However, you then got the second error:

```
Error: INSTALLATION FAILED: cannot re-use a name that is still in use
```

This error means that a Helm release with the name `external-secrets` already exists in the cluster, so Helm refuses to create a new release with the same name[^3].

---

## How to fix these errors

1. **Ensure CRDs are installed before creating ExternalSecret resources**
    - The `installCRDs=true` flag in your Helm install command is correct and required to install the CRDs automatically[^1].
    - However, if you previously attempted an install that failed, the release might be partially installed or stuck.
    - You can try to uninstall the existing release first:

```
helm uninstall external-secrets -n external-secrets
```

Then re-run your install command with `--set installCRDs=true`.
2. **If the release name is in use, either uninstall or use a different release name**
    - The "cannot re-use a name that is still in use" error means the release name `external-secrets` is taken.
    - Run `helm list -n external-secrets` to see existing releases.
    - If the release exists but is broken, uninstall it as above.
    - Alternatively, choose a different release name for installation, e.g.:

```
helm install external-secrets-operator external-secrets/external-secrets -n external-secrets --create-namespace --set installCRDs=true
```

3. **Two-step installation approach (if needed)**
    - Sometimes, CRDs need to be installed first before creating any `ExternalSecret` resources.
    - You can install the CRDs separately first, then install the Helm chart without CRDs:

```
helm install external-secrets-crds external-secrets/external-secrets-crds
helm install external-secrets external-secrets/external-secrets -n external-secrets --create-namespace
```

    - This approach avoids timing issues where the CRDs are not yet available when Helm tries to create `ExternalSecret` resources[^3][^6].

---

## Summary

- Uninstall any existing `external-secrets` Helm release to clear the "name in use" error.
- Reinstall with `--set installCRDs=true` to ensure CRDs are installed.
- If issues persist, consider installing CRDs separately before the main chart.
- Make sure your Kubernetes cluster recognizes the `ExternalSecret` CRD before applying any `ExternalSecret` resources.

This sequence will resolve the "resource mapping not found" error and the "cannot re-use a name" error you encountered.

<div style="text-align: center">⁂</div>

[^1]: https://external-secrets.io/v0.4.4/guides-getting-started/

[^2]: https://github.com/external-secrets/kubernetes-external-secrets/issues/365

[^3]: https://github.com/external-secrets/external-secrets/discussions/1566

[^4]: https://stackoverflow.com/questions/76051610/external-secret-is-not-working-on-kubernetes

[^5]: https://blog.gitguardian.com/how-to-handle-secrets-in-helm/

[^6]: https://github.com/hashicorp/terraform-provider-kubernetes/issues/1380

[^7]: https://www.reddit.com/r/kubernetes/comments/1gvwlis/how_can_i_apply_secrets_to_a_helm_chart/

[^8]: https://stackoverflow.com/questions/67912695/kubernetes-external-secrets-operator-error-invalidclienttokenid-the-security

