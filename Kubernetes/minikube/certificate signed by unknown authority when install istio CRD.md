
# x509: certificate signed by unknown authority when install istio CRD:

Type     Reason                  Age                   From               Message
-------------------------
Normal   Scheduled               10h                   default-scheduler  Successfully assigned istio-system/istiod-5494b44d74-z7hmn to minikube
Normal   Pulling                 10h (x5 over 10h)     kubelet            Pulling image "docker.io/istio/pilot:1.26.1"
Warning  Failed                  10h (x5 over 10h)     kubelet            Failed to pull image "docker.io/istio/pilot:1.26.1": Error response from daemon: Get "https://registry-1.docker.io/v2/": tls: failed to verify certificate: x509: certificate signed by unknown authority
Warning  Failed                  10h (x5 over 10h)     kubelet            Error: ErrImagePull
Normal   BackOff                 10h (x43 over 10h)    kubelet            Back-off pulling image "docker.io/istio/pilot:1.26.1"
Warning  Failed                  10h (x43 over 10h)    kubelet            Error: ImagePullBackOff

The issue you are facing when installing Istio CRDs on Minikube on macOS, where the Istio pods fail to pull images due to the error "tls: failed to verify certificate: x509: certificate signed by unknown authority," is related to TLS certificate verification failure when the kubelet tries to pull the Istio container images from Docker Hub.

### Root Cause

The error indicates that your Minikube environment's container runtime cannot verify the TLS certificate of Docker Hub's registry, likely because the certificate authority (CA) certificates are missing, outdated, or not trusted in your Minikube VM or Docker environment.

### How to Solve This Issue

1. **Verify Minikube and Docker Certificate Trust**
    - Ensure that the Minikube VM or Docker environment used by Minikube has up-to-date CA certificates.
    - You can SSH into the Minikube VM and update CA certificates:

```
minikube ssh
sudo update-ca-certificates
```

    - On macOS, if you are using Docker Desktop with Minikube, ensure Docker Desktop is updated and its certificates are valid.
2. **Check Network and Proxy Settings**
    - If you are behind a corporate proxy or firewall, the proxy might be intercepting TLS traffic and replacing certificates with its own, which are not trusted by default.
    - Configure Docker and Minikube to trust your proxy's CA certificate by adding it to the trusted certificates in the Minikube VM or Docker daemon.
3. **Use a Different Image Registry or Mirror**
    - As a workaround, you can configure Istio to pull images from a different registry or use a local image mirror that does not have TLS issues.
4. **Manually Pull and Load Images**
    - You can manually pull the Istio images on your host machine and load them into Minikube:

```
docker pull docker.io/istio/pilot:1.26.1
minikube image load docker.io/istio/pilot:1.26.1
```

    - This bypasses the need for Minikube to pull the image directly.
5. **Check Istio Version and Compatibility**
    - Make sure you are using a compatible version of Istio with your Kubernetes and Minikube versions.
    - Some users reported similar TLS issues with certain Istio versions and Kubernetes versions, which were resolved by upgrading or downgrading Istio or Kubernetes.
6. **Refer to Istio Certificate Management Docs**
    - If you are generating or plugging in custom CA certificates for Istio, ensure the certificates are correctly configured and trusted in the cluster as per Istio's documentation on plugging in CA certificates[^2].

### Summary

The main fix is to ensure that the Minikube environment trusts the CA certificates used by Docker Hub's registry to avoid the TLS certificate verification error when pulling images. This often involves updating CA certificates inside Minikube, configuring proxy trust, or manually loading images into Minikube[^2][^6][^9].

If the problem persists after these steps, consider checking Istio GitHub issues and forums for similar TLS errors related to your Istio and Kubernetes versions[^1][^9].

<div style="text-align: center">⁂</div>

[^1]: https://github.com/istio/istio/issues/14942

[^2]: https://istio.io/latest/docs/tasks/security/cert-management/plugin-ca-cert/

[^3]: https://istio.io/latest/docs/setup/install/istioctl/

[^4]: https://github.com/istio/istio/issues/25688

[^5]: https://stackoverflow.com/questions/57361168/istio-installation-failed

[^6]: https://discuss.istio.io/t/installing-istio/7617

[^7]: https://istio.io/latest/docs/ops/diagnostic-tools/istioctl/

[^8]: https://kubebyexample.com/learning-paths/istio/install

[^9]: https://stackoverflow.com/questions/72073613/istio-installation-failed-apple-silicon-m1/72837452

[^10]: https://github.com/istio/istio/issues/37531

