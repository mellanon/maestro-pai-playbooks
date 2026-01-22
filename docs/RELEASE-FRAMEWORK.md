# PAI Release Framework

**Purpose**: Step-by-step operational procedure for releasing PAI contributions. Use this for FINALIZE phase validation.

---

## Release Flow Overview

```
feature/branch    → Development (messy OK)
        │
        │  1. File inventory
        │  2. Sanitization check
        │  3. Tests pass
        ▼
contrib-branch    → Clean staging (cherry-pick)
        │
        │  4. PR review
        │  5. Merge
        ▼
main              → Release
```

---

## Pre-Release Checklist

### Phase 1: File Inventory

- [ ] **Explicit include list** defined
- [ ] **Explicit exclude list** defined (credentials, fixtures)
- [ ] **No untracked files** that shouldn't be included

### Phase 2: Sanitization

- [ ] **No PII** (personal paths, usernames)
- [ ] **No secrets** (API keys, tokens)
- [ ] **Config externalized** to .env files

### Phase 3: Scope Check

- [ ] PR matches **original proposal**
- [ ] **No drive-by fixes** (unrelated changes)
- [ ] **Commit history clean**

### Phase 4: Validation

- [ ] **Tests pass** (unit, integration, CLI)
- [ ] **Documentation updated**
- [ ] **CHANGELOG.md updated**

---

## Human Approval Gates

| Gate | Phase | Check |
|------|-------|-------|
| **GATE 1** | Pre-Release | File inventory approved |
| **GATE 2** | Pre-Release | Sanitization check passed |
| **GATE 3** | Pre-Release | Tests pass |
| **GATE 4** | Pre-Release | Documentation complete |
| **GATE 5** | Contrib | Clean commit created |
| **GATE 6** | Contrib | Diff reviewed |
| **GATE 7** | Publish | PR created with description |
| **GATE 8** | Publish | PR approved and merged |

---

## File Inventory Template

```markdown
## File Inventory

### Include (✅)

| # | File/Directory | Reason |
|---|----------------|--------|
| | **Skill Definition** | |
| 1 | `.claude/skills/[Name]/SKILL.md` | Skill triggers |
| 2 | `.claude/skills/[Name]/README.md` | Quick start |
| 3 | `.claude/skills/[Name]/CHANGELOG.md` | Version history |
| 4 | `.claude/skills/[Name]/docs/*.md` | Documentation |
| | **Implementation** | |
| 5 | `bin/[cli]/[cli].ts` | CLI entrypoint |
| 6 | `bin/[cli]/lib/*.ts` | Library modules |
| | **Tests** | |
| 7 | `bin/[cli]/test/*.test.ts` | Test files |

### Exclude (❌)

| Path | Reason |
|------|--------|
| `bin/[cli]/profiles/` | Contains credentials |
| `.env`, `.env.*` | Contains secrets |
| `test/fixtures/` | May contain PII |
```

---

## PR Description Template

```markdown
## Summary

**TL;DR:** [One sentence value proposition]

### How It Works

[ASCII diagram or brief architecture]

### Key Components

| Component | Purpose |
|-----------|---------|
| `[file]` | [Description] |

---

## What This Adds

### Implementation (`bin/`)

| File | Purpose |
|------|---------|
| `[name].ts` | [Description] |

### Skill Definition (`.claude/skills/[Name]/`)

- `SKILL.md` - Workflow routing
- `docs/` - CLI reference, concepts

---

## Test Plan

- [x] Unit tests pass
- [x] Integration tests pass
- [x] Manual testing completed
- [ ] Reviewer tests with own config

---

## Checklist

- [x] Tests passing
- [x] No secrets in code
- [x] Documentation updated
- [x] CHANGELOG updated
```

---

## Version Tagging

```bash
# Tag marks tested code
git tag -a v1.0.0 -m "Release v1.0.0: Description"
git push origin v1.0.0

# Cherry-pick FROM TAG (not feature branch)
git checkout v1.0.0 -- path/to/files/
```

---

## If PR Needs Changes

```bash
# Fix on feature branch (NOT contrib)
git checkout feature/branch
# Make fixes...

# Create new tag
git tag -a v1.0.1 -m "Release v1.0.1: Address feedback"

# Update contrib from NEW tag
git checkout contrib-branch
git checkout v1.0.1 -- path/to/files/
git commit -m "fix: Address PR feedback"
```

---

## Key Principles

1. **Tag = Tested Code** - Tag marks exact code that passed tests
2. **Cherry-pick FROM TAG** - Not from feature branch
3. **File Inventory is Law** - Only inventory files get released
4. **Human Gates** - Each phase requires approval
5. **Never Force Push** - To protected branches

---

## Sanitization Commands

```bash
# Check for secrets
grep -rE "(API_KEY|TOKEN|SECRET)" --include="*.ts"

# Check for personal paths
grep -r "/Users/" --include="*.ts" --include="*.md"

# Check for hardcoded usernames
grep -rE "(andreas|mellanon)" --include="*.ts"
```

---

## Validation Gate

Before creating PR:

- [ ] File inventory created and approved
- [ ] Sanitization check passed
- [ ] All tests passing
- [ ] Documentation updated
- [ ] CHANGELOG entry added
- [ ] Clean commit from tag
- [ ] Diff reviewed

---

## Source

Extracted from: [PAI Skill Release and Contribution Workflow Runbook](https://github.com/danielmiessler/PAI)
