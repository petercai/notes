

# docker host machine ip in colima

To find the Docker container's host machine IP when using Colima, the commonly used IP address inside containers to reach the host is **192.168.5.2**. This IP is assigned by Colima to represent the host machine from within the container network, since `host.docker.internal` DNS name is not set by Colima by default[^1].

### Details and Recommendations:

- **Host IP inside containers:** Use `192.168.5.2` as the IP address to access services running on the host machine from inside Colima containers[^1][^2].
- **No `host.docker.internal` by default:** Unlike Docker Desktop, Colima does not automatically provide the `host.docker.internal` hostname, so you need to use the fixed IP `192.168.5.2` or configure your own DNS/hosts entries[^1][^2].
- **Enable network address for VM:** When starting Colima, use the `--network-address` flag to assign an external IP to the Colima VM, making it reachable from the host and containers. You can check the assigned IP with `colima status` (e.g., `192.168.106.2`)[^4][^5].
- **Bidirectional networking limitations:** Colima currently does not fully support Docker’s `--network=host` mode or bidirectional port sharing like Docker Desktop. Services on the host are reachable inside containers via `192.168.5.2`, but localhost (`127.0.0.1`) inside containers refers to the container itself, not the host[^2].


### Summary

- To reach the host machine from a Docker container running in Colima, use the IP address `192.168.5.2`.
- Start Colima with `colima start --network-address` to enable VM IP assignment and port forwarding.
- Avoid relying on `localhost` or `127.0.0.1` inside containers to reach the host.
- If you want to access the Colima VM itself from the host, check the IP shown by `colima status`.

This approach is the current best practice given Colima’s networking model on macOS and Linux systems[^1][^2][^4][^5].

<div style="text-align: center">⁂</div>

[^1]: https://github.com/abiosoft/colima/issues/857

[^2]: https://github.com/abiosoft/colima/issues/1262

[^3]: https://www.reddit.com/r/docker/comments/1eggtq2/custom_docker_container_ip_on_macos_with_colima/

[^4]: https://stackoverflow.com/questions/76256289/exposing-a-container-in-colima-to-a-docker-container-on-host

[^5]: https://code.saghul.net/2023/09/migrating-from-docker-desktop-to-colima/

[^6]: https://golang.testcontainers.org/system_requirements/using_colima/

[^7]: https://discuss.hashicorp.com/t/nomad-service-discovery-on-macos-returns-loopback-ip/53046

[^8]: https://forums.docker.com/t/how-to-reach-localhost-on-host-from-docker-container/113321

