# Agent Skills

![Release](https://img.shields.io/github/v/release/voku/agent-skills?style=flat-square) ![Stars](https://img.shields.io/github/stars/voku/agent-skills?style=flat-square) ![License](https://img.shields.io/github/license/voku/agent-skills?style=flat-square)

A repository of portable skills for coding agents. This fork keeps the broader
upstream skill catalog and adds voku's evidence-driven engineering skills used by
the `voku/agent-*` workflow.

Skills follow the [Agent Skills](https://agentskills.io/) specification and can
be installed by clients supported by the `skills` CLI.

## Installation

Install the catalog:

```bash
npx skills add voku/agent-skills
```

Install one skill:

```bash
npx skills add voku/agent-skills --skill operational-prompting
npx skills add voku/agent-skills --skill coding-simplicity
npx skills add voku/agent-skills --skill php-best-practices
```

Install globally or for a specific agent:

```bash
npx skills add voku/agent-skills -g
npx skills add voku/agent-skills -a claude-code
```

List the current catalog instead of relying on a manually duplicated README
inventory:

```bash
npx skills add voku/agent-skills --list
```

Manual installation remains ordinary Git:

```bash
git clone https://github.com/voku/agent-skills.git
cp -r agent-skills/skills/* .claude/skills/
```

## voku engineering skills

### `operational-prompting`

Reusable, tool-neutral operational prompting for coding agents. It covers
repo-owned instructions, instruction hierarchy, bounded task contracts,
validation/evidence contracts, machine-readable workflows, stopping conditions,
and portable skill manifests.

The canonical skill deliberately does **not** own the governed
`voku/agent-*` operating-prompt recipe catalog. A machine-readable recipe catalog
whose correctness depends on one compiler belongs with that compiler. For the
current workflow, `voku/agent-recall-compiler` owns recipe definitions, typed
arguments, rendering/template identity, and recipe applicability metadata;
`voku/agent-loop` owns lifecycle and execution authority.

A project-specific execution contract still benefits from the familiar five
concerns:

```text
Goal
Context
Constraints
Verification
Done When
```

`Verification` defines how reality is measured. `Done When` defines the observed
result that permits the task to stop. The skill teaches those reusable principles
without duplicating Recall-owned recipes or Loop-owned lifecycle policy.

### `coding-simplicity`

Implementation-time simplicity without deleting correctness. Before adding new
surface area, search in this order:

1. no code change;
2. existing repository owner/pattern;
3. language standard library;
4. native platform capability;
5. already-installed dependency;
6. one shared root-cause fix;
7. minimum new code.

The skill keeps explicit safety and verification floors. A shorter patch that
moves behavior into the wrong layer, drops a trust-boundary check, or leaves
sibling callers broken is not simpler.

### `code-review-*`

The review skills are **independent targeted lenses**, not a mandatory review
swarm. Start with the dominant concern and keep the review evidence-first. If
another concern becomes primary, hand off to at most one smaller follow-up lens.

Available lenses include architecture, error handling, performance, security,
simplicity, and type safety.

### `php-best-practices`

Modern PHP engineering guidance covering typing, static analysis, maintainable
object design, legacy migration, and repository-native verification.

## Full catalog

The repository contains the complete catalog under [`skills/`](skills/),
including the Laravel, React/TypeScript, testing, security, database, API, Git,
SEO, accessibility, technical-debt, and documentation skills inherited from and
extended beyond the upstream project.

Use the repository itself as the source of truth:

```bash
npx skills add voku/agent-skills --list
```

Each skill's own `SKILL.md`, rules, metadata, and supporting files define its
actual contract. README summaries are intentionally not a second hand-maintained
manifest.

## Using with agent-loop

`voku/agent-loop` can install this catalog as an explicit extra skill root for
portable engineering guidance. Governed workflow prompt recipes are not sourced
from this catalog: `voku/agent-recall-compiler` owns and releases that catalog,
while `voku/agent-loop` owns durable Contract/Run orchestration and the gates that
may authorize execution.

The intended ownership shape is:

```text
tool-neutral engineering principles  -> voku/agent-skills
bounded project context + recipes     -> voku/agent-recall-compiler
Contract / Run / lifecycle authority  -> voku/agent-loop
```

Working-memory packages may carry resumable Session state, but they do not become
the durable authority for approved task policy. Likewise, installing a portable
skill does not grant workflow or mutation authority.

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for voku-specific catalog changes.

## Provenance

This repository is derived from the public `AsyrafHussin/agent-skills` catalog
and retains upstream skill material where useful. voku-specific changes focus on
governed agentic coding, operational prompting, simplicity, review routing, and
integration with the `voku/agent-*` packages.

Individual adapted skills retain their own provenance where applicable.

## License

[MIT](LICENSE)
