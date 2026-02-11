# AdGuard Home DNS

## AdGuard Home networking and Tailscale

Docker by default may set up a Docker proxy between the container and the host.
This leads to the scenario where all inbound connections from Tailscale appear from a single client IP (the proxy IP in the docker network the AdGuard Home container runs in),
thus leading to incorrect client level statistics.

To fix this issue, one has to enforce Docker to not use the proxy component. To do so one needs to follow these steps:
1. Edit `/etc/docker/daemon.json`:
   ```json
   {
     "userland-proxy": false
   }
   ```
1. Restart the Docker daemon
   ```shell
   sudo systemctl restart docker
   ```
