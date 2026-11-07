---
title: "A Docker Compose App Is Running. Is It Production-Ready?"
date: 2026-10-23
tags: ["docker", "linux", "operations", "deployment", "devops"]
description: "Take a single-host Docker Compose application beyond its first successful start: persistent data, controlled updates, health checks, secrets, and recovery."
draft: false
---

`docker compose up -d` finishes without errors. The website loads. The application is running.

Before handing it over to users, there are a few more questions to answer: what survives a container replacement, how do updates work, and who notices when the service stops working?

For a small application, Compose on one Linux host can be a practical deployment choice. The operational work around it determines how dependable that choice becomes.

{{< figure src="featured.svg" alt="Four operational foundations for a Docker Compose deployment: persistent data, controlled releases, monitoring, and recovery." >}}

## Define the failure you can tolerate

A single host is a single failure domain. Restart policies do not help if the VM, its storage, or the network disappears.

That may be acceptable for an internal tool with a recovery window of several hours. It may be unacceptable for a service with a strict availability requirement. Write down the expected downtime and data loss before deciding whether the architecture fits.

Compose is a deployment tool. Availability also depends on the host, storage, dependencies, and the people operating them.

## Make persistent data explicit

List every path the application writes to. Classify each as disposable, configuration, or durable business data.

A named volume keeps data across normal container replacement, but it is not a backup. It also needs deliberate handling during host migration. A bind mount makes the host path visible, but introduces ownership and permissions you must manage.

For each durable path, record:

- Where it lives on the host or in Docker-managed storage.
- Which process owns it and which permissions it needs.
- How it is backed up consistently.
- How it is restored to another machine.

Database files need a database-aware backup procedure. Copying a live volume without coordinating with the database can produce an unusable recovery point.

Avoid `docker compose down -v` in routine deployment instructions: the volume removal is an entirely different operation from replacing containers.

## Deploy a known release

Use a versioned image reference rather than a moving `latest` tag. If reproducibility matters, record or pin the image digest as well; tags can be reassigned.

Build images in a repeatable build process and keep the resulting artifacts accessible. Rebuilding an old Git commit months later may resolve different dependencies.

For an example application image supplied by your own registry, the service configuration could include:

```yaml
services:
  app:
    image: ${APP_IMAGE:?Set APP_IMAGE to the tested release}
    restart: unless-stopped
    ports:
      - "127.0.0.1:8080:8080"
    volumes:
      - app_uploads:/var/lib/app/uploads
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"

volumes:
  app_uploads:
```

This is a configuration fragment, not a complete application stack. It assumes the image listens on port 8080 and writes uploads to the shown path. The loopback binding is intended for a reverse proxy on the same host. A proxy in another container needs a shared Docker network instead.

Log rotation limits disk consumption for this container's logs. Choose the retention according to your troubleshooting needs, and collect logs elsewhere if they must survive host loss.

## Check readiness and failure separately

A container can be running while its application is stuck. A health check adds a useful signal, provided its command exists in the image and tests something meaningful.

For an image that includes `curl` and provides a readiness endpoint:

```yaml
healthcheck:
  test: ["CMD", "curl", "--fail", "--silent", "--show-error", "http://127.0.0.1:8080/ready"]
  interval: 30s
  timeout: 5s
  retries: 3
  start_period: 30s
```

Place this under the application service. Adapt the endpoint and timings to the actual application.

A health check marks a container healthy or unhealthy. Standalone Compose does not automatically restart a container merely because it becomes unhealthy. Restart policies respond to container exits, subject to their policy conditions.

Startup ordering is another concern: `depends_on` with `condition: service_healthy` can wait for a dependency's health check during startup. It does not replace retry logic when that dependency fails later. See Docker's [startup order documentation](https://docs.docker.com/compose/how-tos/startup-order) and [service reference](https://docs.docker.com/reference/compose-file/services/).

Monitor the service from outside the host too. An internal readiness check cannot tell you whether public DNS or the external TLS path is broken.

## Keep secrets out of the repository

Do not commit production passwords into Compose files or bake them into images. Environment files also need protection; calling a file `.env` does not make it secret.

Compose supports file-based secrets mounted into explicitly selected services. The application must support reading those files. Conventions such as `PASSWORD_FILE` are image-specific, not something Compose automatically translates for every application.

Protect the source files on the host, define who can retrieve replacement credentials, and test the recovery process. File-based Compose secrets do not provide encrypted host storage by themselves. Docker describes the mechanism in its [Compose secrets guide](https://docs.docker.com/compose/how-tos/use-secrets/).

## Treat updates as a procedure

A small deployment runbook can be short:

1. Review release notes and database migration requirements.
2. Confirm a recent, tested backup exists.
3. Set the intended image reference and validate configuration.
4. Pull the release and apply the deployment during the agreed window.
5. Check logs, health, and one real user workflow.
6. Record the deployed reference and outcome.

Typical commands, run from the deployment directory with its intended environment, are:

```bash
# Quiet validation avoids printing the resolved configuration and its values.
docker compose config --quiet
docker compose pull
docker compose up -d
docker compose ps
docker compose logs --tail=100 app
```

Review log output before sharing it; applications sometimes log sensitive values. Container replacement can interrupt requests. These commands do not promise a zero-downtime rollout.

Keep the previous image reference, but do not assume changing the image back reverses a database migration. Decide whether recovery means an application rollback, a forward fix, or a database restore before making a risky schema change.

## Prove you can replace the host

The final test is to deploy on a clean machine and restore the required data. Can someone find the configuration, access the registry, recover secrets, and bring the service back without files from the original host?

Use the [restore drill]({{< ref "backup-restore-testing" >}}) to measure that process. If the result fits the application's recovery targets, a single-host Compose deployment can be an understandable and maintainable solution. If it does not, you have specific evidence for changing the architecture.
