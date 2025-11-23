---
title: "Top 20 systemd Commands Every Linux Admin Should Know"
date: 2025-02-23
tags: ["Linux", "systemd", "Sysadmin", "DevOps", "Automation"]
description: "A practical guide to the most important systemd commands every Linux administrator should master — including service management, troubleshooting, logs, targets, and performance tuning."
draft: false
---

Systemd is the backbone of modern Linux distributions — handling services, logging, targets, startup sequences, timers, sockets, crash diagnostics, and more.

Whether you're maintaining servers, containers, homelabs, or enterprise infrastructure, mastering systemd is essential for efficient troubleshooting and smooth operations.

In this guide, you'll learn the *20 most important* systemd commands that every Linux administrator should know, with real examples and best practices.

---

### 🔧 1. Check Service Status  
```bash
systemctl status <service>
```

### ▶️ 2. Start a Service  
```bash
sudo systemctl start <service>
```

### ⏹️ 3. Stop a Service  
```bash
sudo systemctl stop <service>
```

### 🔁 4. Restart a Service  
```bash
sudo systemctl restart <service>
```

### 🔄 5. Reload a Service  
```bash
sudo systemctl reload <service>
```

### 🚀 6. Enable Service at Boot  
```bash
sudo systemctl enable <service>
```

### 🛑 7. Disable Service at Boot  
```bash
sudo systemctl disable <service>
```

### 🧱 8. Mask / Unmask a Service  
```bash
sudo systemctl mask <service>
sudo systemctl unmask <service>
```

### 🧩 9. Check All Failed Services  
```bash
systemctl --failed
```

### 📜 10. View Logs for a Service  
```bash
journalctl -u <service>
```

### 🚨 11. Last 100 Log Lines  
```bash
journalctl -u <service> -n 100
```

### 🕒 12. View Logs Since a Time  
```bash
journalctl -u <service> --since "2 hours ago"
```

### 📅 13. Check Boot History  
```bash
journalctl --list-boots
```

### 📦 14. List All Services  
```bash
systemctl list-units --type=service
```

### 🎯 15. Show Current Target  
```bash
systemctl get-default
```

### 🎛️ 16. Change Boot Target  
```bash
sudo systemctl set-default graphical.target
sudo systemctl set-default multi-user.target
```

### ⏲️ 17. List All Timers  
```bash
systemctl list-timers
```

### ⛔ 18. Shut Down  
```bash
sudo systemctl poweroff
```

### 🔄 19. Reboot  
```bash
sudo systemctl reboot
```

### 💥 20. Analyze Failed Boots  
```bash
systemd-analyze
systemd-analyze blame
systemd-analyze critical-chain
```

---

### 🧠 Best Practices for Using systemd

✔ Use `journalctl -xe` for quick troubleshooting  
✔ Keep overrides in `/etc/systemd/system/<service>.d/override.conf`  
✔ Use timers instead of cron where possible  
✔ Mask unused services  
✔ Test reload before restart using:
```bash
systemctl reload-or-restart <service>
```

---

### 🚀 Need Help Managing Linux Servers?

At **TechZ (techz.at)**, we help companies harden Linux servers, fix failing services, and implement best-practice configurations.

👉 *Need expert help? Reach out anytime.*
