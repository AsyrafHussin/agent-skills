---
title: Observability and Monitoring Gaps
impact: HIGH
impactDescription: "Debt you can't measure can't be paid down; incidents take hours longer to diagnose"
tags: observability, monitoring, alerts, slo
---

## Observability and Monitoring Gaps

**Impact: HIGH (Debt you can't measure can't be paid down; incidents take hours longer to diagnose)**

A system without structured logs, metrics, traces, and alerts is a system that breaks silently. The "we'll add monitoring later" decision is a tax paid during every incident, every customer-reported bug, and every capacity-planning conversation. Observability debt has the unique property of being most expensive to fix *during* an incident.

## How to Detect

For each service in scope, audit:

1. **Structured logs** — JSON, with correlation IDs, request IDs, user IDs (when appropriate)
2. **Application metrics** — request rates, latency percentiles (p50/p95/p99), error rates, queue depths
3. **Tracing** — distributed traces with span IDs spanning service boundaries
4. **Alerts** — paging only on user-facing symptoms; non-paging for early signals
5. **SLOs** — explicit service-level objectives with error budgets

```bash
# Find log statements still using non-structured output
grep -rEn 'echo |print(|print_r(|var_dump(|console\.log\(' app/ src/ | wc -l

# Laravel: is logging configured to JSON channel?
grep -A5 "'channels' =>" config/logging.php

# Are there any Prometheus / OpenTelemetry imports/integrations?
grep -rEn 'prometheus|opentelemetry|datadog|sentry|new-relic' composer.json package.json

# Sentry / Bugsnag / equivalent error tracking installed?
grep -rE 'sentry|bugsnag|rollbar|honeybadger' composer.lock package-lock.json
```

## Incorrect

```php
// ❌ Observability black hole

// 1. Print-style logging — unstructured, no correlation
public function pay(Order $order) {
    echo "Processing payment for order " . $order->id . "\n";
    try {
        $result = $this->stripe->charge($order);
    } catch (Exception $e) {
        echo "FAILED: " . $e->getMessage() . "\n";          // lost on next request
    }
}

// 2. No request IDs
// 3. No latency metrics — "the app feels slow" is the only signal
// 4. Pager rule: "send a Slack message if a single 500 occurs"
//    → wakes people up for every transient hiccup; signal-to-noise = 0
// 5. No SLOs → no shared definition of "the service is broken"
```

**Problems:**
- During an incident, you can't tell which user, which request, or which span failed
- Capacity planning is guesswork ("we think we can handle 2× traffic")
- Slow regressions (p95 creeping from 200ms → 800ms over a quarter) go unnoticed
- On-call burnout from low-quality alerts

## Correct

```php
// ✅ Structured logging with correlation ID + context
public function pay(Order $order): PaymentResult {
    $log = Log::withContext([
        'order_id'    => $order->id,
        'user_id'     => $order->user_id,
        'request_id'  => request()->header('X-Request-ID') ?? Str::uuid(),
    ]);

    $log->info('payment.start', ['amount' => $order->total]);
    $start = microtime(true);

    try {
        $result = $this->stripe->charge($order);
        $log->info('payment.success', [
            'charge_id' => $result->id,
            'duration_ms' => (int) ((microtime(true) - $start) * 1000),
        ]);
        return $result;
    } catch (\Throwable $e) {
        $log->error('payment.failure', [
            'exception' => $e::class,
            'message'   => $e->getMessage(),
            'duration_ms' => (int) ((microtime(true) - $start) * 1000),
        ]);
        report($e);                                          // → Sentry / Bugsnag
        throw $e;
    }
}
```

Minimum viable observability stack to install:
- **Logs:** JSON channel + central log store (CloudWatch, Loki, Datadog Logs)
- **Errors:** Sentry / Bugsnag / Rollbar
- **Metrics + tracing:** OpenTelemetry SDK → vendor or self-hosted (Grafana stack, Datadog, Honeycomb)
- **Uptime:** external prober (UptimeRobot, Pingdom, Datadog Synthetics)
- **Alerts:** routed to a real pager system (PagerDuty, Opsgenie); thresholds based on SLO burn rate, not single events

Define SLOs explicitly (illustrative — real Sloth uses `version: prometheus/v1` plus an
`sli.events` block with `error_query`/`total_query`; Pyrra ships as a Kubernetes CRD —
consult each tool's schema before adopting):

```yaml
# slo.yaml — illustrative shape only
service: checkout
slos:
  - name: availability
    objective: 99.9%
    sli: error_rate < 1% over 28d
  - name: latency
    objective: 99% of requests < 500ms over 28d
```

**Benefits:**
- Incidents resolve faster (mean MTTR drops 50%+ with traces)
- Slow regressions are caught when they're small
- Alerts wake people up only for user-facing problems
- Engineering and product share a quantitative definition of "the service is healthy"

## Remediation Strategy

- **Effort:**
  - **S** — add Sentry + JSON logging to one service
  - **M** — add OpenTelemetry + APM dashboards
  - **L** — full SLO program (objectives, alerts on burn rate, error budgets, blameless postmortems)
- **When to pay down:**
  - **NOW:** any production service without an error-tracker (Sentry-class)
  - **NOW:** any service whose only outage signal is "a user complained"
  - **This quarter:** structured logging + request IDs + p95 latency dashboards
  - **Then:** SLO program; alerts based on burn rates, not raw thresholds

**Anti-patterns:**
- **Alert on everything** — alarms drown signal; only page on customer-impacting symptoms
- **Logs-only observability** — logs are expensive to query at scale; metrics + traces complement them
- **No retention policy** — logs at 1TB/day with infinite retention becomes its own debt
- **Tool sprawl** — one logs vendor, one APM, one error tracker is plenty

Reference: [Google SRE — Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/) · [OpenTelemetry](https://opentelemetry.io/) · [Sloth — SLO Generator](https://sloth.dev/)
