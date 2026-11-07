---
title: "Your Backups Are Green. Can You Actually Restore?"
date: 2026-11-07
tags: ["linux", "backup", "disaster-recovery", "sysadmin", "operations"]
description: "A practical restore drill for a small Linux application: recover files, validate the database, test the application, and measure how long recovery really takes."
draft: false
---

A backup dashboard full of green checkmarks is reassuring. But imagine the server disappears tomorrow morning. Can someone bring the application back on a clean machine, with the credentials and documentation that are still available?

That is a different question from whether last night's backup job finished.

A useful restore test follows the whole path: retrieve the backup, recover the data, start the application, and check that someone can use it. Here is a manageable way to run that test for a small Linux application.

{{< figure src="featured.svg" alt="Recovery drill with four checkpoints: retrieve the backup, restore data, validate the application, and measure recovery time." >}}

## Start with a business question

Before choosing commands, decide what a successful recovery means.

- **How much data can we lose?** This is the recovery point objective, or RPO.
- **How long can the service be unavailable?** This is the recovery time objective, or RTO.
- **What must work when we declare recovery complete?** Name actual user actions.

For an example internal application, the targets might be no more than four hours of lost changes and service restored within two hours. Those are example targets, not a recommendation for every system.

A nightly backup alone cannot guarantee a four-hour RPO. And a two-hour RTO needs to include provisioning, finding credentials, downloading data, restoring it, validating the service, and making it available to users. A fast file copy only measures one part.

## Pick one application and a realistic failure

Start with one service rather than the entire infrastructure. Assume its original machine is gone.

For a typical application, make an inventory of:

| Component | What you need to recover |
| --- | --- |
| Application | Known version, package or container image, startup configuration |
| Database | Application-consistent backup and compatible restore tools |
| Uploaded files | Documents, attachments, images, and their permissions |
| Configuration | Service definitions, environment settings, proxy configuration |
| Access | Backup credentials, encryption keys, secret recovery procedure |
| Dependencies | DNS, certificates, identity provider, external services |

If the backup contains the database but not the uploaded documents, you may recover a working login screen and a broken application.

Also check where recovery instructions live. A runbook stored only on the failed server will not help much.

## Build an isolated recovery environment

Use a disposable VM or a separate test environment with enough disk space. Keep it disconnected from production integrations until you have deliberately configured test replacements.

Recovered applications can immediately start sending email, processing queues, calling payment APIs, or running scheduled jobs. Disable those paths before the first application start. Use restricted access for restored customer data and remove it after the drill according to your retention rules.

Give the test its own database endpoint and hostname. Check the effective configuration, not just the name of the environment file.

Start the timer when recovery work begins. Record provisioning and access delays as well as restore duration.

## Restore a specific backup

Choose a known backup timestamp and record its identifier. Avoid an ambiguous “latest” when a repository contains several hosts or applications.

With restic, for example, inspect the available snapshots and select the appropriate one:

```bash
restic snapshots
restic ls SNAPSHOT_ID
```

Replace `SNAPSHOT_ID` with the selected ID. Configure repository access and the repository password through your established secret handling process.

On the isolated Linux recovery host, create a fresh target directory:

```bash
restore_dir=$(mktemp -d /var/tmp/restore-drill.XXXXXX)
restic restore SNAPSHOT_ID --target "$restore_dir"
printf 'Restored files: %s\n' "$restore_dir"
```

Run these commands in the same shell. The restore recreates backed-up paths beneath that target: `/srv/app/uploads` would appear under `$restore_dir/srv/app/uploads`. Recovering ownership and other metadata may require appropriate privileges and filesystem support; check the result explicitly.

A repository integrity check is another useful layer:

```bash
restic check --read-data
```

This reads and checks the stored data and can take substantial time and bandwidth. Schedule it accordingly. It does not start your application or prove that every required file was included in the backup. See the [restic repository documentation](https://github.com/restic/restic/blob/master/doc/045_working_with_repos.rst) and [restore documentation](https://github.com/restic/restic/blob/master/doc/050_restore.rst) for details.

## Validate the database as a database

Copying files out of a backup is not enough for a running database. Use a supported logical backup or a correctly coordinated physical backup procedure. An arbitrary copy of live database files may be inconsistent.

For example, a PostgreSQL custom-format dump created with `pg_dump -Fc` can be restored with `pg_restore`. On a disposable PostgreSQL instance, with the connection settings explicitly pointed at that test instance:

```bash
# Confirm the destination before creating or restoring anything.
psql -d postgres -c '\conninfo'

createdb restore_drill
pg_restore --exit-on-error --no-owner --no-privileges \
  --dbname=restore_drill /path/to/recovered/app.dump
```

The database name must be unused. This example assumes a trusted dump, a compatible PostgreSQL version, and any required extensions already available. Omitting ownership and privileges simplifies this test, but means it does **not** validate production roles and access rights. Test those separately before treating the exercise as a full recovery rehearsal.

PostgreSQL documents the supported archive formats and restore options in the [pg_restore reference](https://www.postgresql.org/docs/17/app-pgrestore.html).

After the import, check application-specific facts: recent records, expected date ranges, relationships, and whether referenced attachments exist. A successful import proves less than a successful user workflow.

Database and file backups also need a compatible recovery point. A document record restored from one time and an uploads directory restored from another can leave missing attachments even when both backups are individually readable.

## Test a real workflow

Start the recovered application with its test configuration and inspect its logs. Then perform a short acceptance test:

1. Sign in with a designated test account.
2. Open an existing record with an attachment.
3. Download the attachment and check its contents.
4. Create and read back a test record, if the isolated environment permits it.
5. Restart the application and confirm the data persists.

A health endpoint is useful, but it may only prove that a process answers HTTP requests. It might never touch the database, file storage, or authentication path your users depend on.

If external dependencies or traffic cutover are excluded from the drill, record that limitation. Do not report a partial exercise as proof of the full RTO.

## Keep a short recovery record

The output should be evidence another person can use, not just “restore worked.”

```text
Service:
Failure scenario:
Backup ID and timestamp:
Recovery started / finished:
Recovered data timestamp:
Target RPO / RTO:
Measured data loss / recovery duration:
Acceptance checks and results:
Excluded dependencies or cutover steps:
Missing credentials, files, or instructions:
Follow-up owner and due date:
```

Calculate the age of the recovered data relative to the simulated incident, not relative to when you finish the drill. Note if the application data is older than the backup job timestamp.

## Make the next drill easier

For a small environment, a reasonable starting routine is a monthly restore of selected files and a quarterly application recovery drill. Adjust the frequency to the service's importance and rate of change. Repeat after significant changes to storage, authentication, backup tooling, or deployment architecture.

Automate repeatable checks, but occasionally have someone else follow the runbook from a clean machine. That reveals undocumented assumptions about access and setup.

If you already use [ZFS snapshots]({{< ref "zfs-snapshot-automation" >}}), include them in the recovery design. A snapshot on the same pool is useful for rollback, but it will not survive loss of that pool. Keep recoverable copies outside the failure domain you are testing.

For the first drill, choose one important application, one backup, and one isolated machine. Write down what must work, recover it, and measure the result. The gaps you discover become a concrete list of improvements before an outage forces the same exercise under pressure.
