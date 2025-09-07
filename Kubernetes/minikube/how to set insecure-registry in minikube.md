<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# how to set insecure-registry in minikube

To configure an insecure Docker registry in Minikube, you need to start Minikube with the `--insecure-registry` flag specifying the registry IP or CIDR range that should be treated as insecure (i.e., without TLS).

## How to set insecure registry in Minikube

1. **Stop and delete any existing Minikube cluster** (required because the flag only applies at cluster creation):
```bash
minikube stop
minikube delete
```

2. **Start Minikube with the `--insecure-registry` option:**
```bash
minikube start --insecure-registry="192.168.5.0/24"
```

Replace `"192.168.5.0/24"` with the IP or subnet of your insecure registry. For example, if your registry is at `192.168.5.2`, you can specify the subnet it belongs to.

You can also specify multiple insecure registries by separating them with commas:

```bash
minikube start --insecure-registry="192.168.5.0/24,localhost:5000"
```

3. **Verify the configuration:**

After Minikube starts, you can SSH into the Minikube VM and check the Docker daemon configuration to confirm the insecure registry is set:

```bash
minikube ssh
cat /var/lib/boot2docker/profile
```

Look for the `--insecure-registry` flag in the Docker daemon startup options.

---

### Additional notes

- The `--insecure-registry` flag configures the Docker daemon inside the Minikube VM to allow pulling images from registries without TLS.
- This setting must be applied at cluster creation; changing it on a running cluster requires deleting and recreating Minikube.
- If your registry runs on a non-standard port, specify it without the protocol, e.g., `localhost:5000`.
- For registries running inside the cluster (e.g., on service IPs like `10.0.0.0/24`), use the appropriate CIDR.

---

### Example

```bash
minikube stop
minikube delete
minikube start --insecure-registry="192.168.5.0/24"
```

This will allow Minikube’s Docker daemon to pull images from any registry in the `192.168.5.x` subnet without requiring TLS.

---

This approach is based on official Minikube documentation and community best practices[^1][^2][^5].

<div style="text-align: center">⁂</div>

[^1]: https://minikube.sigs.k8s.io/docs/handbook/registry/

[^2]: https://stackoverflow.com/questions/43129110/how-can-i-allow-a-private-insecure-registry-to-be-used-inside-a-minikube-node

[^3]: https://github.com/kubernetes/minikube/issues/17209

[^4]: https://github.com/kubernetes/minikube/issues/15631

[^5]: https://articles.maximemoreillon.com/articles/360

[^6]: https://merative.github.io/spm-kubernetes/prereq/kubernetes/minikube/

[^7]: https://hasura.io/blog/sharing-a-local-registry-for-minikube-37c7240d0615

[^8]: https://minikube.sigs.k8s.io/docs/handbook/addons/custom-images/

