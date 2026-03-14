# unraid-arr

Ansible playbook to deploy a media automation stack on Unraid with VPN protection via Gluetun.

## Stack

| Service | Purpose | Network |
|---|---|---|
| **Gluetun** | VPN gateway (PIA WireGuard) | `vpn_net` |
| **qBittorrent** | Download client | `vpn_net` + `arr_backend` |
| **Prowlarr** | Indexer manager | `vpn_net` + `arr_backend` |
| **Radarr** | Movie management | `arr_backend` |
| **Sonarr** | TV show management | `arr_backend` |

## Network Architecture

- **`vpn_net`** — All traffic routed through Gluetun's VPN tunnel. qBittorrent and Prowlarr use this so download and indexer traffic is protected. If the VPN drops, these containers lose connectivity (kill switch).
- **`arr_backend`** — Internal network for ARR apps to communicate with each other and qBittorrent. Radarr and Sonarr live here with direct port access.

## Prerequisites

- Unraid server with Docker enabled
- Ansible with `community.docker` collection
- PIA subscription with WireGuard support
- Ansible Vault for secrets

## Secrets (ansible-vault)

```yaml
vault_pia_user: "your-pia-username"
vault_pia_pass: "your-pia-password"
vault_wg_key: "your-wireguard-private-key"
```

## Usage

```bash
ansible-playbook arr-stack.yml --ask-vault-pass
```

## Ports

| Port | Service |
|---|---|
| 8080 | qBittorrent (via Gluetun) |
| 9696 | Prowlarr (via Gluetun) |
| 7878 | Radarr |
| 8989 | Sonarr |

## Paths

| Path | Purpose |
|---|---|
| `/mnt/user/appdata/<service>` | Config data |
| `/mnt/user/media/movies` | Movie library |
| `/mnt/user/media/tv` | TV library |
| `/mnt/user/downloads` | Download directory |
