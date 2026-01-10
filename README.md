# X NeoServ IPTV Panel

## The Most Advanced IPTV Management Solution

**Support: Telegram @NEOSERV_ME**

---

## Why X NeoServ is the BEST Choice

### Security First
- Stream source URLs are NEVER exposed to end users
- Tokenized authentication with HMAC-SHA256 signatures
- Rate limiting protection (30 req/min with auto-block)
- IP whitelist/blacklist enforcement
- Connection limit enforcement per user
- Path traversal protection
- No source URLs in browser network inspector

### Lightning Fast Performance
- Instant stream opening (under 1 second)
- Optimized proxy buffering (512KB buffers)
- 2-second connection timeout for fast failover
- CDN-compatible with redirect following
- Segment caching for instant replay

### Complete Device Support
- MAG/STB Emu (Stalker Portal API)
- VLC Media Player
- GSE Smart IPTV
- Perfect Player
- TiviMate
- Enigma2/Dreambox
- Any M3U/M3U8 compatible player

### Professional Admin Features
- Modern dashboard with real-time statistics
- User management with expiration dates
- Live TV stream management
- VOD Movies library
- TV Series with episodes
- Category organization
- Bouquet/Package system
- Multi-server load balancing
- EPG (TV Guide) management
- Active connection monitoring
- Admin/Reseller hierarchy

---

## Installation

### Requirements
- Ubuntu 20.04+ or Debian 11+
- 2GB RAM minimum (4GB recommended)
- 2 CPU cores minimum
- 20GB disk space
- Root access

### One-Click Install

```bash
# Download and extract
wget https://your-download-link/neoserv-panel.zip
unzip neoserv-panel.zip
cd neoserv-panel

# Run installer
chmod +x install.sh
sudo ./install.sh
```

### Manual Installation

1. Install dependencies:
```bash
sudo apt update
sudo apt install -y nodejs npm postgresql nginx
```

2. Setup PostgreSQL:
```bash
sudo -u postgres createuser neoserv
sudo -u postgres createdb neoserv_panel -O neoserv
```

3. Configure environment:
```bash
cp .env.example .env
# Edit .env with your database credentials
```

4. Install and build:
```bash
npm install
npm run build
```

5. Start server:
```bash
npm start
```

---

## Features Overview

### Dashboard
- Total users, streams, movies, series count
- Active connections in real-time
- Server status monitoring
- Quick access to all sections

### User Management
- Create unlimited users
- Set expiration dates
- Connection limits
- Bouquet assignments
- Enable/Disable/Ban users
- Kill active connections
- Download playlists (M3U, M3U Plus, Enigma2)
- Trial account support

### Live TV Streams
- Add unlimited channels
- Assign categories
- EPG channel mapping
- Stream health monitoring
- Start/Stop/Reload controls
- Stream ordering
- Resolution/codec info display

### VOD Movies
- Full movie library
- Category organization
- Movie metadata (year, rating, description)
- Poster/cover images
- Container format support

### TV Series
- Series with seasons
- Episode management
- Series metadata
- Cover images
- Episode ordering

### Categories
- Live TV categories
- Movie categories
- Series categories
- Custom ordering

### Bouquets (Packages)
- Create content packages
- Assign streams, movies, series
- User-bouquet mapping
- Package ordering
- Content deduplication

### Servers
- Main server configuration
- Load balancer support
- Server health monitoring
- SSH credentials storage
- Server status display

### EPG Sources
- Multiple EPG sources
- Auto-refresh scheduling
- Channel mapping
- XML/XMLTV support

### Connections
- Real-time active connections
- User/Stream/IP tracking
- Connection duration
- Kill individual connections

### Settings
- General settings
- Security configuration
- Streaming options
- Server settings
- API compatibility
- Transcoding options

### Admin Management
- Multiple admin accounts
- Reseller accounts
- Permission levels
- Admin activity logging

### MAG Device Support
- Full Stalker Portal API
- Device registration
- MAC address management
- STB Emu compatible

---

## API Endpoints

### Player API (XUI.ONE Compatible)
- `GET /player_api.php` - Full player API
- `GET /get.php` - Playlist generation
- `GET /live/{user}/{pass}/{stream}.ts` - Live streams
- `GET /movie/{user}/{pass}/{movie}.mp4` - VOD movies

### Stalker Portal API (MAG Devices)
- `POST /stalker_portal/server/load.php` - Portal API
- Full STB authentication
- Channel/VOD listings
- EPG data

### Admin API
- Full REST API for all management functions
- Token-based authentication
- Rate limiting protection

---

## Cloudflare Compatible

X NeoServ supports dual-domain Cloudflare setup:

- **Panel Domain** - Protected by Cloudflare (orange cloud proxy enabled)
- **Streaming Domain** - DNS only (gray cloud, no proxy)

This ensures:
- Panel is protected from DDoS attacks
- Streaming works without Cloudflare blocking video traffic
- Server IP remains hidden behind Cloudflare

Configure in Settings > Domains tab after installation.

---

## Architecture

### Hybrid Model
- **Control Plane**: Panel handles authentication, user management, token generation
- **Data Plane**: External Nginx proxy handles stream delivery for production performance

### Security Flow
1. User requests stream via playlist URL
2. Nginx extracts token and signature
3. Panel validates credentials via auth_request
4. Panel returns source URL in header (never to client)
5. Nginx proxies stream from source
6. Client receives stream without seeing source

---

## Auto-Restart Configuration

The panel automatically restarts on server reboot via systemd:

```bash
# Service is enabled by default
sudo systemctl enable neoserv

# Manual control
sudo systemctl start neoserv
sudo systemctl stop neoserv
sudo systemctl restart neoserv
sudo systemctl status neoserv

# View logs
sudo journalctl -u neoserv -f
```

---

## SSL Certificate

For production, always use HTTPS:

```bash
sudo certbot --nginx -d yourdomain.com
```

---

## Load Balancer Setup (True Resource Distribution)

X NeoServ uses a **true load balancing architecture** where LB servers fetch streams directly from source URLs - properly distributing bandwidth, CPU, and RAM across multiple servers.

### How It Works
1. User requests stream from LB server
2. LB server validates user with main panel (lightweight API call)
3. LB server fetches HLS content **directly from source URL**
4. Stream is served to user from LB server's bandwidth
5. LB reports CPU/memory metrics to main panel every 30 seconds

### Setup via Panel
1. Go to **Servers** page in admin panel
2. Add new server with SSH credentials (root access required)
3. Set server type to "Load Balancer"
4. Click **Install LB** button
5. Panel automatically installs: Node.js 20, Nginx, LB Proxy service
6. Server is ready to handle streams!

### What Gets Installed on LB Server
- Nginx (reverse proxy for HLS)
- Node.js 20 (LB proxy script)
- Custom proxy service (systemd managed)
- Secure token authentication

### Monitoring
- Real-time CPU/Memory/Disk usage in Servers page
- Active connections per LB server
- Automatic failover if LB goes offline

---

## Backup & Restore

### Backup Database
```bash
pg_dump neoserv_panel > backup.sql
```

### Restore Database
```bash
psql neoserv_panel < backup.sql
```

---

## Troubleshooting

### Panel not starting
```bash
sudo journalctl -u neoserv -n 50
```

### Database connection issues
```bash
sudo systemctl status postgresql
```

### Nginx errors
```bash
sudo nginx -t
sudo tail -f /var/log/nginx/error.log
```

---

## Support

For all support inquiries:

**Telegram: @NEOSERV_ME**

---

## License

X NeoServ IPTV Panel - Professional IPTV Management Solution

Copyright (c) 2025 NeoServ

---

*Built with performance, security, and reliability in mind.*
