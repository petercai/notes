
# Helm chart to pull images in Minikube

To configure your Helm chart to pull images from `localhost:5000` in Minikube, you need to ensure Minikube can access your local registry and your Helm chart references the image correctly. Here's how to do it:

## 1. Make your local registry accessible to Minikube

- **Minikube runs in a VM or container, so `localhost` inside Minikube is not your host machine.** You need to expose your local registry so Minikube can reach it.
- One common approach is to use your host machine's IP address or a hostname mapped to it instead of `localhost`. For example, add an entry in your `/etc/hosts` like:

```
docker.local 127.0.0.1
```

- Then, inside the Minikube VM, add a hosts entry pointing `docker.local` to your host IP:

```bash
minikube ssh
sudo vi /etc/hosts
# Add line: docker.local <host-machine-ip>
```

- Push your image to the registry still using the localhost in host machine:

```bash
docker tag cdb-int-credential-provisioning localhost:5000/cdb-int-credential-provisioning:latest
docker push localhost:5000/cdb-int-credential-provisioning:latest
```

- In your Helm chart, reference the image as `docker.local:5000/cdb-int-credential-provisioning:latest`.


## 2. Configure Helm chart image values

In your Helm chart's `values.yaml` or via `--set`, specify:

```yaml
image:
  repository: docker.local:5000/cdb-int-credential-provisioning
  tag: latest
  pullPolicy: IfNotPresent
```

Or on the command line:

```bash
helm install myapp ./mychart --set image.repository=docker.local:5000/cdb-int-credential-provisioning --set image.tag=latest --set image.pullPolicy=IfNotPresent
```

Setting `imagePullPolicy` to `IfNotPresent` avoids unnecessary pulls if the image is cached.

## 3. Allow Minikube to pull from an insecure registry

Since local registries often run without TLS, Minikube needs to be started with the insecure registry flag:

```bash
minikube start --insecure-registry="docker.local:5000"
```

If Minikube is already running, you may need to stop and restart it with this flag.

## 4. Alternative: Use Minikube's Docker daemon

To avoid networking issues, you can build and push images directly inside Minikube's Docker environment:

```bash
eval $(minikube docker-env)
docker build -t cdb-int-credential-provisioning:latest .
```

Then set your Helm chart image repository to just `cdb-int-credential-provisioning` and `imagePullPolicy` to `Never` or `IfNotPresent`.

## Summary

| Step | Action |
| :-- | :-- |
| Make local registry reachable | Map hostname (e.g., `docker.local`) to host IP and configure Minikube hosts accordingly |
| Push image using hostname | Tag and push image to `docker.local:5000/...` |
| Set Helm image repository | Use `docker.local:5000/your-image` in Helm values or `--set` |
| Start Minikube with insecure registry flag | `minikube start --insecure-registry="docker.local:5000"` |
| Or build image inside Minikube | Use `eval $(minikube docker-env)` and build image directly |

This setup ensures your Helm chart can pull images from your local registry when deploying to Minikube[^1][^3][^5].

<div style="text-align: center">⁂</div>

[^1]: https://stackoverflow.com/questions/42564058/how-can-i-use-local-docker-images-with-minikube

[^2]: https://octopus.com/blog/local-images-minikube

[^3]: https://minikube.sigs.k8s.io/docs/handbook/pushing/

[^4]: https://www.kenmuse.com/blog/testing-kubernetes-operators-and-controllers-with-minikube/

[^5]: https://minikube.sigs.k8s.io/docs/handbook/registry/

[^6]: https://github.com/dsyer/kubernetes-intro/issues/1

[^7]: https://github.com/kiali/helm-charts

[^8]: https://microcks.io/documentation/guides/installation/minikube-helm/

