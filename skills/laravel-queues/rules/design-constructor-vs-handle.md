---
title: Constructor vs handle() — Keep the Constructor Pure
impact: HIGH
impactDescription: "Constructor runs sync at dispatch; side effects there run inside the HTTP request"
tags: design, constructor, serialization, side-effects
---

## Constructor vs handle() — Keep the Constructor Pure

**Impact: HIGH (Constructor runs sync at dispatch; side effects there run inside the HTTP request)**

A common confusion: which code in a job class runs sync, and which runs on the worker?

| Method | Runs when | Runs where |
|---|---|---|
| **`__construct()`** | At `dispatch()` time | In the dispatching process (HTTP request) |
| `handle()` | When worker pops the job | In the queue worker process |
| `failed(Throwable $e)` | After all `$tries` exhausted | In the queue worker process |

Anything in `__construct()` happens **synchronously, inside the request that dispatches**. DB writes, HTTP calls, file I/O, slow computation — they all add to the request latency, defeating the point of queuing.

The constructor's job: **store primitive args for `handle()` to use.** Nothing else.

## Incorrect — DB writes in constructor

```php
// ❌ Constructor does work — adds latency to every dispatcher

class GenerateInvoicePdf implements ShouldQueue
{
    use Dispatchable, Queueable;

    public readonly string $pdfUrl;

    public function __construct(public readonly int $invoiceId)
    {
        // ⚠️ This runs in the HTTP request, not the worker.
        $invoice = Invoice::findOrFail($invoiceId);
        $renderer = app(InvoiceRenderer::class);
        $this->pdfUrl = $renderer->render($invoice);   // ← 1.2 seconds!

        // ⚠️ Database write in the constructor — caught by transactions, etc.
        $invoice->update(['pdf_generated_at' => now()]);
    }

    public function handle(): void
    {
        // The actual "queued" work is now a one-liner; the heavy lifting already happened.
        Mail::to($invoice->user)->send(new InvoiceReady($this->pdfUrl));
    }
}
```

**Why it breaks:**
- The dispatching controller now waits 1.2 seconds for the PDF render
- If the dispatcher is in a DB transaction, the constructor's `update()` is too — race conditions and rollback weirdness
- "Why is checkout slow?" — answered 3 days later: the constructor of an "async" job

## Correct — constructor stores args, handle() does the work

```php
// ✅ Constructor: pure assignment. handle(): the actual work.

class GenerateInvoicePdf implements ShouldQueue
{
    use Dispatchable, Queueable;

    public function __construct(public readonly int $invoiceId) {}

    public function handle(InvoiceRenderer $renderer): void
    {
        $invoice = Invoice::findOrFail($this->invoiceId);
        $pdfUrl = $renderer->render($invoice);
        $invoice->update(['pdf_generated_at' => now(), 'pdf_url' => $pdfUrl]);
        Mail::to($invoice->user)->send(new InvoiceReady($pdfUrl));
    }
}
```

**Why it works:**
- Dispatching takes microseconds (just enqueues the job ID)
- All the heavy work runs on the worker
- Dependencies (`InvoiceRenderer`) injected via `handle()` method signature — Laravel resolves from container at run time, not at dispatch

## What's OK in the constructor

- Assigning constructor arguments to properties (`public readonly` promotion in PHP 8+)
- Simple computed values from constructor arguments (e.g., calculating a string label from passed values)
- Setting `$this->onQueue('high')` for routing (rare; usually do at dispatch site instead)

What's NOT OK:
- `DB::*` queries
- `Http::*` calls
- `Mail::*`, `Storage::*`, anything that touches I/O
- `findOrFail` / Eloquent reads (defer to `handle()`)
- Heavy computation

## Dependency injection — `handle()` only

```php
// ✅ Inject services into handle(), not the constructor
public function handle(StripeGateway $stripe, OrderRepository $orders): void
{
    $order = $orders->findOrFail($this->orderId);
    $stripe->charge($order);
}
```

Laravel resolves `handle()` dependencies via the container at run time, on the worker. Injecting into the constructor doesn't work cleanly anyway — constructor args become part of the serialised payload, and you can't serialise a `StripeGateway` instance.

## Detection

```bash
# Find jobs whose constructor does more than property assignment
# (heuristic: constructor body > 3 lines is suspect)
for f in app/Jobs/*.php; do
  LINES=$(awk '/public function __construct/,/^[[:space:]]*\}/' "$f" | wc -l)
  if [ "$LINES" -gt 5 ]; then
    echo "SUSPECT (long constructor): $f"
  fi
done

# Find DB / HTTP / Mail calls inside __construct
grep -rEnB1 'public function __construct' --include='*.php' app/Jobs/ | \
  while read line; do
    f=$(echo "$line" | cut -d: -f1)
    awk '/public function __construct\(/,/^\s*\}/' "$f" | \
      grep -qE '(DB::|Http::|Mail::|Storage::|->find|->save|->update|->create)' && echo "SIDE EFFECT IN CTOR: $f"
  done
```

Reference: [Laravel 13 — Queues: Class Structure](https://laravel.com/docs/13.x/queues#class-structure) · [Laravel 13 — Queues: Dependency Injection](https://laravel.com/docs/13.x/queues#dependency-injection)
