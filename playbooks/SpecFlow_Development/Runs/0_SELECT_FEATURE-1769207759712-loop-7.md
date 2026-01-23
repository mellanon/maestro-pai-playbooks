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

- [x] **Create state directory if needed**: ✅ State directory `.maestro/outputs/COMPLETED_FEATURES` already exists. Current feature F-11 (Vector Collector Service) is in progress at phase `none`.
  ```bash
  mkdir -p .maestro/outputs/COMPLETED_FEATURES
  ```

### Task 1: Initialize SpecFlow (Fresh Start Only)

- [x] **Check if SpecFlow is initialized**: ✅ SpecFlow already initialized (`.specflow/` directory exists with features.db). Skipping to Task 2.
  ```bash
  ls -la .specflow/ 2>/dev/null || ls -la .specify/ 2>/dev/null || echo "NOT_INITIALIZED"
  ```

  **If NOT_INITIALIZED:**

  1. Look for a requirements spec in the project:
     ```bash
     ls docs/*.md 2>/dev/null | head -5
     ```

  2. Initialize SpecFlow from the requirements:
     ```bash
     # If requirements file exists
     specflow init --from-spec docs/REQUIREMENTS.md

     # Or if no spec file, initialize empty and add features manually
     specflow init
     ```

  3. Verify features were created:
     ```bash
     specflow status
     ```

  **If already initialized:** Skip to Task 2.

### Task 2: Check Current Feature State

- [x] **Read feature state file** (if exists): ✅ Current feature F-11 (Vector Collector Service) already set, at phase `none`. Skipping to Task 5.

  Check if `.maestro/CURRENT_FEATURE.md` exists:
  ```bash
  cat .maestro/CURRENT_FEATURE.md 2>/dev/null || echo "NO_CURRENT_FEATURE"
  ```

  If it exists and contains a feature ID, that feature is IN PROGRESS. Skip to Task 5.

  If it doesn't exist or is empty, proceed to Task 3 to select a new feature.

### Task 3: Get Next Feature (By ID Order)

- [ ] **List all pending features and sort by ID**:
  ```bash
  # Get JSON output and sort by feature ID number
  specflow status --json | jq -r '.features[] | select(.status == "pending") | .id' | sort -t'-' -k2 -n | head -1
  ```

  **Alternative (human-readable):**
  ```bash
  specflow status --json | jq -r '.features[] | select(.status == "pending") | "\(.id)\t\(.name)"' | sort -t'-' -k2 -n
  ```

  This returns features in F-1, F-2, F-3... order (NOT by priority).

- [ ] **Select the FIRST feature from the sorted list**:

  Record the feature ID (e.g., `F-2`) for the next task.

  **If no pending features exist:** Write "ALL_FEATURES_COMPLETE" to `.maestro/CURRENT_FEATURE.md` and exit playbook.

### Task 4: Initialize Feature Context

- [ ] **Check feature phase and initialize if needed**:
  ```bash
  specflow status <feature-id>
  ```

  Based on the current phase:

  | Current Phase | Action |
  |---------------|--------|
  | `none` | Run `specflow specify <feature-id>` to start |
  | `specify` | Feature has spec, proceed to Step 1 |
  | `plan` | Feature has plan, proceed to Step 2 |
  | `tasks` | Feature has tasks, proceed to Step 3 |
  | `implement` | Feature in progress, proceed to Step 4 |
  | `complete` | Feature done, return to Task 3 for next |

- [ ] **Write current feature to state file**:

  Create/update `.maestro/CURRENT_FEATURE.md`:
  ```markdown
  # Current Feature

  - **Feature ID:** <feature-id>
  - **Feature Name:** <feature-name>
  - **Started:** <date>
  - **Phase:** <current-phase>

  ## Feature Details

  [Copy spec summary here if available]
  ```

### Task 5: Verify Feature Ready

- [x] **Confirm feature is ready for playbook**: ✅ F-11 (Vector Collector Service) verified as pending with phase `none`. 16 features total: 12 complete, 4 pending. F-11 is ready to proceed to Step 1 (SPECIFY).

  ```bash
  specflow status <feature-id>
  ```

  **Required state:** Feature exists and is not `complete`.

  Record current phase for routing:
  - Phase `none` or `specify` → Start at Step 1 (SPECIFY) ← **F-11 is here**
  - Phase `plan` → Start at Step 2 (PLAN)
  - Phase `tasks` → Start at Step 3 (TASKS)
  - Phase `implement` → Start at Step 4 (IMPLEMENT)

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
