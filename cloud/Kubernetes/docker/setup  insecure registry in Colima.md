# setup  insecure registry in Colima?

When running Colima on macOS **without admin access**, configuring Docker to allow pushing/pulling from an **insecure (HTTP) local registry** like `http://localhost:5000` requires editing Colima’s user-level config because you cannot modify the system-wide Docker daemon settings.

Here is how to set up an insecure registry in Colima:

---

## Steps to configure insecure registry in Colima

1. **Edit Colima’s configuration file**

Colima stores its config in your home directory at:

```
$HOME/.colima/default/colima.yaml
```

2. **Add your local registry as an insecure registry**

Open the file and add or edit the `insecure-registries` section under `docker` like this:

```yaml
docker:
  insecure-registries:
    - docker.local:5000
```

Replace `docker.local:5000` with the hostname and port you use for your local registry.
3. **Restart Colima to apply changes**

Run:

```bash
colima stop
colima start
```

or if you want to edit and restart in one step:

```bash
colima start --edit
```

This opens the config in your editor; make the change and save, then Colima restarts with the new config.
4. **Verify Docker daemon recognizes the insecure registry**

Run:

```bash
docker info | grep -A 5 "Insecure Registries"
```

You should see your `docker.local:5000` listed.
5. **Use the hostname in your image tags**

Tag your image with the insecure registry hostname:

```bash
docker tag cdb-int-credential-provisioning docker.local:5000/cdb-int-credential-provisioning:latest
docker push docker.local:5000/cdb-int-credential-provisioning:latest
```


---

## Additional notes

- Since you don’t have admin access, you cannot modify `/etc/docker/daemon.json` or system Docker daemon settings, but Colima’s config is user-level and designed for this.
- If you use `localhost:5000` inside Minikube or Colima, it won’t work because `localhost` inside the VM/container refers to the VM itself, not your host machine.
- Use `docker.local` or `host.docker.internal` as the hostname, and ensure it resolves correctly inside Colima and Minikube.
- You may need to add `docker.local` to your macOS `/etc/hosts` pointing to `127.0.0.1`.

---

## Summary

| Step | Command / Action |
| :-- | :-- |
| Edit Colima config | `vim $HOME/.colima/default/colima.yaml` |
| Add insecure registry | Add under `docker:` section: `insecure-registries: ["docker.local:5000"]` |
| Restart Colima | `colima stop && colima start` |
| Verify insecure registries | `docker info | grep -A 5 "Insecure Registries"` |
| Tag and push image | `docker tag ... docker.local:5000/...` and `docker push ...` |


---

This method configures Colima’s Docker daemon to trust your local HTTP registry without requiring admin rights on macOS.

---

**References:**

- Colima GitHub issue discussion on insecure registries and `colima.yaml` config [\#318](https://github.com/abiosoft/colima/issues/318)
- Colima official README on configuration and usage[^1]

<div style="text-align: center">⁂</div>

[^1]: https://github.com/abiosoft/colima

[^2]: https://github.com/abiosoft/colima/issues/318

[^3]: https://smallsharpsoftwaretools.com/tutorials/use-colima-to-run-docker-containers-on-macos/

[^4]: https://rameezkhan.me/posts/2022/2022-10-28--colima-as-a-docker-desktop-replacement/

[^5]: https://devoriales.com/post/368/running-docker-on-macos-without-docker-desktop-updated-with-kubernetes-installation

[^6]: https://stackoverflow.com/questions/72709388/docker-hub-login-failure-on-mac-using-colima

[^7]: https://golang.testcontainers.org/system_requirements/using_colima/

[^8]: https://marczin.dev/blog/macos-docker-setup/

