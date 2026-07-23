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
