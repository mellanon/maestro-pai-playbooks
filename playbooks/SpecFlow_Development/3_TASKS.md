# Step 3: TASKS - Break Down Implementation

**Phase**: Task Planning | **Gate**: Automated

---

## Context
- **Playbook:** SpecFlow Development
- **State Directory:** `.maestro/` (in project root)

## Objective

Break the feature into reviewable implementation units with dependencies.

## Prerequisites

- Technical plan approved (plan.md exists and human approved)
- Feature phase is "plan"

## Phase Guard (MUST RUN FIRST)

Before doing any work, verify this step should run:

- [ ] **Check if this step should execute**:
  ```bash
  # Read current feature ID
  FEATURE_ID=$(grep "Feature ID:" .maestro/CURRENT_FEATURE.md 2>/dev/null | awk -F: '{print $2}' | tr -d ' ')

  # Check if ALL_FEATURES_COMPLETE
  if grep -q "ALL_FEATURES_COMPLETE" .maestro/CURRENT_FEATURE.md 2>/dev/null; then
    echo "PHASE_GUARD: SKIP - All features complete"
    exit 0
  fi

  # Check feature exists
  if [[ -z "$FEATURE_ID" ]]; then
    echo "PHASE_GUARD: SKIP - No current feature"
    exit 0
  fi

  # Step 3 only runs when phase is "plan"
  PHASE=$(specflow status "$FEATURE_ID" --json 2>/dev/null | jq -r '.phase' 2>/dev/null || echo "none")

  if [[ "$PHASE" != "plan" ]]; then
    echo "PHASE_GUARD: SKIP - Feature not at plan phase (current: $PHASE)"
    exit 0
  fi

  echo "PHASE_GUARD: PROCEED - $FEATURE_ID ready for task breakdown (phase: $PHASE)"
  ```

If the phase guard exits with "SKIP", this document is a NO-OP. Proceed to next document.

## Instructions

### 1. Check Current Status

- [ ] **Verify plan exists**:
  ```bash
  specflow status
  ```
  Feature should show phase: `② plan`

### 2. Create Task Breakdown

- [ ] **Run the tasks phase**:
  ```bash
  specflow tasks <feature-id>
  ```

  Example: `specflow tasks F-001`

  This creates:
  ```
  .specify/<feature-id>/
  ├── spec.md      ← (from step 1)
  ├── plan.md      ← (from step 2)
  └── tasks.md     ← Implementation units
  ```

### 3. Validate Task Quality

- [ ] **Check tasks follow TDD pattern** (`docs/TDD-EVALS.md`):

| Criterion | Check |
|-----------|-------|
| Each task is independently testable | |
| Tasks have clear acceptance criteria | |
| Dependencies are explicit | |
| Tasks are ordered by dependency | |

- [ ] **Check task granularity**:
  - [ ] No task is too large (can be done in one session)
  - [ ] No task is too small (trivial changes grouped)
  - [ ] Each task has a verifiable outcome

### 4. Verify Phase Complete

- [ ] **Check feature status**:
  ```bash
  specflow status
  ```
  Feature should now show phase: `③ tasks`

## Output

- `.specify/<feature-id>/tasks.md` with implementation breakdown
- Feature phase updated to "tasks"

## Next

→ Proceed to Step 4 (IMPLEMENT) to begin TDD implementation.
