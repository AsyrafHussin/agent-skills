---
title: UPPERCASE Naming for Root-Level Conventional Files
impact: CRITICAL
impactDescription: "Hosting platforms, tools, and readers expect specific filenames at root — case matters"
tags: naming, root, conventions
---

## UPPERCASE Naming for Root-Level Conventional Files

**Impact: CRITICAL (Hosting platforms, tools, and readers expect specific filenames at root — case matters)**

GitHub, GitLab, Bitbucket, npm, Packagist, and most static-site generators look for **specific filenames in specific cases** at the repo root. `Readme.md` is not the same as `README.md` to a case-sensitive filesystem, and on case-sensitive CI runners (Linux containers) a mismatch breaks tools that auto-render the file.

## Incorrect

```
❌ Inconsistent / wrong casing
.
├── Readme.md                   # GitHub renders, but tools that grep "README" miss it
├── Changelog.md                # Keep-a-Changelog tooling expects CHANGELOG.md
├── license.md                  # Should be LICENSE (no extension by convention)
├── Contributing.md             # GitHub's "How to contribute" UI looks for CONTRIBUTING.md
└── security.md                 # GitHub security advisories look for SECURITY.md
```

**Problems:**
- GitHub case-sensitively matches `SECURITY.md` for the Security Advisories tab; `security.md` won't link
- Many CI tools (e.g., `markdownlint` rule MD041) and license detectors look up the exact uppercase name
- Linux filesystems treat `README.md` and `Readme.md` as distinct files — switching between editor casings creates phantom duplicates in git

## Correct

```
✅ Conventional UPPERCASE names at root
.
├── README.md                   # uppercase, .md extension
├── CHANGELOG.md
├── LICENSE                     # no .md extension (long-standing convention)
├── CONTRIBUTING.md
├── SECURITY.md
├── CODE_OF_CONDUCT.md          # underscores, not hyphens
└── AUTHORS.md                  # or MAINTAINERS.md
```

**Benefits:**
- GitHub auto-detects each file for its respective tab/feature (Security, Contributing, Code of Conduct)
- License-detection tools (GitHub Linguist, choosealicense.com) recognize `LICENSE` without extension
- Case-sensitive CI environments behave the same as your dev machine

## The `LICENSE` extension convention

`LICENSE` (no extension) is the long-standing convention from open-source culture. GitHub auto-detects the license type from the file content regardless of extension, but `LICENSE` without extension is the dominant pattern. `LICENSE.md` and `LICENSE.txt` also work; pick one and be consistent across your repos.

## Detection

```bash
# Find mis-cased conventional root files
ls *.md LICENSE* 2>/dev/null | \
  awk 'BEGIN{IGNORECASE=1}
       /^(readme|changelog|contributing|security|code[-_]of[-_]conduct|authors|maintainers)\.md$/ \
       && $0 !~ /^(README|CHANGELOG|CONTRIBUTING|SECURITY|CODE_OF_CONDUCT|AUTHORS|MAINTAINERS)\.md$/ \
       { print "MIS-CASED: " $0 }'

# Or simply:
ls Readme.md Changelog.md Contributing.md Security.md 2>/dev/null
```

Reference: [GitHub — Adding a security policy](https://docs.github.com/en/code-security/getting-started/adding-a-security-policy-to-your-repository) · [Keep a Changelog — naming](https://keepachangelog.com/)
