# Step 4: Complete Feature

**Phase**: Completion | **Purpose**: Mark feature done, generate docs

---

## Context
- **Playbook:** SpecFlow 3: Implement
- **Agent:** {{AGENT_NAME}}
- **Project:** {{AGENT_PATH}}
- **Auto Run Folder:** {{AUTORUN_FOLDER}}
- **Date:** {{DATE}}

## Prerequisites

- All tasks in tasks.md are marked `[x]`
- All tests passing
- Step 3 verification complete

## Tasks

### Task 1: Final Validation

- [ ] **Run specflow validate**:
  ```bash
  specflow validate <feature-id>
  ```

  Confirm:
  - [ ] spec.md exists and valid
  - [ ] plan.md exists and valid
  - [ ] tasks.md exists and all tasks complete
  - [ ] Tests pass

### Task 2: Mark Feature Complete

- [ ] **Execute specflow complete**:
  ```bash
  specflow complete <feature-id>
  ```

  This:
  - Validates all SpecFlow phases
  - Runs Doctorow Gate checklist
  - Updates feature status to `complete`
  - Updates phase to `implement` (done)

### Task 3: Generate Documentation

- [ ] **Create/update docs.md** (if specflow doesn't auto-generate):

  Write to `.specflow/specs/<feature-slug>/docs.md`:
  ```markdown
  # <Feature Name> - Documentation

  ## Overview
  [Summary from spec.md]

  ## Usage
  [How to use this feature]

  ## API Reference
  [If applicable - functions, types, etc.]

  ## Examples
  [Code examples]

  ## Testing
  - Test file: [path]
  - Tests: X passing
  ```

### Task 4: Log Completion

- [ ] **Append to playbook log**:

  Write to `{{AUTORUN_FOLDER}}/IMPLEMENT_LOG.md` (append):
  ```markdown
  ## {{DATE}} - <feature-id> COMPLETE

  - **Feature:** <feature-name>
  - **Tasks Completed:** X of X
  - **Tests:** X passing
  - **Duration:** [time from start]
  - **Files Created/Modified:**
    - [list of files]
  ```

### Task 5: Update Feature State

- [ ] **Update CURRENT_FEATURE.md**:
  ```markdown
  # Current Feature

  - **Feature ID:** <feature-id>
  - **Feature Name:** <feature-name>
  - **Phase:** complete
  - **Status:** DONE
  - **Completed:** {{DATE}}
  ```

---

**Next:** Step 5 (NEXT_OR_EXIT) - Check for more features or exit
