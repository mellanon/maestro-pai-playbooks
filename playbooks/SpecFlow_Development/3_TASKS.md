# Step 3: TASKS - Break Down Implementation

**Phase**: Task Planning | **Gate**: Human Approval Required

---

## Context
- **Playbook:** SpecFlow Development
- **Agent:** {{AGENT_NAME}}
- **Project:** {{AGENT_PATH}}
- **Auto Run Folder:** {{AUTORUN_FOLDER}}
- **Loop:** {{LOOP_NUMBER}}

## Objective

Break the feature into reviewable implementation units with dependencies.

## Prerequisites

- Technical plan approved (plan.md exists and human approved)
- Feature phase is "plan"

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
  specflow/<feature-id>/
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

- `specflow/<feature-id>/tasks.md` with implementation breakdown
- Feature phase updated to "tasks"

## Human Gate

**STOP** - Present task breakdown to human for review before implementation.

Show:
1. `tasks.md` breakdown
2. Task dependencies
3. Estimated scope

**Only proceed to Step 4 after human approval.**
