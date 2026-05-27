# Operational Prompting

Repo-owned operational prompting for coding agents. 8 rules across 4 categories.

**Version:** 1.0.0

## Overview

This skill provides guidance for:
- Replacing persona-heavy mega-prompts with repo-owned instruction files
- Layering agent rules across global, repo, path, and task scopes
- Defining exact validation commands and stopping conditions
- Requiring evidence-first summaries instead of theatrical process narration
- Publishing portable, vendor-neutral skill manifests in YAML

## Categories

### 1. Repo-Owned Control (Critical)
Keep durable agent behavior in repository files and define an instruction hierarchy.

### 2. Task Contracts (Critical)
Limit edits to the smallest affected scope and stop when requirements are unclear.

### 3. Validation & Evidence (High)
Bind work to exact commands, deterministic outputs, and auditable summaries.

### 4. Portability (Medium)
Publish portable skill manifests for Copilot, Codex, Claude, Gemini, and similar tools.

## Rules

| Rule | Category | Impact |
|------|----------|--------|
| `repo-owned-instructions` | Repo-Owned Control | CRITICAL |
| `instruction-hierarchy` | Repo-Owned Control | CRITICAL |
| `scope-minimal-diff` | Task Contracts | CRITICAL |
| `stop-ask-conditions` | Task Contracts | CRITICAL |
| `validation-contracts` | Validation & Evidence | HIGH |
| `evidence-output` | Validation & Evidence | HIGH |
| `machine-readable-first` | Validation & Evidence | HIGH |
| `portable-skill-manifests` | Portability | MEDIUM |
