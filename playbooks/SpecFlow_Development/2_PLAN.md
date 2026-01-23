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
