# Sections

This file defines all sections, their ordering, impact levels, and descriptions.
The section ID (in parentheses) is the filename prefix used to group rules.

---

## 1. Driver & Config (config)

**Impact:** CRITICAL
**Description:** The choices made in `config/queue.php` shape every downstream concern. Wrong driver at scale (database when redis is needed) becomes the bottleneck; wrong `after_commit` setting causes jobs to fail mysteriously when they reference uncommitted rows; missing failed-jobs storage means failures are silently lost.

## 2. Job Design (design)

**Impact:** CRITICAL
**Description:** The shape of the job class itself — does it implement `ShouldQueue`, does it pass IDs or models, is it idempotent, is the constructor pure? Get this wrong and the job either doesn't queue at all, fails on retry, or duplicates side effects under load.

## 3. Retry & Failure (retry)

**Impact:** HIGH
**Description:** Every job will eventually fail. Configuring `$tries` and `$backoff`, implementing `failed()`, distinguishing transient from permanent errors, and avoiding the "burn all 5 attempts on a hung HTTP call" trap (`#[FailOnTimeout]`) decide whether failures become noise or pages.

## 4. Scaling & Workers (scaling)

**Impact:** HIGH
**Description:** Worker process management — Supervisor or systemd config, queue priority lanes, recycling workers to combat PHP memory leaks. The same job code can be fast or impossibly slow depending on how the workers are tuned around it.

## 5. Batching & Chaining (bus)

**Impact:** HIGH
**Description:** When you need to dispatch many related jobs — choosing `Bus::batch` (parallel, progress tracking, then/catch/finally) vs `Bus::chain` (strict sequential, abort on failure), how to handle partial failures with `allowFailures()`, and how to chunk thousands of items without overwhelming the queue.

## 6. Testing & Operations (ops)

**Impact:** MEDIUM
**Description:** Asserting queue behaviour in tests (`Queue::fake`, `Bus::fake`), wiring queueable jobs into the scheduler with overlap prevention, and the recurring question of when Horizon's complexity earns its keep over plain `queue:work`.
