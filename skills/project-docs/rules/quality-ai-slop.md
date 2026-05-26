---
title: AI Slop in Documentation Content
impact: HIGH
impactDescription: "AI-generated content has telltale fingerprints that erode docs quality file-by-file"
tags: quality, ai-generated, slop, cleanup
---

## AI Slop in Documentation Content

**Impact: HIGH (AI-generated content has telltale fingerprints that erode docs quality file-by-file)**

[`cleanup-ai-junk`](cleanup-ai-junk.md) covers whole files agents leave behind. This rule covers **content inside otherwise-legitimate docs** — a README, an architecture overview, a runbook — that has been bulked up with AI-style filler. The patterns are recognizable; once you see them, you can't unsee them.

## Slop fingerprints in content

### 1. Filler phrases — delete on sight

```
"It's worth noting that..."
"It's important to understand..."
"In summary..."  /  "To summarize..."
"Let's dive into..."  /  "Let's take a look at..."
"As we'll see..."  /  "As mentioned earlier..."
"That being said..."  /  "Having said that..."
"At its core..."  /  "Essentially..."
"In essence..."  /  "Fundamentally..."
"It goes without saying..."
"Needless to say..." (if it's needless to say, don't say it)
```

These phrases add words without adding meaning. A human technical writer doesn't sprinkle "in essence" through a deployment guide.

### 2. Bullet-list explosions

```markdown
❌ ## Why we chose Redis

   Redis was chosen for the following reasons:

   - **Performance** — Redis is very fast
   - **Reliability** — Redis is highly reliable
   - **Scalability** — Redis can scale to large workloads
   - **Community** — Redis has a large and active community
   - **Documentation** — Redis is well-documented
   - **Ecosystem** — Redis has a rich ecosystem of tools
   - **Maturity** — Redis is a mature and proven technology
   - **Versatility** — Redis supports many use cases
```

Eight bullets, zero specifics. Replace with one paragraph that says *what we actually needed* (sub-millisecond GET latency + pub-sub for session invalidation) and *why Redis fit* (battle-tested at our scale; team has prod experience). The version with bullets *looks* informative; the prose version *is* informative.

### 3. Closing sign-offs

```markdown
❌ ## Conclusion

   I hope this guide has been helpful in setting up your development
   environment. If you have any questions, feel free to reach out to
   the team. Happy coding! 🎉
```

Documentation isn't a blog post. Cut the goodbye.

### 4. Past-tense narration where present tense fits

```markdown
❌ When we started this project, we considered using Laravel because
   it offered a good balance of features. We decided that Laravel was
   the right choice. We chose MySQL as our database because of its
   maturity.
```

```markdown
✅ The application uses Laravel for its mature ecosystem and MySQL
   for its operational track record. See [ADR-0002](adr/0002-...) for
   the full decision context.
```

The "we did X" narrative belongs in commit messages or ADRs, not in the current architecture doc.

### 5. Generic praise / hedging

```
"leveraging industry best practices"
"following modern development standards"
"a robust and scalable solution"
"a powerful and flexible framework"
"state-of-the-art technology"
```

These phrases are zero-content. If "industry best practices" matters, name the specific practice (e.g., "OWASP ASVS Level 2", "12-factor app", "Conventional Commits"). If "scalable" matters, name the actual numbers (e.g., "10k req/s with p95 < 200ms").

### 6. Markdown formatting overuse

- **Random words bolded for no reason**
- Excessive heading nesting (`####` after only one `##`)
- Bullet lists where prose flows better
- Tables for two items

### 7. The 100-line answer to a 3-line question

A "How do I run tests?" section that takes 100 lines to say `php artisan test`.

## Detection

```bash
# Filler-phrase fingerprints
grep -rEi --include='*.md' \
  "it's worth noting|let's (dive|take a look|start)|it goes without saying|needless to say|that being said|having said that|at its core|in essence|fundamentally" \
  docs/ README.md

# Closing sign-offs in docs
grep -rEi --include='*.md' \
  "happy coding|hope this (was )?help|if you have any questions|feel free to reach out" \
  docs/ README.md

# Generic praise (likely AI-slop)
grep -rEi --include='*.md' \
  "industry best practices|state-of-the-art|robust and scalable|powerful and flexible" \
  docs/ README.md

# Many emojis = often AI-generated (humans rarely emoji-pepper technical docs).
# `grep -roE` prints one match per emoji occurrence (not per line), so the
# per-file count below is the true emoji count.
find docs/ README.md -name '*.md' -type f 2>/dev/null | while read f; do
  N=$(grep -oE '🤖|✨|🚀|🎉|💡|⚡|📝|🔥|🎯|💪' "$f" | wc -l | tr -d ' ')
  if [ "$N" -gt 5 ]; then
    echo "MANY EMOJIS ($N): $f"
  fi
done
```

## How to triage a slop-flagged doc

1. **Read the whole doc end to end** — is the content valid under the slop?
2. **If valid** — rewrite the prose: cut filler, name specifics, prefer present tense
3. **If hollow** — the slop was hiding that the doc has no real content. Either rewrite from scratch with real content, or delete.

**Never** rewrite docs *with another LLM* without close review — you'll just produce different slop. Use the model to draft, then read every line as a human and cut ruthlessly.

Reference: Internal: [`cleanup-ai-junk`](cleanup-ai-junk.md) (junk files) · [`quality-conciseness`](quality-conciseness.md) (general bloat) · [On Bullshit (Frankfurt)](https://en.wikipedia.org/wiki/On_Bullshit) — yes, really
