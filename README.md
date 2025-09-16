# metabase Installation Guide

metabase is a free and open-source business intelligence. Metabase helps everyone make sense of their data with visualization

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 2+ cores
  - RAM: 2GB minimum
  - Storage: 5GB for data
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 3000 (default metabase port)
  - None
- **Dependencies**:
  - See official documentation for specific requirements
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install metabase
sudo dnf install -y metabase

# Enable and start service
sudo systemctl enable --now metabase

# Configure firewall
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload

# Verify installation
metabase --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install metabase
sudo apt install -y metabase

# Enable and start service
sudo systemctl enable --now metabase

# Configure firewall
sudo ufw allow 3000

# Verify installation
metabase --version
```

### Arch Linux

```bash
# Install metabase
sudo pacman -S metabase

# Enable and start service
sudo systemctl enable --now metabase

# Verify installation
metabase --version
```

### Alpine Linux

```bash
# Install metabase
apk add --no-cache metabase

# Enable and start service
rc-update add metabase default
rc-service metabase start

# Verify installation
metabase --version
```

### openSUSE/SLES

```bash
# Install metabase
sudo zypper install -y metabase

# Enable and start service
sudo systemctl enable --now metabase

# Configure firewall
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload

# Verify installation
metabase --version
```

### macOS

```bash
# Using Homebrew
brew install metabase

# Start service
brew services start metabase

# Verify installation
metabase --version
```

### FreeBSD

```bash
# Using pkg
pkg install metabase

# Enable in rc.conf
echo 'metabase_enable="YES"' >> /etc/rc.conf

# Start service
service metabase start

# Verify installation
metabase --version
```

### Windows

```bash
# Using Chocolatey
choco install metabase

# Or using Scoop
scoop install metabase

# Verify installation
metabase --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/metabase

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
metabase --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable metabase

# Start service
sudo systemctl start metabase

# Stop service
sudo systemctl stop metabase

# Restart service
sudo systemctl restart metabase

# Check status
sudo systemctl status metabase

# View logs
sudo journalctl -u metabase -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add metabase default

# Start service
rc-service metabase start

# Stop service
rc-service metabase stop

# Restart service
rc-service metabase restart

# Check status
rc-service metabase status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'metabase_enable="YES"' >> /etc/rc.conf

# Start service
service metabase start

# Stop service
service metabase stop

# Restart service
service metabase restart

# Check status
service metabase status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start metabase
brew services stop metabase
brew services restart metabase

# Check status
brew services list | grep metabase
```

### Windows Service Manager

```powershell
# Start service
net start metabase

# Stop service
net stop metabase

# Using PowerShell
Start-Service metabase
Stop-Service metabase
Restart-Service metabase

# Check status
Get-Service metabase
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream metabase_backend {
    server 127.0.0.1:3000;
}

server {
    listen 80;
    server_name metabase.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name metabase.example.com;

    ssl_certificate /etc/ssl/certs/metabase.example.com.crt;
    ssl_certificate_key /etc/ssl/private/metabase.example.com.key;

    location / {
        proxy_pass http://metabase_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName metabase.example.com
    Redirect permanent / https://metabase.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName metabase.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/metabase.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/metabase.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:3000/
    ProxyPassReverse / http://127.0.0.1:3000/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend metabase_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/metabase.pem
    redirect scheme https if !{ ssl_fc }
    default_backend metabase_backend

backend metabase_backend
    balance roundrobin
    server metabase1 127.0.0.1:3000 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R metabase:metabase /etc/metabase
sudo chmod 750 /etc/metabase

# Configure firewall
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status metabase

# View logs
sudo journalctl -u metabase -f

# Monitor resource usage
top -p $(pgrep metabase)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/metabase"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/metabase-backup-$DATE.tar.gz" /etc/metabase /var/lib/metabase

echo "Backup completed: $BACKUP_DIR/metabase-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop metabase

# Restore from backup
tar -xzf /backup/metabase/metabase-backup-*.tar.gz -C /

# Start service
sudo systemctl start metabase
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u metabase -n 100
sudo tail -f /var/log/metabase/metabase.log

# Check configuration
metabase --version

# Check permissions
ls -la /etc/metabase
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 3000

# Test connectivity
telnet localhost 3000

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep metabase)

# Check disk I/O
iotop -p $(pgrep metabase)

# Check connections
ss -an | grep 3000
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  metabase:
    image: metabase:latest
    ports:
      - "3000:3000"
    volumes:
      - ./config:/etc/metabase
      - ./data:/var/lib/metabase
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update metabase

# Debian/Ubuntu
sudo apt update && sudo apt upgrade metabase

# Arch Linux
sudo pacman -Syu metabase

# Alpine Linux
apk update && apk upgrade metabase

# openSUSE
sudo zypper update metabase

# FreeBSD
pkg update && pkg upgrade metabase

# Always backup before updates
tar -czf /backup/metabase-pre-update-$(date +%Y%m%d).tar.gz /etc/metabase

# Restart after updates
sudo systemctl restart metabase
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/metabase

# Clean old logs
find /var/log/metabase -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/metabase
```

## Additional Resources

- Official Documentation: https://docs.metabase.org/
- GitHub Repository: https://github.com/metabase/metabase
- Community Forum: https://forum.metabase.org/
- Best Practices Guide: https://docs.metabase.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
