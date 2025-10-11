<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# When should I use helm upgrade instead of reinstalling with helm install

You should use **`helm upgrade`** instead of reinstalling with **`helm install`** when you want to modify or update an existing Helm release without downtime or losing the current state. Specifically:

- **To apply configuration changes or update chart versions** to an existing deployment while keeping your application running. `helm upgrade` performs a rolling update by upgrading replicas one by one, which keeps the service available during the process[^1].
- When you want to **incrementally update resources** managed by the release (e.g., change values, update images, add/remove resources) without deleting and recreating everything from scratch[^2][^4][^5].
- To **preserve release history and enable rollbacks**. `helm upgrade` tracks release versions, so you can roll back to previous versions if needed, which is not possible if you delete and reinstall[^4].
- When you want to **avoid the failure that occurs if you try to `helm install` a release name that already exists**. `helm install` is only for fresh installs and will fail if the release name is taken[^2].
- If you want to **combine upgrade and install in one command**, you can use `helm upgrade --install`, which upgrades if the release exists or installs if it does not[^4].


### When to avoid `helm upgrade` and use reinstall (`helm install`) instead:

- If the release is in a **failed or corrupted state** that cannot be fixed by upgrade, you might delete the release (`helm uninstall`) and do a fresh install[^6].
- If you want to **reset everything to a clean state** rather than patching changes incrementally.

---

### Summary

| Use Case | Recommended Command | Reason |
| :-- | :-- | :-- |
| Fresh deployment | `helm install` | Creates new release; fails if release exists |
| Change configuration or update | `helm upgrade` | Applies incremental changes, keeps service available, preserves release history |
| Upgrade or install if missing | `helm upgrade --install` | Combines upgrade and install in one command |
| Release in broken state | `helm uninstall` + `install` | Clean slate reinstall if upgrade can't fix issues |


---

In short, **use `helm upgrade` to safely and efficiently update existing deployments** without downtime or losing history, and reserve `helm install` for initial deployments or when you want to start fresh. This approach aligns with Helm best practices and ensures smooth lifecycle management of your Kubernetes applications[^1][^2][^4][^5].

<div style="text-align: center">⁂</div>

[^1]: https://www.microfocus.com/documentation/securelogin/9.1/slae_installation_config_guide/t4fyvi485xwx.html

[^2]: https://kodekloud.com/community/t/helm-intall-vs-helm-upgrade/209362

[^3]: https://stackoverflow.com/questions/61992095/does-helm-upgrade-reinstall-entire-chart-or-only-altered-manifests

[^4]: https://helm.sh/docs/helm/helm_upgrade/

[^5]: https://helm.sh/docs/intro/using_helm/

[^6]: https://github.com/helm/helm/issues/3353

[^7]: https://developer.harness.io/docs/continuous-delivery/deploy-srv-diff-platforms/helm/helm-deployment-faqs/

[^8]: https://www.reddit.com/r/kubernetes/comments/1b428t3/newbie_question_whats_the_actual_point_of_helm/

