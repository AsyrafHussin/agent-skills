# Changelog

Notable changes to the `voku/agent-skills` catalog are documented here.

This repository is a skill catalog rather than a Composer package, so entries are
dated and tied to Git commits instead of inventing a package version that no
runtime consumes.

## 2026-08-09 - Governed operational prompting and coding simplicity

### Added

- Added `coding-simplicity`, an implementation-time skill that searches for the
  smallest correct solution in this order: no change, existing repository owner,
  standard library, native platform capability, installed dependency, shared
  root-cause fix, then minimum new code. Safety and verification floors remain
  mandatory.
- Added a reusable `operational-prompting/operating-prompts.json` catalog with
  explicit L1/L2 recipe levels. Most engineering recipes are L2 so current
  project recall can turn reusable method into a project-specific L1 execution
  contract instead of shipping generic prompts with placeholders.
- Added L2 recipes for planning horizons, regression hunting, coverage plus
  mutation testing, deletion-first review, missingness audits, adversarial review,
  reproduce-before-fix, reject-and-restart, and multi-pass
  correctness/simplification work.
- Added context-independent L1 controls for continuation, evidence reporting, and
  bounded retry/stop behavior.

### Changed

- Operational contracts now use the five-part shape `Goal + Context + Constraints
  + Verification + Done When`. Verification defines the measurement procedure;
  Done When defines the observable stopping condition.
- Code-review skills are targeted independent lenses rather than an automatic
  review swarm. Start with the dominant lens and allow at most one evidence-backed
  handoff when another concern becomes primary.
- Reusable recipes carry no hidden task thresholds or project commands. Numeric
  floors, retry limits, mutation commands, horizons, and other policy are explicit
  caller arguments; repository facts come from the consuming project's recall.

### Fixed

- Repository badges, installation commands, clone examples, and per-skill `npx`
  examples now point at `voku/agent-skills` instead of the upstream fork owner.
