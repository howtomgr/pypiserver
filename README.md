# pypiserver Installation Guide

pypiserver is a free and open-source Python package server. pypiserver provides minimal PyPI server for private Python packages

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
  - CPU: 1 core minimum
  - RAM: 256MB minimum
  - Storage: 10GB for packages
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8080 (default pypiserver port)
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

# Install pypiserver
sudo dnf install -y pypiserver

# Enable and start service
sudo systemctl enable --now pypiserver

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
pypiserver --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install pypiserver
sudo apt install -y pypiserver

# Enable and start service
sudo systemctl enable --now pypiserver

# Configure firewall
sudo ufw allow 8080

# Verify installation
pypiserver --version
```

### Arch Linux

```bash
# Install pypiserver
sudo pacman -S pypiserver

# Enable and start service
sudo systemctl enable --now pypiserver

# Verify installation
pypiserver --version
```

### Alpine Linux

```bash
# Install pypiserver
apk add --no-cache pypiserver

# Enable and start service
rc-update add pypiserver default
rc-service pypiserver start

# Verify installation
pypiserver --version
```

### openSUSE/SLES

```bash
# Install pypiserver
sudo zypper install -y pypiserver

# Enable and start service
sudo systemctl enable --now pypiserver

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
pypiserver --version
```

### macOS

```bash
# Using Homebrew
brew install pypiserver

# Start service
brew services start pypiserver

# Verify installation
pypiserver --version
```

### FreeBSD

```bash
# Using pkg
pkg install pypiserver

# Enable in rc.conf
echo 'pypiserver_enable="YES"' >> /etc/rc.conf

# Start service
service pypiserver start

# Verify installation
pypiserver --version
```

### Windows

```bash
# Using Chocolatey
choco install pypiserver

# Or using Scoop
scoop install pypiserver

# Verify installation
pypiserver --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/pypiserver

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
pypiserver --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable pypiserver

# Start service
sudo systemctl start pypiserver

# Stop service
sudo systemctl stop pypiserver

# Restart service
sudo systemctl restart pypiserver

# Check status
sudo systemctl status pypiserver

# View logs
sudo journalctl -u pypiserver -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add pypiserver default

# Start service
rc-service pypiserver start

# Stop service
rc-service pypiserver stop

# Restart service
rc-service pypiserver restart

# Check status
rc-service pypiserver status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'pypiserver_enable="YES"' >> /etc/rc.conf

# Start service
service pypiserver start

# Stop service
service pypiserver stop

# Restart service
service pypiserver restart

# Check status
service pypiserver status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start pypiserver
brew services stop pypiserver
brew services restart pypiserver

# Check status
brew services list | grep pypiserver
```

### Windows Service Manager

```powershell
# Start service
net start pypiserver

# Stop service
net stop pypiserver

# Using PowerShell
Start-Service pypiserver
Stop-Service pypiserver
Restart-Service pypiserver

# Check status
Get-Service pypiserver
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream pypiserver_backend {
    server 127.0.0.1:8080;
}

server {
    listen 80;
    server_name pypiserver.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name pypiserver.example.com;

    ssl_certificate /etc/ssl/certs/pypiserver.example.com.crt;
    ssl_certificate_key /etc/ssl/private/pypiserver.example.com.key;

    location / {
        proxy_pass http://pypiserver_backend;
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
    ServerName pypiserver.example.com
    Redirect permanent / https://pypiserver.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName pypiserver.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/pypiserver.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/pypiserver.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8080/
    ProxyPassReverse / http://127.0.0.1:8080/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend pypiserver_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/pypiserver.pem
    redirect scheme https if !{ ssl_fc }
    default_backend pypiserver_backend

backend pypiserver_backend
    balance roundrobin
    server pypiserver1 127.0.0.1:8080 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R pypiserver:pypiserver /etc/pypiserver
sudo chmod 750 /etc/pypiserver

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
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
sudo systemctl status pypiserver

# View logs
sudo journalctl -u pypiserver -f

# Monitor resource usage
top -p $(pgrep pypiserver)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/pypiserver"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/pypiserver-backup-$DATE.tar.gz" /etc/pypiserver /var/lib/pypiserver

echo "Backup completed: $BACKUP_DIR/pypiserver-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop pypiserver

# Restore from backup
tar -xzf /backup/pypiserver/pypiserver-backup-*.tar.gz -C /

# Start service
sudo systemctl start pypiserver
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u pypiserver -n 100
sudo tail -f /var/log/pypiserver/pypiserver.log

# Check configuration
pypiserver --version

# Check permissions
ls -la /etc/pypiserver
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8080

# Test connectivity
telnet localhost 8080

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep pypiserver)

# Check disk I/O
iotop -p $(pgrep pypiserver)

# Check connections
ss -an | grep 8080
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  pypiserver:
    image: pypiserver:latest
    ports:
      - "8080:8080"
    volumes:
      - ./config:/etc/pypiserver
      - ./data:/var/lib/pypiserver
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update pypiserver

# Debian/Ubuntu
sudo apt update && sudo apt upgrade pypiserver

# Arch Linux
sudo pacman -Syu pypiserver

# Alpine Linux
apk update && apk upgrade pypiserver

# openSUSE
sudo zypper update pypiserver

# FreeBSD
pkg update && pkg upgrade pypiserver

# Always backup before updates
tar -czf /backup/pypiserver-pre-update-$(date +%Y%m%d).tar.gz /etc/pypiserver

# Restart after updates
sudo systemctl restart pypiserver
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/pypiserver

# Clean old logs
find /var/log/pypiserver -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/pypiserver
```

## Additional Resources

- Official Documentation: https://docs.pypiserver.org/
- GitHub Repository: https://github.com/pypiserver/pypiserver
- Community Forum: https://forum.pypiserver.org/
- Best Practices Guide: https://docs.pypiserver.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
