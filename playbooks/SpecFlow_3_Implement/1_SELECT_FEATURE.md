# Step 1: Select Feature Ready for Implementation

**Phase**: Selection | **Purpose**: Find next feature with tasks ready to implement

---

## Context
- **Playbook:** SpecFlow 3: Implement
- **Agent:** {{AGENT_NAME}}
- **Project:** {{AGENT_PATH}}
- **Auto Run Folder:** {{AUTORUN_FOLDER}}
- **Date:** {{DATE}}

## Objective

Find the next feature that has tasks and is ready for implementation (phase = `tasks`).

## Tasks

### Task 1: List Features Ready for Implementation

- [ ] **Query for features with tasks (by ID order)**:
  ```bash
  # Get features with phase "tasks" ready for implementation
  specflow status --json | jq -r '
    .features[]
    | select(.status == "pending" and .phase == "tasks")
    | "\(.id)\t\(.name)"
  ' | sort -t'-' -k2 -n
  ```

  **If features found:** Continue to Task 2

  **If NO features ready:**
  - Write to `{{AUTORUN_FOLDER}}/CURRENT_FEATURE.md`:
    ```markdown
    # Current Feature
    ALL_FEATURES_IMPLEMENTED

    All features have been implemented. Playbook complete.
    Date: {{DATE}}
    ```
  - This signals the playbook to exit

### Task 2: Select First Feature

- [ ] **Record the first feature from the sorted list**:

  Write to `{{AUTORUN_FOLDER}}/CURRENT_FEATURE.md`:
  ```markdown
  # Current Feature

  - **Feature ID:** <feature-id>
  - **Feature Name:** <feature-name>
  - **Phase:** tasks
  - **Action:** Implement via TDD
  - **Started:** {{DATE}}
  - **Tasks Completed:** 0 of X
  ```

### Task 3: Load Feature Context

- [ ] **Read the spec, plan, and tasks**:
  ```bash
  cat .specflow/specs/<feature-slug>/spec.md
  cat .specflow/specs/<feature-slug>/plan.md
  cat .specflow/specs/<feature-slug>/tasks.md
  ```

- [ ] **Identify first incomplete task**:
  Look for tasks marked `[ ]` (unchecked) in tasks.md

---

**Next:** Step 2 (IMPLEMENT) - Execute TDD cycle for next task
