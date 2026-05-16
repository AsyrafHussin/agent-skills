---
title: Supervisor Config — Don't Kill Mid-Job on Deploy
impact: HIGH
impactDescription: "Wrong stopwaitsecs cuts jobs mid-execution on every deploy — corrupted state, duplicate charges"
tags: scaling, supervisor, workers, deployment
---

## Supervisor Config — Don't Kill Mid-Job on Deploy

**Impact: HIGH (Wrong stopwaitsecs cuts jobs mid-execution on every deploy — corrupted state, duplicate charges)**

Laravel queue workers run forever. They need a process manager to start them, restart them if they crash, and stop them gracefully on deploy. **Supervisor** is the standard (also valid: systemd, Procfile on Heroku, Forge's UI which wraps Supervisor).

The most common production bug: **`stopwaitsecs` set too low.** When Supervisor sends SIGTERM on `php artisan queue:restart`, the worker finishes its current job before exiting. If Supervisor kills the worker (SIGKILL) before the job finishes, you get partial state — half-charged customer, half-imported CSV, exception logged but row not updated.

## Anatomy of a correct Supervisor config

```ini
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/app/artisan queue:work redis --queue=high,default,low --sleep=3 --tries=3 --max-time=3600 --backoff=3
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=forge
numprocs=8
redirect_stderr=true
stdout_logfile=/var/www/app/storage/logs/worker.log
stopwaitsecs=3600
```

**What each directive does:**

| Directive | Meaning | Critical to get right? |
|---|---|---|
| `command` | The worker invocation | **Yes** — see flags below |
| `numprocs` | Number of worker processes (parallel job slots) | Yes — undersized = backed-up queues |
| `autorestart=true` | If a worker crashes, restart it | Yes |
| `stopasgroup=true` + `killasgroup=true` | Stop all child processes too | Yes |
| `stopwaitsecs` | How long to wait for graceful shutdown before SIGKILL | **Critical** — must exceed your longest job's `$timeout` |
| `redirect_stderr` + `stdout_logfile` | Capture worker output | Useful for debugging |

## The `stopwaitsecs` trap

A job has `$timeout = 30`. The worker is running it. Deploy triggers `supervisorctl restart laravel-worker`. Supervisor sends SIGTERM:

- **`stopwaitsecs = 10`** ❌ — Supervisor waits 10s, then SIGKILL. Job is mid-charge. Customer charged, your DB not updated. Manual reconciliation tomorrow.
- **`stopwaitsecs = 3600`** ✅ — Supervisor waits up to an hour. Worker finishes its current 30s job, exits cleanly. Deploy proceeds.

**Rule:** `stopwaitsecs > max(jobs' $timeout values)` with healthy margin. **Default to 3600 (1 hour)** unless you have a specific reason otherwise. Idle workers exit immediately on SIGTERM; only busy ones wait.

## Worker command flags

```bash
php artisan queue:work redis \
    --queue=high,default,low \      # priority lanes (see scaling-multi-queue-priority)
    --sleep=3 \                     # how long to sleep when no jobs available
    --tries=3 \                     # default $tries unless job overrides
    --max-time=3600 \               # restart worker after 1h (combat memory leaks)
    --backoff=3                     # default backoff unless job overrides
```

The two most important flags for production:

- **`--max-time=3600`** — recycles the worker process every hour. PHP isn't great at long-running processes; recycling combats memory drift.
- **`--queue=high,default,low`** — pull from `high` first, then `default`, then `low`. Without this you have no priority lanes.

## Systemd alternative (modern servers)

If you don't want Supervisor, systemd works:

```ini
# /etc/systemd/system/laravel-worker@.service
[Unit]
Description=Laravel queue worker %i
After=redis.service mysql.service

[Service]
User=forge
Group=forge
Restart=always
RestartSec=5
ExecStart=/usr/bin/php /var/www/app/artisan queue:work redis --queue=high,default,low --sleep=3 --tries=3 --max-time=3600 --backoff=3
TimeoutStopSec=3600
KillMode=mixed

[Install]
WantedBy=multi-user.target
```

```bash
# Enable 8 workers
systemctl enable laravel-worker@{1..8}.service
systemctl start laravel-worker@{1..8}.service
```

Equivalent semantically; pick based on team familiarity.

## Avoiding the "jobs not running" mystery

Symptoms: dispatched jobs appear in `jobs` table / Redis but never run. Diagnosis order:

1. **Is the worker running?** `supervisorctl status laravel-worker:*`
2. **Is it pulling the right queue?** Check `--queue=` flag matches your `onQueue(...)` dispatches
3. **Is it pulling the right connection?** Check first arg to `queue:work` (e.g., `redis` vs `database`)
4. **Did you forget `queue:restart` after a deploy?** Workers boot the old code until restarted
5. **Did the worker crash?** Check `worker.log` and Supervisor's `autostartretries`

## Detection

```bash
# Find supervisor config files
find /etc/supervisor /etc/supervisord.d ~/supervisor 2>/dev/null -name '*laravel*' -o -name '*queue*'

# Validate stopwaitsecs is reasonable
grep -nH 'stopwaitsecs' /etc/supervisor/conf.d/*.conf 2>/dev/null \
  | awk -F= '{ if ($2 < 60) print "TOO LOW: " $0 }'

# On Forge: the worker config is in the panel under "Daemons"; verify Time Out is high enough.
```

Reference: [Laravel 13 — Queues: Supervisor Configuration](https://laravel.com/docs/13.x/queues#supervisor-configuration) · [Supervisor docs](http://supervisord.org/)
