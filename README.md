# consul-connect Installation Guide

consul-connect is a free and open-source service mesh. HashiCorp Consul Connect provides service mesh with service discovery

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
  - Network: Various protocols
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8500 (default consul-connect port)
  - Mesh on various
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

# Install consul-connect
sudo dnf install -y consul_connect

# Enable and start service
sudo systemctl enable --now consul-connect

# Configure firewall
sudo firewall-cmd --permanent --add-port=8500/tcp
sudo firewall-cmd --reload

# Verify installation
consul-connect --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install consul-connect
sudo apt install -y consul_connect

# Enable and start service
sudo systemctl enable --now consul-connect

# Configure firewall
sudo ufw allow 8500

# Verify installation
consul-connect --version
```

### Arch Linux

```bash
# Install consul-connect
sudo pacman -S consul_connect

# Enable and start service
sudo systemctl enable --now consul-connect

# Verify installation
consul-connect --version
```

### Alpine Linux

```bash
# Install consul-connect
apk add --no-cache consul_connect

# Enable and start service
rc-update add consul-connect default
rc-service consul-connect start

# Verify installation
consul-connect --version
```

### openSUSE/SLES

```bash
# Install consul-connect
sudo zypper install -y consul_connect

# Enable and start service
sudo systemctl enable --now consul-connect

# Configure firewall
sudo firewall-cmd --permanent --add-port=8500/tcp
sudo firewall-cmd --reload

# Verify installation
consul-connect --version
```

### macOS

```bash
# Using Homebrew
brew install consul_connect

# Start service
brew services start consul_connect

# Verify installation
consul-connect --version
```

### FreeBSD

```bash
# Using pkg
pkg install consul_connect

# Enable in rc.conf
echo 'consul-connect_enable="YES"' >> /etc/rc.conf

# Start service
service consul-connect start

# Verify installation
consul-connect --version
```

### Windows

```bash
# Using Chocolatey
choco install consul_connect

# Or using Scoop
scoop install consul_connect

# Verify installation
consul-connect --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/consul_connect

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
consul-connect --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable consul-connect

# Start service
sudo systemctl start consul-connect

# Stop service
sudo systemctl stop consul-connect

# Restart service
sudo systemctl restart consul-connect

# Check status
sudo systemctl status consul-connect

# View logs
sudo journalctl -u consul-connect -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add consul-connect default

# Start service
rc-service consul-connect start

# Stop service
rc-service consul-connect stop

# Restart service
rc-service consul-connect restart

# Check status
rc-service consul-connect status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'consul-connect_enable="YES"' >> /etc/rc.conf

# Start service
service consul-connect start

# Stop service
service consul-connect stop

# Restart service
service consul-connect restart

# Check status
service consul-connect status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start consul_connect
brew services stop consul_connect
brew services restart consul_connect

# Check status
brew services list | grep consul_connect
```

### Windows Service Manager

```powershell
# Start service
net start consul-connect

# Stop service
net stop consul-connect

# Using PowerShell
Start-Service consul-connect
Stop-Service consul-connect
Restart-Service consul-connect

# Check status
Get-Service consul-connect
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream consul_connect_backend {
    server 127.0.0.1:8500;
}

server {
    listen 80;
    server_name consul_connect.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name consul_connect.example.com;

    ssl_certificate /etc/ssl/certs/consul_connect.example.com.crt;
    ssl_certificate_key /etc/ssl/private/consul_connect.example.com.key;

    location / {
        proxy_pass http://consul_connect_backend;
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
    ServerName consul_connect.example.com
    Redirect permanent / https://consul_connect.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName consul_connect.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/consul_connect.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/consul_connect.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8500/
    ProxyPassReverse / http://127.0.0.1:8500/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend consul_connect_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/consul_connect.pem
    redirect scheme https if !{ ssl_fc }
    default_backend consul_connect_backend

backend consul_connect_backend
    balance roundrobin
    server consul_connect1 127.0.0.1:8500 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R consul_connect:consul_connect /etc/consul_connect
sudo chmod 750 /etc/consul_connect

# Configure firewall
sudo firewall-cmd --permanent --add-port=8500/tcp
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
sudo systemctl status consul-connect

# View logs
sudo journalctl -u consul-connect -f

# Monitor resource usage
top -p $(pgrep consul_connect)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/consul_connect"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/consul_connect-backup-$DATE.tar.gz" /etc/consul_connect /var/lib/consul_connect

echo "Backup completed: $BACKUP_DIR/consul_connect-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop consul-connect

# Restore from backup
tar -xzf /backup/consul_connect/consul_connect-backup-*.tar.gz -C /

# Start service
sudo systemctl start consul-connect
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u consul-connect -n 100
sudo tail -f /var/log/consul_connect/consul_connect.log

# Check configuration
consul-connect --version

# Check permissions
ls -la /etc/consul_connect
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8500

# Test connectivity
telnet localhost 8500

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep consul_connect)

# Check disk I/O
iotop -p $(pgrep consul_connect)

# Check connections
ss -an | grep 8500
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  consul_connect:
    image: consul_connect:latest
    ports:
      - "8500:8500"
    volumes:
      - ./config:/etc/consul_connect
      - ./data:/var/lib/consul_connect
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update consul_connect

# Debian/Ubuntu
sudo apt update && sudo apt upgrade consul_connect

# Arch Linux
sudo pacman -Syu consul_connect

# Alpine Linux
apk update && apk upgrade consul_connect

# openSUSE
sudo zypper update consul_connect

# FreeBSD
pkg update && pkg upgrade consul_connect

# Always backup before updates
tar -czf /backup/consul_connect-pre-update-$(date +%Y%m%d).tar.gz /etc/consul_connect

# Restart after updates
sudo systemctl restart consul-connect
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/consul_connect

# Clean old logs
find /var/log/consul_connect -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/consul_connect
```

## Additional Resources

- Official Documentation: https://docs.consul_connect.org/
- GitHub Repository: https://github.com/consul_connect/consul_connect
- Community Forum: https://forum.consul_connect.org/
- Best Practices Guide: https://docs.consul_connect.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
