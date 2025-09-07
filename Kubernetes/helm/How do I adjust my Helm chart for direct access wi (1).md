<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# How do I adjust my Helm chart for direct access without Istio in minikube

To adjust your Helm chart for **direct access without Istio** in your local Minikube environment, you should remove or disable Istio-specific resources (like VirtualService, Gateway, DestinationRule) and instead rely on native Kubernetes service exposure methods. Here’s how to do it:

---

### 1. Add a flag to enable/disable Istio in your Helm chart

In your `values.yaml`, add:

```yaml
istio:
  enabled: false
```

Set this to `true` in your production (GKE) environment and `false` for Minikube.

---

### 2. Wrap Istio-specific templates with a conditional check

In your Helm templates (e.g., `virtualservice.yaml`), wrap Istio resources like this:

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

This ensures Istio resources are only created when Istio is enabled.

---

### 3. Expose your service directly with a Kubernetes Service

Create or modify a Service manifest like this:

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
      nodePort: 30080   # Optional fixed port for easy access
  selector:
    app: {{ include "mychart.name" . }}
```

This exposes your app directly on a NodePort accessible via Minikube IP.

---

### 4. (Optional) Use Minikube Ingress addon for HTTP routing

If you want hostname-based routing similar to Istio ingress:

- Enable ingress addon:

```bash
minikube addons enable ingress
minikube tunnel
```

- Create a Kubernetes Ingress resource (instead of Istio VirtualService/Gateway):

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

- Add an entry in your `/etc/hosts` pointing `myapp.local` to your Minikube IP (`minikube ip`).

---

### Summary

| Step | Production (GKE with Istio) | Local Minikube (No Istio) |
| :-- | :-- | :-- |
| Istio resources | Enabled (`istio.enabled: true`) | Disabled (`istio.enabled: false`) |
| Ingress routing | Istio VirtualService + Gateway | Kubernetes Ingress or NodePort |
| Service exposure | ClusterIP + Istio ingress | NodePort or Ingress addon |
| DNS | Real domain names | `/etc/hosts` mapping to Minikube IP |


---

### References

- Istio Helm customization and conditional installation [^1]
- Minikube ingress addon documentation [^6][^7]
- Kubernetes native Ingress and Service exposure best practices

---

**In essence:**
Use Helm conditionals to disable Istio manifests locally, expose your app via Kubernetes native Service (NodePort or Ingress), and configure Minikube networking accordingly to test your micro-service without Istio overhead.

<div style="text-align: center">⁂</div>

[^1]: https://istio.io/latest/docs/setup/additional-setup/customize-installation-helm/

[^2]: https://istio.io/latest/docs/tasks/traffic-management/ingress/ingress-control/

[^3]: https://www.apollographql.com/docs/graphos/routing/self-hosted/containerization/kubernetes

[^4]: https://github.com/istio/istio/issues/42954

[^5]: https://docs.solo.io/gloo-mesh-enterprise/main/istio/manual/manual_deploy/

[^6]: https://spacelift.io/blog/kubernetes-ingress

[^7]: https://minikube.sigs.k8s.io/docs/handbook/accessing/

[^8]: https://github.com/istio/istio/issues/6463

