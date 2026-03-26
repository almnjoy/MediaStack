# 🎬 Media Stack (Plex + Automation + Requests)

A clean, modern, self-hosted media automation stack using Docker.

This setup allows users to:
- Request movies and TV shows via a web UI
- Automatically search and download content
- Organize media into a Plex library
- Stream content through Plex
- Keep torrent traffic behind a VPN

---

# 🧠 Overview

This stack is built around a simple flow:
User → Seerr → Radarr/Sonarr → Prowlarr → qBittorrent (VPN) → Import → Plex


### What this means:
- Users request content in **Seerr**
- **Radarr (movies)** or **Sonarr (TV)** handle automation
- **Prowlarr** provides indexers (Yts, ez.tv, etc.)
- **qBittorrent** downloads content (through VPN)
- Media is imported into organized folders
- **Plex** serves the final content

---

# 🧱 Services Used

| Service       | Purpose |
|--------------|--------|
| gluetun      | VPN container for torrent traffic |
| prowlarr     | Indexer manager |
| qbittorrent  | Torrent client |
| radarr       | Movie automation |
| sonarr       | TV automation |
| seerr        | Request UI (user-facing) |
| tautulli     | Plex monitoring |

---

# ⚙️ Requirements

## Hardware / Host
- Linux host (recommended: Ubuntu/Debian)
- Docker + Docker Compose installed
- Enough storage for media

## External Requirements
- Plex server (local or remote)
- VPN provider (for Gluetun)
- Indexers (public or private trackers)
- Domain (optional, for web access)
- Reverse proxy (Traefik recommended)

---

# 📦 Folder Structure (Host)
/opt/docker/media-stack/
├── docker-compose.yml
├── gluetun/
├── prowlarr/
├── qbittorrent/
├── radarr/
├── sonarr/
├── seerr/
├── tautulli/


---

# 💾 Media Storage
Media is stored on a shared location (local disk or network share).

Example:
/mnt/media
├── downloads
│ ├── movies
│ └── tv
├── Movies
└── TV

---
# 🔗 Container Path Mapping
All containers use:
/mnt/media → /media


So inside apps:
/media/downloads
/media/downloads/movies
/media/downloads/tv
/media/Movies
/media/TV


---

# 🚀 Docker Compose (Example)

```yaml
version: "3.8"

services:

  gluetun:
    image: qmcgaw/gluetun
    container_name: gluetun
    cap_add:
      - NET_ADMIN
    environment:
      - VPN_SERVICE_PROVIDER=yourvpn
      - OPENVPN_USER=youruser
      - OPENVPN_PASSWORD=yourpass
    volumes:
      - ./gluetun:/gluetun
    restart: unless-stopped

  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent
    container_name: qbittorrent
    network_mode: "service:gluetun"
    environment:
      - PUID=1000
      - PGID=1000
    volumes:
      - ./qbittorrent:/config
      - /mnt/media:/media
    restart: unless-stopped

  prowlarr:
    image: lscr.io/linuxserver/prowlarr
    container_name: prowlarr
    environment:
      - PUID=1000
      - PGID=1000
    volumes:
      - ./prowlarr:/config
    ports:
      - "9696:9696"
    restart: unless-stopped

  radarr:
    image: lscr.io/linuxserver/radarr
    container_name: radarr
    environment:
      - PUID=1000
      - PGID=1000
    volumes:
      - ./radarr:/config
      - /mnt/media:/media
    ports:
      - "7878:7878"
    restart: unless-stopped

  sonarr:
    image: lscr.io/linuxserver/sonarr
    container_name: sonarr
    environment:
      - PUID=1000
      - PGID=1000
    volumes:
      - ./sonarr:/config
      - /mnt/media:/media
    ports:
      - "8989:8989"
    restart: unless-stopped

  seerr:
    image: ghcr.io/seerr-team/seerr:latest
    container_name: seerr
    volumes:
      - ./seerr:/app/config
    ports:
      - "5055:5055"
    restart: unless-stopped

  tautulli:
    image: lscr.io/linuxserver/tautulli
    container_name: tautulli
    volumes:
      - ./tautulli:/config
    ports:
      - "8181:8181"
    restart: unless-stopped


🔁 Application Setup

1. qBittorrent

Set paths:

/media/downloads
/media/downloads/movies
/media/downloads/tv

Create categories:

movies → /media/downloads/movies
tv → /media/downloads/tv
2. Radarr
Root folder:
/media/Movies
Enable:
Use Hardlinks Instead of Copy
3. Sonarr
Root folder:
/media/TV
Enable:
Use Hardlinks Instead of Copy
4. Prowlarr
Add indexers
Sync to Radarr + Sonarr
5. Seerr
Connect Plex
Add Radarr + Sonarr
Set root folders:
/media/Movies
/media/TV
6. Plex
Point libraries to:
/media/Movies
/media/TV
🔐 VPN Setup
qBittorrent runs through Gluetun
Other services do NOT use VPN
Keeps torrent traffic isolated
🌐 Optional: Reverse Proxy

Use Traefik or Nginx to expose:

request.yourdomain.com → Seerr

Users:

login with Plex
request content
🔄 Media Flow
Movie Example
User → Seerr → Radarr → qBittorrent
→ /media/downloads/movies
→ /media/Movies
→ Plex
TV Example
User → Seerr → Sonarr → qBittorrent
→ /media/downloads/tv
→ /media/TV
→ Plex
💡 Important Notes
Hardlinks

Files appear twice but are not duplicated on disk.

Paths Must Match

Always use /media/... inside containers.

Folder Creation

Make sure these exist:

/media/downloads/movies
/media/downloads/tv
⚠️ Common Issues
Issue	Cause
Permission denied	Wrong folder ownership
Downloads fail	Incorrect path
No results	Indexer issues
Import fails	Wrong root folder
