---
author: Ali Abharya
pubDatetime: 2024-02-18T19:12:37.909Z
modDatetime: 2024-02-18T19:12:37.909Z
title: Nginx security practices
slug: nginx-security-practices
featured: true
draft: false
tags:
  - nginx
  - security
  - deployment
  - configuration
description: ""
---

# Hardening Nginx Server and Security Practices

## 1. Update and Upgrade System Packages

Before setting up Nginx, ensure your server's operating system and packages are up-to-date. Use the following commands to update and upgrade:

```bash
sudo apt update
sudo apt upgrade
```

## 2. Install Nginx with Secure Configuration

Install Nginx and choose secure configuration options. Disable unnecessary modules and services, leaving only the essentials. The following command installs Nginx on Ubuntu:

```bash
sudo apt install nginx
```

## 3. Configure Firewall Rules

Use a firewall to control incoming and outgoing traffic. UFW (Uncomplicated Firewall) is a simple option:

```bash
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable
```

Replace 'Nginx Full' with 'Nginx HTTP' for a basic HTTP setup.

## 4. Enable TLS/SSL Encryption

Secure data in transit by implementing TLS/SSL certificates. Obtain and install a certificate using Let's Encrypt:

```bash
sudo apt install certbot
sudo certbot --nginx -d your_domain.com
```

## 5. Implement Security Headers and Hardening Techniques

Enhance security by adding headers and implementing best practices. Modify your Nginx configuration file (`/etc/nginx/nginx.conf` or site-specific files) to include:

```bash
  server {
    # Security Headers
    add_header X-Content-Type-Options "nosniff";
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload";

    # Disable unnecessary features
    server_tokens off;
    fastcgi_hide_header X-Powered-By;
  }
```
