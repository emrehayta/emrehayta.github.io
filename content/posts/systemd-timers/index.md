---
title: "From Cron to systemd Timers: Making Scheduled Jobs Easier to Operate"
date: 2026-09-18
tags: ["linux", "systemd", "automation", "operations"]
description: "Build a systemd timer with a separate service, bounded execution, journal logs, and an explicit policy for missed runs."
draft: false
---

A scheduled script is easy to forget until someone asks why yesterday's report never arrived. At that point, you need to know whether the job started, what it printed, and how it exited.

On a systemd-based Linux server, a timer and a service can make those questions easier to answer. The timer handles scheduling. The service describes the work and exposes its status and logs.

Cron remains useful for simple tasks and environments without systemd. Moving a job should solve an operational problem, rather than merely replace one syntax with another.

{{< figure src="featured.svg" alt="A paper-style schedule with three daily execution windows and one missed run caught after the server starts." >}}

## Start with the job's contract

Before writing unit files, decide:

- Which account runs the command?
- What files, credentials, and network services does it need?
- What counts as success?
- How long may it run?
- Should a missed execution happen after the machine comes back?

For this example, assume a root-owned executable at `/usr/local/bin/daily-report`, an existing service account named `reportjob`, and a writable output directory at `/var/lib/daily-report` owned by that account. The executable runs in the foreground and returns a nonzero exit status on failure.

The script must validate its own output. A timer cannot tell whether a report contains the expected records.

## Describe execution in a service

Create `/etc/systemd/system/daily-report.service`:

```ini
[Unit]
Description=Generate the daily operations report

[Service]
Type=oneshot
User=reportjob
Group=reportjob
WorkingDirectory=/var/lib/daily-report
ExecStart=/usr/local/bin/daily-report
TimeoutStartSec=20min
UMask=0077
NoNewPrivileges=true
PrivateTmp=true
```

`Type=oneshot` lets systemd track the foreground command until it finishes. For this service type, `TimeoutStartSec` bounds the execution time; choose a limit that allows normal work to finish while detecting a stuck job.

The isolation settings are a starting point, not a complete sandbox. In particular, `PrivateTmp` changes the temporary directory view. Test scripts that exchange files with other processes before enabling it.

Use absolute executable paths and configure required settings explicitly. `ExecStart` does not run arbitrary shell syntax by default. Put pipelines and conditional shell logic in a maintained script rather than assuming the unit is a crontab.

## Add the schedule

Create `/etc/systemd/system/daily-report.timer`:

```ini
[Unit]
Description=Schedule the daily operations report

[Timer]
OnCalendar=*-*-* 02:00:00 UTC
RandomizedDelaySec=10min
Persistent=true
Unit=daily-report.service

[Install]
WantedBy=timers.target
```

The timer uses an explicit timezone. It aims for a daily run around 02:00 UTC, with up to ten minutes of random delay and additional scheduling tolerance. This spreads load if the same job runs across a fleet. It is unsuitable when downstream consumers require an exact start time.

`Persistent=true` allows a missed calendar activation to trigger after the timer becomes active again. It does not replay every missed day or generate historical reports automatically. The script needs its own date-range logic if backfilling matters.

A timer also does not start another instance of the same service while that service is already active. That helps prevent timer-triggered overlap, but does not lock out someone executing the script directly. These behaviours are documented in the upstream [systemd timer reference](https://www.freedesktop.org/software/systemd/man/257/systemd.timer.html).

## Validate before enabling

Check the unit files and calendar expression on the target host:

```bash
sudo systemd-analyze verify /etc/systemd/system/daily-report.service /etc/systemd/system/daily-report.timer
systemd-analyze calendar '*-*-* 02:00:00 UTC'
sudo systemctl daemon-reload
```

Then run the service deliberately, after confirming its effects are safe:

```bash
sudo systemctl start daily-report.service
systemctl status daily-report.service --no-pager
journalctl -u daily-report.service --since '30 minutes ago' --no-pager
```

A completed oneshot service normally returns to an inactive state. That is expected. Inspect the command's result and output instead of treating `inactive` alone as failure.

The [systemd service reference](https://www.freedesktop.org/software/systemd/man/latest/systemd.service.html) explains oneshot execution and timeout options. Check the installed manual for version-specific support.

## Switch schedules without running both

Once the manual test passes, remove or disable the old cron entry in the agreed change window. Then enable the timer:

```bash
sudo systemctl enable --now daily-report.timer
systemctl list-timers --all daily-report.timer
```

With persistence enabled, starting the timer may initiate catch-up work. Account for that during migration.

Observe the first scheduled execution. Test a controlled failure in a non-production environment too: does the command fail visibly, and does the responsible person receive a notification?

## Monitor completion, not just scheduling

An enabled timer proves a schedule exists. It does not prove the report succeeded or reached its recipient.

Record the time of successful completion only after the required output has been validated. Alert when that timestamp becomes too old for the schedule. Keep journal retention long enough to investigate failures, and add explicit alert delivery; a failed unit does not notify a person by itself.

For the broader monitoring setup, see [monitoring a small Linux fleet]({{< ref "monitoring-small-linux-fleet" >}}).

The useful outcome is a job with a clear owner, inspectable execution, and a known response to missed or failed runs. The timer syntax is only one part of that.
