## 👋 Welcome to proxmox 🚀

<<<<<<< HEAD
This project provides a full **Proxmox VE** cluster solution running entirely within Docker containers.  
It includes a Datacenter Manager (PDM) and three Proxmox VE nodes, simulating a real-world cluster environment for testing and development.

## � Prerequisites
- **Linux Host**: Required for cgroup and kernel module mapping.
- **Docker**: With `docker-compose` support.
- **Hardware**: At least 4GB RAM recommended (1GB per node assigned).
- **Kernel Modules**:
  - `kvm` (Required)
  - `vhost_net` (Optional, for network acceleration. Uncomment in `docker-compose.yaml` if loaded on host)

## 🚀 Quick Start
Start the cluster with:
```bash
docker compose up -d
```

## 🖥️ Services & Access

| Node | Service | Internal IP | IPv6 | SSH Port | HTTP/HTTPS Port | Proxy/SPICE Port |
|------|---------|-------------|------|----------|-----------------|------------------|
| **Manager** (`pdm`) | Proxmox Datacenter Manager | `10.0.99.1` | `fd00::1` | `2222` | `8443` (https) | N/A |
| **Node 1** (`pve-1`) | Proxmox VE | `10.0.99.2` | `fd00::2` | `2223` | `8006` (https) | `3128` |
| **Node 2** (`pve-2`) | Proxmox VE | `10.0.99.3` | `fd00::3` | `2224` | `8006` (https) | `3129` |
| **Node 3** (`pve-3`) | Proxmox VE | `10.0.99.4` | `fd00::4` | `2225` | `8006` (https) | `3130` |

> **Note**: 
> - **Manager** (`pdm`): Access via `https://172.17.0.1:8443` or your host's Docker bridge IP.
> - **Node 1** (`pve-1`): Access via `https://172.17.0.1:8006` or your host's Docker bridge IP.
> - **Node 2** (`pve-2`): Access via `https://172.17.0.1:8007` or your host's Docker bridge IP.
> - **Node 3** (`pve-3`): Access via `https://172.17.0.1:8008` or your host's Docker bridge IP.
> - **SSH & Web UI** ports are bound specifically to `172.17.0.1`. Access them via `https://172.17.0.1:<port>` or your host's Docker bridge IP.
> - **Proxy/SPICE** ports (`3128-3130`) are bound to all interfaces (`0.0.0.0`).
> - Node 1 Web UI is mapped to host port `8006`, Node 2 to `8007`, and Node 3 to `8008`.

## 💾 Storage & Volumes
The nodes share storage volumes to simulate shared cluster resources:
- `/var/lib/vz/dump`: Shared backups.
- `/var/lib/vz/template/iso`: Shared ISO images.

System volumes mapped from host (Read-Only where applicable):
- `/sys/fs/cgroup`: Required for systemd.
- `/usr/lib/modules`: Required for loading kernel modules.

## 🌐 Networking
The cluster operates on a **dual-stack** Docker network (`dual_stack`):
- **IPv4 Subnet**: `10.0.99.0/24` (Gateway: `.99`)
- **IPv6 Subnet**: `fd00::/64` (Gateway: `::99`)

## 🔒 Nginx Reverse Proxy Example

Here is a comprehensive Nginx configuration to proxy these services. It handles the required WebSocket headers for Proxmox consoles.

```nginx
# Define upstreams using the host mappings
upstream proxmox_pdm    { server 172.17.0.1:8443; }
upstream proxmox_node_1 { server 172.17.0.1:8006; }
upstream proxmox_node_2 { server 172.17.0.1:8007; }
upstream proxmox_node_3 { server 172.17.0.1:8008; }

map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}

server {
    listen 443 ssl;
    server_name pdm.*;
    
    ssl_certificate /etc/letsencrypt/live/domain/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/domain/privkey.pem;

    location / {
        proxy_pass https://proxmox_pdm;
        proxy_ssl_verify off;
        
        # Standard Proxy Headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # WebSocket Headers (Critical for VNC/SPICE)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
    }
}

```

## Author  
=======
proxmox - Self-hosted Docker Compose deployment

## 📋 Description

Proxmox is a containerized service deployed using Docker Compose. This setup provides a complete, production-ready deployment with proper security defaults, logging, and configuration management.

## 🚀 Services

This deployment includes the following services:

- **pve-1**: Service container
- **pve-2**: Service container
- **pve-3**: Service container
- **pdm**: Service container

## 📦 Installation

### Using curl

```shell
curl -q -LSsf "https://raw.githubusercontent.com/composemgr/proxmox/main/docker-compose.yaml" -o compose.yml
```

### Using git

```shell
git clone "https://github.com/composemgr/proxmox" ~/.local/srv/docker/proxmox
cd ~/.local/srv/docker/proxmox
docker compose up -d
```

### Using composemgr

```shell
composemgr install proxmox
```

## 🔧 Configuration

### Directory Structure

The project follows a standardized rootfs layout:

```
.
├── docker-compose.yaml
└── rootfs/
    ├── config/          # Application configuration files
    ├── data/            # Application data and logs
```

### Environment Variables

Key environment variables (with defaults):

```shell
# Core Settings
TZ=America/New_York                    # Timezone
BASE_HOST_NAME=${HOSTNAME}             # Hostname for the service
BASE_DOMAIN_NAME=                      # Domain name (optional)
```

All variables have sane defaults and can be overridden via `.env` or `app.env` files.

## 🌐 Access

- **Web Interface**: http://172.17.0.1:8006
- **Production**: Configure your reverse proxy to forward to port 8006

For production deployments, use a reverse proxy (nginx, traefik, caddy) to handle SSL/TLS.

## 📂 Volumes

Data persistence locations:

- `./rootfs/config/` - Application configuration
- `./rootfs/data/` - Application data and logs

## 🔐 Security

- All secrets use secure defaults with `changeme_*` prefix for easy identification
- No hardcoded passwords in compose file  
- Environment-based configuration for sensitive data
- Logging configured with rotation (5MB max, 1 file retained)

## 🔍 Logging

All services use standardized logging:
- **Driver**: json-file
- **Max Size**: 5MB per file
- **Max Files**: 1 (rotated)

View logs:
```shell
docker compose logs -f
docker compose logs -f [service_name]
```

## 🛠️ Management

### Start services
```shell
docker compose up -d
```

### Stop services
```shell
docker compose down
```

### Restart services
```shell
docker compose restart
```

### Update images
```shell
docker compose pull
docker compose up -d
```

### View status
```shell
docker compose ps
```

### Execute commands in container
```shell
docker compose exec [service_name] [command]
```

## 🔄 Backup & Restore

### Backup
```shell
# Backup volumes
tar -czf proxmox-backup-$(date +%Y%m%d).tar.gz rootfs/
```

### Restore
```shell
# Restore from backup
tar -xzf proxmox-backup-YYYYMMDD.tar.gz
docker compose up -d
```

## 📋 Requirements

- Docker Engine 20.10+
- Docker Compose V2+
- Sufficient disk space for data and logs

## 🆘 Troubleshooting

### Check service status
```shell
docker compose ps
```

### View detailed logs
```shell
docker compose logs --tail=100 -f
```

### Restart a specific service
```shell
docker compose restart [service_name]
```

### Reset and start fresh
```shell
docker compose down -v
docker compose up -d
```

## 🤝 Author
>>>>>>> bcb1fdf462eb (🗃️ Major updates 🗃️)

🤖 casjay: [Github](https://github.com/casjay) 🤖  
🦄 composemgr: [Github](https://github.com/composemgr) 🦄
