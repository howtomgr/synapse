# Synapse Installation Guide

Synapse is a free and open-source Matrix Server. Matrix homeserver written in Python

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
  - CPU: 2 cores minimum (4+ cores recommended)
  - RAM: 2GB minimum (4GB+ recommended)
  - Storage: 1GB for installation
  - Network: 8008/8448 ports
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8008/8448 (default synapse port)
- **Dependencies**:
  - python3, postgresql
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

# Install synapse
sudo dnf install -y synapse python3, postgresql

# Enable and start service
sudo systemctl enable --now matrix-synapse

# Configure firewall
sudo firewall-cmd --permanent --add-service=synapse
sudo firewall-cmd --reload

# Verify installation
synapse --version || systemctl status matrix-synapse
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install synapse
sudo apt install -y synapse python3, postgresql

# Enable and start service
sudo systemctl enable --now matrix-synapse

# Configure firewall
sudo ufw allow 8008/8448

# Verify installation
synapse --version || systemctl status matrix-synapse
```

### Arch Linux

```bash
# Install synapse
sudo pacman -S synapse

# Enable and start service
sudo systemctl enable --now matrix-synapse

# Verify installation
synapse --version || systemctl status matrix-synapse
```

### Alpine Linux

```bash
# Install synapse
apk add --no-cache synapse

# Enable and start service
rc-update add matrix-synapse default
rc-service matrix-synapse start

# Verify installation
synapse --version || rc-service matrix-synapse status
```

### openSUSE/SLES

```bash
# Install synapse
sudo zypper install -y synapse python3, postgresql

# Enable and start service
sudo systemctl enable --now matrix-synapse

# Configure firewall
sudo firewall-cmd --permanent --add-service=synapse
sudo firewall-cmd --reload

# Verify installation
synapse --version || systemctl status matrix-synapse
```

### macOS

```bash
# Using Homebrew
brew install synapse

# Start service
brew services start synapse

# Verify installation
synapse --version
```

### FreeBSD

```bash
# Using pkg
pkg install synapse

# Enable in rc.conf
echo 'matrix-synapse_enable="YES"' >> /etc/rc.conf

# Start service
service matrix-synapse start

# Verify installation
synapse --version || service matrix-synapse status
```

### Windows

```powershell
# Using Chocolatey
choco install synapse

# Or using Scoop
scoop install synapse

# Verify installation
synapse --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory if needed
sudo mkdir -p /etc/matrix-synapse

# Set up basic configuration
sudo tee /etc/matrix-synapse/synapse.conf << 'EOF'
# Synapse Configuration
event_cache_size: 10K
EOF

# Test configuration
sudo synapse -t || sudo matrix-synapse configtest

# Reload service
sudo systemctl reload matrix-synapse
```

### Security Hardening

```bash
# Set appropriate permissions
sudo chown -R synapse:synapse /etc/matrix-synapse
sudo chmod 750 /etc/matrix-synapse

# Enable security features
# See security section for detailed hardening steps
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable matrix-synapse

# Start service
sudo systemctl start matrix-synapse

# Stop service
sudo systemctl stop matrix-synapse

# Restart service
sudo systemctl restart matrix-synapse

# Reload configuration
sudo systemctl reload matrix-synapse

# Check status
sudo systemctl status matrix-synapse

# View logs
sudo journalctl -u matrix-synapse -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add matrix-synapse default

# Start service
rc-service matrix-synapse start

# Stop service
rc-service matrix-synapse stop

# Restart service
rc-service matrix-synapse restart

# Check status
rc-service matrix-synapse status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'matrix-synapse_enable="YES"' >> /etc/rc.conf

# Start service
service matrix-synapse start

# Stop service
service matrix-synapse stop

# Restart service
service matrix-synapse restart

# Check status
service matrix-synapse status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start synapse
brew services stop synapse
brew services restart synapse

# Check status
brew services list | grep synapse
```

### Windows Service Manager

```powershell
# Start service
net start matrix-synapse

# Stop service
net stop matrix-synapse

# Using PowerShell
Start-Service matrix-synapse
Stop-Service matrix-synapse
Restart-Service matrix-synapse

# Check status
Get-Service matrix-synapse
```

## Advanced Configuration

### Performance Optimization

```bash
# Configure performance settings
cat >> /etc/matrix-synapse/synapse.conf << 'EOF'
event_cache_size: 10K
EOF

# Apply system tuning
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535

# Restart service
sudo systemctl restart matrix-synapse
```

### Clustering and High Availability

```bash
# Configure clustering (if supported)
# See official documentation for cluster setup

# Basic load balancing setup example
# Configure multiple instances on different ports
```

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream synapse_backend {
    server 127.0.0.1:8008/8448;
    server 127.0.0.1:{default_port}1 backup;
}

server {
    listen 80;
    server_name synapse.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name synapse.example.com;

    ssl_certificate /etc/ssl/certs/synapse.example.com.crt;
    ssl_certificate_key /etc/ssl/private/synapse.example.com.key;

    location / {
        proxy_pass http://synapse_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support (if needed)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName synapse.example.com
    Redirect permanent / https://synapse.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName synapse.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/synapse.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/synapse.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8008/8448/
    ProxyPassReverse / http://127.0.0.1:8008/8448/
    
    # WebSocket support (if needed)
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/?(.*) "ws://127.0.0.1:8008/8448/$1" [P,L]
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend synapse_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/synapse.pem
    redirect scheme https if !{ ssl_fc }
    default_backend synapse_backend

backend synapse_backend
    balance roundrobin
    option httpchk GET /health
    server synapse1 127.0.0.1:8008/8448 check
    server synapse2 127.0.0.1:{default_port}1 check backup
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R synapse:synapse /etc/matrix-synapse
sudo chmod 750 /etc/matrix-synapse

# Configure firewall
sudo firewall-cmd --permanent --add-service=synapse
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on

# Configure fail2ban
sudo tee /etc/fail2ban/jail.d/synapse.conf << 'EOF'
[synapse]
enabled = true
port = 8008/8448
filter = synapse
logpath = /var/log/matrix-synapse/*.log
maxretry = 5
bantime = 3600
EOF
```

### SSL/TLS Configuration

```bash
# Generate SSL certificates
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/synapse.key \
    -out /etc/ssl/certs/synapse.crt

# Configure SSL in synapse
# See official documentation for SSL configuration
```

## Database Setup

### PostgreSQL Backend (if applicable)

```bash
# Create database and user
sudo -u postgres psql << EOF
CREATE DATABASE synapse_db;
CREATE USER synapse_user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE synapse_db TO synapse_user;
EOF

# Configure synapse to use PostgreSQL
# See official documentation for database configuration
```

### MySQL/MariaDB Backend (if applicable)

```bash
# Create database and user
sudo mysql << EOF
CREATE DATABASE synapse_db;
CREATE USER 'synapse_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON synapse_db.* TO 'synapse_user'@'localhost';
FLUSH PRIVILEGES;
EOF
```

## Performance Optimization

### System Tuning

```bash
# Kernel parameters
sudo tee -a /etc/sysctl.conf << EOF
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.ip_local_port_range = 1024 65535
net.core.netdev_max_backlog = 5000
vm.swappiness = 10
EOF

sudo sysctl -p

# Synapse specific tuning
event_cache_size: 10K
```

### Resource Limits

```bash
# Configure system limits
sudo tee -a /etc/security/limits.conf << EOF
synapse soft nofile 65535
synapse hard nofile 65535
synapse soft nproc 32768
synapse hard nproc 32768
EOF
```

## Monitoring

### Prometheus Integration

```yaml
# prometheus.yml configuration
scrape_configs:
  - job_name: 'synapse'
    static_configs:
      - targets: ['localhost:8008/8448']
    metrics_path: '/metrics'
```

### Health Checks

```bash
# Basic health check script
#!/bin/bash
if systemctl is-active --quiet matrix-synapse; then
    echo "Synapse is running"
    exit 0
else
    echo "Synapse is not running"
    exit 1
fi
```

### Log Monitoring

```bash
# Configure log rotation
sudo tee /etc/logrotate.d/synapse << 'EOF'
/var/log/matrix-synapse/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 synapse synapse
    postrotate
        systemctl reload matrix-synapse > /dev/null 2>&1 || true
    endscript
}
EOF
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Synapse backup script
BACKUP_DIR="/backup/synapse"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Stop service (if required)
systemctl stop matrix-synapse

# Backup configuration
tar -czf "$BACKUP_DIR/synapse-config-$DATE.tar.gz" /etc/matrix-synapse

# Backup data (adjust paths as needed)
tar -czf "$BACKUP_DIR/synapse-data-$DATE.tar.gz" /var/lib/synapse

# Start service
systemctl start matrix-synapse

# Clean old backups (keep 30 days)
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_DIR"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop matrix-synapse

# Restore configuration
sudo tar -xzf /backup/synapse/synapse-config-*.tar.gz -C /

# Restore data
sudo tar -xzf /backup/synapse/synapse-data-*.tar.gz -C /

# Set permissions
sudo chown -R synapse:synapse /etc/matrix-synapse
sudo chown -R synapse:synapse /var/lib/synapse

# Start service
sudo systemctl start matrix-synapse
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u matrix-synapse -n 100
sudo tail -f /var/log/matrix-synapse/*.log

# Check configuration
sudo synapse -t || sudo matrix-synapse configtest

# Check permissions
ls -la /etc/matrix-synapse
ls -la /var/lib/synapse
```

2. **Connection refused**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8008/8448
sudo netstat -tlnp | grep 8008/8448

# Check firewall
sudo firewall-cmd --list-all
sudo iptables -L -n

# Test connection
telnet localhost 8008/8448
nc -zv localhost 8008/8448
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep python)
htop -p $(pgrep python)

# Check connections
ss -ant | grep :8008/8448 | wc -l

# Monitor I/O
iotop -p $(pgrep python)
```

### Debug Mode

```bash
# Run in debug mode
sudo synapse -d
# or
sudo matrix-synapse debug

# Increase log verbosity
# Edit configuration to enable debug logging
```

## Integration Examples

### Docker Compose

```yaml
version: '3.8'
services:
  synapse:
    image: synapse:latest
    container_name: synapse
    ports:
      - "8008/8448:8008/8448"
    volumes:
      - ./config:/etc/matrix-synapse
      - ./data:/var/lib/synapse
    environment:
      - synapse_CONFIG=/etc/matrix-synapse/synapse.conf
    restart: unless-stopped
    networks:
      - synapse_net

networks:
  synapse_net:
    driver: bridge
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: synapse
spec:
  replicas: 3
  selector:
    matchLabels:
      app: synapse
  template:
    metadata:
      labels:
        app: synapse
    spec:
      containers:
      - name: synapse
        image: synapse:latest
        ports:
        - containerPort: 8008/8448
        volumeMounts:
        - name: config
          mountPath: /etc/matrix-synapse
      volumes:
      - name: config
        configMap:
          name: synapse-config
---
apiVersion: v1
kind: Service
metadata:
  name: synapse
spec:
  selector:
    app: synapse
  ports:
  - port: 8008/8448
    targetPort: 8008/8448
  type: LoadBalancer
```

### Ansible Playbook

```yaml
---
- name: Install and configure Synapse
  hosts: all
  become: yes
  tasks:
    - name: Install synapse
      package:
        name: synapse
        state: present
    
    - name: Configure synapse
      template:
        src: synapse.conf.j2
        dest: /etc/matrix-synapse/synapse.conf
        owner: synapse
        group: synapse
        mode: '0640'
      notify: restart synapse
    
    - name: Start and enable synapse
      systemd:
        name: matrix-synapse
        state: started
        enabled: yes
  
  handlers:
    - name: restart synapse
      systemd:
        name: matrix-synapse
        state: restarted
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update synapse

# Debian/Ubuntu
sudo apt update && sudo apt upgrade synapse

# Arch Linux
sudo pacman -Syu synapse

# Alpine Linux
apk update && apk upgrade synapse

# openSUSE
sudo zypper update synapse

# FreeBSD
pkg update && pkg upgrade synapse

# Always backup before updates
tar -czf /backup/synapse-pre-update-$(date +%Y%m%d).tar.gz /etc/matrix-synapse

# Restart after updates
sudo systemctl restart matrix-synapse
```

### Regular Maintenance Tasks

```bash
# Clean logs
find /var/log/matrix-synapse -name "*.log" -mtime +30 -delete

# Verify integrity
sudo synapse --verify || sudo matrix-synapse check

# Update databases (if applicable)
sudo synapse-update-db

# Optimize performance
sudo synapse-optimize

# Check for security updates
sudo synapse --security-check
```

## Additional Resources

- Official Documentation: https://docs.synapse.org/
- GitHub Repository: https://github.com/synapse/synapse
- Community Forum: https://forum.synapse.org/
- Wiki: https://wiki.synapse.org/
- Comparison vs Dendrite, Conduit, Construct: https://docs.synapse.org/comparison

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
