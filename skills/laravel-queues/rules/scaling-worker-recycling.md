---
title: Worker Recycling — --max-jobs and --max-time
impact: MEDIUM
impactDescription: "Long-running PHP processes leak memory; recycling combats drift before it bites"
tags: scaling, workers, memory-leaks
---

## Worker Recycling — --max-jobs and --max-time

**Impact: MEDIUM (Long-running PHP processes leak memory; recycling combats drift before it bites)**

PHP wasn't designed for forever-running processes. Workers accumulate memory over time — Eloquent's IdentityMap, container-bound singletons, Carbon objects in static caches, third-party library state. After a few thousand jobs or a few hours, a worker can be using 4× the RAM it started with.

Laravel's queue worker has two flags that solve this **by gracefully exiting and letting Supervisor restart the process**:

- **`--max-jobs=N`** — exit after processing N jobs
- **`--max-time=N`** — exit after N seconds

The worker finishes its current job, exits, and Supervisor immediately restarts a fresh process. Memory resets to baseline.

## The default (without these flags)

```bash
# ❌ Worker runs forever — memory grows
php artisan queue:work redis
```

Symptoms:
- `htop` shows worker processes at 800MB-1.5GB after a day
- OOM-killed by the kernel on memory-constrained servers
- Worker performance degrades — late-life job latency 2-3× higher than fresh-process latency
- Occasional `Allowed memory size exhausted` in failed jobs

## Correct

```bash
# ✅ Recycle every hour (most common)
php artisan queue:work redis --sleep=3 --tries=3 --max-time=3600

# ✅ Or recycle every 1000 jobs (when traffic is bursty)
php artisan queue:work redis --sleep=3 --tries=3 --max-jobs=1000

# ✅ Both (whichever hits first)
php artisan queue:work redis --sleep=3 --tries=3 --max-time=3600 --max-jobs=1000
```

When the limit hits, the worker exits cleanly between jobs (it doesn't interrupt the current one). Supervisor then restarts it.

## Why `--memory` isn't enough

Laravel also has `--memory=128`, which kills the worker if memory usage exceeds 128MB. This works but:

- It triggers AFTER the leak has grown to the limit (degraded performance period)
- It can cut a job mid-execution if the job itself spikes memory
- It depends on accurate memory reporting (sometimes off on container platforms)

`--max-time` and `--max-jobs` are **preventive**; `--memory` is **emergency brake**. Use the preventive flags by default; use `--memory` as a belt-and-braces upper bound.

```bash
# All three
php artisan queue:work redis --sleep=3 --tries=3 --max-time=3600 --max-jobs=1000 --memory=256
```

## Picking the right values

| Worker type | `--max-time` | `--max-jobs` |
|---|---|---|
| Default | 3600 (1h) | 1000 |
| Memory-heavy jobs (PDF, video) | 600 (10min) | 50 |
| Lightweight jobs (notifications, cache warmers) | 7200 (2h) | 5000 |
| Burst processing (data import) | 1800 (30min) | 500 |

These are starting points. Watch your worker memory over time and adjust.

## What happens during recycle

1. Worker finishes its current job (or exits during the idle sleep if no job)
2. Worker process exits cleanly (zero exit code)
3. Supervisor (or systemd) immediately starts a new worker
4. Cold-start cost: ~200-500ms while the new worker boots Laravel
5. Net effect: a few hundred ms of "no available worker" per recycle, far less than the memory-leak cost

For an 8-worker pool with `--max-time=3600`, you get roughly one cold-start every 7.5 minutes across the fleet — negligible.

## On Horizon

Horizon manages worker lifecycle differently — workers exit when Horizon decides via the configured `nice` and `tries` settings. The equivalent recycling happens via Horizon's auto-balancer and restart on deploy. You typically don't set `--max-time` on Horizon-managed workers; let Horizon handle it.

## Detection

```bash
# Workers running without recycling flags
grep -nH 'queue:work' /etc/supervisor/conf.d/*.conf 2>/dev/null \
  | grep -v -- '--max-time\|--max-jobs'

# Current worker memory usage (sanity check)
ps aux | grep 'queue:work' | awk '{ printf "%s\t%.1f MB\n", $11, $6/1024 }'
```

If any active worker is using > 500MB, recycling is overdue.

Reference: [Laravel 13 — Queues: Processing a Specified Number of Jobs](https://laravel.com/docs/13.x/queues#the-queue-work-command) · [Laravel 13 — Queues: Resource Considerations](https://laravel.com/docs/13.x/queues#queue-workers-and-deployment)
