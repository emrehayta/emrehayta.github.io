---
title: "Docker Is Filling the Disk: A Cleanup Plan That Keeps Your Data"
date: 2026-09-25
tags: ["docker", "linux", "storage", "troubleshooting", "operations"]
description: "Diagnose Docker disk growth, distinguish logs and caches from durable volumes, and reclaim reviewed resources without blindly pruning application data."
draft: false
---

A Docker host runs out of disk space. A search turns up a one-line prune command. It is tempting to run it immediately.

First establish what is growing. Images, build cache, container logs, writable layers, and volumes have different lifecycles. Removing the wrong category can turn a storage problem into a recovery exercise.

This walkthrough assumes a Linux Docker Engine host. Docker Desktop adds a VM and virtual disk layer, so host filesystem accounting can look different.

{{< figure src="featured.svg" alt="A storage map separates image layers, build cache, logs, and durable application data, with data volumes marked for review." >}}

## Find the filesystem under pressure

Start with the host and the daemon's reported data directory:

```bash
docker info --format '{{.DockerRootDir}}'
df -hT
df -ih
docker system df -v
```

Docker's data root is an important clue, but not the entire footprint. Bind mounts, external volume drivers, and some image-store arrangements can keep data elsewhere. Inspect actual mount locations before concluding which filesystem is full.

`docker system df -v` shows Docker's view of objects and reclaimable storage. It does not replace filesystem accounting. Shared image layers also mean individual image sizes are not simply additive.

If inodes are exhausted, freeing a few large files may not solve the problem. The [Linux no-space guide]({{< ref "linux-no-space-left" >}}) covers that distinction.

## Separate data from disposable resources

Use a working classification:

| Category | Question before removal |
| --- | --- |
| Image | Is it needed for the running release or a planned rollback? |
| Build cache | Can the build process regenerate it within an acceptable time? |
| Stopped container | Does its writable layer contain anything that was never persisted? |
| Container log | What retention is required, and is rotation configured? |
| Volume | Which application owns the data, and is recovery tested? |

A stopped container is not necessarily disposable. An unused volume is not necessarily unimportant. “Unused” describes Docker's references, not business value.

## Investigate logs and writable layers

List containers, including stopped ones, and inspect their writable layer sizes:

```bash
docker ps -a --size
```

For a selected container:

```bash
docker inspect --format '{{json .HostConfig.LogConfig}}' CONTAINER
```

Replace `CONTAINER` with the intended name or ID. A large writable layer can indicate that uploads, caches, or generated files are going inside the container instead of a deliberate mount.

For a service using the `json-file` driver, configure bounded retention in Compose:

```yaml
logging:
  driver: json-file
  options:
    max-size: "10m"
    max-file: "3"
```

This belongs under the relevant service. Applying changed container logging settings generally requires recreating that container; plan for the service interruption and verify durable data first. Setting a daemon default does not retroactively update existing containers.

Do not manually delete files from Docker's managed storage directory. For logging-driver behaviour and configuration, consult Docker's [JSON file logging documentation](https://docs.docker.com/engine/logging/drivers/json-file/).

## Remove reviewed resources in small steps

Start with an explicitly reviewed image or stopped container rather than a system-wide prune. Keep the active release and any image needed for rollback available.

If a builder's cache is the confirmed problem, inspect the relevant builder before pruning. Buildx may use different builders and storage from the daemon's default build path. Match the cleanup command to the builder that owns the cache.

For the default builder, one possible bounded cleanup is:

```bash
# Removes eligible build cache unused for at least seven days.
# Review the interactive prompt before confirming.
docker builder prune --filter 'until=168h'
```

This is a removal command, not a preview. Cache cleanup can make subsequent builds slower or require network downloads.

Docker's [pruning guide](https://docs.docker.com/engine/manage-resources/pruning/) explains the categories targeted by the different commands. Inspect the installed CLI help too, especially on older hosts.

After each change, check the affected filesystem again. If space barely moves, revisit the diagnosis instead of progressively widening deletion flags.

## Treat volumes as application data

Inspect ownership and container references before considering a volume removable:

```bash
docker volume ls
docker volume inspect VOLUME
docker ps -a --filter volume=VOLUME
```

For a currently detached volume, ask whether its Compose project was temporarily removed, whether it belongs to a migration, or whether it contains the previous database state.

Modern `docker volume prune` defaults to unused anonymous volumes; `--all` broadens this to unused named volumes. Neither definition means the content is safe to lose. See the [volume prune reference](https://docs.docker.com/reference/cli/docker/volume/prune/).

Avoid broad volume pruning as a routine emergency response. If a particular volume is confirmed obsolete, document its owner, backup status, and removal decision, then remove that specific object in the agreed window.

## Check for deleted files that remain open

A process can retain disk blocks after a file is unlinked. If filesystem usage and visible files disagree, investigate open deleted files with an available tool such as:

```bash
sudo lsof +L1
```

Review the output locally because filenames and process details may be sensitive. Coordinate any process restart with its owner; killing a process simply to release space can interrupt service or unfinished work.

## Prevent the next emergency

Set separate retention policies for logs, build caches, releases, and durable data. Alert on available bytes and inodes for the filesystems that actually hold them, with enough lead time to act.

When a host is critically full, temporary capacity expansion may be safer than rushed deletion. Once stable, move unintended application writes into managed persistent paths and test recovery on another machine.

The [Compose production checklist]({{< ref "docker-compose-production" >}}) covers those deployment decisions. A useful cleanup leaves both more free space and a clear explanation of what was removed.
