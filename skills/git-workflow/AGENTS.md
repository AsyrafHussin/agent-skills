# Git Workflow - Agent Guidelines

This document provides guidance for AI agents on how to effectively use the git-workflow skill to review and improve git practices in codebases.

## Skill Overview

**Purpose:** Help teams maintain excellent git hygiene through commit conventions, branching strategies, and PR workflows.

**When to Use:**
- Reviewing commit messages for clarity and convention compliance
- Analyzing branching strategies and suggesting improvements
- Evaluating pull request practices and templates
- Auditing git history for cleanliness and maintainability
- Setting up git workflows for new or existing projects
- Troubleshooting git collaboration issues

## Agent Capabilities

### 1. Commit Message Analysis

**What to Check:**
- Conventional commit format compliance
- Subject line clarity and length (50 char ideal, 72 max)
- Imperative mood usage
- Body presence and quality for complex changes
- Issue/PR references
- Breaking change documentation

**Example Analysis:**
```
Input: "fixed bug in login"

Agent Response:
[commit] Use conventional commit format: "fix(auth): resolve login timeout issue"
[commit] Add commit body explaining the root cause and solution
[commit] Reference the related issue: "Fixes #123"
```

### 2. Branch Strategy Evaluation

**What to Check:**
- Branch naming conventions
- Branch protection on main/master
- Feature branch workflow adoption
- Branch lifecycle (short-lived vs long-lived)
- Cleanup of merged branches
- Release branch strategy

**Example Analysis:**
```
Input: Repository with 50+ stale merged branches and inconsistent naming

Agent Response:
[branch] 45 merged branches still exist - implement automatic deletion
[branch] Inconsistent naming detected (fix-bug, feature_new, FIX-123) - adopt consistent convention
[branch] Main branch lacks protection - enable PR requirements and CI checks
```

### 3. Pull Request Review

**What to Check:**
- PR size (lines changed, files modified)
- Description quality and template usage
- Reviewer assignment strategy
- CI check requirements
- Merge strategy consistency
- Draft PR usage for WIP

**Example Analysis:**
```
Input: PR with 2000+ lines, no description, bypassing CI

Agent Response:
[pr] PR is too large (2000+ lines, 50 files) - consider splitting into smaller PRs
[pr] Missing PR description - add context, testing notes, and related issues
[pr] CI checks failed but merge was forced - require CI passing before merge
```

### 4. History Management Review

**What to Check:**
- Rebase vs merge strategy appropriateness
- Force push patterns on shared branches
- Commit cleanliness before merging
- Tag usage for releases
- Semantic versioning compliance

**Example Analysis:**
```
Input: Multiple force pushes to main, inconsistent merge strategy, no release tags

Agent Response:
[history] 5 force pushes detected on main branch - disable force push and use revert instead
[history] Mixing merge and rebase strategies - adopt consistent squash merge for PRs
[history] No release tags found - implement semantic versioning with annotated tags
```

### 5. Collaboration Assessment

**What to Check:**
- Code review practices
- Merge conflict resolution patterns
- Team communication through git
- CODEOWNERS file usage
- Review response times

## Analysis Workflow

### Step 1: Context Gathering
```
1. Identify repository platform (GitHub, GitLab, etc.)
2. Check repository size and team structure
3. Review existing git configuration
4. Examine recent commit history (last 50-100 commits)
5. Analyze open and recent PRs
```

### Step 2: Rule Application
```
1. Start with CRITICAL priority rules (commit messages)
2. Move to HIGH priority rules (branching, PRs)
3. Finish with MEDIUM priority rules (history, collaboration)
4. Consider project context and team size
```

### Step 3: Findings Presentation
```
Format: [category] Description of issue or recommendation

Priority order:
1. Critical issues affecting team workflow
2. High-impact improvements
3. Nice-to-have enhancements

Include:
- Specific examples from the codebase
- Concrete recommendations
- Configuration snippets when applicable
```

## Common Patterns and Solutions

### Pattern: Messy Commit History
**Indicators:**
- WIP commits in main branch
- No conventional commit format
- Commit messages like "fix", "update", "changes"

**Solution:**
```bash
# Implement commitlint
npm install --save-dev @commitlint/cli @commitlint/config-conventional
echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js

# Setup git hooks with husky
npm install --save-dev husky
npx husky install
echo '#!/bin/sh\nnpx commitlint --edit $1' > .husky/commit-msg

# Use squash merge for PRs
gh api repos/:owner/:repo --method PATCH -f squash_merge_commit_title=PR_TITLE
```

### Pattern: Unprotected Main Branch
**Indicators:**
- Direct pushes to main
- No required reviews
- CI bypassed
- Force push allowed

**Solution:**
```bash
# Enable branch protection via GitHub CLI
gh api repos/:owner/:repo/branches/main/protection --method PUT --input - <<EOF
{
  "required_pull_request_reviews": {
    "required_approving_review_count": 1,
    "dismiss_stale_reviews": true
  },
  "required_status_checks": {
    "strict": true,
    "contexts": ["ci/test", "ci/lint"]
  },
  "enforce_admins": true,
  "restrictions": null,
  "allow_force_pushes": false
}
EOF
```

### Pattern: Large, Unfocused PRs
**Indicators:**
- PRs with 1000+ lines changed
- Multiple unrelated changes
- Slow review cycles
- High conflict rate

**Solution:**
```markdown
# Create PR template
## .github/PULL_REQUEST_TEMPLATE.md

## Summary
Brief description (1-2 sentences)

## Changes
- Focused list of changes
- Max 5-10 items suggests good scope

## Size Guidelines
- ✅ Small: < 200 lines
- ⚠️ Medium: 200-400 lines
- ❌ Large: > 400 lines (should be split)

## Testing
- [ ] Unit tests added/updated
- [ ] Manual testing completed
- [ ] No breaking changes OR migration guide included

## Related Issues
Fixes #issue-number
```

### Pattern: Inconsistent Branch Naming
**Indicators:**
- Branches named "test", "fix", "my-branch"
- Mix of formats (snake_case, PascalCase, kebab-case)
- No type prefixes

**Solution:**
```bash
# Document branch naming convention in CONTRIBUTING.md
# Enforce with CI check

# .github/workflows/branch-naming.yml
name: Branch Naming
on: pull_request

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - name: Check branch name
        run: |
          if [[ ! "${{ github.head_ref }}" =~ ^(feature|fix|hotfix|refactor|docs|test|chore)/.+ ]]; then
            echo "Branch name must match pattern: type/description"
            echo "Valid types: feature, fix, hotfix, refactor, docs, test, chore"
            exit 1
          fi
```

## Example Workflows

### Scenario 1: New Project Setup
```
Agent Task: "Set up git workflow for new team repository"

Actions:
1. Create commitlint.config.js with conventional commits
2. Set up Husky hooks for commit-msg validation
3. Create PR template in .github/PULL_REQUEST_TEMPLATE.md
4. Configure branch protection for main
5. Create CONTRIBUTING.md with git guidelines
6. Set up semantic-release for automated versioning
7. Document workflow in README.md

Deliverable: "Git workflow configured with conventional commits,
PR templates, branch protection, and automated releases"
```

### Scenario 2: Git History Audit
```
Agent Task: "Review last 100 commits and identify git hygiene issues"

Analysis Steps:
1. Check commit message format
   git log --oneline -100 --format="%s" | analyze format

2. Identify WIP/fixup commits in main
   git log -100 --grep="WIP\|fixup\|wip" --oneline

3. Check for force pushes
   git reflog main | grep "force"

4. Analyze commit size distribution
   git log -100 --stat | analyze changes

5. Review merge commit patterns
   git log -100 --merges --oneline

Output: Categorized findings with specific examples and recommendations
```

### Scenario 3: PR Workflow Optimization
```
Agent Task: "Improve PR workflow for faster reviews"

Recommendations:
1. Implement PR size checks in CI
2. Set up CODEOWNERS for automatic reviewer assignment
3. Create PR templates for different change types
4. Configure required status checks
5. Set up draft PR workflow documentation
6. Implement squash merge by default
7. Add PR metrics tracking

Outcome: "Reduced average PR review time from 3 days to 1 day"
```

## Best Practices for Agents

### DO:
- Provide specific, actionable recommendations
- Show concrete examples from the codebase
- Offer configuration snippets and commands
- Explain WHY each practice matters
- Consider team size and project context
- Prioritize critical issues first
- Link to authoritative resources

### DON'T:
- Recommend practices that don't fit the team's workflow
- Overwhelm with too many changes at once
- Suggest configuration without explaining benefits
- Ignore existing team conventions
- Apply rules dogmatically without context
- Forget to explain migration paths

## Integration Points

### With CI/CD
```yaml
# Example GitHub Actions integration
name: Git Hygiene Check
on: [pull_request]

jobs:
  commit-lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: wagoid/commitlint-github-action@v5

  pr-size:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Check PR size
        run: |
          FILES_CHANGED=$(git diff --name-only origin/${{ github.base_ref }}...HEAD | wc -l)
          LINES_CHANGED=$(git diff --stat origin/${{ github.base_ref }}...HEAD | tail -1 | awk '{print $4+$6}')
          if [ $LINES_CHANGED -gt 500 ]; then
            echo "PR is too large ($LINES_CHANGED lines). Consider splitting."
            exit 1
          fi
```

### With Project Tools
- **GitHub/GitLab:** Use API for branch protection, PR analysis
- **Jira/Linear:** Link commits to issues for traceability
- **Slack/Discord:** Notify team of workflow violations
- **Documentation:** Auto-generate changelog from conventional commits

## Metrics to Track

Track these metrics to measure git workflow health:

1. **Commit Quality**
   - % of commits following conventional format
   - Average commit message length
   - % of commits with body text

2. **PR Velocity**
   - Average time to first review
   - Average time to merge
   - Average PR size (lines, files)

3. **Branch Health**
   - Number of stale branches
   - Average branch lifetime
   - % of branches following naming convention

4. **Release Cadence**
   - Time between releases
   - % of releases with proper tags
   - Semantic version compliance

## Learning Resources for Agents

When providing guidance, reference these authoritative sources:

- **Git Official Docs:** https://git-scm.com/doc
- **Conventional Commits:** https://www.conventionalcommits.org
- **Pro Git Book:** https://git-scm.com/book/en/v2
- **GitHub Best Practices:** https://docs.github.com/en/get-started/quickstart/github-flow
- **Code Review Guide:** https://google.github.io/eng-practices/review/

## Output Format

Always structure your analysis as:

```
# Git Workflow Analysis

## Summary
Brief overview of findings (2-3 sentences)

## Critical Issues
[commit] Issue description with example
[branch] Issue description with example

## Recommendations

### Immediate Actions
1. [Action with command or config]
2. [Action with command or config]

### Long-term Improvements
1. [Improvement suggestion]
2. [Improvement suggestion]

## Configuration Changes

### commitlint.config.js
```javascript
// configuration
```

### .github/workflows/ci.yml
```yaml
# workflow
```

## Resources
- [Link to relevant documentation]
- [Link to example implementation]
```

---

## Agent Self-Check

Before completing analysis, verify:
- [ ] Checked all critical priority rules
- [ ] Provided specific examples from codebase
- [ ] Included actionable recommendations
- [ ] Offered configuration snippets
- [ ] Explained WHY for each suggestion
- [ ] Prioritized issues appropriately
- [ ] Considered team/project context
- [ ] Linked to authoritative resources
