---
title: "When a Linux Service Keeps Restarting: Debugging systemd Restart Loops"
date: 2026-11-15
tags: ["linux", "systemd", "troubleshooting", "operations"]
description: "Trace a systemd restart loop from the first failure, inspect effective unit settings, and add bounded retries without hiding a broken application."
draft: false
---

A service starts, fails, and starts again. A few seconds later the status changes to “start request repeated too quickly.” Increasing the retry limit may make the message disappear temporarily, but it does not explain why the application exits.

The useful investigation starts with the first failure and separates three things: what the application did, how systemd interpreted it, and what the restart policy asked systemd to do next.

{{< figure src="featured.svg" alt="A circular restart path passes through start, exit, and retry before a rate-limit stop breaks the loop." >}}

## Capture evidence before another restart

Use the actual unit name in place of `example-app.service`:

```bash
systemctl status example-app.service --no-pager -l
journalctl -u example-app.service --since '30 minutes ago' \
  --no-pager -o short-iso

systemctl show example-app.service \
  -p Result -p ExecMainCode -p ExecMainStatus -p NRestarts
```

Keep the relevant output before resetting state or changing configuration. Look earlier than the final rate-limit message: the first attempt may contain the useful error, while later attempts only repeat its consequences.

Logs can include credentials and request data, so inspect and redact them before sharing. Their availability also depends on permissions and retention. The [journalctl guide]({{< ref "journalctl" >}}) has additional filtering examples.

If repeated attempts are actively causing harm, coordinate a deliberate stop with the service owner. Stopping a production service changes availability; it should be an incident decision, not a reflexive diagnostic step.

## Read the effective unit

Inspect the service and its drop-ins:

```bash
systemctl cat example-app.service
systemctl show example-app.service \
  -p User -p Group -p WorkingDirectory -p ExecStart \
  -p Restart -p RestartUSec \
  -p StartLimitIntervalUSec -p StartLimitBurst
```

Local overrides can change the packaged unit. `systemctl cat` shows backing files, while `systemctl show` reports loaded properties. If files were edited without a daemon reload, the two views may not yet agree.

Check the executable path, account, working directory, environment-file references, and writable paths. A command that works in an administrator's interactive shell can fail under a service account with a different environment and filesystem access.

Avoid collecting environment values into a broad support bundle unless needed; they may contain secrets. Document how the service receives credentials without unnecessarily copying their contents.

## Match the failure to a hypothesis

A few common failure patterns narrow the next check:

| Evidence | Next question |
| --- | --- |
| Configuration parser error | Did the deployed file pass the application's checker? |
| Permission denied | Can the configured service account access the exact path? |
| Address already in use | Which process owns the intended listener? |
| Missing library or executable | Does the installed release match its runtime dependencies? |
| Database connection failure | Are endpoint, credentials, network, and application retries correct? |
| Process killed unexpectedly | Do kernel logs or resource controls explain the termination? |

Use evidence to choose the next command. An exit status is a clue interpreted in the context of the application; it rarely proves the whole diagnosis on its own.

Do not run a second production instance manually just to reproduce the error. It may bind the same port, consume the same queue, or execute startup migrations. Reproduce safely in a test environment, or use a supported validation mode that does not start the workload.

## Confirm the process model

A service definition must match how the application runs. A traditional daemon that backgrounds itself needs different supervision from a foreground process. A readiness-notifying service must actually send the notification expected by its unit type.

An incorrect process model can produce timeouts or apparent exits even when part of the application remains running. Prefer the application's documented foreground invocation for a simple service, and verify how child processes are managed.

For a conventional long-running foreground process, a policy fragment could be:

```ini
[Unit]
StartLimitIntervalSec=300
StartLimitBurst=5

[Service]
Restart=on-failure
RestartSec=15s
```

This illustrates bounded retries, not a complete service definition. It allows a limited number of start attempts within the configured interval and spaces automatic retries. Pick values based on startup cost, dependency recovery, and the service's availability target.

`Restart=on-failure` covers recognised failure cases, not every form of application unhealthiness. A hung process may remain alive without triggering this policy. A deliberate `systemctl stop` does not cause an automatic restart. See the [service reference](https://www.freedesktop.org/software/systemd/man/257/systemd.service.html).

## Understand the rate limit

The start limit protects the system from rapid repeated starts. Manual starts count too. Once the limit is reached, systemd does not automatically schedule a new attempt merely because the interval later expires.

`systemctl reset-failed` clears the failed state and start-rate counters for the selected unit. It does not repair a bad configuration or restore missing credentials. The [unit reference](https://www.freedesktop.org/software/systemd/man/257/systemd.unit.html) describes the limit and reset behaviour.

Dependency ordering also needs care. Starting after a network-related target does not establish that a remote database is accepting authenticated connections. Use bounded application retry behaviour for transient dependency outages and observe when those retries are exhausted.

## Recover deliberately and verify usefulness

After correcting the actual cause, validate any edited unit files and application configuration. Reload systemd if the unit definition changed. During the agreed recovery action, clear the rate limit if necessary and start the service:

```bash
sudo systemctl daemon-reload
sudo systemctl reset-failed example-app.service
sudo systemctl start example-app.service
```

These commands change service state. Use them after the cause is addressed and with the intended unit selected.

Inspect logs and perform a real application check. Then watch for long enough to cover the previous failure pattern: a service that used to fail after ten minutes has not demonstrated stability after thirty seconds.

Add an alert for persistent unavailability and unexpected restart activity. Pair it with a user-facing check, because a process that stays alive can still serve errors. The [small-fleet monitoring article]({{< ref "monitoring-small-linux-fleet" >}}) explains that distinction.

Keep the incident's original error, root cause, effective configuration change, and validation result. A good restart policy buys recovery time from transient failures; a useful investigation explains why this failure happened at all.
