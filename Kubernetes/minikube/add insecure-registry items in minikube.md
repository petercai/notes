<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# add insecure-registry items in minikube

To add insecure registries in Minikube, you need to specify the `--insecure-registry` flag when starting Minikube. This flag configures the Docker daemon inside the Minikube VM to allow pulling images from registries without TLS verification.

## How to add insecure registries in Minikube

### 1. Delete existing Minikube cluster (required)

The `--insecure-registry` flag only takes effect when creating a new cluster. If you already have a running or stopped cluster, delete it first:

```bash
minikube stop
minikube delete
```


### 2. Start Minikube with `--insecure-registry`

Use the `minikube start` command with the `--insecure-registry` option specifying the registry IP or CIDR range:

```bash
minikube start --insecure-registry="192.168.5.0/24"
```

Replace `"192.168.5.0/24"` with your registry’s IP address or subnet.

You can specify multiple insecure registries by separating them with commas:

```bash
minikube start --insecure-registry="192.168.5.0/24,localhost:5000"
```


### 3. Verify the setting

After Minikube starts, SSH into the Minikube VM and check Docker daemon options:

```bash
minikube ssh
cat /var/lib/boot2docker/profile
```

Look for the `--insecure-registry` flag in the Docker daemon startup arguments.

---

## Additional notes

- The insecure registry flag expects a **CIDR range** or IP without port (e.g., `192.168.5.0/24`), not a URL or hostname.
- If your registry runs on a non-standard port, you can specify it as `localhost:5000`, but this may not always work reliably.
- You can also manually edit `~/.minikube/machines/minikube/config.json` to add multiple insecure registries under `"EngineOptions": { "InsecureRegistry": [...] }`, but this is less common.
- Minikube’s registry addon does not automatically add insecure registries; you must specify them explicitly.
- For local development, using the subnet that includes your registry IP is a common approach.

---

### Summary

```bash
minikube stop
minikube delete
minikube start --insecure-registry="192.168.5.0/24"
```

This enables Docker inside Minikube to pull images from insecure registries in the specified subnet.

---

This method is based on official Minikube documentation and community best practices[^1][^2][^5].

<div style="text-align: center">⁂</div>

[^1]: https://minikube.sigs.k8s.io/docs/handbook/registry/

[^2]: https://stackoverflow.com/questions/43129110/how-can-i-allow-a-private-insecure-registry-to-be-used-inside-a-minikube-node

[^3]: https://github.com/kubernetes/minikube/issues/17209

[^4]: https://github.com/kubernetes/minikube/issues/15631

[^5]: https://merative.github.io/spm-kubernetes/prereq/kubernetes/minikube/

[^6]: https://minikube.sigs.k8s.io/docs/handbook/addons/registry-aliases/

[^7]: https://minikube.sigs.k8s.io/docs/handbook/addons/custom-images/

[^8]: https://hasura.io/blog/sharing-a-local-registry-for-minikube-37c7240d0615

