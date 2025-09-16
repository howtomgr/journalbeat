# journalbeat Installation Guide

journalbeat is a free and open-source systemd journal shipper. Journalbeat ships systemd journal entries to Elasticsearch

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
  - Storage: 1GB for data
  - Network: Systemd journal
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 5066 (default journalbeat port)
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

# Install journalbeat
sudo dnf install -y journalbeat

# Enable and start service
sudo systemctl enable --now journalbeat

# Configure firewall
sudo firewall-cmd --permanent --add-port=5066/tcp
sudo firewall-cmd --reload

# Verify installation
journalbeat --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install journalbeat
sudo apt install -y journalbeat

# Enable and start service
sudo systemctl enable --now journalbeat

# Configure firewall
sudo ufw allow 5066

# Verify installation
journalbeat --version
```

### Arch Linux

```bash
# Install journalbeat
sudo pacman -S journalbeat

# Enable and start service
sudo systemctl enable --now journalbeat

# Verify installation
journalbeat --version
```

### Alpine Linux

```bash
# Install journalbeat
apk add --no-cache journalbeat

# Enable and start service
rc-update add journalbeat default
rc-service journalbeat start

# Verify installation
journalbeat --version
```

### openSUSE/SLES

```bash
# Install journalbeat
sudo zypper install -y journalbeat

# Enable and start service
sudo systemctl enable --now journalbeat

# Configure firewall
sudo firewall-cmd --permanent --add-port=5066/tcp
sudo firewall-cmd --reload

# Verify installation
journalbeat --version
```

### macOS

```bash
# Using Homebrew
brew install journalbeat

# Start service
brew services start journalbeat

# Verify installation
journalbeat --version
```

### FreeBSD

```bash
# Using pkg
pkg install journalbeat

# Enable in rc.conf
echo 'journalbeat_enable="YES"' >> /etc/rc.conf

# Start service
service journalbeat start

# Verify installation
journalbeat --version
```

### Windows

```bash
# Using Chocolatey
choco install journalbeat

# Or using Scoop
scoop install journalbeat

# Verify installation
journalbeat --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/journalbeat

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
journalbeat --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable journalbeat

# Start service
sudo systemctl start journalbeat

# Stop service
sudo systemctl stop journalbeat

# Restart service
sudo systemctl restart journalbeat

# Check status
sudo systemctl status journalbeat

# View logs
sudo journalctl -u journalbeat -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add journalbeat default

# Start service
rc-service journalbeat start

# Stop service
rc-service journalbeat stop

# Restart service
rc-service journalbeat restart

# Check status
rc-service journalbeat status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'journalbeat_enable="YES"' >> /etc/rc.conf

# Start service
service journalbeat start

# Stop service
service journalbeat stop

# Restart service
service journalbeat restart

# Check status
service journalbeat status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start journalbeat
brew services stop journalbeat
brew services restart journalbeat

# Check status
brew services list | grep journalbeat
```

### Windows Service Manager

```powershell
# Start service
net start journalbeat

# Stop service
net stop journalbeat

# Using PowerShell
Start-Service journalbeat
Stop-Service journalbeat
Restart-Service journalbeat

# Check status
Get-Service journalbeat
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream journalbeat_backend {
    server 127.0.0.1:5066;
}

server {
    listen 80;
    server_name journalbeat.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name journalbeat.example.com;

    ssl_certificate /etc/ssl/certs/journalbeat.example.com.crt;
    ssl_certificate_key /etc/ssl/private/journalbeat.example.com.key;

    location / {
        proxy_pass http://journalbeat_backend;
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
    ServerName journalbeat.example.com
    Redirect permanent / https://journalbeat.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName journalbeat.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/journalbeat.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/journalbeat.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:5066/
    ProxyPassReverse / http://127.0.0.1:5066/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend journalbeat_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/journalbeat.pem
    redirect scheme https if !{ ssl_fc }
    default_backend journalbeat_backend

backend journalbeat_backend
    balance roundrobin
    server journalbeat1 127.0.0.1:5066 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R journalbeat:journalbeat /etc/journalbeat
sudo chmod 750 /etc/journalbeat

# Configure firewall
sudo firewall-cmd --permanent --add-port=5066/tcp
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
sudo systemctl status journalbeat

# View logs
sudo journalctl -u journalbeat -f

# Monitor resource usage
top -p $(pgrep journalbeat)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/journalbeat"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/journalbeat-backup-$DATE.tar.gz" /etc/journalbeat /var/lib/journalbeat

echo "Backup completed: $BACKUP_DIR/journalbeat-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop journalbeat

# Restore from backup
tar -xzf /backup/journalbeat/journalbeat-backup-*.tar.gz -C /

# Start service
sudo systemctl start journalbeat
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u journalbeat -n 100
sudo tail -f /var/log/journalbeat/journalbeat.log

# Check configuration
journalbeat --version

# Check permissions
ls -la /etc/journalbeat
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 5066

# Test connectivity
telnet localhost 5066

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep journalbeat)

# Check disk I/O
iotop -p $(pgrep journalbeat)

# Check connections
ss -an | grep 5066
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  journalbeat:
    image: journalbeat:latest
    ports:
      - "5066:5066"
    volumes:
      - ./config:/etc/journalbeat
      - ./data:/var/lib/journalbeat
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update journalbeat

# Debian/Ubuntu
sudo apt update && sudo apt upgrade journalbeat

# Arch Linux
sudo pacman -Syu journalbeat

# Alpine Linux
apk update && apk upgrade journalbeat

# openSUSE
sudo zypper update journalbeat

# FreeBSD
pkg update && pkg upgrade journalbeat

# Always backup before updates
tar -czf /backup/journalbeat-pre-update-$(date +%Y%m%d).tar.gz /etc/journalbeat

# Restart after updates
sudo systemctl restart journalbeat
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/journalbeat

# Clean old logs
find /var/log/journalbeat -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/journalbeat
```

## Additional Resources

- Official Documentation: https://docs.journalbeat.org/
- GitHub Repository: https://github.com/journalbeat/journalbeat
- Community Forum: https://forum.journalbeat.org/
- Best Practices Guide: https://docs.journalbeat.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
