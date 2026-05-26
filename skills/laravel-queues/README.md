# Laravel Queues & Jobs

Production-grade queue patterns for **Laravel 13 (MySQL + Redis)**. Covers driver choice, job design, retry/failure, worker scaling, batching/chaining, and testing. 20 rules across 6 categories.

**Version:** 1.0.0

## Overview

- Driver choice: when `database` is enough, when to move to `redis`
- Job design: `ShouldQueue`, IDs-not-models, idempotency, constructor vs `handle()`
- Retry & failure: `#[Backoff]`, `#[FailOnTimeout]`, `failed()`, transient vs permanent
- Scaling: Supervisor config, multi-queue priority lanes, worker recycling
- Batching & chaining: `Bus::batch` vs `Bus::chain`, failure handling, chunking
- Testing & ops: `Queue::fake()`, `Bus::fake()`, scheduling, when to adopt Horizon

## Categories

### 1. Driver & Config (CRITICAL)
Driver choice (database/redis/sqs), `after_commit`, failed-jobs storage.

### 2. Job Design (CRITICAL)
`ShouldQueue`, pass IDs not models, idempotency on payment/external-API jobs, constructor vs handle.

### 3. Retry & Failure (HIGH)
Tries + backoff (incl. `#[Backoff]` attribute), `failed()` method, transient vs permanent, `#[FailOnTimeout]`.

### 4. Scaling & Workers (HIGH)
Supervisor config, multi-queue priority, worker recycling to avoid memory leaks.

### 5. Batching & Chaining (HIGH)
`Bus::batch` vs `Bus::chain`, failure handling with `allowFailures()`, chunking large sets.

### 6. Testing & Operations (MEDIUM)
`Queue::fake()`, scheduled jobs with `withoutOverlapping`, when Horizon is warranted.

## Usage

```
Audit our queue setup
Review this job class
Should this be queued or run sync?
Why is this job retrying forever?
Set up Supervisor for queue workers
When should we adopt Horizon?
```

## References

- [Laravel 13 — Queues](https://laravel.com/docs/13.x/queues)
- [Laravel 13 — Horizon](https://laravel.com/docs/13.x/horizon)
- [Laravel 13 — Task Scheduling](https://laravel.com/docs/13.x/scheduling)
- [Supervisor — A Process Control System](http://supervisord.org/)
