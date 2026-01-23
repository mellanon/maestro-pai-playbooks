# Step 0: SELECT FEATURE - Feature Selection and Initialization

**Phase**: Selection | **Gate**: None (Automated)

---

## Context
- **Playbook:** SpecFlow Development
- **Agent:** Signal-1
- **Project:** /Users/andreas/Developer/pai/versions/worktrees/signal-agent-1
- **Auto Run Folder:** /Users/andreas/Developer/maestro-pai-playbooks/playbooks/SpecFlow_Development
- **Date:** 2026-01-23
- **Loop:** 00001

## Purpose

Select the next feature to implement and initialize it for the playbook workflow. This document runs at the START of each feature cycle (including the first run and after completing each feature).

## Prerequisites

- Project repository exists
- Requirements specification available (e.g., `docs/REQUIREMENTS.md`)

## Tasks

### Task 0: Initialize SpecFlow (Fresh Start Only)

- [x] **Check if SpecFlow is initialized**: ✅ Already initialized (.specflow/ exists)
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

  **If already initialized:** Skip to Task 1.

### Task 1: Check Current Feature State

- [x] **Read feature state file** (if exists): ✅ NO_CURRENT_FEATURE - proceeding to Task 2

  Check if `/Users/andreas/Developer/maestro-pai-playbooks/playbooks/SpecFlow_Development/CURRENT_FEATURE.md` exists:
  ```bash
  cat /Users/andreas/Developer/maestro-pai-playbooks/playbooks/SpecFlow_Development/CURRENT_FEATURE.md 2>/dev/null || echo "NO_CURRENT_FEATURE"
  ```

  If it exists and contains a feature ID, that feature is IN PROGRESS. Skip to Task 4.

  If it doesn't exist or is empty, proceed to Task 2 to select a new feature.

### Task 2: Get Next Feature (By ID Order)

- [x] **List all pending features and sort by ID**: ✅ Found 14 pending features (F-2 through F-15)
  ```bash
  # Get JSON output and sort by feature ID number
  specflow status --json | jq -r '.features[] | select(.status == "pending") | .id' | sort -t'-' -k2 -n | head -1
  ```

  **Alternative (human-readable):**
  ```bash
  specflow status --json | jq -r '.features[] | select(.status == "pending") | "\(.id)\t\(.name)"' | sort -t'-' -k2 -n
  ```

  This returns features in F-1, F-2, F-3... order (NOT by priority).

  **Example output:**
  ```
  F-2	Event Logging Library
  F-3	Session Lifecycle Hooks
  F-4	Tool Use Instrumentation
  ...
  ```

  **Note:** SpecFlow natively orders by priority. The `jq` + `sort` pipeline re-orders by ID.

- [x] **Select the FIRST feature from the sorted list**: ✅ Selected F-2 (Event Logging Library)

  Record the feature ID (e.g., `F-2`) for the next task.

  **If no pending features exist:** Write "ALL_FEATURES_COMPLETE" to `/Users/andreas/Developer/maestro-pai-playbooks/playbooks/SpecFlow_Development/CURRENT_FEATURE.md` and exit playbook.

### Task 3: Initialize Feature Context

- [x] **Check feature phase and initialize if needed**: ✅ Phase is "none" - needs specification (Step 1)
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
  | `complete` | Feature done, return to Task 2 for next |

- [x] **Write current feature to state file**: ✅ Created CURRENT_FEATURE.md for F-2

  Create/update `/Users/andreas/Developer/maestro-pai-playbooks/playbooks/SpecFlow_Development/CURRENT_FEATURE.md`:
  ```markdown
  # Current Feature

  - **Feature ID:** <feature-id>
  - **Feature Name:** <feature-name>
  - **Started:** 2026-01-23
  - **Phase:** <current-phase>

  ## Feature Details

  [Copy spec summary here if available]
  ```

### Task 4: Verify Feature Ready

- [x] **Confirm feature is ready for playbook**: ✅ F-2 exists, status=pending, phase=none → Start at Step 1 (SPECIFY)
  ```bash
  specflow status <feature-id>
  ```

  **Required state:** Feature exists and is not `complete`.

  Record current phase for routing:
  - Phase `none` or `specify` → Start at Step 1 (SPECIFY)
  - Phase `plan` → Start at Step 2 (PLAN)
  - Phase `tasks` → Start at Step 3 (TASKS)
  - Phase `implement` → Start at Step 4 (IMPLEMENT)

## Output

- `/Users/andreas/Developer/maestro-pai-playbooks/playbooks/SpecFlow_Development/CURRENT_FEATURE.md` - Contains the active feature ID and context

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

**Note:** Maestro Auto Run processes documents in order (0 → 1 → 2...). The routing above is for HUMAN understanding - agents should check `CURRENT_FEATURE.md` at each step to determine if work is needed.

---

## Feature Ordering

This playbook uses **ID-based ordering** (F-1, F-2, F-3...) NOT priority-based ordering.

**Why ID order?**
- Predictable sequence for multi-feature development
- Allows planning work in feature dependency order
- Priority can be used for manual overrides

**To override:** Manually edit `/Users/andreas/Developer/maestro-pai-playbooks/playbooks/SpecFlow_Development/CURRENT_FEATURE.md` with your preferred feature ID before running.

---

**Next:** Step 1 (SPECIFY) - Begin requirements elicitation for selected feature
