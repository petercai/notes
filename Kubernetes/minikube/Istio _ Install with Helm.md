1. [Documentation](https://istio.io/latest/docs/ "Learn how to deploy, use, and operate Istio.")
2. [Sidecar Mode](https://istio.io/latest/docs/setup/ "Information for setting up and operating Istio in sidecar mode.")
3. [Install](https://istio.io/latest/docs/setup/install/ "Choose the guide that best suits your needs and platform.")
4. Install with Helm

Follow this guide to install and configure an Istio mesh using [Helm](https://helm.sh/docs/).

The Helm charts used in this guide are the same as those used when installing Istio via [Istioctl](https://istio.io/latest/docs/setup/install/istioctl/), with the exception of the `gateway` chart.

Istioctl uses a different [gateway chart](https://github.com/istio/istio/tree/release-1.26/manifests/charts/gateways/istio-ingress) than the [gateway chart](https://github.com/istio/istio/tree/release-1.26/manifests/charts/gateway) described in this guide

Prerequisites
-------------

1. Perform any necessary [platform-specific setup](https://istio.io/latest/docs/setup/platform-setup/).

2. Check the [Requirements for Pods and Services](https://istio.io/latest/docs/ops/deployment/application-requirements/).

3. [Install the latest Helm client](https://helm.sh/docs/intro/install/). Helm versions released before the [oldest currently-supported Istio release](https://istio.io/latest/docs/releases/supported-releases/#support-status-of-istio-releases) are not tested, supported, or recommended.

4. Configure the Helm repository:
    ```
    helm repo add istio https://istio-release.storage.googleapis.com/charts
    helm repo update
    ```
    
Installation steps
------------------

This section describes the procedure to install Istio using Helm. The general syntax for helm installation is:

The variables specified in the command are as follows:

* `<chart>` A path to a packaged chart, a path to an unpacked chart directory or a URL.
* `<release>` A name to identify and manage the Helm chart once installed.
* `<namespace>` The namespace in which the chart is to be installed.

Default configuration values can be changed using one or more `--set <parameter>=<value>` arguments. Alternatively, you can specify several parameters in a custom values file using the `--values <file>` argument.

1. Install the Istio base chart which contains cluster-wide Custom Resource Definitions (CRDs) which must be installed prior to the deployment of the Istio control plane:
   
   ```
   helm install istio-base istio/base -n istio-system --set defaultRevision=default --create-namespace

To learn more about the release, try:
  $ helm status istio-base -n istio-system
  $ helm get all istio-base -n istio-system
   ```

2. Validate the CRD installation with the `helm ls` command:
   
   ```
   helm ls -n istio-system
   ```
   
   In the output locate the entry for `istio-base` and make sure the status is set to `deployed`.

3. If you intend to use Istio CNI chart you must do so now. See [Install Istio with the CNI plugin](https://istio.io/latest/docs/setup/additional-setup/cni/#installing-with-helm) for more info.

4. Install the Istio discovery chart which deploys the `istiod` service:
   
   ```
   helm install istiod istio/istiod -n istio-system --wait
   helm upgrade istiod istio/istiod -n istio-system --wait --debug --timeout 10m0s


   ```
    4.1 troubelshooting
        ```
        kubectl get pods -n istio-system 
        kubectl describe pod istiod-74f575db47-smvtf -n istio-system
    ```
    5. Verify the Istio discovery chart installation:
   
   ```
   helm ls -n istio-system
   ```

6. Get the status of the installed helm chart to ensure it is deployed:
   
   ```
   helm status istiod -n istio-system
   ```

7. Check `istiod` service is successfully installed and its pods are running:
   
   ```
   kubectl get deployments -n istio-system --output wide
   ```

8. (Optional) Install an ingress gateway:
   
   ```
   kubectl create namespace istio-ingress
   helm install istio-ingress istio/gateway -n istio-ingress --wait
   ```
   
   See [Installing Gateways](https://istio.io/latest/docs/setup/additional-setup/gateway/) for in-depth documentation on gateway installation.

Updating your Istio configuration
---------------------------------

You can provide override settings specific to any Istio Helm chart used above and follow the Helm upgrade workflow to customize your Istio mesh installation. The available configurable options can be found by using `helm show values istio/<chart>`; for example `helm show values istio/gateway`.

### Migrating from non-Helm installations

If you’re migrating from a version of Istio installed using `istioctl` to Helm (Istio 1.5 or earlier), you need to delete your current Istio control plane resources and re-install Istio using Helm as described above. When deleting your current Istio installation, you must not remove the Istio Custom Resource Definitions (CRDs) as that can lead to loss of your custom Istio resources.

You can follow steps mentioned in the [Istioctl uninstall guide](https://istio.io/latest/docs/setup/install/istioctl/#uninstall-istio).

Uninstall
---------

You can uninstall Istio and its components by uninstalling the charts installed above.

1. List all the Istio charts installed in `istio-system` namespace:
   
   ```
   helm ls -n istio-system
   ```
2. (Optional) Delete any Istio gateway chart installations:
   
   ```
   helm delete istio-ingress -n istio-ingress
   kubectl delete namespace istio-ingress
   ```
3. Delete Istio discovery chart:
   
   ```
   helm delete istiod -n istio-system
   ```
4. Delete Istio base chart:
   By design, deleting a chart via Helm doesn’t delete the installed Custom Resource Definitions (CRDs) installed via the chart.
   
   ```
   helm delete istio-base -n istio-system
   ```
5. Delete the `istio-system` namespace:
   
   ```
   kubectl delete namespace istio-system
   ```
   
   Uninstall stable revision label resources

-----------------------------------------

If you decide to continue using the old control plane, instead of completing the update, you can uninstall the newer revision and its tag by first issuing `helm template istiod istio/istiod -s templates/revision-tags.yaml --set revisionTags={prod-canary} --set revision=canary -n istio-system | kubectl delete -f -`. You must then uninstall the revision of Istio that it pointed to by following the uninstall procedure above.

If you installed the gateway(s) for this revision using in-place upgrades, you must also reinstall the gateway(s) for the previous revision manually. Removing the previous revision and its tags will not automatically revert the previously upgraded gateway(s).

### (Optional) Deleting CRDs installed by Istio

Deleting CRDs permanently removes any Istio resources you have created in your cluster. To delete Istio CRDs installed in your cluster:

```
kubectl get crd -oname | grep --color=never 'istio.io' | xargs kubectl delete
```

Generate a manifest before installation
---------------------------------------

You can generate the manifests for each component before installing Istio using the `helm template` sub-command. For example, to generate a manifest that can be installed with `kubectl` for the `istiod` component:

```
helm template istiod istio/istiod -n istio-system --kube-version {Kubernetes version of target cluster} > istiod.yaml
```

The generated manifest can be used to inspect what exactly is installed as well as to track changes to the manifest over time.

To install the manifest generated above, which will create the `istiod` component in the target cluster:

```
kubectl apply -f istiod.yaml
```

> If attempting to install and manage Istio using helm template, please note the following caveats:
> The Istio namespace (istio-system by default) must be created manually.
> Resources may not be installed with the same sequencing of dependencies as helm install
> This method is not tested as part of Istio releases.
> While helm install will automatically detect environment specific settings from your Kubernetes context, helm template cannot as it runs offline, which may lead to unexpected results. In particular, you must ensure that you follow these steps if your Kubernetes environment does not support third party service account tokens.
> kubectl apply of the generated manifest may show transient errors due to resources not being available in the cluster in the correct order.
> elm install automatically prunes any resources that should be removed when the configuration changes (e.g. if you remove a gateway). This does not happen when you use helm template with kubectl, and these resources must be removed manually.

