# Step 6: FINALIZE

**Phase**: Archive & Release | **Gate**: 100%

**SpecFirst Skill**: `/openspec-archive`, `/specfirst-release`

---

## Objective

Archive completed specs and prepare for release/PR.

## Prerequisites

- All tasks COMPLETE
- All tests passing
- `specfirst gate` shows 100%

## Execute

### 1. Archive Specs

```
/openspec-archive
> Which change? pai-observability-stage1
```

SpecFirst handles:
- Verifying all tasks complete
- Merging spec deltas to `openspec/specs/`
- Updating `CHANGELOG.md`
- Moving to `openspec/archive/`

### 2. Prepare Release (if needed)

```
/specfirst-release
> Version? v1.0.0
> Skill? PAI Observability
```

Creates:
- File inventory
- Pre-release checklist
- Sanitization verification

### 3. Create PR Description

Generate `PR_DESCRIPTION.md`:

```markdown
## Summary

[Auto-generated from proposal.md]

## Changes

[Auto-generated from tasks.md]

## Test Results

[Auto-generated from test output]
```

## Output

- Archived specs in `openspec/archive/`
- Updated `CHANGELOG.md`
- `PR_DESCRIPTION.md`
- Ready for `git commit` and PR

## Complete

Playbook finished. Create PR when ready.
