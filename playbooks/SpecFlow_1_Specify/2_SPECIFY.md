# Step 2: Create Specification

**Phase**: Specification | **Purpose**: Generate spec.md via interview

---

## Context
- **Playbook:** SpecFlow 1: Specify
- **Agent:** {{AGENT_NAME}}
- **Project:** {{AGENT_PATH}}
- **Auto Run Folder:** {{AUTORUN_FOLDER}}
- **Date:** {{DATE}}

## Prerequisites

- `{{AUTORUN_FOLDER}}/CURRENT_FEATURE.md` exists with feature ID
- Feature phase is `none`

## Tasks

### Task 1: Check Exit Condition

- [ ] **Read current feature state**:
  ```bash
  cat {{AUTORUN_FOLDER}}/CURRENT_FEATURE.md
  ```

  **If contains `ALL_SPECS_COMPLETE`:** Skip remaining tasks, proceed to Step 3.

### Task 2: Run Specification Interview

- [ ] **Execute specflow specify**:
  ```bash
  specflow specify <feature-id>
  ```

  This runs the 8-phase interview protocol:
  1. Problem Statement
  2. User Stories
  3. Acceptance Criteria
  4. Technical Constraints
  5. Dependencies
  6. Non-Functional Requirements
  7. Edge Cases
  8. Success Metrics

### Task 3: Verify Spec Created

- [ ] **Confirm spec.md exists**:
  ```bash
  ls -la .specflow/specs/<feature-slug>/spec.md
  ```

- [ ] **Check spec quality** (brief review):
  ```bash
  head -50 .specflow/specs/<feature-slug>/spec.md
  ```

  Verify it contains:
  - [ ] Problem statement
  - [ ] User stories
  - [ ] Acceptance criteria
  - [ ] Technical approach overview

### Task 4: Update Feature Phase

- [ ] **Mark feature as specified**:
  ```bash
  specflow phase <feature-id> specify
  ```

### Task 5: Log Completion

- [ ] **Append to playbook log**:

  Write to `{{AUTORUN_FOLDER}}/SPECIFY_LOG.md` (append):
  ```markdown
  ## {{DATE}} - <feature-id>

  - **Feature:** <feature-name>
  - **Spec Path:** .specflow/specs/<feature-slug>/spec.md
  - **Status:** Specification complete
  ```

---

**Next:** Step 3 (NEXT_OR_EXIT) - Check for more features or exit
