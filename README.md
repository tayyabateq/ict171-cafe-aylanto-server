# Ict171-cafe-aylanto-server
# ICT171 Cloud Server Project - Documentation

| Parameter | Details |
| :--- | :--- |
| **Student Name** | [Insert Your Name Here] |
| **Student ID** | [Insert Your Student ID Here] |
| **Unit Code & Name** | ICT171 Introduction to Server Environments and Architectures |
| **Server Global IP** | [Insert Your Server Public IP Address] |
| **Domain Name** | https://cafeaylanto.duckdns.org/ |
| **GitHub Repository** | [Insert GitHub Repository URL Here] |
| **Video Explainer Link**| [Insert YouTube / Google Drive / Video Link Here] |

---

## 1. Project Architecture & Overview
This project documents the deployment, security hardening, and configuration of an Infrastructure as a Service (IaaS) cloud server running on Ubuntu 22.04 LTS. 

The server hosts **Cafe Aylanto**, a multi-purpose internet presence integrated with an automated real-time server telemetry and health monitoring subsystem.

### Multi-Purpose Server Architecture:
1. **Infrastructure Layer (IaaS):** Cloud VM configured manually via SSH terminal.
2. **Web Service Engine:** NGINX with custom Virtual Host block configurations.
3. **Domain Name System:** Dynamic DNS mapping managed via DuckDNS (`cafeaylanto.duckdns.org`)[cite: 1].
4. **Transport Layer Security:** Free SSL/TLS encryption via Let's Encrypt (Certbot) with enforced HTTP-to-HTTPS redirection[cite: 1].
5. **Scripting & Monitoring Subsystem:** Custom Bash telemetry script generating live HTML server metrics directly integrated into the web application (`/status.html`)[cite: 1].

---

## 2. Infrastructure Provisioning & Hardening (IaaS)

### 2.1 SSH Access
Access to the remote server was established via SSH terminal[cite: 1]:
```bash
ssh ubuntu@YOUR_SERVER_PUBLIC_IP
# Update system dependencies
sudo apt update && sudo apt upgrade -y
#!/bin/bash
# ==============================================================================
# ICT171 Cloud Server Master Deployment Script (Apache Edition)
# Target Domain: cafeaylanto.duckdns.org
# ==============================================================================

# Exit immediately if any command fails
set -e

# ==============================================================================
# CONFIGURATION VARIABLES (Update these before running)
# ==============================================================================
DOMAIN="cafeaylanto.duckdns.org"
WEB_ROOT="/var/www/cafeaylanto"
DUCKDNS_TOKEN="YOUR_DUCKDNS_TOKEN"        # Replace with your actual DuckDNS token
SSL_EMAIL="your-email@example.com"        # Replace with your actual email for SSL alerts

echo "=================================================="
echo "Starting Full Apache Deployment for $DOMAIN"
echo "=================================================="

# ------------------------------------------------------------------------------
# PART 3: APACHE WEB SERVER INSTALLATION & SITE DEPLOYMENT
# ------------------------------------------------------------------------------
echo "[1/4] Installing Apache2 and Setting Up Directory Structure..."

sudo apt update
sudo apt install -y apache2

# Enable rewrite module for SSL redirects later
sudo a2enmod rewrite

# Create web root and set permissions
sudo mkdir -p $WEB_ROOT
sudo chown -R $USER:$USER $WEB_ROOT
sudo chmod -R 755 $WEB_ROOT

# Configure Apache Virtual Host
echo "Configuring Apache Virtual Host file..."
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

# Enable new site and disable default Apache site
sudo a2ensite cafeaylanto.conf
sudo a2dissite 000-default.conf
sudo apache2ctl configtest
sudo systemctl restart apache2

# Create main website HTML page
echo "Deploying main index.html file..."
cat << 'EOF' > $WEB_ROOT/index.html
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

# ------------------------------------------------------------------------------
# PART 4: DUCKDNS CONFIGURATION & AUTOMATION
# ------------------------------------------------------------------------------
echo "[2/4] Setting Up DuckDNS Automatic IP Updates..."

mkdir -p ~/duckdns
cat << EOF > ~/duckdns/duck.sh
echo url="https://www.duckdns.org/update?domains=cafeaylanto&token=$DUCKDNS_TOKEN&ip=" | curl -k -K -
EOF

chmod 700 ~/duckdns/duck.sh

# Run once manually to sync IP
~/duckdns/duck.sh

# Add DuckDNS cron job (runs every 5 minutes)
(crontab -l 2>/dev/null | grep -v "duck.sh"; echo "*/5 * * * * ~/duckdns/duck.sh >/dev/null 2>&1") | crontab -

# ------------------------------------------------------------------------------
# PART 5: CUSTOM SCRIPTING & TELEMETRY DASHBOARD
# ------------------------------------------------------------------------------
echo "[3/4] Creating Custom Server Telemetry Script & Web Status Dashboard..."

mkdir -p ~/scripts

cat << 'EOF' > ~/scripts/server_telemetry.sh
#!/bin/bash
# Script: server_telemetry.sh
# Purpose: Gathers system performance metrics and formats them into an integrated web report.

OUTPUT_FILE="/var/www/cafeaylanto/status.html"
UPTIME=$(uptime -p)
MEMORY=$(free -m | awk 'NR==2{printf "Memory: %sMB / %sMB (%.2f%%)", $3,$2,$3*100/$2 }')
DISK=$(df -h | awk '$NF=="/"{printf "Disk: %s / %s (%s used)", $3,$2,$5}')
APACHE_STATUS=$(systemctl is-active apache2)

cat << HTML_EOF > $OUTPUT_FILE
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
HTML_EOF
EOF

chmod +x ~/scripts/server_telemetry.sh

# Run once immediately to generate initial status.html
~/scripts/server_telemetry.sh

# Add Telemetry cron job (runs every minute)
(crontab -l 2>/dev/null | grep -v "server_telemetry.sh"; echo "* * * * * /bin/bash ~/scripts/server_telemetry.sh >/dev/null 2>&1") | crontab -

# ------------------------------------------------------------------------------
# PART 6: SSL/TLS ENCRYPTION SETUP (Certbot for Apache)
# ------------------------------------------------------------------------------
echo "[4/4] Installing Certbot and Configuring SSL/TLS Encryption for Apache..."

sudo apt install -y certbot python3-certbot-apache

if [ "$SSL_EMAIL" != "your-email@example.com" ]; then
    # Non-interactive certificate generation
    sudo certbot --apache -d $DOMAIN --non-interactive --agree-tos -m $SSL_EMAIL --redirect
else
    # Interactive prompt fallback if default email wasn't changed
    echo "Running Certbot interactively..."
    sudo certbot --apache -d $DOMAIN
fi

echo "=================================================="
echo "APACHE DEPLOYMENT COMPLETED SUCCESSFULLY!"
echo "Main Website: https://$DOMAIN"
echo "Telemetry Page: https://$DOMAIN/status.html"
echo "=================================================="
# Configure firewall access ports
sudo ufw allow 22/tcp    # SSH
sudo ufw allow 80/tcp    # HTTP
sudo ufw allow 443/tcp   # HTTPS
sudo ufw enable
# Create directory structure for Cafe Aylanto
sudo mkdir -p /var/www/cafeaylanto

# Assign ownership to the active user
sudo chown -R $USER:$USER /var/www/cafeaylanto
sudo chmod -R 755 /var/www/cafeaylanto
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
sudo a2ensite cafeaylanto.conf
sudo a2dissite 000-default.conf
sudo apache2ctl configtest
sudo systemctl restart apache2
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
dig cafeaylanto.duckdns.org
mkdir -p ~/duckdns
cat << 'EOF' > ~/duckdns/duck.sh
echo url="[https://www.duckdns.org/update?domains=cafeaylanto&token=YOUR_DUCKDNS_TOKEN&ip=](https://www.duckdns.org/update?domains=cafeaylanto&token=YOUR_DUCKDNS_TOKEN&ip=)" | curl -k -K -
EOF

chmod 700 ~/duckdns/duck.sh

# Automate update script via crontab (runs every 5 minutes)
(crontab -l 2>/dev/null; echo "*/5 * * * * ~/duckdns/duck.sh >/dev/null 2>&1") | crontab -
# Install Certbot for Apache
sudo apt install certbot python3-certbot-apache -y

# Obtain SSL Certificate and automatically configure HTTP-to-HTTPS redirect
sudo certbot --apache -d cafeaylanto.duckdns.org
# Test automated certificate renewal procedure
sudo certbot renew --dry-run
6. Custom Scripting Component & Integrated Visualization
6.1 Script Purpose & Logic
A custom Bash telemetry script (server_telemetry.sh) was created to gather real-time server health metrics (CPU uptime, memory utilization, disk space, and Apache service status)[cite: 1].
To fulfill the rubric requirement for verifiable visual output, the script generates an HTML status page directly into /var/www/cafeaylanto/status.html[cite: 1].
6.2 Script Code (scripts/server_telemetry.sh)
Bash
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
6.3 Execution & Cron Automation
Bash
# Make script executable
chmod +x ~/scripts/server_telemetry.sh

# Automate telemetry regeneration every minute via crontab
(crontab -l 2>/dev/null; echo "* * * * * /bin/bash /home/ubuntu/scripts/server_telemetry.sh >/dev/null 2>&1") | crontab -
6.4 Verifiable Output
The output of this script can be verified live on the web server at[cite: 1]:
https://cafeaylanto.duckdns.org/status.html
7. References
Apache HTTP Server Project. (2026). Apache HTTP Server Version 2.4 Documentation. https://httpd.apache.org/docs/2.4/
Certbot Documentation. (2026). User Guide — Certbot Apache Instructions. https://certbot.eff.org/instructions
DuckDNS. (2026). DuckDNS Installation & Specifications. https://www.duckdns.org/install.jsp
Ubuntu Documentation. (2026). Ubuntu Server Guide - Web Servers. https://ubuntu.com/server/docs
