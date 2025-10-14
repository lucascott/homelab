# Home Lab

My personal homelab setup. Nothing fancy but it gets the job done 😎. Reliable, secure, and easy to maintain.

For more details on my hardware specifications see [here](#my-current-hardware-specifications).

## Architecture Overview

The infrastructure is built around Docker containers orchestrated using Docker Compose.
All services are exposed with Traefik as a reverse proxy, providing SSL termination and automatic service discovery via Docker container labels.

The homelab uses two main Docker networks:
- `reverse-proxy` - For services exposed through Traefik reverse proxy
- `prometheus` - For services integrating with Prometheus for monitoring and metrics collection

Web services are organized with hostname-based routing (*.lucascott.com). They are automatically discovered by Traefik using Docker labels. Internal-only services are protected with the `internalOnly@file` middleware which restricts access to the local network and Tailscale IP range.
In addition, public internet-facing services are protected by Cloudflare's WAF and DDoS protection.

All web services exposed via Traefik use the same wildcard Let's Encrypt certificate with DNS-01 challenge via Cloudflare.
This allows secure access to all public and internal only services without needing to manage individual certificates.


### Third-Party Services
* [Cloudflare](https://www.cloudflare.com/en-gb/) for domain registration and public DNS hosting. Cloudflare's free tier provides sufficient features for my needs.
* [Tailscale](https://tailscale.com/) for secure remote access to internal services when away from home.
* GitHub + Github Actions to host all my code and automate deployment of my personal website.
* [Srenity.online](https://www.srenity.online/) to find serenity in this self-hosting journey!

## Quick Start

1. **Initial Setup**
   ```bash
   ./setup.sh
   ```
   This creates the required Docker networks (`reverse-proxy` and `prometheus`).

2. **Service Configuration**
   Each service directory contains its own `compose.yaml` and configuration files. Navigate to the specific service directory and run:
   ```bash
   docker compose up -d
   ```

## Services

### Public Services
| Service                                                     | Description                  | Directory                                                                    |
|-------------------------------------------------------------|------------------------------|------------------------------------------------------------------------------|
| [**www.lucascott.com**](https://www.lucascott.com)          | My personal website          | [lucascott.github.io repo](https://github.com/lucascott/lucascott.github.io) |
| [**mixes.lucascott.com**](https://mixes.lucascott.com)      | My DJ mixes archive          | Coming soon!                                                                 |
| [**YourSpotify**](https://github.com/Yooooomi/your_spotify) | Spotify listening statistics | [`yourspotify/`](./yourspotify/)                                             |

### Infrastructure & Core Services

| Service                                                        | Description                                                        | Directory                |
|----------------------------------------------------------------|--------------------------------------------------------------------|--------------------------|
| [**Traefik**](https://github.com/traefik/traefik)              | Reverse proxy with automatic SSL and service discovery             | [`traefik/`](./traefik/) |
| [**AdGuard Home**](https://github.com/AdguardTeam/AdGuardHome) | DNS server for ad and tracker blocking and internal domain routing | [`adguard/`](./adguard/) |

### Monitoring & Observability

| Service                                                       | Description                         | Directory                      |
|---------------------------------------------------------------|-------------------------------------|--------------------------------|
| [**Prometheus**](https://prometheus.io/)                      | Metrics collection and storage      | [`prometheus/`](./prometheus/) |
| [**Grafana**](https://grafana.com/oss/grafana/)               | Metrics visualization dashboard     | [`grafana/`](./grafana/)       |
| [**What's Up Docker (WUD)**](https://getwud.github.io/wud/#/) | Container update monitoring service | [`wud/`](./wud/)               |
| [**Dockge**](https://github.com/louislam/dockge)              | Docker Compose stack manager        | [`dockge/`](./dockge/)         |
| [**Portainer**](https://github.com/portainer/portainer)       | Docker container management UI      | [`portainer/`](./portainer/)   |

### Security & Privacy

| Service                                                       | Description                            | Directory                        |
|---------------------------------------------------------------|----------------------------------------|----------------------------------|
| [**Vaultwarden**](https://github.com/dani-garcia/vaultwarden) | Self-hosted Bitwarden password manager | [`vaultwarden/`](./vaultwarden/) |

### Media & Personal Services

| Service                                                     | Description                        | Directory                          |
|-------------------------------------------------------------|------------------------------------|------------------------------------|
| [**Homepage**](https://gethomepage.dev/)                    | Homepage dashboard for services    | [`homepage/`](./homepage/)         |
| [**Immich**](https://immich.app/)                           | Self-hosted photo and video backup | [`immich/`](./immich/)             |
| [**SilverBullet**](https://silverbullet.md/)                | Personal knowledge management      | [`silverbullet/`](./silverbullet/) |

### Smart Home & IoT

| Service                            | Description                               | Directory                |
|------------------------------------|-------------------------------------------|--------------------------|
| [**ESPHome**](https://esphome.io/) | ESP32 home environment monitoring station | [`esphome/`](./esphome/) |

## Security Considerations

- **Internal Services**: All services not meant to be publicly exposed are configured with IP whitelisting via Traefik's `internalOnly@file` middleware. This restricts access to the local network and Tailscale IP range only.
- **SSL/TLS**: All web services are exposed via Traefik and use a Let's Encrypt wildcard certificate managed by Traefik.
- **Firewall**: The host machine runs a firewall allowing only necessary incoming ports (80, 443, 53 if using AdGuard Home).
- **User Permissions**: Docker containers run with least privilege, avoiding root user where possible.
- **Regular Updates**: Services are monitored for updates using WUD, and updates are applied regularly to ensure security patches are in place.
- **Strong Passwords**: Services requiring authentication (e.g., Vaultwarden, Portainer) are secured with strong, unique passwords.
- **Secrets Management**: Sensitive data stored in Docker secrets files (e.g., Cloudflare API tokens) in the host machine with restricted user permissions.
- **Network Isolation**: Services use dedicated networks (created automatically by Docker Compose) for secure internal communication. When necessary, individual services are connected to both `reverse-proxy` and `prometheus` networks.

## Reliability & Backups
- **Data Persistence**: Critical services use Docker volumes for data persistence in the host filesystem.
- **Backups**: Regular backups of important data are performed using custom scripts (not included in this repository) and stored offsite.
- **Uptime monitoring**: Uptime Kuma (not included in this repository yet) monitors public and internal services for availability and sends alerts via Telegram.
- **Failure recovery**: In case of a service failure, the service is automatically restarted based on Docker Compose `restart` policy.

## Updates
The What's Up Docker (WUD) service monitors for container updates. 
Update notifications are sent to me by a personal Telegram bot integrated with WUD.

Most services are configured with:
- `wud.watch.digest=true` - Monitor for image updates
- `wud.tag.include/exclude` - Control which tags to monitor
- `wud.link.template` - Direct links to release pages

Updates can be applied via WUD UI's Docker trigger button, via Dockge UI, or manually by navigating to the service directory and running:
```bash
cd <service-directory>
docker compose pull && docker compose up -d --remove-orphans
```

## Monitoring
- **Prometheus**: Collects metrics from cAdvisor and other exporters
- **Grafana**: Visualizes system and application metrics
- **Logs**: Container logs available via `docker compose logs`

## Contributing

When adding new services:
1. Create a new directory with descriptive name
2. Include `compose.yaml` with proper network configuration
3. Add Traefik labels for automatic routing
4. Create service-specific README with setup instructions
5. Update this main README with service information

## Future Improvements
- Include missing services in this repository:
  - Uptime Kuma configuration for service uptime monitoring
  - Backup scripts/configuration and documentation
- Explore hosting an SSO identity provider (e.g., Keycloak) and integrate it with services for unified authentication

## My current hardware specifications

### Main Server

**HP EliteDesk 800 65W G3 Mini**
- **Host OS**: Debian GNU/Linux 12 (bookworm) on bare metal
- **CPU**: Intel(R) Core(TM) i5-7500 CPU @ 3.40GHz (4 cores / 4 threads)
- **RAM**: 16GB
- **Storage**:
  - 256GB NVMe SSD for OS and Docker containers
  - 500GB SSD for Docker volumes and persistent data

### Secondary Servers

**Raspberry Pi 3b+**
- **Host OS**: Raspberry Pi OS Lite 64-bit - Debian 12 (bookworm)
- **Storage**: 32GB microSD card
- **Purpose**: Dedicated AdGuard Home DNS server for Tailscale clients

**Raspberry Pi 3b+**
- **Host OS**: Raspberry Pi OS Lite 64-bit - Debian 12 (bookworm)
- **Storage**: 
  - 32GB microSD card
  - External 1TB USB SSD for media storage backup
- **Purpose**: Offsite backup server and uptime monitoring server

## License

This is a personal homelab configuration. Use at your own discretion.
