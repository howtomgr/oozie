# oozie Installation Guide

oozie is a free and open-source workflow scheduler. Oozie provides workflow scheduler for Hadoop jobs

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
  - RAM: 4GB minimum
  - Storage: 10GB for data
  - Network: HTTP/REST
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 11000 (default oozie port)
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

# Install oozie
sudo dnf install -y oozie

# Enable and start service
sudo systemctl enable --now oozie

# Configure firewall
sudo firewall-cmd --permanent --add-port=11000/tcp
sudo firewall-cmd --reload

# Verify installation
oozie --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install oozie
sudo apt install -y oozie

# Enable and start service
sudo systemctl enable --now oozie

# Configure firewall
sudo ufw allow 11000

# Verify installation
oozie --version
```

### Arch Linux

```bash
# Install oozie
sudo pacman -S oozie

# Enable and start service
sudo systemctl enable --now oozie

# Verify installation
oozie --version
```

### Alpine Linux

```bash
# Install oozie
apk add --no-cache oozie

# Enable and start service
rc-update add oozie default
rc-service oozie start

# Verify installation
oozie --version
```

### openSUSE/SLES

```bash
# Install oozie
sudo zypper install -y oozie

# Enable and start service
sudo systemctl enable --now oozie

# Configure firewall
sudo firewall-cmd --permanent --add-port=11000/tcp
sudo firewall-cmd --reload

# Verify installation
oozie --version
```

### macOS

```bash
# Using Homebrew
brew install oozie

# Start service
brew services start oozie

# Verify installation
oozie --version
```

### FreeBSD

```bash
# Using pkg
pkg install oozie

# Enable in rc.conf
echo 'oozie_enable="YES"' >> /etc/rc.conf

# Start service
service oozie start

# Verify installation
oozie --version
```

### Windows

```bash
# Using Chocolatey
choco install oozie

# Or using Scoop
scoop install oozie

# Verify installation
oozie --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/oozie

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
oozie --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable oozie

# Start service
sudo systemctl start oozie

# Stop service
sudo systemctl stop oozie

# Restart service
sudo systemctl restart oozie

# Check status
sudo systemctl status oozie

# View logs
sudo journalctl -u oozie -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add oozie default

# Start service
rc-service oozie start

# Stop service
rc-service oozie stop

# Restart service
rc-service oozie restart

# Check status
rc-service oozie status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'oozie_enable="YES"' >> /etc/rc.conf

# Start service
service oozie start

# Stop service
service oozie stop

# Restart service
service oozie restart

# Check status
service oozie status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start oozie
brew services stop oozie
brew services restart oozie

# Check status
brew services list | grep oozie
```

### Windows Service Manager

```powershell
# Start service
net start oozie

# Stop service
net stop oozie

# Using PowerShell
Start-Service oozie
Stop-Service oozie
Restart-Service oozie

# Check status
Get-Service oozie
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream oozie_backend {
    server 127.0.0.1:11000;
}

server {
    listen 80;
    server_name oozie.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name oozie.example.com;

    ssl_certificate /etc/ssl/certs/oozie.example.com.crt;
    ssl_certificate_key /etc/ssl/private/oozie.example.com.key;

    location / {
        proxy_pass http://oozie_backend;
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
    ServerName oozie.example.com
    Redirect permanent / https://oozie.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName oozie.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/oozie.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/oozie.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:11000/
    ProxyPassReverse / http://127.0.0.1:11000/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend oozie_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/oozie.pem
    redirect scheme https if !{ ssl_fc }
    default_backend oozie_backend

backend oozie_backend
    balance roundrobin
    server oozie1 127.0.0.1:11000 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R oozie:oozie /etc/oozie
sudo chmod 750 /etc/oozie

# Configure firewall
sudo firewall-cmd --permanent --add-port=11000/tcp
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
sudo systemctl status oozie

# View logs
sudo journalctl -u oozie -f

# Monitor resource usage
top -p $(pgrep oozie)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/oozie"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/oozie-backup-$DATE.tar.gz" /etc/oozie /var/lib/oozie

echo "Backup completed: $BACKUP_DIR/oozie-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop oozie

# Restore from backup
tar -xzf /backup/oozie/oozie-backup-*.tar.gz -C /

# Start service
sudo systemctl start oozie
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u oozie -n 100
sudo tail -f /var/log/oozie/oozie.log

# Check configuration
oozie --version

# Check permissions
ls -la /etc/oozie
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 11000

# Test connectivity
telnet localhost 11000

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep oozie)

# Check disk I/O
iotop -p $(pgrep oozie)

# Check connections
ss -an | grep 11000
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  oozie:
    image: oozie:latest
    ports:
      - "11000:11000"
    volumes:
      - ./config:/etc/oozie
      - ./data:/var/lib/oozie
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update oozie

# Debian/Ubuntu
sudo apt update && sudo apt upgrade oozie

# Arch Linux
sudo pacman -Syu oozie

# Alpine Linux
apk update && apk upgrade oozie

# openSUSE
sudo zypper update oozie

# FreeBSD
pkg update && pkg upgrade oozie

# Always backup before updates
tar -czf /backup/oozie-pre-update-$(date +%Y%m%d).tar.gz /etc/oozie

# Restart after updates
sudo systemctl restart oozie
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/oozie

# Clean old logs
find /var/log/oozie -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/oozie
```

## Additional Resources

- Official Documentation: https://docs.oozie.org/
- GitHub Repository: https://github.com/oozie/oozie
- Community Forum: https://forum.oozie.org/
- Best Practices Guide: https://docs.oozie.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
