# Step 3: TASKS

**Phase**: Task Breakdown | **Gate**: 80% | **Human Approval**: Required

---

## Objective

Break architecture into atomic, dependency-tracked tasks using OpenSpec `tasks.md`.

## Prerequisites

- `PLAN.md` approved
- `specfirst status` shows PLAN phase complete

## Execute

### 1. Populate OpenSpec Tasks

Update `openspec/changes/pai-observability-stage1/tasks.md`:

```markdown
# Implementation Tasks

## Task Overview

| ID | Task | Dependencies | Status |
|----|------|--------------|--------|
| T-001 | Create event types | None | PENDING |
| T-002 | Implement event emitter | T-001 | PENDING |
| T-003 | Add SessionStart hook | T-002 | PENDING |

## Task Details

### T-001: Create event types
**Requirements**: REQ-OBS-001, REQ-OBS-002
**Files**: types/events.ts
**Status**: PENDING

...
```

### 2. Quality Gate

```bash
specfirst gate
```

## Output

- Updated `openspec/changes/pai-observability-stage1/tasks.md`
- `.specfirst/state.json` updated

## Next

After human approval → Proceed to Step 4 (IMPLEMENT)
