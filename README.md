# unraid-arr

Ansible playbook to deploy a media automation stack on Unraid with VPN protection via Gluetun. Follows [TRaSH Guides](https://trash-guides.info/) best practices for folder structure and hardlinks.

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

## Folder Structure (TRaSH Guides)

Uses a single `/data` share to enable **hardlinks and atomic moves** — no extra disk space used when importing, and instant file operations instead of slow copy+delete.

```
/mnt/user/data/
├── torrents/
│   ├── movies/
│   └── tv/
└── media/
    ├── movies/
    └── tv/
```

| Container | Volume Mount | Why |
|---|---|---|
| qBittorrent | `/data/torrents` | Only needs torrent downloads |
| Radarr / Sonarr | `/data` | Needs both torrents + media for hardlinks |

**Important:** In Radarr/Sonarr, set the root folder to `/data/media/movies` and `/data/media/tv`. In qBittorrent, set the default save path to `/data/torrents` with category subfolders (`movies`, `tv`).

## Prerequisites

- Unraid server with Docker enabled
- Ansible with `community.docker` collection
- PIA subscription with WireGuard support
- Ansible Vault for secrets
- "Tunable (support Hard Links)" enabled in Unraid Settings > Global Share Settings

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
