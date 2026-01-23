# Step 2: Create Architecture Plan

**Phase**: Planning | **Purpose**: Generate plan.md with technical design

---

## Context
- **Playbook:** SpecFlow 2: Plan
- **Agent:** {{AGENT_NAME}}
- **Project:** {{AGENT_PATH}}
- **Auto Run Folder:** {{AUTORUN_FOLDER}}
- **Date:** {{DATE}}

## Prerequisites

- `{{AUTORUN_FOLDER}}/CURRENT_FEATURE.md` exists with feature ID
- Feature has spec.md (phase = `specify`)

## Tasks

### Task 1: Check Exit Condition

- [ ] **Read current feature state**:
  ```bash
  cat {{AUTORUN_FOLDER}}/CURRENT_FEATURE.md
  ```

  **If contains `ALL_PLANS_COMPLETE`:** Skip remaining tasks, proceed to Step 4.

### Task 2: Run Planning

- [ ] **Execute specflow plan**:
  ```bash
  specflow plan <feature-id>
  ```

  This generates:
  - Data models and schemas
  - API design (if applicable)
  - Component architecture
  - Error handling strategy
  - Failure modes analysis
  - Dependencies identification
  - Testing strategy overview

### Task 3: Verify Plan Created

- [ ] **Confirm plan.md exists**:
  ```bash
  ls -la .specflow/specs/<feature-slug>/plan.md
  ```

- [ ] **Check plan quality** (brief review):
  ```bash
  head -100 .specflow/specs/<feature-slug>/plan.md
  ```

  Verify it contains:
  - [ ] Architecture overview
  - [ ] Data models
  - [ ] Error handling approach
  - [ ] Testing strategy

### Task 4: Update Feature Phase

- [ ] **Mark feature as planned**:
  ```bash
  specflow phase <feature-id> plan
  ```

---

**Next:** Step 3 (TASKS) - Create implementation task breakdown
