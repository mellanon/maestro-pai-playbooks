# Step 2: PLAN - Create Technical Architecture

**Phase**: Planning | **Gate**: Automated

---

## Context
- **Playbook:** SpecFlow Development
- **State Directory:** `.maestro/` (in project root)

## Objective

Create a technical architecture plan including data models, failure modes, and design decisions.

## Prerequisites

- Specification approved (spec.md exists and human approved)
- Feature phase is "specify"

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

  # Step 2 only runs when phase is "specify"
  PHASE=$(specflow status "$FEATURE_ID" --json 2>/dev/null | jq -r '.phase' 2>/dev/null || echo "none")

  if [[ "$PHASE" != "specify" ]]; then
    echo "PHASE_GUARD: SKIP - Feature not at specify phase (current: $PHASE)"
    exit 0
  fi

  echo "PHASE_GUARD: PROCEED - $FEATURE_ID ready for planning (phase: $PHASE)"
  ```

If the phase guard exits with "SKIP", this document is a NO-OP. Proceed to next document.

## Instructions

### 1. Check Current Status

- [ ] **Verify spec exists**:
  ```bash
  specflow status
  ```
  Feature should show phase: `① specify`

### 2. Create Technical Plan

- [ ] **Run the plan phase**:
  ```bash
  specflow plan <feature-id>
  ```

  Example: `specflow plan F-001`

  This creates:
  ```
  .specify/<feature-id>/
  ├── spec.md      ← (from step 1)
  └── plan.md      ← Architecture decisions
  ```

### 3. Validate Plan Quality

- [ ] **Check plan covers**:

| Section | Included? |
|---------|-----------|
| Data models | |
| API/Interface design | |
| Error handling strategy | |
| Failure modes | |
| Dependencies | |
| Testing strategy | |

- [ ] **Check design against PAI principles**:

| Principle | Applied? |
|-----------|----------|
| Code before prompts | |
| Deterministic where possible | |
| CLI-first interfaces | |

### 4. Verify Phase Complete

- [ ] **Check feature status**:
  ```bash
  specflow status
  ```
  Feature should now show phase: `② plan`

## Output

- `.specify/<feature-id>/plan.md` with technical architecture
- Feature phase updated to "plan"

## Next

→ Proceed to Step 3 (TASKS) to break down implementation units.
