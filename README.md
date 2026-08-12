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

Reusable operational prompting for coding agents. The main design is not a pile
of polished generic prompts. Most reusable engineering recipes are **L2
meta-prompts** that tell the consuming agent how to create a project-specific L1
contract from current repository recall.

The generated L1 contract has exactly five parts:

```text
Goal
Context
Constraints
Verification
Done When
```

`Verification` defines how reality is measured. `Done When` defines the observed
result that permits the task to stop.

The versioned recipe catalog lives at:

```text
skills/operational-prompting/operating-prompts.json
```

Current L2 recipes include:

- `adversarial-review`
- `coverage-mutation`
- `deletion-first`
- `missingness-audit`
- `multi-pass-correctness-simplify`
- `plan-horizon`
- `regression-hunt`
- `reproduce-before-fix`
- `reject-and-restart`

Context-independent controls remain L1, including continuation, evidence
reporting, and bounded retry/stop behavior.

Recipes deliberately contain no hidden repository commands or task thresholds.
The caller supplies hard policy such as horizons, minimum coverage increases,
mutation commands, retry limits, and stopping conditions. Concrete files,
symbols, callers, tests, project conventions, risks, and executable validation
come from the consuming repository's recall.

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

`voku/agent-loop` can install this catalog as an explicit extra skill root. The
`operational-prompting` manifest is also consumed explicitly by the governed
L2 workflow:

```text
approved task policy
+ project recall
+ selected L2 recipe
        ↓
project-specific L1 execution contract
        ↓
implementation + verification
```

The wording above is intentionally about semantic ownership, not a particular
working-memory file. Approved task policy must come from the governed workflow's
current durable source; pruneable Session state is not the canonical source for
an L2 execution contract.

The skill repository owns reusable engineering semantics.
`agent-recall-compiler` owns deterministic project context and recipe rendering,
and `agent-loop` owns durable task/Run orchestration and execution-contract
gates. Working-memory packages may carry resumable Session state, but they do
not become the durable authority for approved L2 policy merely because an older
integration passed a WorkBrief through them.

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
