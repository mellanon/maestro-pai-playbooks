# Step 1: Select Feature Needing Plan

**Phase**: Selection | **Purpose**: Find next feature with spec but no plan

---

## Context
- **Playbook:** SpecFlow 2: Plan
- **Agent:** {{AGENT_NAME}}
- **Project:** {{AGENT_PATH}}
- **Auto Run Folder:** {{AUTORUN_FOLDER}}
- **Date:** {{DATE}}

## Objective

Find the next feature that has a specification but needs planning (phase = `specify`).

## Tasks

### Task 1: List Features Needing Plans

- [ ] **Query for features with specs but no plans (by ID order)**:
  ```bash
  # Get features with phase "specify" that need planning
  specflow status --json | jq -r '
    .features[]
    | select(.status == "pending" and .phase == "specify")
    | "\(.id)\t\(.name)"
  ' | sort -t'-' -k2 -n
  ```

  **If features found:** Continue to Task 2

  **If NO features need plans:**
  - Write to `{{AUTORUN_FOLDER}}/CURRENT_FEATURE.md`:
    ```markdown
    # Current Feature
    ALL_PLANS_COMPLETE

    All features have plans and task breakdowns. Run SpecFlow_3_Implement next.
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
  - **Phase:** specify
  - **Action:** Create plan and task breakdown
  - **Started:** {{DATE}}
  ```

### Task 3: Load Feature Context

- [ ] **Read the existing spec**:
  ```bash
  cat .specflow/specs/<feature-slug>/spec.md
  ```

  This provides context for planning.

---

**Next:** Step 2 (PLAN) - Create the architecture plan
