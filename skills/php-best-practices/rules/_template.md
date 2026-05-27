---
title: Rule Title Here
impact: MEDIUM
impactDescription: Optional description of impact
tags: tag1, tag2, tag3
---

# Rule Title Here

## Why it matters

One short paragraph on the real engineering cost of getting this wrong. Be specific about what breaks, what hides, or what becomes unanalyzable when this rule is ignored.

## Rule

A blunt, testable recommendation in one or two sentences.

## Bad

```php
<?php

declare(strict_types=1);

// Minimal bad example — show the anti-pattern, not an essay
```

## Better

```php
<?php

declare(strict_types=1);

// Improved — addresses the worst problem but may still have gaps
```

## Best

```php
<?php

declare(strict_types=1);

// Preferred modern approach — idiomatic, analyzable, copy-pasteable
```

## Exceptions / trade-offs

Where the rule does not apply, with concrete limits. Keep this honest — not every rule applies everywhere.

## Static-analysis notes

What PHPStan / Psalm / IDEs can verify automatically with this pattern in place.

## Version notes

Exact PHP version floor if the feature is version-specific. Use format: `PHP 8.x+`

## Related topics

- [related-rule.md](related-rule.md) — one-line description of the relationship
