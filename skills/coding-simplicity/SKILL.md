---
name: coding-simplicity
description: Implementation-time simplicity for coding, fixing, and refactoring. Use when changing code and the goal is the smallest correct solution without sacrificing safety or verification. Do not use for non-coding requests or as an always-on writing/persona mode.
license: MIT
metadata:
  author: Agent Skills Team
  version: "1.0.0"
---

# Coding Simplicity

Build the smallest **correct** change after understanding the real flow. Short code is an outcome, not the objective.

## Before choosing the solution

- Read the task and the code path it actually touches.
- Trace callers when changing shared behavior or fixing a bug.
- Identify explicit requirements, trust boundaries, data-loss risks, and repository validation before optimizing for size.

## Search order

Stop at the first option that fully satisfies the verified requirement:

1. no code change;
2. reuse the existing repository owner/pattern;
3. language standard library;
4. native platform, database, shell, browser, or protocol capability;
5. already-installed dependency;
6. one root-cause fix at the shared owner rather than repeated symptom patches;
7. minimum new code.

A one-liner is good only when it is the clearest correct form left by this search. Never prefer one line over required guards, readable control flow, or correct edge-case behavior.

## Do not manufacture surface area

- No speculative abstraction, interface, factory, config, compatibility layer, dependency, or cleanup.
- Prefer deletion and reuse over moving complexity into a helper or manager.
- Touch the fewest files that correctly own the behavior.
- Do not broaden a requested fix into adjacent modernization.

## Safety floor

Never simplify away explicit requirements, trust-boundary validation, authorization/security controls, error handling that prevents data loss, required transaction/concurrency guarantees, accessibility, or necessary compatibility.

If the smallest-looking patch sits in the wrong layer or leaves sibling callers broken, it is not the simpler solution.

## Verification floor

- Non-trivial changed logic leaves the smallest runnable proof already appropriate for the repository.
- Trivial changes do not justify inventing a test framework or ceremony.
- Run the repository's existing relevant gates and report only observed results.

## Deliberate simplifications

When a real corner is intentionally cut, record the known ceiling and an observable revisit trigger in task/workflow evidence. Do not leave tool-branded product-code comments unless the repository explicitly wants them.

## Boundary

This skill governs implementation choices only. It does not impose a persona, terse prose, persistent intensity mode, or session-wide behavior on unrelated work.

## Provenance

This skill adapts the implementation-order, root-cause, safety-floor, and verification ideas from DietrichGebert/ponytail while intentionally dropping its persona, intensity modes, output-style rules, and session persistence. Ponytail is MIT licensed.
