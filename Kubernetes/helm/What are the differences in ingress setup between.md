<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# What are the differences in ingress setup between GKE and minikube

Here are the key differences in **Ingress setup between GKE and Minikube**, based on the search results and Kubernetes best practices:

---

## 1. **Ingress Controller Implementation**

| Aspect | GKE | Minikube |
| :-- | :-- | :-- |
| **Ingress Controller** | Uses the **GKE Ingress Controller** integrated with Google Cloud Load Balancing (GLBC). When you create an Ingress resource, GKE automatically provisions a Google Cloud HTTP(S) Load Balancer. This is a managed, cloud-native load balancer with advanced features like SSL termination, global load balancing, and health checks. [^4][^6] | Uses the **NGINX Ingress Controller** (community-maintained) typically enabled via the Minikube addon (`minikube addons enable ingress`). This runs as pods inside the cluster and routes traffic internally. It does not provision external cloud load balancers. [^2][^5] |
| **Load Balancer Type** | Cloud HTTP(S) Load Balancer (external, managed) | Internal NGINX reverse proxy running as a pod |
| **Annotations** | GKE-specific annotations for Google Cloud load balancer configuration | NGINX-specific annotations for ingress-nginx controller. Different annotation syntax and features compared to GKE. [^1][^3] |


---

## 2. **Ingress Resource Behavior**

- **GKE**
    - Creating an Ingress resource triggers automatic creation of a Google Cloud Load Balancer with backend services mapped to Kubernetes Services of type `NodePort` or using container-native load balancing with Network Endpoint Groups (NEGs).
    - Supports advanced features like SSL certificates via Google-managed certs, global load balancing, and health checks based on pod readiness probes. [^4]
- **Minikube**
    - Ingress resource is fulfilled by the NGINX ingress controller pod inside the cluster.
    - Requires enabling the ingress addon and sometimes running `minikube tunnel` to expose services externally.
    - Does not create external load balancers; ingress is accessible via Minikube IP or localhost with port forwarding or tunneling. [^2][^5]

---

## 3. **Annotations and Configuration Differences**

- GKE and Minikube ingress controllers use **different annotation formats** to configure ingress behavior such as redirects, rewrites, SSL, and timeouts.
- For example, Minikube’s ingress-nginx uses annotations like:

```
nginx.ingress.kubernetes.io/rewrite-target: /$1
```

- GKE’s ingress controller may use different annotations or none, relying on Google Cloud defaults.
- Mistakes happen when applying Minikube-style ingress manifests directly on GKE and vice versa, causing ingress resources not to work properly. [^1][^3]

---

## 4. **DNS and Access**

- **GKE**
    - Ingress is assigned an external IP by Google Cloud Load Balancer.
    - DNS records can be pointed to this IP for public access.
    - Supports TLS termination with managed certificates. [^4]
- **Minikube**
    - Ingress is accessible via Minikube’s IP address or localhost with port forwarding/tunnel.
    - You often need to add entries to your local `/etc/hosts` file to map hostnames to Minikube IP for testing. [^5]

---

## 5. **Setup and Maintenance**

- **GKE**
    - Ingress controller is pre-installed and managed by Google Cloud.
    - Users mainly create Ingress resources and configure annotations.
    - Requires enabling the HTTP Load Balancing add-on if not enabled by default. [^4]
- **Minikube**
    - User must enable ingress addon explicitly:

```
minikube addons enable ingress
```

    - Monitor ingress-nginx pods for readiness.
    - May require running `minikube tunnel` to expose LoadBalancer services. [^2][^5]

---

## Summary Table

| Feature | GKE Ingress | Minikube Ingress |
| :-- | :-- | :-- |
| Controller | GKE Ingress Controller (GLBC) | NGINX Ingress Controller (community) |
| Load Balancer | External Google Cloud HTTP(S) Load Balancer | Internal NGINX reverse proxy pod |
| Resource Provisioning | Automatic cloud load balancer creation | Manual addon enable, pod-based ingress |
| Annotations | GKE-specific, Google Cloud annotations | NGINX-specific annotations |
| DNS | External IP with public DNS | Minikube IP + `/etc/hosts` mapping |
| SSL/TLS | Managed certificates supported | Self-managed certificates or none |
| Setup | Enabled by default or via GKE console/CLI | `minikube addons enable ingress` + `minikube tunnel` |
| Use case | Production-grade, cloud-native ingress | Local development and testing |


---

## References

- [StackOverflow: NGINX Ingress redirect differences Minikube vs GKE](https://stackoverflow.com/questions/66833790/nginx-ingress-redirect-www-to-non-www-on-minikube-vs-gke) [^1]
- [Minikube Ingress Setup](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/) [^5]
- [GKE Ingress with External Load Balancer](https://cloud.google.com/kubernetes-engine/docs/tutorials/http-balancer) [^4]
- [Spacelift Blog on Kubernetes Ingress](https://spacelift.io/blog/kubernetes-ingress) [^2]
- [StackOverflow: Ingress YAML issues on GKE vs Minikube](https://stackoverflow.com/questions/66199574/ingress-yml-file-isnt-being-applied-to-gke-but-works-fine-in-minikube) [^3]

---

### In brief

- **GKE Ingress** is tightly integrated with Google Cloud Load Balancing and provides a managed, scalable, and production-ready ingress solution with cloud-native features.
- **Minikube Ingress** uses the NGINX ingress controller running inside the cluster, suitable for local development and testing, requiring manual addon enablement and local network configuration.

Adjust your ingress manifests and annotations accordingly depending on the environment to ensure correct behavior.

<div style="text-align: center">⁂</div>

[^1]: https://stackoverflow.com/questions/66833790/nginx-ingress-redirect-www-to-non-www-on-minikube-vs-gke

[^2]: https://spacelift.io/blog/kubernetes-ingress

[^3]: https://stackoverflow.com/questions/66199574/ingress-yml-file-isnt-being-applied-to-gke-but-works-fine-in-minikube

[^4]: https://cloud.google.com/kubernetes-engine/docs/tutorials/http-balancer

[^5]: https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/

[^6]: https://www.spectrocloud.com/blog/getting-started-with-kubernetes-ingress

[^7]: https://kubernetes.io/docs/concepts/services-networking/ingress/

[^8]: https://serverfault.com/questions/1147219/set-up-minikube-installation-with-gke

