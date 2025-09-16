# freeipa Installation Guide

freeipa is a free and open-source integrated identity and authentication solution. FreeIPA provides centralized authentication, authorization, and account information, serving as an open-source alternative to Microsoft Active Directory

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
  - CPU: 2+ cores recommended
  - RAM: 2GB minimum (4GB+ recommended)
  - Storage: 10GB for installation
  - Network: LDAP, Kerberos, DNS, HTTPS
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 389 (default freeipa port)
  - Ports 636, 88, 464, 80, 443
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

# Install freeipa
sudo dnf install -y freeipa-server

# Enable and start service
sudo systemctl enable --now ipa

# Configure firewall
sudo firewall-cmd --permanent --add-port=389/tcp
sudo firewall-cmd --reload

# Verify installation
ipa --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install freeipa
sudo apt install -y freeipa-server

# Enable and start service
sudo systemctl enable --now ipa

# Configure firewall
sudo ufw allow 389

# Verify installation
ipa --version
```

### Arch Linux

```bash
# Install freeipa
sudo pacman -S freeipa-server

# Enable and start service
sudo systemctl enable --now ipa

# Verify installation
ipa --version
```

### Alpine Linux

```bash
# Install freeipa
apk add --no-cache freeipa-server

# Enable and start service
rc-update add ipa default
rc-service ipa start

# Verify installation
ipa --version
```

### openSUSE/SLES

```bash
# Install freeipa
sudo zypper install -y freeipa-server

# Enable and start service
sudo systemctl enable --now ipa

# Configure firewall
sudo firewall-cmd --permanent --add-port=389/tcp
sudo firewall-cmd --reload

# Verify installation
ipa --version
```

### macOS

```bash
# Using Homebrew
brew install freeipa-server

# Start service
brew services start freeipa-server

# Verify installation
ipa --version
```

### FreeBSD

```bash
# Using pkg
pkg install freeipa-server

# Enable in rc.conf
echo 'ipa_enable="YES"' >> /etc/rc.conf

# Start service
service ipa start

# Verify installation
ipa --version
```

### Windows

```bash
# Using Chocolatey
choco install freeipa-server

# Or using Scoop
scoop install freeipa-server

# Verify installation
ipa --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/freeipa-server

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
ipa --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable ipa

# Start service
sudo systemctl start ipa

# Stop service
sudo systemctl stop ipa

# Restart service
sudo systemctl restart ipa

# Check status
sudo systemctl status ipa

# View logs
sudo journalctl -u ipa -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add ipa default

# Start service
rc-service ipa start

# Stop service
rc-service ipa stop

# Restart service
rc-service ipa restart

# Check status
rc-service ipa status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'ipa_enable="YES"' >> /etc/rc.conf

# Start service
service ipa start

# Stop service
service ipa stop

# Restart service
service ipa restart

# Check status
service ipa status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start freeipa-server
brew services stop freeipa-server
brew services restart freeipa-server

# Check status
brew services list | grep freeipa-server
```

### Windows Service Manager

```powershell
# Start service
net start ipa

# Stop service
net stop ipa

# Using PowerShell
Start-Service ipa
Stop-Service ipa
Restart-Service ipa

# Check status
Get-Service ipa
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream freeipa-server_backend {
    server 127.0.0.1:389;
}

server {
    listen 80;
    server_name freeipa-server.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name freeipa-server.example.com;

    ssl_certificate /etc/ssl/certs/freeipa-server.example.com.crt;
    ssl_certificate_key /etc/ssl/private/freeipa-server.example.com.key;

    location / {
        proxy_pass http://freeipa-server_backend;
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
    ServerName freeipa-server.example.com
    Redirect permanent / https://freeipa-server.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName freeipa-server.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/freeipa-server.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/freeipa-server.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:389/
    ProxyPassReverse / http://127.0.0.1:389/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend freeipa-server_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/freeipa-server.pem
    redirect scheme https if !{ ssl_fc }
    default_backend freeipa-server_backend

backend freeipa-server_backend
    balance roundrobin
    server freeipa-server1 127.0.0.1:389 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R freeipa-server:freeipa-server /etc/freeipa-server
sudo chmod 750 /etc/freeipa-server

# Configure firewall
sudo firewall-cmd --permanent --add-port=389/tcp
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
sudo systemctl status ipa

# View logs
sudo journalctl -u ipa -f

# Monitor resource usage
top -p $(pgrep freeipa-server)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/freeipa-server"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/freeipa-server-backup-$DATE.tar.gz" /etc/freeipa-server /var/lib/freeipa-server

echo "Backup completed: $BACKUP_DIR/freeipa-server-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop ipa

# Restore from backup
tar -xzf /backup/freeipa-server/freeipa-server-backup-*.tar.gz -C /

# Start service
sudo systemctl start ipa
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u ipa -n 100
sudo tail -f /var/log/freeipa-server/freeipa-server.log

# Check configuration
ipa --version

# Check permissions
ls -la /etc/freeipa-server
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 389

# Test connectivity
telnet localhost 389

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep freeipa-server)

# Check disk I/O
iotop -p $(pgrep freeipa-server)

# Check connections
ss -an | grep 389
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  freeipa-server:
    image: freeipa-server:latest
    ports:
      - "389:389"
    volumes:
      - ./config:/etc/freeipa-server
      - ./data:/var/lib/freeipa-server
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update freeipa-server

# Debian/Ubuntu
sudo apt update && sudo apt upgrade freeipa-server

# Arch Linux
sudo pacman -Syu freeipa-server

# Alpine Linux
apk update && apk upgrade freeipa-server

# openSUSE
sudo zypper update freeipa-server

# FreeBSD
pkg update && pkg upgrade freeipa-server

# Always backup before updates
tar -czf /backup/freeipa-server-pre-update-$(date +%Y%m%d).tar.gz /etc/freeipa-server

# Restart after updates
sudo systemctl restart ipa
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/freeipa-server

# Clean old logs
find /var/log/freeipa-server -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/freeipa-server
```

## Additional Resources

- Official Documentation: https://docs.freeipa-server.org/
- GitHub Repository: https://github.com/freeipa-server/freeipa-server
- Community Forum: https://forum.freeipa-server.org/
- Best Practices Guide: https://docs.freeipa-server.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
