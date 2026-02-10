# Ubuntu Server Hardening

A practical hardening reference for Ubuntu Server, covering SSH, firewall, intrusion prevention, and OS hardening techniques. Based on academic research and tested configurations.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
![Platform](https://img.shields.io/badge/platform-Ubuntu%20Server-E95420.svg)
![Purpose](https://img.shields.io/badge/purpose-educational-green.svg)

---

## Contents

1. [SSH Hardening](#1-ssh-hardening)
2. [Host-based Firewall (ufw)](#2-host-based-firewall-ufw)
3. [Intrusion Prevention (Fail2Ban)](#3-intrusion-prevention-fail2ban)
4. [AppArmor](#4-apparmor)
5. [General Hardening](#5-general-hardening)
6. [Automated Security (Ubuntu Pro)](#6-automated-security-ubuntu-pro)

---

## 1. SSH Hardening

### Enable and verify SSH

```bash
sudo systemctl enable ssh
systemctl list-unit-files | grep ssh
sudo sshd -t -f /etc/ssh/sshd_config
```

If `/run/sshd` is missing:

```bash
sudo mkdir -p /run/sshd
sudo chmod 0755 /run/sshd
sudo chown root:root /run/sshd
sudo systemctl restart sshd
```

### Backup the original config

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.original
sudo chmod a-w /etc/ssh/sshd_config.original
```

### Restrict SSH access to a group

```bash
sudo groupadd sshlogin
sudo usermod -aG sshlogin <username>
```

### Configure `/etc/ssh/sshd_config`

```bash
sudo nano /etc/ssh/sshd_config
```

Key settings:

```
PermitRootLogin no
PasswordAuthentication no
AllowGroups sshlogin
```

### Generate and copy SSH keys (Ed25519)

```bash
ssh-keygen -t ed25519 -C "user@hostname"
ssh-copy-id user@<server-ip>
```

---

## 2. Host-based Firewall (ufw)

### Install and configure

```bash
sudo apt install ufw -y
sudo systemctl start ufw

sudo ufw default deny incoming
sudo ufw default allow outgoing

sudo ufw app list
sudo ufw allow OpenSSH
sudo ufw allow 'Apache Secure'

sudo ufw logging medium
sudo ufw limit 22/tcp

sudo ufw enable
sudo ufw status
```

---

## 3. Intrusion Prevention (Fail2Ban)

### Install and verify

```bash
sudo apt install fail2ban -y
sudo systemctl status fail2ban
```

### Configure SSH jail

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

Add or update the `[sshd]` section:

```ini
[sshd]
enabled = true
port    = ssh
filter  = sshd
maxretry = 3
bantime  = 3600
```

```bash
sudo systemctl restart fail2ban
```

---

## 4. AppArmor

AppArmor ships with Ubuntu and provides mandatory access control (MAC) for applications.

```bash
sudo systemctl status apparmor
```

If not active:

```bash
sudo systemctl enable --now apparmor
```

---

## 5. General Hardening

### Check and disable unused services

```bash
sudo ss -tulnp | grep LISTEN

sudo systemctl stop <service>
sudo systemctl disable <service>
```

### Keep the system updated

```bash
sudo apt update && sudo apt upgrade -y
```

### Install unattended security upgrades

```bash
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

---

## 6. Automated Security (Ubuntu Pro)

With an Ubuntu Pro subscription, the Ubuntu Security Guide (USG) automates CIS benchmark hardening:

```bash
sudo apt install ubuntu-security-guide -y
sudo usg fix cis_level1_server
```

---

## Research

This guide is based on an academic project *Secure Network* (February 2025), which demonstrates a layered security approach for a small company's Linux server infrastructure.

Full report available under research.

---

## Related Projects

- [homelab-scripts](https://github.com/ZalPika/homelab-scripts) — Infrastructure automation
- [usb-dropper-research](https://github.com/ZalPika/usb-dropper-research) — USB malware research
