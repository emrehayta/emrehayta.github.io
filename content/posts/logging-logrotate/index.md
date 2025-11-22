---
title: "Linux Log Management: journalctl, Logrotate & Best Practices"
date: 2025-02-22
tags: ["Linux", "Logging", "Sysadmin", "journalctl", "logrotate", "Troubleshooting", "DevOps"]
description: "A complete guide to managing logs on Linux systems using journalctl, logrotate, persistent storage, cleanup automation, and best practices for production environments."
draft: false
---

### 📝 Introduction

Logs are one of the most important tools for debugging, auditing, and monitoring Linux systems.  
But without proper log management, your server can quickly run into problems:

- disks filling up  
- slow journal queries  
- missing historic logs  
- bloated `/var/log/` directories  
- performance issues  

In this guide, you'll learn how to efficiently manage logs using **journalctl**, **systemd-journald**, **logrotate**, and important best practices for production servers.

---

### 📚 1. Understanding journald & journalctl

Most modern Linux distributions use **systemd-journald** for log collection.

#### View logs (basic)

```bash
journalctl
```

#### Follow logs live

```bash
journalctl -f
```

#### Show logs for a specific service

```bash
journalctl -u sshd
journalctl -u nginx
journalctl -u docker
```

#### Show logs since boot

```bash
journalctl -b
```

#### Show logs for the last hour

```bash
journalctl --since "1 hour ago"
```

---

### 💾 2. Enable Persistent Logging

By default, some distros log *only to memory* (volatile).  
To enable persistent logs:

```bash
sudo mkdir -p /var/log/journal
sudo systemctl restart systemd-journald
```

Now journald writes logs to disk.

#### Check journald storage mode

```bash
journalctl --disk-usage
```

---

### 🛠️ 3. Configure journald (Storage, Compression, Limits)

Edit:

```bash
sudo nano /etc/systemd/journald.conf
```

Important options:

```
Storage=persistent
SystemMaxUse=500M
SystemKeepFree=1G
SystemMaxFileSize=50M
RuntimeMaxUse=200M
Compress=yes
```

Apply changes:

```bash
sudo systemctl restart systemd-journald
```

---

### 🔄 4. Vacuum (Cleanup) Old Logs

Delete logs older than 7 days:

```bash
journalctl --vacuum-time=7d
```

Delete logs until usage < 200M:

```bash
journalctl --vacuum-size=200M
```

Delete oldest files until only recent 10 files remain:

```bash
journalctl --vacuum-files=10
```

---

### 📂 5. Understanding logrotate

Traditional logs (non-journald) live in:

```
/var/log/*.log
```

`logrotate` handles:

- rotation  
- compression  
- retention  
- permissions  

### Example rotation config (Nginx)

File:

```
/etc/logrotate.d/nginx
```

Content:

```
/var/log/nginx/*.log {
    daily
    missingok
    rotate 14
    compress
    delaycompress
    notifempty
    sharedscripts
    postrotate
        systemctl reload nginx
    endscript
}
```

Run manually:

```bash
sudo logrotate -f /etc/logrotate.conf
```

Check status:

```bash
cat /var/lib/logrotate/status
```

---

### 🔧 6. Troubleshooting Common Log Problems

#### Disk full because journald grew too large

```bash
journalctl --disk-usage
journalctl --vacuum-size=200M
```

#### Logrotate not running?

```bash
sudo systemctl status logrotate.timer
```

#### Logs missing after reboot?

Check if persistent logging is enabled.

#### Service not logging?

```bash
journalctl -u <service> -e
```

#### Check file permissions

```bash
ls -la /var/log
```

---

### 🧰 7. Automating Log Cleanup (Example Script)

Add this to your scripts repo:

```bash
#!/usr/bin/env bash

# cleanup-logs.sh — safe log cleanup

journalctl --vacuum-size=200M
journalctl --vacuum-time=14d

logrotate -f /etc/logrotate.conf

echo "Log cleanup completed."
```

---

### ⭐ 8. Best Practices for Production

- Enable persistent journald logs  
- Set size limits to avoid disk explosions  
- Rotate logs regularly  
- Monitor `/var/log/` usage  
- Don’t keep logs forever (privacy + disk usage)  
- Use central logging for important systems (ELK, Loki, CloudWatch, etc.)  

---

### 🎯 Conclusion

Log management is essential for stable Linux systems.  
With the right combination of **journalctl**, **journald limits**, and **logrotate**, you ensure your system stays clean, fast, and predictable — even under heavy load.

---

### 🚀 Need Help Managing Linux Servers?

Managing logs, storage, and system performance can be time-consuming.

**TechZ (techz.at)** helps companies:

- secure & maintain Linux servers  
- optimize log retention  
- prevent storage outages  
- automate monitoring & maintenance  

👉 *Need expert help? Contact us anytime.*
