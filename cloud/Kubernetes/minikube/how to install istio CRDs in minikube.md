
# how to install istio CRDs in minikube

To install Istio Custom Resource Definitions (CRDs) in Minikube, follow these steps based on the official Istio installation process and best practices:

## 1. Prepare Minikube with sufficient resources

Istio requires a Minikube cluster with enough CPU and memory. For example:

```bash
minikube start --memory=16384 --cpus=4 --kubernetes-version=v1.26.1
```

Adjust memory, CPU, and Kubernetes version as needed[^1][^2].

## 2. Download and install `istioctl`

Download the latest Istio release which includes the `istioctl` CLI:

```bash
curl -L https://istio.io/downloadIstio | sh -
cd istio-<version>
export PATH=$PWD/bin:$PATH
```

Replace `<version>` with the actual downloaded folder name[^5].

## 3. Install Istio CRDs

You can install the Istio CRDs separately before installing the full Istio control plane:

```bash
kubectl apply -f manifests/crd.yaml
```

Or, if you have the Istio release directory:

```bash
kubectl apply -f install/kubernetes/helm/istio/templates/crds.yaml
```

This applies all the required Istio CRDs to your cluster[^7].

## 4. Install Istio control plane with a profile (e.g., demo)

After CRDs are installed, install Istio components using `istioctl`:

```bash
istioctl install --set profile=demo -y
```

The `demo` profile is recommended for evaluation and includes key Istio features and sample apps[^5].

## 5. Verify installation

Check that Istio pods are running in the `istio-system` namespace:

```bash
kubectl get pods -n istio-system
```

You should see pods like `istiod`, `istio-ingressgateway`, etc., in `Running` state.

---

## Summary of commands

```bash
minikube start --memory=16384 --cpus=4 --kubernetes-version=v1.26.1

curl -L https://istio.io/downloadIstio | sh -
cd istio-<version>
export PATH=$PWD/bin:$PATH

kubectl apply -f install/kubernetes/helm/istio/templates/crds.yaml

istioctl install --set profile=demo -y

kubectl get pods -n istio-system
```


---

## Notes

- Minikube provides an Istio addon, but it is often outdated and not recommended for recent Kubernetes versions[^4].
- Installing CRDs separately helps avoid issues with CRD installation timing.
- The `istioctl install` command installs CRDs automatically if not already installed, but applying CRDs explicitly can help with troubleshooting.
- For more details, see the official Istio docs: https://istio.io/latest/docs/setup/platform-setup/minikube/

This approach installs Istio CRDs properly on Minikube, enabling you to run Istio service mesh and sample applications.

<div style="text-align: center">⁂</div>

[^1]: https://istio.io/latest/docs/setup/platform-setup/minikube/

[^2]: https://github.com/drabo/istio

[^3]: https://istio.io/v1.3/docs/setup/install/kubernetes/

[^4]: https://kubebyexample.com/learning-paths/istio/install

[^5]: https://faun.pub/istio-step-by-step-part-10-installing-istio-1-4-in-minikube-ebce9a4e99c

[^6]: https://www.solo.io/topics/istio/istio-kubernetes

[^7]: http://haralduebele.github.io/2019/02/22/install-istio-and-kiali-on-ibm-cloud-or-minikube/

[^8]: https://github.com/kubernetes/minikube/issues/2199

# from copilot

To install Istio CRDs (Custom Resource Definitions) in Minikube, follow these steps:

1. **Start Minikube**:
   ```bash
   minikube start --memory=8192mb --cpus=4
   ```

2. **Download Istio**:
   Download the latest Istio release package from the Istio website.

3. **Install Istioctl**:
   Extract the downloaded package and move the `istioctl` binary to your PATH:
   ```bash
   export PATH=$PATH:/path/to/istio/bin
   ```

4. **Install Istio CRDs**:
   Use `istioctl` to install Istio CRDs and core components:
   ```bash
   istioctl install --set profile=default
   ```

5. **Verify Installation**:
   Check that the Istio components are running:
   ```bash
   kubectl get pods -n istio-system
   ```

6. **Label the Namespace**:
   Label the default namespace to enable automatic sidecar injection:
   ```bash
   kubectl label namespace default istio-injection=enabled
   ```

These steps will set up Istio in your Minikube cluster, including the necessary CRDs [1](https://istio.io/latest/docs/setup/platform-setup/minikube/) [2](https://minikube.sigs.k8s.io/docs/handbook/addons/istio/).

If you need more detailed guidance or run into any issues, feel free to ask!