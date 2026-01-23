# Step 3: Create Task Breakdown

**Phase**: Task Definition | **Purpose**: Generate tasks.md with implementation units

---

## Context
- **Playbook:** SpecFlow 2: Plan
- **Agent:** {{AGENT_NAME}}
- **Project:** {{AGENT_PATH}}
- **Auto Run Folder:** {{AUTORUN_FOLDER}}
- **Date:** {{DATE}}

## Prerequisites

- Feature has plan.md (phase = `plan`)

## Tasks

### Task 1: Check Exit Condition

- [ ] **Read current feature state**:
  ```bash
  cat {{AUTORUN_FOLDER}}/CURRENT_FEATURE.md
  ```

  **If contains `ALL_PLANS_COMPLETE`:** Skip remaining tasks, proceed to Step 4.

### Task 2: Run Task Generation

- [ ] **Execute specflow tasks**:
  ```bash
  specflow tasks <feature-id>
  ```

  This generates:
  - Numbered task list (T-1.1, T-1.2, etc.)
  - Task groups for parallelization
  - Dependencies between tasks
  - Acceptance criteria per task
  - Estimated complexity

### Task 3: Verify Tasks Created

- [ ] **Confirm tasks.md exists**:
  ```bash
  ls -la .specflow/specs/<feature-slug>/tasks.md
  ```

- [ ] **Check task quality** (brief review):
  ```bash
  cat .specflow/specs/<feature-slug>/tasks.md
  ```

  Verify it contains:
  - [ ] Numbered tasks (T-X.Y format)
  - [ ] Clear acceptance criteria
  - [ ] Task dependencies noted
  - [ ] Parallel groups identified

### Task 4: Update Feature Phase

- [ ] **Mark feature as task-ready**:
  ```bash
  specflow phase <feature-id> tasks
  ```

### Task 5: Log Completion

- [ ] **Append to playbook log**:

  Write to `{{AUTORUN_FOLDER}}/PLAN_LOG.md` (append):
  ```markdown
  ## {{DATE}} - <feature-id>

  - **Feature:** <feature-name>
  - **Plan Path:** .specflow/specs/<feature-slug>/plan.md
  - **Tasks Path:** .specflow/specs/<feature-slug>/tasks.md
  - **Task Count:** [X tasks in Y groups]
  - **Status:** Planning complete
  ```

---

**Next:** Step 4 (NEXT_OR_EXIT) - Check for more features or exit
