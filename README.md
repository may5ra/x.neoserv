# X NeoServ Control Panel v3.0

A modern, full-stack IPTV/streaming management dashboard with advanced security, load balancing, and comprehensive audit logging.

## Features

- **User Management**: Full CRUD with ban/enable/disable, connection limits, playlist downloads (M3U/M3U Plus/Enigma2)
- **Stream Management**: Live TV streams with real-time status, technical info (resolution, codecs, FPS, bitrate)
- **VOD & Series**: Movies and TV series management with categories
- **Bouquets**: Content packages for organized channel/VOD distribution
- **Reseller System**: Credit-based reseller accounts with filtered access
- **MAG Device Support**: Set-top box management with token security
- **Load Balancer**: Custom proxy with source URL hiding, HLS rewriting, caching
- **EPG Sources**: Electronic Program Guide management
- **Audit Logging**: Complete activity tracking for all admin/reseller actions
- **Security**: Rate limiting, IP whitelist/blacklist, connection enforcement
- **Dark/Light Theme**: Modern UI inspired by Vercel/Linear design

---

## System Requirements

- **OS**: Ubuntu 20.04/22.04/24.04 or Debian 11/12
- **RAM**: Minimum 2GB (4GB recommended)
- **Storage**: 20GB+ free space
- **CPU**: 2+ cores recommended
- **Ports**: 80 (default, configurable)

---

## Installation

### Option 1: Automated Installation (Recommended)

The fastest way to get started. Run as root:

```bash
# Create directory and download
mkdir -p /opt/neoserv
cd /opt/neoserv

# Download panel package
wget https://neoserv-panel.com/v2/neoserv-panel.zip

# Extract and run installer
unzip neoserv-panel.zip
chmod +x install.sh
sudo bash install.sh
```

The installer will prompt you for:
1. **Database password** - Strong password for PostgreSQL
2. **Admin password** - Password for the admin panel (username: `admin`)
3. **Panel port** - Default is 80

After installation, access your panel at: `http://YOUR_SERVER_IP`

---

### Option 2: Manual Installation

For those who prefer step-by-step control.

#### Step 1: Install System Dependencies

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install required packages
sudo apt install -y curl wget unzip git postgresql postgresql-contrib

# Install Node.js 20
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo bash -
sudo apt install -y nodejs

# Verify installation
node -v  # Should show v20.x.x
npm -v   # Should show 10.x.x
```

#### Step 2: Setup PostgreSQL Database

```bash
# Start PostgreSQL
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Create database user and database
sudo -u postgres psql -c "CREATE USER neoserv WITH ENCRYPTED PASSWORD 'YOUR_DB_PASSWORD';"
sudo -u postgres psql -c "CREATE DATABASE neoserv_panel OWNER neoserv;"
sudo -u postgres psql -d neoserv_panel -c "GRANT ALL ON SCHEMA public TO neoserv;"
```

Replace `YOUR_DB_PASSWORD` with a strong password.

#### Step 3: Download and Extract Panel

```bash
# Create installation directory
sudo mkdir -p /opt/neoserv
cd /opt/neoserv

# Download panel (or upload manually)
wget -O neoserv-panel.zip https://neoserv-panel.com/v2/neoserv-panel.zip
unzip neoserv-panel.zip
```

#### Step 4: Configure Environment

```bash
# Generate session secret
SESSION_SECRET=$(openssl rand -hex 32)

# Create .env file
cat > /opt/neoserv/.env <<EOF
DATABASE_URL=postgresql://neoserv:YOUR_DB_PASSWORD@localhost:5432/neoserv_panel
SESSION_SECRET=${SESSION_SECRET}
NODE_ENV=production
PORT=80
ADMIN_PASSWORD=YOUR_ADMIN_PASSWORD
EOF

# Secure the file
chmod 600 /opt/neoserv/.env
```

Replace:
- `YOUR_DB_PASSWORD` with your PostgreSQL password
- `YOUR_ADMIN_PASSWORD` with your desired admin panel password

#### Step 5: Install Dependencies and Build

```bash
cd /opt/neoserv

# Install Node.js packages
npm install

# Initialize database schema
npm run db:push

# Build the application
npm run build
```

#### Step 6: Create Systemd Service

```bash
# Create service file
cat > /etc/systemd/system/neoserv.service <<EOF
[Unit]
Description=X NeoServ Panel
After=network.target postgresql.service

[Service]
Type=simple
User=root
WorkingDirectory=/opt/neoserv
EnvironmentFile=/opt/neoserv/.env
ExecStart=/usr/bin/node dist/index.js
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF

# Enable and start service
sudo systemctl daemon-reload
sudo systemctl enable neoserv
sudo systemctl start neoserv
```

#### Step 7: Verify Installation

```bash
# Check service status
sudo systemctl status neoserv

# View logs if needed
journalctl -u neoserv -f
```

Access your panel at: `http://YOUR_SERVER_IP`
- **Username**: admin
- **Password**: (what you set in ADMIN_PASSWORD)

---

## Updating

To update an existing installation:

```bash
cd /opt/neoserv

# Upload new neoserv-panel-v3.0.zip to /opt/neoserv/
# Then run:
chmod +x UPDATE.sh
sudo bash UPDATE.sh
```

Or manually:

```bash
cd /opt/neoserv

# Stop service
sudo systemctl stop neoserv

# Backup .env
cp .env .env.backup

# Extract new files (will not overwrite .env)
unzip -o neoserv-panel-v3.0.zip

# Restore .env if needed
cp .env.backup .env

# Update dependencies and database
npm install
npm run db:push
npm run build

# Restart service
sudo systemctl start neoserv
```

---

## Useful Commands

| Command | Description |
|---------|-------------|
| `systemctl status neoserv` | Check panel status |
| `systemctl restart neoserv` | Restart panel |
| `systemctl stop neoserv` | Stop panel |
| `systemctl start neoserv` | Start panel |
| `journalctl -u neoserv -f` | View live logs |
| `journalctl -u neoserv --since "1 hour ago"` | View recent logs |

---

## Configuration

### Environment Variables

Located in `/opt/neoserv/.env`:

| Variable | Description |
|----------|-------------|
| `DATABASE_URL` | PostgreSQL connection string |
| `SESSION_SECRET` | Secret for session encryption (auto-generated) |
| `NODE_ENV` | Set to `production` |
| `PORT` | Panel port (default: 80) |
| `ADMIN_PASSWORD` | Initial admin password |

### Changing Panel Port

1. Edit `/opt/neoserv/.env` and change `PORT=80` to desired port
2. Restart: `sudo systemctl restart neoserv`

### Firewall Configuration

If using UFW:

```bash
sudo ufw allow 80/tcp   # HTTP
sudo ufw allow 443/tcp  # HTTPS (if using SSL)
sudo ufw reload
```

---

## Nginx Reverse Proxy (Optional)

If you want SSL/HTTPS with a domain, first change the panel port to 5000:

```bash
# Edit .env and set PORT=5000
nano /opt/neoserv/.env
sudo systemctl restart neoserv
```

Then configure Nginx:

```nginx
server {
    listen 80;
    server_name panel.yourdomain.com;
    
    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Install SSL with Certbot:

```bash
sudo apt install nginx certbot python3-certbot-nginx
sudo certbot --nginx -d panel.yourdomain.com
```

---

## Troubleshooting

### Panel won't start

```bash
# Check logs for errors
journalctl -u neoserv -n 50 --no-pager

# Verify database connection
cd /opt/neoserv
node -e "require('pg').Pool({connectionString: process.env.DATABASE_URL}).query('SELECT 1')"

# Check if port is in use
netstat -tlnp | grep 80
```

### Database connection failed

```bash
# Verify PostgreSQL is running
sudo systemctl status postgresql

# Test connection manually
psql -U neoserv -h localhost -d neoserv_panel
```

### Permission denied errors

```bash
# Ensure proper ownership
sudo chown -R root:root /opt/neoserv
chmod 600 /opt/neoserv/.env
```

---

## Directory Structure

```
/opt/neoserv/
├── .env                 # Environment configuration
├── package.json         # Node.js dependencies
├── dist/                # Compiled application
│   └── index.js         # Main entry point
├── client/              # Frontend source
├── server/              # Backend source
├── shared/              # Shared types/schemas
├── install.sh           # Installation script
└── UPDATE.sh            # Update script
```

---

## License

X NeoServ Control Panel requires a valid license for use.

After installation, navigate to **Settings > License** in the admin panel and enter your license key.

---

## Support

For support, feature requests, or to purchase a license:

**Telegram**: [@neoserv_me](https://t.me/neoserv_me)

---

**X NeoServ v3.0** - Developed by NeoServ Solutions

**DISCLAIMER**: This software is intended solely for managing legal IPTV content. The developer does not provide, distribute or support illegal streaming content. The user is fully responsible for the content transmitted through this software and must ensure compliance with all applicable copyright and broadcasting laws in their jurisdiction. The developer assumes no liability for unlawful use of this software.

2026 All rights reserved. Unauthorized use is prohibited.
