## 👋 Welcome to proxmox 🚀

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

🤖 casjay: [Github](https://github.com/casjay) 🤖  
🦄 composemgr: [Github](https://github.com/composemgr) 🦄
