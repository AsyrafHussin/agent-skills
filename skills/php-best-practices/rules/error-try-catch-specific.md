---
title: Catch the Narrowest Exception You Can Recover From
impact: HIGH
impactDescription: Robust error handling and exception design
tags: php, error-handling, exceptions
---

# Catch the Narrowest Exception You Can Recover From

## Why it matters
Catching `\Exception` or `\Throwable` below the top-level boundary silently absorbs `TypeError`, `Error`, out-of-memory conditions, and programmer mistakes — turning them into false successes or wrong responses. Debugging a system that swallows errors is expensive; discovering the swallow six months later is worse.

## Rule
Catch the narrowest exception type you can actually handle at that call site. Reserve broad catches for top-level error boundaries only, and always log or rethrow there.

## Bad
```php
<?php

declare(strict_types=1);

// Swallows TypeErrors, OOM, programmer mistakes — all treated the same
try {
    $user = $repository->find($id);
    $mailer->sendWelcome($user);
} catch (\Throwable $e) {
    return null;
}
```

## Better
```php
<?php

declare(strict_types=1);

// Better: catches a named exception, but still too wide if multiple
// distinct failures need different responses
try {
    $user = $repository->find($id);
    $mailer->sendWelcome($user);
} catch (\Exception $e) {
    $logger->error('Operation failed', ['error' => $e->getMessage()]);
    return null;
}
```

## Best
```php
<?php

declare(strict_types=1);

// Catch each recoverable exception separately with its own response
try {
    $user = $repository->find($id);
} catch (UserNotFoundException $e) {
    $logger->warning('User not found', ['id' => $id]);
    return null;
}

try {
    $mailer->sendWelcome($user);
} catch (MailerException $e) {
    // Email failure is non-critical — log and continue
    $logger->error('Welcome email failed', [
        'user_id' => $id,
        'error'   => $e->getMessage(),
    ]);
}

// Multi-catch (PHP 8.0+) for genuinely equivalent failures
try {
    $data = $api->fetch($endpoint);
} catch (ConnectionException | TimeoutException $e) {
    $logger->error('API unreachable', ['error' => $e->getMessage()]);
    throw new ServiceUnavailableException('External service down', previous: $e);
}

// Broad catch is correct only at the top-level boundary
try {
    $response = $kernel->handle($request);
} catch (\Throwable $e) {
    $logger->critical('Unhandled exception', [
        'class'   => $e::class,
        'message' => $e->getMessage(),
        'trace'   => $e->getTraceAsString(),
    ]);
    $response = new Response('Internal Server Error', 500);
}
```

## Exceptions / trade-offs
A global error handler, framework middleware, or CLI command runner is the one legitimate place for `catch (\Throwable $e)`. Everywhere else, catching broadly is a code smell.

## Static-analysis notes
PHPStan rule `catchWithoutVariable` and Psalm's `forbiddenFunctions` config can be extended to flag `catch (\Exception` or `catch (\Throwable` outside designated boundary classes.

## Related topics
- [error-custom-exceptions.md](error-custom-exceptions.md) — when to create a catchable exception type
- [error-exception-hierarchy.md](error-exception-hierarchy.md) — structuring exception families for layered catching
- [error-never-suppress.md](error-never-suppress.md) — the @ operator is the silent version of the same problem
