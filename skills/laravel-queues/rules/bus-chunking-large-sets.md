---
title: Chunking Large Datasets — Don't Dispatch One Job Per Row
impact: HIGH
impactDescription: "Dispatching 100k jobs at once floods Redis, overwhelms workers, and starves SLA traffic"
tags: bus, chunking, scale, batch
---

## Chunking Large Datasets — Don't Dispatch One Job Per Row

**Impact: HIGH (Dispatching 100k jobs at once floods Redis, overwhelms workers, and starves SLA traffic)**

For "process 100k orders" or "send emails to all users", the naive approach is one job per record. With 100k jobs:

- Dispatch itself blocks for seconds (network round-trips to Redis)
- The queue fills with 100k entries; other jobs wait their turn
- Each job has overhead (~10-50ms even for trivial work) — totalling 16-80 minutes regardless of worker count
- Failed-jobs table can grow by tens of thousands of rows if a downstream service hiccups

**Chunk it.** Process N items per job. Dispatch fewer, larger jobs. Get a 10×-100× efficiency gain.

## Incorrect

```php
// ❌ One job per record at scale
$users = User::all();   // 100,000 rows loaded into memory
foreach ($users as $user) {
    SendMarketingEmail::dispatch($user->id);
}
// 100,000 dispatches; takes ~30s just to enqueue; queue worker overhead dominates
```

## Correct — chunk inside a Bus::batch

```php
// ✅ Chunk to 500 rows per job; ~200 batch jobs instead of 100k

use App\Jobs\SendMarketingEmailChunk;
use Illuminate\Support\Facades\Bus;

$batch = Bus::batch([])
    ->name('Marketing send — Q3 newsletter')
    ->allowFailures()
    ->then(function ($batch) use ($campaign) {
        $campaign->markComplete();
    })
    ->dispatch();

User::where('subscribed', true)
    ->select('id')
    ->chunkById(500, function ($chunkOfUsers) use ($batch) {
        $batch->add(new SendMarketingEmailChunk(
            userIds: $chunkOfUsers->pluck('id')->all()
        ));
    });
```

```php
// The chunk job processes its slice
class SendMarketingEmailChunk implements ShouldQueue
{
    use Dispatchable, Queueable;

    /** @param int[] $userIds */
    public function __construct(public readonly array $userIds) {}

    public function handle(): void
    {
        User::whereIn('id', $this->userIds)
            ->select(['id', 'email', 'name'])
            ->chunkById(100, function ($users) {
                foreach ($users as $user) {
                    Mail::to($user)->send(new MarketingEmail($user));
                }
            });
    }
}
```

**Wins:**
- 200 batch jobs instead of 100,000 dispatches
- Each batch job processes 500 users in <30 seconds
- Total throughput: same; queue overhead: drastically reduced
- Batch progress tracking shows "180 / 200 chunks done" instead of "84,000 / 100,000"

## Chunk size — picking N

| Job characteristic | Chunk size |
|---|---|
| Lightweight (in-memory transformation) | 1000–5000 |
| External API per item (Stripe, etc.) | 50–200 |
| Email sends | 100–500 |
| DB writes per item | 200–1000 |
| Complex business logic per item | 25–100 |

Aim for **each chunk job running 10s–60s.** Shorter and queue overhead dominates; longer and you lose recoverability (a single failed item burns the whole chunk).

## chunkById vs cursor()

```php
// ✅ chunkById — most efficient for "iterate all rows"
User::query()->select('id')->chunkById(500, fn ($chunk) => /* ... */);

// ✅ cursor() — for streaming inside a chunk (one record at a time, low memory)
foreach (User::cursor() as $user) { /* ... */ }   // 100k records, ~10MB memory

// ❌ All() — loads everything into memory
foreach (User::all() as $user) { /* ... */ }      // 100k records, ~1.5GB memory
```

Pattern: use `chunkById` at the **dispatch** site to build the batch; use `chunkById` again inside the chunk job for memory efficiency processing those 500.

## Idempotency in chunk jobs

A chunk job may run twice (the queue's at-least-once guarantee). Make the chunk's effect idempotent:

```php
public function handle(): void
{
    User::whereIn('id', $this->userIds)
        ->whereDoesntHave('emailLogs', function ($q) {
            $q->where('campaign_id', $this->campaignId);
        })
        ->each(function ($user) {
            Mail::to($user)->send(new MarketingEmail($user));
            $user->emailLogs()->create([
                'campaign_id' => $this->campaignId,
                'sent_at' => now(),
            ]);
        });
}
```

The `whereDoesntHave` filter ensures the email is sent at most once per user per campaign, even if the chunk job retries.

## Don't ship Eloquent collections in the constructor

```php
// ❌ Passes a Collection — entire object serialised
new SendMarketingEmailChunk($users);

// ✅ Pass IDs — serialised as int[]
new SendMarketingEmailChunk(userIds: $users->pluck('id')->all());
```

See [`design-pass-ids-not-models`](design-pass-ids-not-models.md).

## Detection

```bash
# Suspicious patterns — foreach + dispatch
grep -rEnB1 -A2 'foreach\s*\(.*\$\w+\s+as\s+\$\w+\)' --include='*.php' app/ | \
  grep -B1 'dispatch\('
# Manual review: anything iterating > 100 records and dispatching one job each
```

Reference: [Laravel 13 — Queues: Adding Jobs to Batches](https://laravel.com/docs/13.x/queues#adding-jobs-to-batches) · [Laravel 13 — Eloquent: Chunking Results](https://laravel.com/docs/13.x/eloquent#chunking-results)
