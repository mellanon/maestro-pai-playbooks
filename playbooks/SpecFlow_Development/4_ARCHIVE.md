# Step 4: ARCHIVE - Merge Specs and Update CHANGELOG

**Phase**: Archive | **Gate**: All Tests Pass

---

## Context
- **Playbook:** SpecFirst Development
- **Agent:** {{AGENT_NAME}}
- **Project:** {{AGENT_PATH}}
- **Auto Run Folder:** {{AUTORUN_FOLDER}}
- **Loop:** {{LOOP_NUMBER}}

## Objective

Merge completed specs to the canonical location and update CHANGELOG.

## Prerequisites

- All tasks COMPLETE
- All tests passing
- `specfirst gate` shows ≥80%

## Instructions

### 1. Final Test Verification

- [ ] **Run all tests** one final time:
  ```bash
  [project-specific test command]
  ```

  All tests must pass before proceeding.

### 2. Invoke OpenSpec Archive

- [ ] **Run `/openspec-archive`** to merge and archive:

```
/openspec-archive
> Which change? [change-name]
```

The command handles:
1. Verifying all tasks complete
2. Merging spec deltas → `openspec/specs/`
3. Updating `CHANGELOG.md` with entry
4. Moving folder → `openspec/archive/YYYY-MM-DD-[change-name]/`

### 3. Verify Archive Results

- [ ] **Check that specs were merged**:
  - `openspec/specs/` contains merged specs
  - No orphaned references

- [ ] **Check CHANGELOG was updated**:
  - New entry added for this change
  - Entry follows project format

- [ ] **Check archive created**:
  - Original change folder moved to `openspec/archive/`
  - Archive folder has date prefix

### 4. Validate Against SKILL-BEST-PRACTICES

If building a skill, check against `docs/SKILL-BEST-PRACTICES.md`:

| Check | Status |
|-------|--------|
| SKILL.md exists | |
| Description has USE WHEN triggers | |
| Under 500 lines | |
| Examples provided | |
| Out of scope defined | |

## Output

- Specs merged to `openspec/specs/`
- Updated `CHANGELOG.md`
- Archive at `openspec/archive/YYYY-MM-DD-[change-name]/`

## Next

→ Proceed to Step 5 (RELEASE)
