# Step 3: Verify Progress

**Phase**: Verification | **Purpose**: Run tests, check task completion, decide next action

---

## Context
- **Playbook:** SpecFlow 3: Implement
- **Agent:** {{AGENT_NAME}}
- **Project:** {{AGENT_PATH}}
- **Auto Run Folder:** {{AUTORUN_FOLDER}}
- **Date:** {{DATE}}
- **Loop:** {{LOOP_NUMBER}}

## Objective

Verify implementation quality and determine whether to continue with more tasks or complete the feature.

## Tasks

### Task 1: Run Full Test Suite

- [ ] **Execute all tests**:
  ```bash
  bun test
  ```

- [ ] **Record results**:
  - Total tests
  - Passed
  - Failed
  - Duration

  **If tests fail:** Note failures, may need to return to Step 2

### Task 2: Check Task Completion Status

- [ ] **Count completed vs total tasks**:
  ```bash
  TOTAL=$(grep -cE "^\s*- \[.\]" .specflow/specs/<feature-slug>/tasks.md)
  DONE=$(grep -cE "^\s*- \[x\]" .specflow/specs/<feature-slug>/tasks.md)
  echo "Tasks: $DONE of $TOTAL complete"
  ```

### Task 3: Verify Determinism (3 runs)

- [ ] **Run tests 3 times to confirm determinism**:
  ```bash
  bun test && bun test && bun test
  ```

  All 3 runs should have identical results.

### Task 4: Decision Point

- [ ] **Evaluate and act**:

  | Condition | Action |
  |-----------|--------|
  | Tests failing | → Loop to Step 2 (fix) |
  | Tasks incomplete | → Loop to Step 2 (next task) |
  | All tasks done, tests pass | → Proceed to Step 4 (complete) |

  **If looping to Step 2:**
  - Update `{{AUTORUN_FOLDER}}/LOOP_{{LOOP_NUMBER}}_PROGRESS.md` with current state
  - Continue with next task

  **If proceeding to Step 4:**
  - All tasks complete
  - All tests passing
  - Ready to mark feature done

### Task 5: Update Progress File

- [ ] **Write/Update loop progress**:

  Create/update `{{AUTORUN_FOLDER}}/LOOP_{{LOOP_NUMBER}}_PROGRESS.md`:
  ```markdown
  # Implementation Progress - Loop {{LOOP_NUMBER}}

  **Feature:** <feature-id> - <feature-name>
  **Date:** {{DATE}}

  ## Test Results
  - **Total:** X tests
  - **Passed:** X
  - **Failed:** 0
  - **Deterministic:** Yes (3 identical runs)

  ## Task Progress
  - **Completed:** X of Y tasks
  - **Remaining:** [list of remaining task IDs]

  ## Decision
  [CONTINUE to next task / COMPLETE feature]
  ```

---

**Loop Control:**
- If tasks remain → Loop to Step 2
- If feature complete → Proceed to Step 4
