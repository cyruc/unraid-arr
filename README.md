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

## Getting Started

### 1. Prepare Unraid

1. **Enable Docker** — Settings > Docker > Enable Docker: Yes
2. **Install NerdTools** — Apps > search "NerdTools" > install
3. **Enable Python 3** — Settings > Nerd Tools > toggle Python 3 on
4. **Enable SSH** — Settings > Management Access > enable SSH
5. **Enable hardlinks** — Settings > Global Share Settings > Tunable (support Hard Links): Yes
6. **Create the data share** — Shares > Add Share > name it `data`

### 2. Set up your workstation

```bash
# Install Ansible
brew install ansible  # macOS
# or: sudo apt install ansible  # Debian/Ubuntu

# Install the Docker collection
ansible-galaxy collection install community.docker

# Clone the repo
git clone git@github.com:cyruc/unraid-arr.git
cd unraid-arr
```

### 3. Configure

```bash
# Set your Unraid IP
nano inventory.yml

# Create and encrypt your secrets
cp vault.yml.example vault.yml
nano vault.yml  # fill in your PIA credentials and WireGuard private key
ansible-vault encrypt vault.yml
```

### 4. Deploy

```bash
ansible-playbook arr-stack.yml -i inventory.yml -e @vault.yml --ask-vault-pass
```

This creates all directories, Docker networks, and containers automatically.

### 5. Configure the apps

**qBittorrent** (`:8080`)
- Set default save path to `/data/torrents`
- Create category `movies` with path `/data/torrents/movies`
- Create category `tv` with path `/data/torrents/tv`

**Prowlarr** (`:9696`)
- Add your indexers
- Settings > Apps > add Radarr and Sonarr

**Radarr** (`:7878`)
- Set root folder to `/data/media/movies`
- Add qBittorrent as download client, set category to `movies`

**Sonarr** (`:8989`)
- Set root folder to `/data/media/tv`
- Add qBittorrent as download client, set category to `tv`

Search for something in Radarr or Sonarr and the full pipeline runs: Prowlarr finds it > qBittorrent downloads it through VPN > Radarr/Sonarr hardlinks it into your media library.

## Ports

| Port | Service |
|---|---|
| 8080 | qBittorrent (via Gluetun) |
| 9696 | Prowlarr (via Gluetun) |
| 7878 | Radarr |
| 8989 | Sonarr |
