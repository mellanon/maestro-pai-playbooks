# Step 0: SELECT FEATURE - Feature Selection and Initialization

**Phase**: Selection | **Gate**: None (Automated)

---

## Context
- **Playbook:** SpecFlow Development
- **State Directory:** `.maestro/` (in project root - created if missing)

## Purpose

Select the next feature to implement and initialize it for the playbook workflow. This document runs at the START of each feature cycle (including the first run and after completing each feature).

## Prerequisites

- Project repository exists
- Requirements specification available (e.g., `docs/REQUIREMENTS.md`)

## Tasks

### Task 0: Initialize State Directory

- [x] **Create state directory if needed**: ✅ Already exists with CURRENT_FEATURE.md and outputs/
  ```bash
  mkdir -p .maestro/outputs/COMPLETED_FEATURES
  ```

### Task 1: Initialize SpecFlow (Fresh Start Only)

- [x] **Check if SpecFlow is initialized**: ✅ SpecFlow already initialized (`.specflow/` directory exists with features.db and evals.db)

  **If already initialized:** Skip to Task 2.

### Task 2: Check Current Feature State

- [x] **Read feature state file** (if exists): ✅ `.maestro/CURRENT_FEATURE.md` exists with F-7 (PreToolUse Hook Instrumentation) at phase `none`. Feature is IN PROGRESS. Skipping to Task 5.

### Task 3: Get Next Feature (By ID Order)

- [x] **List all pending features and sort by ID**: ⏭️ SKIPPED - Current feature F-7 already set in CURRENT_FEATURE.md

- [x] **Select the FIRST feature from the sorted list**: ⏭️ SKIPPED - F-7 already selected

### Task 4: Initialize Feature Context

- [x] **Check feature phase and initialize if needed**: ⏭️ SKIPPED - F-7 already initialized in CURRENT_FEATURE.md

- [x] **Write current feature to state file**: ⏭️ SKIPPED - CURRENT_FEATURE.md already exists with F-7

### Task 5: Verify Feature Ready

- [x] **Confirm feature is ready for playbook**: ✅ F-7 (PreToolUse Hook Instrumentation) verified:
  - Status: `pending` (not complete)
  - Phase: `none`
  - **Routing:** Phase `none` → Start at Step 1 (SPECIFY)

## Output

- `.maestro/CURRENT_FEATURE.md` - Contains the active feature ID and context

## Routing Logic

After this document completes, the playbook proceeds based on feature phase:

```
IF feature.phase == "none" OR feature.phase == "specify":
    → Proceed to 1_SPECIFY.md
ELIF feature.phase == "plan":
    → Proceed to 2_PLAN.md (skip Step 1)
ELIF feature.phase == "tasks":
    → Proceed to 3_TASKS.md (skip Steps 1-2)
ELIF feature.phase == "implement":
    → Proceed to 4_IMPLEMENT.md (skip Steps 1-3)
ELIF feature == "ALL_FEATURES_COMPLETE":
    → Exit playbook (no more work)
```

**Note:** Maestro Auto Run processes documents in order (0 → 1 → 2...). The routing above is for HUMAN understanding - agents should check `.maestro/CURRENT_FEATURE.md` at each step to determine if work is needed.

---

## Feature Ordering

This playbook uses **ID-based ordering** (F-1, F-2, F-3...) NOT priority-based ordering.

**Why ID order?**
- Predictable sequence for multi-feature development
- Allows planning work in feature dependency order
- Priority can be used for manual overrides

**To override:** Manually edit `.maestro/CURRENT_FEATURE.md` with your preferred feature ID before running.

---

**Next:** Step 1 (SPECIFY) - Begin requirements elicitation for selected feature
