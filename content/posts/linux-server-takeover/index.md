---
title: "What I Check Before Taking Over a Linux Server"
date: 2026-10-09
tags: ["linux", "sysadmin", "operations", "infrastructure", "documentation"]
description: "A structured Linux server handover: identify dependencies, review access and storage, verify backups, and build a prioritised operating plan before making changes."
draft: false
---

Taking responsibility for an existing Linux server starts with understanding what depends on it. An unfamiliar service, an undocumented mount, or a scheduled script may be more important than its name suggests.

The first goal is a reliable picture of the system: what it does, who uses it, how it is maintained, and how it could be recovered. Changes become easier to plan once that picture exists.

{{< figure src="featured.svg" alt="A Linux server handover organised into four areas: purpose, access, dependencies, and recovery." >}}

## Establish the service boundary

Ask the current owner a few concrete questions:

- Which business process stops if this machine disappears?
- Who owns the application and its data?
- Which users and other systems connect to it?
- What are the support hours and maintenance windows?
- Where are credentials, documentation, and recovery instructions stored?

Record what is confirmed and what is still an assumption. A hostname such as `test-old` is not evidence that a machine is unused.

Get access through the agreed administration process. Keep an existing working access path available while testing any replacement, and confirm how console access works if SSH fails.

## Take a read-only baseline

On a typical systemd-based Linux host, these commands provide a starting point:

```bash
hostnamectl
cat /etc/os-release
uname -r
uptime
systemctl --failed
systemctl list-units --type=service --state=running
```

They identify the system and active services, but they do not establish whether the host is supported or correctly configured. Compare the operating system release with its vendor's lifecycle and the organisation's support arrangements.

Also identify virtualisation, cloud instance details where relevant, and any configuration management agent. If Ansible or another system owns a configuration file, changing it manually may only last until the next run.

Keep collected information in a restricted handover record. Logs, process arguments, unit files, and container configuration can contain sensitive data.

## Map listeners to actual services

With suitable privileges, inspect listening TCP and UDP sockets:

```bash
sudo ss -lntup
systemctl list-sockets
```

For each expected listener, record the service, bind address, intended clients, and network restrictions. Listening on all interfaces does not prove a port is reachable from the internet; host firewalls, cloud controls, routing, and upstream devices also matter.

Compare host findings with the network configuration and an authorised check from the relevant client network. Do not assume a service is internal because its users usually access it over a VPN.

If Docker is installed, inspect the running workload without changing it:

```bash
docker ps
docker compose ls
```

Docker access is highly privileged. Use the established access model rather than granting broad permissions just to collect an inventory. For containers, locate the Compose project, image references, persistent volumes, and the actual deployment procedure.

## Understand storage before touching it

Check devices, mounts, capacity, and inodes:

```bash
lsblk -f
findmnt
df -hT
df -ih
```

Look for separate application volumes, remote mounts, unexpected read-only filesystems, and paths that can grow without bounds. A missing mount can leave an application writing to the underlying root filesystem, so verify important paths against the intended mount layout.

Ask what manages the storage: cloud volumes, LVM, RAID, ZFS, or an external platform. Record the expansion and recovery procedure. A snapshot and a second disk in the same machine solve different problems from an independent backup.

Do not begin by deleting old-looking directories or rotating unfamiliar logs. First establish ownership, retention requirements, and whether a running process still uses them.

## Find the work that runs without a login

Scheduled tasks often contain the least documented business logic.

```bash
systemctl list-timers --all
crontab -l
sudo crontab -l
```

These commands cover system timers and the current user's and root's crontabs. They do not enumerate every user's jobs. Also review `/etc/crontab`, `/etc/cron.d/`, periodic cron directories, relevant user timers, and application or container schedulers.

For important jobs, record the input, output, credentials, owner, expected schedule, and failure notification. Check whether the job actually completed recently. An enabled timer only establishes that a schedule exists.

Pay attention to jobs that copy data to other machines or pull files from external systems. Those are dependencies you will need during recovery and change planning.

## Review access with context

Inventory administrative accounts, sudo rules, SSH access, service accounts, and the identity provider. Confirm which accounts belong to active people and which belong to automation.

Do not remove an unfamiliar account until its owner and dependencies are understood. An account with no recent interactive login may still run a deployment or a file transfer.

Establish how access is approved, revoked, and recovered during an outage. If the only administrator credentials live behind an identity service hosted on this same machine, document and resolve that circular dependency.

Separate discovery from remediation. Access changes should have a tested alternative path and an agreed change window where required.

## Read logs for patterns

Start with failed services and a bounded time range:

```bash
journalctl -p err -b --no-pager
journalctl -u nginx --since '24 hours ago' --no-pager
```

Replace `nginx` with an actual service. These examples cover journal entries available to the current user; retention, permissions, and application-specific log files affect what you can see. A lack of old errors may mean the logs were not retained.

Look for recurring restarts, storage errors, certificate failures, failed scheduled work, and repeated authentication problems. Correlate them with monitoring and user reports. One noisy log line is not enough to diagnose the system.

The [journalctl guide]({{< ref "journalctl" >}}) covers more targeted queries. Upstream documentation describes [systemctl](https://www.freedesktop.org/software/systemd/man/latest/systemctl.html) and [journalctl](https://www.freedesktop.org/software/systemd/man/latest/journalctl.html) options; availability depends on the installed version.

## Verify recovery and change procedures

Ask for the backup scope, retention, storage location, and the latest restore-test result. Confirm that configuration, database data, uploads, and required secrets are recoverable together.

If the answer is only a successful backup notification, schedule an isolated [restore drill]({{< ref "backup-restore-testing" >}}). Keep the distinction between a configured backup and demonstrated recovery explicit.

Then understand updates: package sources, patch cadence, pending reboot expectations, application releases, and rollback limitations. Plan a reboot test if the service's boot behaviour has not been verified, rather than discovering dependencies during an unplanned restart.

## Deliver a prioritised handover

A useful handover ends with a short operating record:

| Area | Evidence to retain |
| --- | --- |
| Purpose | Service owner, users, support window, dependencies |
| Access | Approved administration path and console recovery |
| Operation | Deployment, patching, monitoring, scheduled jobs |
| Data | Storage layout, backup scope, restore evidence |
| Open work | Impact, proposed action, owner, target date |

Prioritise according to impact and evidence. An actively failing backup needs a different response from a cosmetic configuration inconsistency. Label unknowns clearly so they do not silently become accepted facts.

The first successful outcome is a server another engineer can understand and operate. From there, hardening, automation, and modernisation can follow a deliberate plan with fewer surprises.
