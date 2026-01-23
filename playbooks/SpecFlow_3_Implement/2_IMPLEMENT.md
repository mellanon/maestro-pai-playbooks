# Step 2: Implement Next Task (TDD)

**Phase**: Implementation | **Purpose**: Execute RED-GREEN-BLUE cycle

---

## Context
- **Playbook:** SpecFlow 3: Implement
- **Agent:** {{AGENT_NAME}}
- **Project:** {{AGENT_PATH}}
- **Auto Run Folder:** {{AUTORUN_FOLDER}}
- **Date:** {{DATE}}
- **Loop:** {{LOOP_NUMBER}}

## Prerequisites

- `{{AUTORUN_FOLDER}}/CURRENT_FEATURE.md` exists with feature ID
- Feature has tasks.md with incomplete tasks

## Tasks

### Task 1: Check Exit Condition

- [ ] **Read current feature state**:
  ```bash
  cat {{AUTORUN_FOLDER}}/CURRENT_FEATURE.md
  ```

  **If contains `ALL_FEATURES_IMPLEMENTED`:** Skip remaining tasks, proceed to Step 5.

### Task 2: Get Next Incomplete Task

- [ ] **Read tasks.md and find first unchecked task**:
  ```bash
  cat .specflow/specs/<feature-slug>/tasks.md | grep -E "^\s*- \[ \]" | head -1
  ```

  **If no unchecked tasks:** This feature is complete, proceed to Step 4.

- [ ] **Load task context**:
  - Task ID (e.g., T-1.1)
  - Task description
  - Acceptance criteria
  - Dependencies (must be complete first)

### Task 3: RED - Write Failing Test

- [ ] **Create test file** (if doesn't exist):
  ```bash
  # Location based on project structure
  touch src/<feature>/<component>.test.ts
  ```

- [ ] **Write test that defines expected behavior**:
  - Test should fail initially (no implementation)
  - Test should be specific to this task
  - Test should be deterministic

- [ ] **Run test to confirm failure**:
  ```bash
  bun test <test-file>
  ```

  Expected: Test FAILS (RED)

### Task 4: GREEN - Implement Minimal Code

- [ ] **Write minimal implementation** that makes test pass:
  - Only enough code to pass the test
  - Don't add extra features
  - Don't optimize yet

- [ ] **Run test to confirm pass**:
  ```bash
  bun test <test-file>
  ```

  Expected: Test PASSES (GREEN)

### Task 5: BLUE - Refactor

- [ ] **Clean up code** (if needed):
  - Remove duplication
  - Improve naming
  - Extract helpers if valuable

- [ ] **Run ALL tests to ensure nothing broke**:
  ```bash
  bun test
  ```

  Expected: ALL tests pass

### Task 6: Mark Task Complete

- [ ] **Update tasks.md** - mark this task as done:
  Change `- [ ] T-X.Y: ...` to `- [x] T-X.Y: ...`

- [ ] **Update progress in CURRENT_FEATURE.md**:
  ```markdown
  - **Tasks Completed:** X of Y
  - **Last Task:** T-X.Y - [description]
  - **Last Updated:** {{DATE}}
  ```

---

**Next:** Step 3 (VERIFY) - Run full test suite and check progress
