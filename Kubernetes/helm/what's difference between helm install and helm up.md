<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# what's difference between helm install and helm upgrade

The difference between `helm install` and `helm upgrade` is as follows:

- **`helm install`** is used to create a new release by installing a Helm chart into your Kubernetes cluster. It deploys a new instance of the chart with the specified release name and configuration. If the release name already exists, the install will fail.
Example usage:

```
helm install my-release my-chart
```

This command installs a new release named `my-release` from the chart `my-chart`[^1][^2].
- **`helm upgrade`** is used to update an existing release with changes from a new or updated chart or configuration. It modifies the resources of the existing release to match the new chart version or values. If the release does not exist, `helm upgrade` by itself will fail unless combined with the `--install` flag, which allows it to install the release if missing.
Example usage:

```
helm upgrade my-release my-chart
```

This command upgrades the existing release `my-release` with changes from `my-chart`[^1][^3].


### Key points:

- `helm install` creates a new release; `helm upgrade` modifies an existing release.
- `helm upgrade` can optionally install if the release does not exist using `--install`.
- Both commands support similar flags for setting values, namespaces, and dependency updates.
- `helm upgrade` sends only changed manifests to Kubernetes, not reinstalling everything from scratch[^5].


### Summary table

| Command | Purpose | Behavior if release exists | Behavior if release missing |
| :-- | :-- | :-- | :-- |
| `helm install` | Install a new release | Fails with error | Installs new release |
| `helm upgrade` | Upgrade an existing release | Applies updates | Fails unless `--install` flag used |
| `helm upgrade --install` | Upgrade or install if missing | Applies updates or installs | Installs new release |

This distinction helps manage lifecycle of Helm releases efficiently in Kubernetes clusters[^1][^3].

<div style="text-align: center">⁂</div>

[^1]: https://glasskube.dev/blog/what-is-helm-in-kubernetes/

[^2]: https://kodekloud.com/community/t/helm-intall-vs-helm-upgrade/209362

[^3]: https://helm.sh/docs/helm/helm_upgrade/

[^4]: https://www.microfocus.com/documentation/securelogin/9.1/slae_installation_config_guide/t4fyvi485xwx.html

[^5]: https://stackoverflow.com/questions/61992095/does-helm-upgrade-reinstall-entire-chart-or-only-altered-manifests

[^6]: https://helm.sh/docs/intro/using_helm/

[^7]: https://github.com/helm/helm/issues/3353

[^8]: https://www.reddit.com/r/kubernetes/comments/10efdmq/helm_makes_it_overly_complex_or_is_it_just_me/

