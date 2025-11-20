
# Standard Operating Procedure: Setup a Virtual Linux Server for Web Application Testing

**Document Version:** 1.0  
**Effective Date:** November 19, 2025  
**Author:** Yihan Liu  
**Status:** Approved

---

## Document Approval

| Role | Name | Signature | Date |
|------|------|------------|------|
| Author | Yihan Liu | | 2025-11-19 |

---

## Table of Contents
1. [Purpose](#purpose)
2. [Scope](#scope)
3. [Definitions](#definitions)
4. [Prerequisites](#prerequisites)
5. [Resource Requirements](#resource-requirements)
6. [Procedure Steps](#procedure-steps)
7. [Software Installation](#software-installation)
8. [Validation Testing](#validation-testing)
9. [Troubleshooting](#troubleshooting)
10. [References](#references)
11. [Revision History](#revision-history)

---

## Purpose

This Standard Operating Procedure (SOP) establishes the standardized process for deploying virtual Linux servers dedicated to web application testing environments. The procedure ensures consistent, secure, and reproducible server configurations that meet organizational standards for quality assurance and development testing.

## Scope

This SOP applies to:
- Quality Assurance Engineers
- DevOps Engineers
- System Administrators
- Development Teams
- IT Operations Staff

The procedure covers environments used for:
- Functional testing
- Integration testing
- Performance testing
- Security vulnerability testing
- User acceptance testing (UAT)

## Definitions

- **LEMP Stack**: Linux, Nginx, MySQL/MariaDB, PHP/Python
- **CI/CD**: Continuous Integration/Continuous Deployment
- **VM**: Virtual Machine
- **Containerization**: OS-level virtualization method
- **Reverse Proxy**: Server that handles client requests and forwards them to backend services

## Prerequisites

### Access Requirements
- VMware vSphere 7.0+ or equivalent hypervisor access
- SSH key-based authentication setup
- Organizational VPN access (if applicable)
- Git repository access

### Technical Requirements
- Ubuntu Server 22.04 LTS ISO image
- Network configuration details
- Storage allocation approval
- Backup strategy defined

## Resource Requirements

### Virtual Machine Specifications
| Resource | Development | Testing | Production-like |
|----------|-------------|---------|-----------------|
| vCPUs | 2 | 4 | 8 |
| Memory | 4GB | 8GB | 16GB |
| Storage | 50GB | 100GB | 200GB |
| Network | NAT | Bridged | Bridged |

### Recommended Configuration
- **Hostname**: `web-test-app-01`
- **OS**: Ubuntu Server 22.04 LTS
- **Disk Layout**: 
  - Root partition: 40GB
  - Swap: 4GB
  - /var: 6GB (extra space for logs)
- **Network**: Static IP assignment preferred

## Procedure Steps

### Phase 1: Virtual Machine Deployment

#### Step 1.1: VM Creation
1. Access VMware vSphere Client
2. Navigate to target cluster/datacenter
3. Right-click → "New Virtual Machine"
4. Select "Custom" configuration type
5. Specify name: `web-test-app-01`

#### Step 1.2: Resource Allocation
1. Configure hardware compatibility: ESXi 7.0+
2. Set guest OS: Ubuntu Linux (64-bit)
3. Allocate resources per specifications
4. Select thin provisioned disk
5. Configure network adapter

#### Step 1.3: OS Installation
1. Mount Ubuntu 22.04 LTS ISO
2. Power on VM and begin installation
3. Configure:
   - Language: English
   - Keyboard: US English
   - Network: Static IP (if required)
   - Storage: Use entire disk with LVM
   - Profile: Set user credentials
   - SSH: Install OpenSSH server

### Phase 2: System Configuration

#### Step 2.1: Initial Setup

# Update system
sudo apt update && sudo apt upgrade -y

# Set hostname
sudo hostnamectl set-hostname web-test-app-01

# Configure static IP (if needed)
sudo nano /etc/netplan/00-installer-config.yaml
Step 2.2: Security Hardening
bash
# Configure firewall
sudo ufw enable
sudo ufw allow OpenSSH
sudo ufw allow from 192.168.1.0/24 to any port 22

# Create testing user
sudo adduser tester
sudo usermod -aG sudo tester
Phase 3: Web Stack Installation
Step 3.1: Install Nginx
bash
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
Step 3.2: Install Database
bash
# Install MariaDB
sudo apt install mariadb-server -y
sudo mysql_secure_installation

# Create test database
sudo mysql -e "CREATE DATABASE webapp_test;"
sudo mysql -e "CREATE USER 'test_user'@'localhost' IDENTIFIED BY 'secure_password';"
sudo mysql -e "GRANT ALL PRIVILEGES ON webapp_test.* TO 'test_user'@'localhost';"
Step 3.3: Install PHP
bash
sudo apt install php-fpm php-mysql php-curl php-json php-gd php-mbstring php-xml -y
sudo systemctl enable php8.1-fpm
Software Installation
Essential Testing Tools
Version Control
bash
sudo apt install git -y
git config --global user.name "Test User"
git config --global user.email "tester@company.com"
Programming Languages
bash
# Node.js for modern web apps
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install nodejs -y

# Python for scripting
sudo apt install python3 python3-pip -y
Testing Dependencies
bash
# Selenium for browser automation
sudo apt install default-jdk -y
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb

# API testing tools
sudo apt install curl jq -y
npm install -g newman  # Postman CLI
Containerization
bash
# Docker for container testing
sudo apt install docker.io -y
sudo systemctl enable docker
sudo usermod -aG docker tester
Configuration Files
Nginx Virtual Host
bash
sudo nano /etc/nginx/sites-available/webapp-test
nginx
server {
    listen 80;
    server_name web-test-app-01;
    root /var/www/html;
    index index.php index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
    }
}
Validation Testing
Service Verification
Web Server Test

bash
curl -I http://localhost
# Expected: HTTP/1.1 200 OK
Database Connection

bash
mysql -u test_user -p -e "SHOW DATABASES;"
PHP Processing

bash
echo "<?php phpinfo(); ?>" | sudo tee /var/www/html/test.php
curl http://localhost/test.php
Performance Baseline
Load Testing

bash
sudo apt install apache2-utils -y
ab -n 1000 -c 10 http://localhost/
Resource Monitoring

bash
sudo apt install htop iotop -y
Troubleshooting
Common Issues
Service Failures
bash
# Check service status
sudo systemctl status nginx
sudo systemctl status mariadb
sudo systemctl status php8.1-fpm

# View logs
sudo journalctl -u nginx -f
sudo tail -f /var/log/nginx/error.log
Permission Problems
bash
# Set correct ownership
sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 755 /var/www/html
Network Issues
bash
# Verify connectivity
ping -c 4 google.com
netstat -tulpn
ss -tulpn
References
Ubuntu Server Guide

Nginx Documentation

MariaDB Knowledge Base

Mozilla Web Security Guidelines

Revision History
| Version | Date | Author | Changes |
|------|------|------------|------|
| Author | Yihan Liu | | 2025-11-19 |
