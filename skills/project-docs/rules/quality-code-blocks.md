---
title: Code Blocks — Language Tags and Copy-Pasteability
impact: MEDIUM
impactDescription: "Untagged code blocks lose syntax highlighting; broken commands waste reader time"
tags: quality, code-blocks, language-tags
---

## Code Blocks — Language Tags and Copy-Pasteability

**Impact: MEDIUM (Untagged code blocks lose syntax highlighting; broken commands waste reader time)**

A code block without a language tag renders as monospace text with no syntax highlighting. A "command" that contains an unexplained placeholder (`<your-token-here>`) or a typo wastes every future reader's time. Code blocks in docs are contracts: if you paste them, they should work.

## Rules

### 1. Every code block has a language tag

````markdown
❌ ```
   php artisan migrate
   ```

✅ ```bash
   php artisan migrate
   ```
````

Common tags:

| Tag | Use for |
|---|---|
| `bash` / `sh` | Shell commands, terminals |
| `php` | PHP code |
| `js` / `ts` / `tsx` | JavaScript / TypeScript / TSX |
| `json` | JSON config |
| `yaml` / `yml` | YAML (CI configs, etc.) |
| `sql` | SQL queries |
| `markdown` / `md` | Nested markdown examples (use 4-backtick outer fence) |
| `diff` | Patches / before-after |
| `text` or no tag | Genuinely plain text only |

### 2. Commands you intend to be copy-pasted must actually run

```bash
❌ git clone <your-repo-url>            # placeholder; reader has to figure out what
❌ npm install your-package             # 'your-package' isn't a real package

✅ git clone git@github.com:your-org/your-repo.git
✅ npm install                           # no args — installs from package.json
```

If a placeholder is unavoidable (real credentials, secret URL), surround it with a comment that makes the substitution obvious:

```bash
✅ # Replace YOUR_API_TOKEN with the token from Settings → API
   curl -H "Authorization: Bearer YOUR_API_TOKEN" https://api.example.com/...
```

### 3. Multi-line commands use proper line continuation

```bash
❌ git commit -m "feat: add user export"
   --no-verify

✅ git commit -m "feat: add user export" \
              --no-verify
```

Without the backslash, the second line is a separate command (and will fail).

### 4. Don't include `$` or `>` prompts in copy-pasteable blocks

```bash
❌ $ npm install
   $ npm run dev

✅ npm install
   npm run dev
```

The `$` is fine if you're showing input/output together (where output lines have no `$`), but for copy-paste-friendly blocks, omit the prompt.

### 5. Show output separately from input

````markdown
❌ ```bash
   $ php artisan about
   Laravel ............. 11.0
   PHP ................. 8.3
   ```

✅ ```bash
   php artisan about
   ```

   Output:

   ```
   Laravel ............. 11.0
   PHP ................. 8.3
   ```
````

This way the reader can copy the command without dragging output along.

### 6. Test the commands

Before publishing a guide, run every command in it from scratch (clean shell, clean checkout). The number of bugs you find on the first run is sobering.

## Detection

```bash
# Code blocks with no language tag
for f in $(find docs/ README.md -name '*.md' 2>/dev/null); do
  awk -v file="$f" '
    /^```$/ && !in_code { print file ":" NR ": untagged code block"; in_code=1; next }
    /^```/  && !in_code { in_code=1; next }
    /^```$/ &&  in_code { in_code=0 }
  ' "$f"
done

# Markdownlint rule MD040 (fenced-code-language) catches this automatically
npx markdownlint-cli2 --config '.markdownlint.json' '**/*.md'
```

Add to `.markdownlint.json`:

```json
{
  "MD040": true        // fenced-code-language — language tags required
}
```

Reference: [Markdownlint MD040](https://github.com/DavidAnson/markdownlint/blob/main/doc/Rules.md#md040) · [GitHub — Syntax highlighting](https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/creating-and-highlighting-code-blocks)
