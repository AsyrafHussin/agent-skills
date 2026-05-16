---
title: Bus::batch vs Bus::chain — Pick the Right One
impact: HIGH
impactDescription: "Wrong choice means parallel work runs serial (slow) or sequential work runs in parallel (race conditions)"
tags: bus, batching, chaining
---

## Bus::batch vs Bus::chain — Pick the Right One

**Impact: HIGH (Wrong choice means parallel work runs serial (slow) or sequential work runs in parallel (race conditions))**

When dispatching multiple related jobs, Laravel offers two APIs that look similar but behave very differently:

- **`Bus::chain([...])`** — strictly sequential. Each job runs only after the previous one succeeds. If any job fails, the rest are abandoned.
- **`Bus::batch([...])`** — parallel by default. All jobs queue immediately; workers pick them up concurrently. Tracks aggregate progress; offers `then`, `catch`, `finally` callbacks.

Picking the wrong one creates real bugs.

## Decision tree

| Scenario | Use |
|---|---|
| Step B depends on step A's side effects | `Bus::chain` (sequential, abort on failure) |
| Process 1000 items, order doesn't matter | `Bus::batch` (parallel) |
| Want a "done when all are done" callback | `Bus::batch` |
| Want a progress bar / status page | `Bus::batch` |
| Workflow with branching ("do A, then B and C in parallel, then D") | `Bus::chain` containing `Bus::batch` |
| One job's failure should NOT stop the others | `Bus::batch->allowFailures()` |
| One job's failure SHOULD stop everything after | `Bus::chain` |

## Bus::chain — strict sequence

```php
// ✅ Each step depends on the previous
use App\Jobs\OptimizePodcast;
use App\Jobs\ProcessPodcast;
use App\Jobs\ReleasePodcast;
use Illuminate\Support\Facades\Bus;

Bus::chain([
    new ProcessPodcast(podcastId: 42),     // step 1
    new OptimizePodcast(podcastId: 42),    // step 2 — runs only if step 1 succeeded
    new ReleasePodcast(podcastId: 42),     // step 3 — runs only if step 2 succeeded
])->dispatch();
```

If `ProcessPodcast` fails (all tries exhausted), `OptimizePodcast` and `ReleasePodcast` are never dispatched. The chain stops dead.

You can add `catch` to handle chain failure:

```php
Bus::chain([
    new ProcessPodcast(42),
    new OptimizePodcast(42),
    new ReleasePodcast(42),
])->catch(function (Throwable $e) {
    // Chain failed; clean up partial state
    Podcast::find(42)?->markProcessingFailed($e->getMessage());
})->dispatch();
```

## Bus::batch — parallel + aggregate tracking

```php
// ✅ Process 500 CSV rows in parallel, with progress callbacks
use App\Jobs\ImportCsvRow;
use Illuminate\Bus\Batch;
use Illuminate\Support\Facades\Bus;
use Throwable;

$batch = Bus::batch(
    Csv::rows($file)->map(fn ($row) => new ImportCsvRow($row))
)->name('CSV import — ' . $file->name)
 ->before(function (Batch $batch) {
    // Batch has been created, jobs not yet added
 })
 ->progress(function (Batch $batch) {
    // Called after each job completes
    broadcast(new BatchProgressUpdated($batch));
 })
 ->then(function (Batch $batch) {
    // All jobs succeeded
    Notification::send($user, new CsvImportComplete($batch->id));
 })
 ->catch(function (Batch $batch, Throwable $e) {
    // First job failure
    Log::error('CSV batch failed', ['batch_id' => $batch->id, 'error' => $e->getMessage()]);
 })
 ->finally(function (Batch $batch) {
    // Always called — success or failure
    Storage::delete($file->path);
 })
 ->onQueue('imports')
 ->dispatch();

return ['batch_id' => $batch->id];   // give the user a way to poll progress
```

By default, **a single failure cancels the rest of the batch.** Use `allowFailures()` to keep going (see next rule).

## Mixing them — chain that contains batches

```php
// ✅ Step 1 (single), then step 2 (parallel batch), then step 3 (single)
Bus::chain([
    new FlushPodcastCache(42),
    Bus::batch([
        new ReleasePodcast(42, region: 'us-east'),
        new ReleasePodcast(42, region: 'eu-west'),
        new ReleasePodcast(42, region: 'ap-south'),
    ]),
    new NotifyPodcastReleased(42),
])->dispatch();
```

This runs:
1. `FlushPodcastCache` first
2. Then three `ReleasePodcast` jobs in parallel (batch waits for all to finish)
3. Then `NotifyPodcastReleased` after the batch completes

## Incorrect

```php
// ❌ Using chain when jobs are independent — slow

Bus::chain([
    new SendOrderConfirmation(1),   // independent
    new SendOrderConfirmation(2),   // independent
    new SendOrderConfirmation(3),   // independent
    // ... 100 more
])->dispatch();
// Result: 100 emails sent one at a time. The 100th waits ~5 minutes.
// Should have been Bus::batch — parallel.
```

```php
// ❌ Using batch when there's a dependency — race condition

Bus::batch([
    new CreateInvoice($orderId),   // creates invoice row
    new EmailInvoiceLink($orderId), // needs the invoice row to exist
])->dispatch();
// Race: EmailInvoiceLink may run before CreateInvoice finishes. Email links to a row
// that doesn't exist yet. Should have been Bus::chain.
```

## Detection

```bash
# Find Bus::chain with many jobs — likely should be Bus::batch
grep -rEnA20 'Bus::chain\(' --include='*.php' app/ \
  | awk '/Bus::chain/,/\)\->/{print}' | grep -c 'new '

# Find Bus::batch where dependency between jobs is implied by name
# (manual review — look at the job names in batch arrays)
grep -rEnA5 'Bus::batch\(' --include='*.php' app/ | head -30
```

Reference: [Laravel 13 — Queues: Job Chaining](https://laravel.com/docs/13.x/queues#job-chaining) · [Laravel 13 — Queues: Job Batching](https://laravel.com/docs/13.x/queues#job-batching)
