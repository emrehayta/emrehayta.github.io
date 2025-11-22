---
title: "SSH Security: Keys, Hardening, Fail2ban, and Best Practices"
date: 2025-02-22
tags: ["SSH", "Linux Security", "Sysadmin", "Hardening", "Fail2ban", "Public Key Authentication", "DevOps"]
description: "A practical guide to securing SSH on Linux systems, covering key-based authentication, hardening techniques, Fail2ban configuration, and recommended security practices."
draft: false
---

### 🔐 Introduction

Secure Shell (SSH) is the backbone of Linux server administration.  
Because it provides direct remote access, SSH is also a **prime attack target** for bots, scanners, and brute-force attempts.

In this guide, you’ll learn:

- how to secure SSH access using **public key authentication**
- how to harden the SSH daemon configuration
- how to use **Fail2ban** to block brute‑force attacks
- additional best practices to reduce exposure and risk

---

### 🔑 1. Use SSH Keys Instead of Passwords

Password authentication is easy — but extremely insecure.  
SSH keys are far stronger and immune to brute-force attacks.

#### Generate a new keypair (on your client)

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

Your keys are stored in:

```
~/.ssh/id_ed25519        # private key
~/.ssh/id_ed25519.pub    # public key
```

#### Copy your key to the server

```bash
ssh-copy-id user@server
```

Alternatively:

```bash
cat ~/.ssh/id_ed25519.pub | ssh user@server "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

---

### 🔧 2. Harden SSHD Configuration

Edit the SSH daemon config:

```bash
sudo nano /etc/ssh/sshd_config
```

Recommended settings:

```
Protocol 2
PasswordAuthentication no
PermitRootLogin no
PubkeyAuthentication yes
PermitEmptyPasswords no
ChallengeResponseAuthentication no
X11Forwarding no
AllowAgentForwarding no
LoginGraceTime 30
MaxAuthTries 3
MaxSessions 2
ClientAliveInterval 300
ClientAliveCountMax 2
```

Reload SSH:

```bash
sudo systemctl reload sshd
```

---

### 🔐 3. Restrict Access with AllowUsers or AllowGroups

Limit SSH to specific users:

```
AllowUsers emre backupadmin
```

Or limit SSH to a group:

```
AllowGroups sshusers
```

---

### 🛡️ 4. Change SSH Port (Optional)

This does **not** replace real security — but it removes bot noise.

Edit:

```
Port 2222
```

Then:

```bash
sudo systemctl restart sshd
```

Update firewall:

```bash
sudo ufw allow 2222/tcp
```

---

### 🚫 5. Enable Fail2ban to Block Brute-force Attacks

Install Fail2ban:

```bash
sudo apt install fail2ban -y   # Debian/Ubuntu
sudo yum install fail2ban -y   # RHEL/Rocky
```

Enable and start:

```bash
sudo systemctl enable --now fail2ban
```

#### Configure SSH jail

Edit:

```bash
sudo nano /etc/fail2ban/jail.local
```

Add:

```
[sshd]
enabled = true
port = ssh
maxretry = 3
bantime = 1h
findtime = 10m
```

Reload:

```bash
sudo systemctl restart fail2ban
```

Check status:

```bash
sudo fail2ban-client status sshd
```

---

### 🧱 6. Additional Best Practices

#### ✔ Use a firewall  
Only allow necessary ports:

```bash
sudo ufw allow 2222/tcp
sudo ufw enable
```

#### ✔ Disable SSH entirely if unused  
For containers or specialized hosts:

```bash
sudo systemctl disable --now sshd
```

#### ✔ Use 2FA for SSH  
Using Google Authenticator or Duo.

#### ✔ Monitor login attempts  
With system logs:

```bash
sudo journalctl -u sshd -f
```

---

### 🏁 Conclusion

SSH is extremely powerful — and equally dangerous if left unsecured.  
By following this guide you significantly reduce your attack surface and protect your Linux infrastructure from automated and targeted attacks.

---

### 🚀 Need Help Securing Your Servers?

If your business relies on Linux servers, secure SSH access is **non‑negotiable**.

**At TechZ (techz.at), we help companies harden their Linux infrastructure, secure remote access, and implement best‑practice server configurations.**

👉 *Need professional help? Reach out anytime.*
