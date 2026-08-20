# Chapter 13: Scheduling Scripts

A script that isn't scheduled has to be run by a human, which defeats a lot of the point of
automation. This chapter covers the three standard ways to schedule shell scripts on Linux.

## 1. `cron` — the classic scheduler

Every user (and the system) can have a **crontab** — a list of scheduled jobs.

```bash
crontab -e          # edit your own crontab
crontab -l           # list your current crontab
sudo crontab -u postgres -e   # edit another user's crontab (needs privilege)
```

### Cron syntax

```
* * * * * command_to_run
│ │ │ │ │
│ │ │ │ └── day of week (0–7, 0 and 7 = Sunday)
│ │ │ └──── month (1–12)
│ │ └────── day of month (1–31)
│ └──────── hour (0–23)
└────────── minute (0–59)
```

### Examples

```cron
# Run every day at 2:00 AM
0 2 * * * /usr/local/bin/backup.sh

# Run every 15 minutes
*/15 * * * * /usr/local/bin/healthcheck.sh

# Run every weekday (Mon–Fri) at 6 PM
0 18 * * 1-5 /usr/local/bin/report.sh

# Run on the 1st of every month at midnight
0 0 1 * * /usr/local/bin/monthly_cleanup.sh

# Run every 6 hours
0 */6 * * * /usr/local/bin/sync.sh
```

### Best practices for cron jobs

```cron
# Always redirect output to a log — cron emails errors by default, which is easy to miss
30 2 * * * /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1

# Set PATH explicitly — cron runs with a minimal environment, not your interactive shell's
PATH=/usr/local/bin:/usr/bin:/bin
30 2 * * * backup.sh >> /var/log/backup.log 2>&1
```

A script that works perfectly when run by hand but silently fails under cron is almost
always an environment problem (missing `$PATH`, missing environment variables like `$HOME`
or `$PGPASSWORD`) — always test with `env -i` to simulate cron's minimal environment:
```bash
env -i /usr/local/bin/backup.sh
```

## 2. `at` — run a script once, in the future

```bash
echo "/usr/local/bin/one_time_task.sh" | at 23:00
at now + 30 minutes -f /usr/local/bin/reminder.sh
atq                 # list pending 'at' jobs
atrm 3               # cancel job number 3
```
Useful for one-off scheduled maintenance ("run this specific script tonight at 11 PM") when
you don't want a permanent cron entry.

## 3. systemd timers — the modern alternative to cron

A systemd timer pairs a `.timer` unit (the schedule) with a `.service` unit (what to run).
More powerful than cron: better logging (via `journalctl`), dependency management, and
the ability to catch up on missed runs.

**`/etc/systemd/system/pg-backup.service`:**
```ini
[Unit]
Description=PostgreSQL backup job

[Service]
Type=oneshot
User=postgres
ExecStart=/usr/local/bin/pg_backup.sh
```

**`/etc/systemd/system/pg-backup.timer`:**
```ini
[Unit]
Description=Run pg-backup daily at 2 AM

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now pg-backup.timer
systemctl list-timers                       # see all scheduled timers and next run time
journalctl -u pg-backup.service              # view the job's logs
```

## 4. Choosing between cron and systemd timers

| Need | Use |
|---|---|
| Simple, quick schedule, works everywhere | `cron` |
| Need to guarantee a missed run catches up after reboot | systemd timer (`Persistent=true`) |
| Want centralized logging via `journalctl` | systemd timer |
| One-off future run | `at` |
| Managing many interdependent scheduled jobs | systemd timers (dependency support) |

## 5. Verifying a scheduled job actually ran

```bash
# Cron: check the log you redirected output to
tail -50 /var/log/backup.log

# systemd: check the unit's status and recent logs
systemctl status pg-backup.service
journalctl -u pg-backup.service --since "1 hour ago"
```

### Key takeaways

- `cron` is simple and universal; always redirect output and set `PATH` explicitly.
- `at` is for one-off future runs, not recurring schedules.
- systemd timers are the modern choice when you want better logging, missed-run catch-up, or
  dependency ordering between jobs.
- Always verify a scheduled job actually ran and succeeded — don't assume silence means
  success.

## Hands-On Exercises — PostgreSQL DBA Bootcamp Style

1. Schedule your `pg_conf_lookup.sh` from Chapter 3 to run every morning at 8 AM and log its
   `work_mem` check to a file:
   ```cron
   0 8 * * * postgres /home/postgres/pg_conf_lookup.sh work_mem >> /var/log/pg_conf_check.log 2>&1
   ```
   Add it with `crontab -e` as the `postgres` user (or `sudo crontab -u postgres -e`).

2. Test the cron-environment problem directly: run your script with `env -i` and see if it
   still finds `psql`/`grep` correctly, or whether you need to hardcode full paths.

3. Convert exercise 1 into a systemd timer (`pg-conf-check.service` +
   `pg-conf-check.timer`), then verify it with `systemctl list-timers` and
   `journalctl -u pg-conf-check.service`.

**Next:** [Chapter 14 — Best Practices & Advanced Techniques](14-best-practices-and-advanced-techniques.md)
**Previous:** [Chapter 12 — Working with Files](12-working-with-files.md)
