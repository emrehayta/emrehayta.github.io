---
title: "Automating ZFS Snapshots with a Simple Bash Script"
description: "Learn how to automate daily ZFS snapshots with a small, generic Bash script that includes automatic cleanup of old snapshots."
date: 2025-11-24
tags: ["zfs", "linux", "backup", "automation", "devops"]
---

ZFS offers powerful snapshot functionality that makes backups fast, atomic, and space efficient.  
But in many environments, snapshots are still created manually – or not at all.

In this post, we will:

- create a **generic Bash script** to automate ZFS snapshots
- support **multiple datasets**
- implement a **retention policy** (e.g. keep the last 7 days)
- integrate the script with **cron** for daily backups

No additional tools, just `bash` and `zfs`.

---

## ✅ The generic `zfs-snapshot.sh` script

Below is a generic script you can drop onto any ZFS-based system.  
You only need to adjust the dataset names and retention period at the top.

```bash
#!/usr/bin/env bash
set -euo pipefail

########################################
# ZFS Snapshot Automation Script
#
# - Creates snapshots for one or more datasets
# - Uses a simple naming convention: <dataset>@<prefix>-<timestamp>
# - Deletes old snapshots based on a retention window (in days)
#
# Requirements:
# - Linux with ZFS tools installed
# - `date` with `-d` support (e.g. GNU date)
########################################

# --- Configuration ----------------------------------------------------------

# List of ZFS datasets to snapshot
DATASETS=(
  "tank/data"
  "tank/vms"
)

# How many days snapshots should be kept
RETENTION_DAYS=7

# Prefix for automatically created snapshots
SNAPSHOT_PREFIX="auto"

# --- Implementation ---------------------------------------------------------

TIMESTAMP=$(date +"%Y-%m-%d_%H-%M-%S")

for dataset in "${DATASETS[@]}"; do
    SNAPSHOT_NAME="${dataset}@${SNAPSHOT_PREFIX}-${TIMESTAMP}"
    echo "[INFO] Creating snapshot: ${SNAPSHOT_NAME}"
    zfs snapshot "${SNAPSHOT_NAME}"

    echo "[INFO] Cleaning up old snapshots for dataset: ${dataset}"
    # List snapshots for this dataset with the configured prefix, oldest first
    zfs list -H -t snapshot -o name -s creation | grep "^${dataset}@${SNAPSHOT_PREFIX}-" | while read -r SNAP; do
        CREATION_STR=$(zfs get -H -o value creation "$SNAP")
        SNAP_TS=$(date -d "$CREATION_STR" +%s)
        CUTOFF_TS=$(date -d "${RETENTION_DAYS} days ago" +%s)

        if (( SNAP_TS < CUTOFF_TS )); then
            echo "[CLEANUP] Destroying old snapshot: $SNAP"
            zfs destroy "$SNAP"
        fi
    done
done
```

Save the script as `zfs-snapshot.sh` and make it executable:

```bash
chmod +x zfs-snapshot.sh
```

---

## 🕒 Automating snapshots with `cron`

To run the script automatically every night at 01:00, add a cronjob:

```bash
sudo crontab -e
```

Add this line (adjust the path if needed):

```bash
0 1 * * * /usr/local/sbin/zfs-snapshot.sh >> /var/log/zfs-snapshot.log 2>&1
```

This will:

- execute the script daily at 01:00
- log output and errors to `/var/log/zfs-snapshot.log`

---

## 🛠 Customization ideas

You can easily adapt the script to your environment:

- Change `DATASETS` to match your pools and datasets
- Adjust `RETENTION_DAYS` (e.g. 3, 7, 14, 30)
- Use different prefixes per environment, e.g.
  - `SNAPSHOT_PREFIX="auto-dev"`
  - `SNAPSHOT_PREFIX="auto-prd"`

For more advanced setups, you could:

- push snapshots to a **backup server** via `zfs send` / `zfs receive`
- create **hourly snapshots** with a different cron schedule
- send metrics/logs to your monitoring system

---

## 🔚 Summary

With less than 50 lines of Bash, you get:

- consistent, automatic ZFS snapshots
- an easy retention policy
- a simple and transparent backup mechanism

Snapshotted filesystems are one of the biggest advantages of ZFS —  
so it makes sense to automate them properly instead of relying on manual commands.

If you want, we can extend this setup in a follow-up post with:

- ZFS replication to a second server
- off-site backups
- or a full snapshot/replication strategy for production environments.
