<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# How do I adjust my Helm chart for direct access without Istio in minikube

To adjust your Helm chart for **direct access without Istio** in your local Minikube environment, you need to remove or disable Istio-specific resources (like VirtualService, Gateway, DestinationRule) and instead rely on native Kubernetes service exposure methods such as:

- Using a Kubernetes **Service** of type `NodePort` or `ClusterIP` for internal access.
- Optionally enabling and using the Minikube **ingress addon** with a standard Kubernetes Ingress resource if you want HTTP routing.

---

### Steps to adjust your Helm chart for Minikube without Istio

1. **Remove or conditionally disable Istio CRDs and resources**
    - In your Helm chart templates, wrap Istio-specific manifests (VirtualService, Gateway, DestinationRule) in a conditional block controlled by a value, e.g., `.Values.istio.enabled`.
    - For Minikube testing, set `istio.enabled=false` to skip these resources.

Example in your template YAML:

```yaml
{{- if .Values.istio.enabled }}
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: {{ include "mychart.fullname" . }}-vs
spec:
  gateways:
    - istio-ingress/ingress-gateway
  hosts:
    - "myapp.local"
  http:
    - match:
        - uri:
            prefix: /
      route:
        - destination:
            host: {{ include "mychart.fullname" . }}
            port:
              number: 80
{{- end }}
```

2. **Expose your service using a Kubernetes Service**
    - Define a `Service` of type `NodePort` or `LoadBalancer` (if supported) to expose your app.
    - Example Service YAML:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "mychart.fullname" . }}
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080  # Optional fixed node port
  selector:
    app: {{ include "mychart.name" . }}
```

3. **Enable Minikube ingress addon (optional)**
    - If you want to use HTTP routing similar to Istio ingress, enable Minikube ingress:

```bash
minikube addons enable ingress
minikube tunnel
```

    - Create a standard Kubernetes Ingress resource instead of Istio VirtualService/Gateway:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ include "mychart.fullname" . }}-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: myapp.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: {{ include "mychart.fullname" . }}
                port:
                  number: 80
```

    - Update your local `/etc/hosts` to map `myapp.local` to Minikube IP (found via `minikube ip`).
4. **Adjust values.yaml**

Add a flag to control Istio usage:

```yaml
istio:
  enabled: false
```

Set this to `true` in your GKE production values and `false` in your Minikube testing values.

---

### Summary

| Aspect | Production (GKE with Istio) | Local Minikube (No Istio) |
| :-- | :-- | :-- |
| Ingress routing | Istio VirtualService + Gateway | Kubernetes Ingress or NodePort Service |
| Service exposure | ClusterIP service, routed by Istio ingress | NodePort or LoadBalancer service |
| Helm chart configuration | `istio.enabled: true` to deploy Istio CRDs | `istio.enabled: false` to skip Istio resources |
| DNS / Hosts | Use real domain names | Use Minikube IP + `/etc/hosts` for local DNS |


---

### References and further reading

- [Istio Helm Chart Customization](https://istio.io/latest/docs/setup/additional-setup/customize-installation-helm/)
- [Minikube Ingress Addon](https://minikube.sigs.k8s.io/docs/handbook/ingress/)
- [Kubernetes Ingress with NGINX](https://spacelift.io/blog/kubernetes-ingress)
- [Helm Conditional Templates](https://helm.sh/docs/chart_template_guide/control_structures/#if-else)

---

**In brief:**
Modify your Helm chart to conditionally include Istio resources only when enabled, and for Minikube testing disable Istio and expose your service via Kubernetes native Service and optionally Ingress, enabling you to test without the overhead of Istio.

<div style="text-align: center">⁂</div>

[^1]: https://istio.io/latest/docs/setup/additional-setup/customize-installation-helm/

[^2]: https://istio.io/latest/docs/tasks/traffic-management/ingress/ingress-control/

[^3]: https://www.apollographql.com/docs/graphos/routing/self-hosted/containerization/kubernetes

[^4]: https://github.com/istio/istio/issues/42954

[^5]: https://docs.solo.io/gloo-mesh-enterprise/main/istio/manual/manual_deploy/

[^6]: https://spacelift.io/blog/kubernetes-ingress

[^7]: https://minikube.sigs.k8s.io/docs/handbook/accessing/

[^8]: https://github.com/istio/istio/issues/6463

