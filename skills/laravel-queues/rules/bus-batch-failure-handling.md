---
title: Batch Failure Handling — allowFailures() and Callbacks
impact: HIGH
impactDescription: "Default batch behaviour cancels remaining jobs on first failure; allowFailures() keeps them running"
tags: bus, batching, failure-handling
---

## Batch Failure Handling — allowFailures() and Callbacks

**Impact: HIGH (Default batch behaviour cancels remaining jobs on first failure; allowFailures() keeps them running)**

`Bus::batch` has an opinion about failures: **by default, a single job failure cancels the rest of the batch.** Any unstarted jobs in the batch are silently dropped; in-flight jobs complete but don't trigger further dispatches.

For a "process these 1000 CSV rows" batch where one row failing shouldn't stop the other 999, that default is wrong. Use `allowFailures()`.

For a "create these 3 resources" batch where if any fails, the others are pointless, the default IS right.

## Default: fail-fast

```php
// ✅ Fail-fast: if one job fails, cancel the rest

Bus::batch([
    new CreateOrderRecord($data),
    new ReserveInventory($data),
    new ChargePayment($data),
])->catch(function (Batch $batch, Throwable $e) {
    // Triggered on the FIRST failure; remaining jobs cancelled
    OrderFlow::rollback($data);
})->dispatch();
```

Right call here: you don't want `ChargePayment` to run if `ReserveInventory` failed.

## allowFailures(): keep going

```php
// ✅ Resilient batch: one row failing doesn't stop the import

$batch = Bus::batch(
    $rows->map(fn ($row) => new ImportCsvRow($row))
)->name('CSV import')
 ->allowFailures()                            // ← critical
 ->then(function (Batch $batch) {
    // All processed (some may have failed)
    $report = "Imported {$batch->processedJobs()} / {$batch->totalJobs}; failed: {$batch->failedJobs}";
    Mail::to($user)->send(new CsvImportReport($report));
 })
 ->dispatch();
```

`allowFailures()` semantics:
- Failed jobs go to `failed_jobs` (same as a normal job failure)
- The batch's `then` callback fires when all jobs have either succeeded OR failed
- The batch's `catch` callback NEVER fires (since failures are allowed)
- `$batch->failedJobs` and `$batch->processedJobs()` show the split

## Querying batch state

For UI / monitoring:

```php
$batch = Bus::findBatch($batchId);

$batch->pendingJobs;         // not yet processed
$batch->totalJobs;           // total count
$batch->processedJobs();     // pendingJobs done (success + fail)
$batch->failedJobs;          // count of failures
$batch->progress();          // 0-100 percentage
$batch->finished();          // bool: all jobs processed
$batch->cancelled();         // bool: cancel() was called
```

Use this to render a progress page:

```php
// routes/web.php
Route::get('/batches/{id}/progress', function (string $id) {
    $batch = Bus::findBatch($id);
    abort_unless($batch, 404);
    return [
        'total' => $batch->totalJobs,
        'done' => $batch->processedJobs(),
        'failed' => $batch->failedJobs,
        'progress' => $batch->progress(),
        'finished' => $batch->finished(),
    ];
});
```

## Cancelling a batch

```php
// Cancel any remaining queued jobs (in-flight jobs still complete)
Bus::findBatch($batchId)->cancel();

// In a job, check before starting work
public function handle(): void
{
    if ($this->batch()->cancelled()) {
        return;
    }
    // do the work
}
```

Useful pattern: a user clicks "cancel import" on the progress page, you call `cancel()`, jobs see the cancelled state and exit early.

## Incorrect — common mistakes

```php
// ❌ Using catch but not realising it doesn't fire if allowFailures()
Bus::batch($jobs)
    ->allowFailures()
    ->catch(function (Batch $batch, Throwable $e) {
        // NEVER FIRES because allowFailures() is set
        // (failed jobs go to failed_jobs, but the batch carries on)
    })
    ->dispatch();
```

```php
// ❌ Forgetting allowFailures on a "best effort" batch
Bus::batch($csvJobs)->dispatch();   // first failure → remaining cancelled
// Customer reported one row failed, 5000 others never imported.
```

```php
// ❌ Trying to "retry the batch" by re-dispatching all jobs
// Bus::batch doesn't have built-in retry. If you want retry semantics,
// either:
//   - Set $tries on the individual jobs (each retries independently)
//   - Filter for failed jobs after the batch and dispatch a NEW batch with just those
```

## When the batch's `then` should run

`then` callback fires when:
- All jobs succeeded (no failures); OR
- `allowFailures()` is set AND all jobs have completed (some passed, some failed)

`catch` fires when:
- A job fails AND `allowFailures()` is NOT set (then cancels remaining)

`finally` always fires after `then` or `catch`.

## Detection

```bash
# Batches without explicit failure mode choice — manual review
grep -rEnA10 'Bus::batch\(' --include='*.php' app/ | grep -B2 -A5 '->dispatch()'

# Look for batches that DON'T have a callback (then/catch/finally) — fire-and-forget batches
# are usually fine, but worth checking they're intentional
```

Reference: [Laravel 13 — Queues: Allowing Failures](https://laravel.com/docs/13.x/queues#batch-failures) · [Laravel 13 — Queues: Inspecting Batches](https://laravel.com/docs/13.x/queues#inspecting-batches)
