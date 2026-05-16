# Sections

This file defines all sections, their ordering, impact levels, and descriptions.
The section ID (in parentheses) is the filename prefix used to group rules.

---

## 1. Comments (comments)

**Impact:** CRITICAL
**Description:** The single loudest AI tell. Models love to narrate the next line in the comment above it, write empty docblocks that restate the function signature, and leave placeholder comments like `// TODO: implement` long after the code shipped. Cleaning these out is the fastest path to "this looks human-written".

## 2. Naming (naming)

**Impact:** CRITICAL
**Description:** AI-generated code swings between two opposite failure modes — generic placeholders (`data`, `result`, `info`) and over-descriptive run-ons (`theUserWhoIsCurrentlyLoggedIn`) — often in the same file. A human teammate converges on a register. Suffix abuse (`*Helper`, `*Manager`, `*Util`) and type-in-name patterns (`userObject`, `resultArray`) round out the family.

## 3. Over-engineering (over-eng)

**Impact:** HIGH
**Description:** Adding layers nobody asked for. Premature interfaces with one implementation, single-method classes that should be functions, wrappers called from one place, and pulling in a new dependency when an existing one does the job. AI defaults to "enterprise-grade" because its training distribution is enterprise-grade.

## 4. Defensive overdose (defensive)

**Impact:** HIGH
**Description:** Try/catch wrapped around code that can't throw. Null checks after a non-null assertion. `if (array && array.length > 0)` three times in the same function. Meanwhile real defenses (timeouts on external calls, rate limits, circuit breakers) are missing. AI is defensive in the wrong places.

## 5. Test slop (test)

**Impact:** HIGH
**Description:** Tests that mock everything and assert nothing. Tests that mirror the implementation's logic — they pass because they re-encode the same code, not because they verify behaviour. "Doesn't throw" assertions that don't check anything meaningful. Snapshot abuse instead of behavioural assertions.

## 6. Style fingerprints (style)

**Impact:** MEDIUM
**Description:** The small tells: hyper-consistent formatting (no human drift), `as any` / `@ts-ignore` sprinkled where types are hard, *absence* of `// HACK:` / `// XXX:` scars (real codebases have geology), stray `console.log` / `dd()` debug artifacts, and trivial boilerplate like `if (x) return true; else return false`.
