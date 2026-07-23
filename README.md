# ICT171 Cloud Server Project - Documentation

| Parameter | Details |
| :--- | :--- |
| **Student Name** | [Muhammad Tayyab Attiq] |
| **Student ID** | [36086315] |
| **Unit Code & Name** | ICT171 Introduction to Server Environments and Architectures |
| **Server Global IP** | [107.22.2.30/] |
| **Domain Name** | https://cafeaylanto.duckdns.org/ |
| **GitHub Repository** | https://github.com/tayyabateq/ict171-cafe-aylanto-server |
| **Video Explainer Link**| [Insert Video] |

---

## 1. Project Architecture & Overview
This project documents the deployment, security hardening, and configuration of an Infrastructure as a Service (IaaS) cloud server running on Ubuntu 22.04 LTS[cite: 1]. 

The server hosts **Cafe Aylanto**, a multi-purpose internet presence integrated with an automated real-time server telemetry and health monitoring subsystem[cite: 1].

### Multi-Purpose Server Architecture:
1. **Infrastructure Layer (IaaS):** Cloud Virtual Machine configured manually via SSH terminal[cite: 1].
2. **Web Service Engine:** Apache2 with custom Virtual Host configuration files[cite: 1].
3. **Domain Name System:** Dynamic DNS mapping managed via DuckDNS (`cafeaylanto.duckdns.org`)[cite: 1].
4. **Transport Layer Security:** Free SSL/TLS encryption via Let's Encrypt (Certbot for Apache) with enforced HTTP-to-HTTPS redirection[cite: 1].
5. **Scripting & Monitoring Subsystem:** Custom Bash telemetry script generating live HTML server metrics directly integrated into the web application (`/status.html`)[cite: 1].

---

## 2. Infrastructure Provisioning & Hardening (IaaS)

### 2.1 SSH Access
Access to the remote server was established via SSH terminal[cite: 1]:
```bash
ssh ubuntu@YOUR_SERVER_PUBLIC_IP

```

### 2.2 Server Updates & Firewall Configuration

System repositories were updated, and Uncomplicated Firewall (UFW) rules were configured to secure network access:

```bash
# Update system dependencies
sudo apt update && sudo apt upgrade -y

# Configure firewall access ports
sudo ufw allow 22/tcp    # SSH
sudo ufw allow 80/tcp    # HTTP
sudo ufw allow 443/tcp   # HTTPS
sudo ufw enable

```

---

## 3. Web Server Installation & Site Deployment (Apache2)

### 3.1 Installing Apache2

```bash
sudo apt update
sudo apt install apache2 -y

# Enable rewrite module for SSL redirects
sudo a2enmod rewrite

```

### 3.2 Setting Up Web Directory & Permissions

```bash
# Create directory structure for Cafe Aylanto
sudo mkdir -p /var/www/cafeaylanto

# Assign ownership to the active user
sudo chown -R $USER:$USER /var/www/cafeaylanto
sudo chmod -R 755 /var/www/cafeaylanto

```

### 3.3 Configuring Apache Virtual Host

Create a custom site configuration file in `/etc/apache2/sites-available/cafeaylanto.conf`:

```bash
sudo bash -c "cat << 'EOF' > /etc/apache2/sites-available/cafeaylanto.conf
<VirtualHost *:80>
    ServerName cafeaylanto.duckdns.org
    DocumentRoot /var/www/cafeaylanto

    <Directory /var/www/cafeaylanto>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog \${APACHE_LOG_DIR}/cafeaylanto_error.log
    CustomLog \${APACHE_LOG_DIR}/cafeaylanto_access.log combined
</VirtualHost>
EOF"

```

Enable the new site, disable default site, and restart Apache:

```bash
sudo a2ensite cafeaylanto.conf
sudo a2dissite 000-default.conf
sudo apache2ctl configtest
sudo systemctl restart apache2

```

### 3.4 Deploying Main Web Page

```bash
cat << 'EOF' > /var/www/cafeaylanto/index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Cafe Aylanto | Fine Dining</title>
    <style>
        body { font-family: Arial, sans-serif; background: #f8f9fa; color: #333; text-align: center; padding: 50px; }
        h1 { color: #8b0000; }
        .card { background: white; padding: 30px; border-radius: 10px; max-width: 600px; margin: 0 auto; box-shadow: 0 4px 8px rgba(0,0,0,0.1); }
        .btn { display: inline-block; padding: 10px 20px; background: #8b0000; color: white; text-decoration: none; border-radius: 5px; margin-top: 20px; }
    </style>
</head>
<body>
    <div class="card">
        <h1>Welcome to Cafe Aylanto</h1>
        <p>Experience Mediterranean dining and culinary excellence.</p>
        <hr>
        <a href="/status.html" class="btn">View Integrated Server Health Metrics</a>
    </div>
</body>
</html>
EOF

```

---

## 4. DNS Configuration & Dynamic Update Automation

### 4.1 DuckDNS Domain Mapping

1. Mapped dynamic domain `cafeaylanto.duckdns.org` to server IP `107.22.2.30/` on DuckDNS.


2. Verified DNS resolution:

```bash
dig cafeaylanto.duckdns.org

```

### 4.2 Dynamic IP Update Script & Cron Automation

To ensure continuous domain accessibility:

```bash
mkdir -p ~/duckdns
cat << 'EOF' > ~/duckdns/duck.sh
echo url="[https://www.duckdns.org/update?domains=cafeaylanto&token=YOUR_DUCKDNS_TOKEN&ip=](https://www.duckdns.org/update?domains=cafeaylanto&token=YOUR_DUCKDNS_TOKEN&ip=)" | curl -k -K -
EOF

chmod 700 ~/duckdns/duck.sh

# Automate update script via crontab (runs every 5 minutes)
(crontab -l 2>/dev/null; echo "*/5 * * * * ~/duckdns/duck.sh >/dev/null 2>&1") | crontab -

```

---

## 5. SSL/TLS Encryption & HTTPS Setup (Certbot)

### 5.1 Provisioning SSL Certificate for Apache

```bash
# Install Certbot for Apache
sudo apt install certbot python3-certbot-apache -y

# Obtain SSL Certificate and automatically configure HTTP-to-HTTPS redirect
sudo certbot --apache -d cafeaylanto.duckdns.org

```

### 5.2 Certificate Auto-Renewal Verification

```bash
# Test automated certificate renewal procedure
sudo certbot renew --dry-run

```

---

## 6. Custom Scripting Component & Integrated Visualization

### 6.1 Script Purpose & Logic

A custom Bash telemetry script (`server_telemetry.sh`) was created to gather real-time server health metrics (CPU uptime, memory utilization, disk space, and Apache service status).

To fulfill the rubric requirement for **verifiable visual output**, the script generates an HTML status page directly into `/var/www/cafeaylanto/status.html`.

### 6.2 Script Code (`scripts/server_telemetry.sh`)

```bash
#!/bin/bash
# Script: server_telemetry.sh
# Purpose: Gathers system performance metrics and formats them into an integrated web report.

OUTPUT_FILE="/var/www/cafeaylanto/status.html"
UPTIME=$(uptime -p)
MEMORY=$(free -m | awk 'NR==2{printf "Memory: %sMB / %sMB (%.2f%%)", $3,$2,$3*100/$2 }')
DISK=$(df -h \vert{} awk '$NF=="/"{printf "Disk: %s / %s (%s used)", $3,$2,$5}')
APACHE_STATUS=$(systemctl is-active apache2)

cat << EOF > $OUTPUT_FILE
<!DOCTYPE html>
<html>
<head>
    <title>Cafe Aylanto - Server Telemetry</title>
    <meta http-equiv="refresh" content="15">
    <style>
        body { font-family: monospace; background-color: #121212; color: #00ffcc; padding: 40px; }
        .dashboard { border: 2px solid #00ffcc; padding: 25px; border-radius: 10px; max-width: 650px; margin: 0 auto; }
        h2 { border-bottom: 1px solid #00ffcc; padding-bottom: 10px; color: #ffffff; }
        .status-ok { color: #00ff00; font-weight: bold; }
    </style>
</head>
<body>
    <div class="dashboard">
        <h2>Cafe Aylanto Real-Time Server Telemetry</h2>
        <p><strong>System Uptime:</strong> $UPTIME</p>
        <p><strong>RAM Allocation:</strong> $MEMORY</p>
        <p><strong>Disk Utilization:</strong> $DISK</p>
        <p><strong>Apache2 Service:</strong> <span class="status-ok">$APACHE_STATUS</span></p>
        <hr>
        <p><em>Last Automation Execution: $(date)</em></p>
        <a href="/" style="color:#ffffff;">&larr; Return to Main Cafe Aylanto Website</a>
    </div>
</body>
</html>
EOF

```

### 6.3 Execution & Cron Automation

```bash
# Make script executable
chmod +x ~/scripts/server_telemetry.sh

# Automate telemetry regeneration every minute via crontab
(crontab -l 2>/dev/null; echo "* * * * * /bin/bash /home/ubuntu/scripts/server_telemetry.sh >/dev/null 2>&1") | crontab -

```

### 6.4 Verifiable Output

The output of this script can be verified live on the web server at:
`https://cafeaylanto.duckdns.org/status.html`

---

## 7. References

* Apache HTTP Server Project. (2026). *Apache HTTP Server Version 2.4 Documentation*. https://httpd.apache.org/docs/2.4/
* Certbot Documentation. (2026). *User Guide — Certbot Apache Instructions*. https://certbot.eff.org/instructions
* DuckDNS. (2026). *DuckDNS Installation & Specifications*. https://www.duckdns.org/install.jsp
* Ubuntu Documentation. (2026). *Ubuntu Server Guide - Web Servers*. https://ubuntu.com/server/docs

```

```
