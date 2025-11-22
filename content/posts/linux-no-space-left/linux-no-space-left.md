---
title: "How to Fix ‘No Space Left on Device’ on Linux — 10 Common Causes"
date: 2025-11-21
tags: ["Linux", "Troubleshooting", "Sysadmin", "Filesystems", "Storage", "DevOps", "Disk Management"]
description: "A complete guide to fixing the 'No Space Left on Device' error on Linux, including 10 real-world causes and step-by-step solutions for storage, inodes, logs, Docker, and more."
draft: false
---

### Introduction

The infamous **“No space left on device”** error in Linux can appear even when the disk *looks* like it has free space.  
This error is extremely common across servers, homelabs, containers, CI pipelines, and production systems.

In this guide, we break down the **10 most common causes**—and how to fix each one.

---

### 1. The Disk Is Actually Full

#### Check Disk Usage

```bash
df -h
```

Look for partitions at **100%** usage.

#### Fix

- Clean logs (see section 3)
- Remove old backups, cache, temp files
- Move large data to another volume

---

### 2. Inodes Are Full (Even If Disk Space Is Not)

Linux tracks files using **inodes**.  
When inodes are exhausted, you *cannot create new files* even if you have plenty of disk space left.

#### Check inode usage

```bash
df -i
```

If it shows **100%**, that’s the cause.

#### Fix

Find directories with huge numbers of tiny files:

```bash
sudo find / -xdev -type d -exec sh -c 'printf "%10s  %s
" "$(ls -1A "$1" | wc -l)" "$1"' sh {} \; | sort -n
```

Then clean the culprit directory.

---

### 3. Log Files Grew Out of Control

Typical locations:

- /var/log/syslog
- /var/log/messages
- /var/log/journal/

#### Check largest logs

```bash
sudo du -sh /var/log/*
```

#### Fix

```bash
sudo truncate -s 0 /var/log/syslog
sudo systemctl restart rsyslog
```

For journald:

```bash
sudo journalctl --vacuum-size=200M
```

---

### 4. Docker Consuming All Disk Space

#### Check Docker disk usage

```bash
docker system df
```

#### Clean unused images & containers

```bash
docker system prune -a
docker volume prune
```

---

### 5. Snapshots Filling Storage (ZFS, Btrfs, LVM)

#### Check ZFS usage

```bash
zfs list -t snapshot
```

Remove snapshots:

```bash
sudo zfs destroy tank/data@snap1
```

#### Check Btrfs usage

```bash
sudo btrfs filesystem usage /
```

---

### 6. A Mounted Volume Is Full

#### Check mounts

```bash
mount | column -t
```

If the *target mount* is full, you get the error even when `/` has free space.

---

### 7. Tmpfs or /tmp Is Full

#### Check

```bash
df -h /tmp
```

#### Fix

- Clear /tmp
- Increase tmpfs size in /etc/fstab

---

### 8. A Process Holding Deleted Files

#### Check open deleted files

```bash
sudo lsof | grep deleted
```

#### Fix

Restart the relevant service:

```bash
sudo systemctl restart <service>
```

---

### 9. Quotas Reached

#### Check quotas

```bash
sudo repquota -a
```

#### Fix

```bash
sudo edquota <username>
```

---

### 10. Wrong Partition Used Due to Missing Mount

If a mount fails, applications may write to the underlying root filesystem.

#### Check

```bash
mount | grep /data
```

---

### Conclusion

The **“No space left on device”** error can come from many different sources—not just lack of space.  
By checking disk usage, inodes, logs, mounts, Docker, and snapshots, you can diagnose and fix it quickly.

---

### Need Help?

If your business relies on Linux systems and you want professional support:

**👉 TechZ — We solve Linux problems.**  
https://techz.at

We help with:

- Storage issues  
- Production incidents  
- Filesystems (ZFS, LVM, ext4, Btrfs)  
- Docker & Kubernetes  
- Automation & DevOps

Reach out anytime.
