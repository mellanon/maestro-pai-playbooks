# Step 1: Select Feature Needing Specification

**Phase**: Selection | **Purpose**: Find next feature without a spec

---

## Context
- **Playbook:** SpecFlow 1: Specify
- **Agent:** {{AGENT_NAME}}
- **Project:** {{AGENT_PATH}}
- **Auto Run Folder:** {{AUTORUN_FOLDER}}
- **Date:** {{DATE}}

## Objective

Find the next feature that needs a specification (phase = `none` or no spec.md exists).

## Tasks

### Task 1: List Features Needing Specification

- [ ] **Query for features needing specs (by ID order)**:
  ```bash
  # Get features with phase "none" or "specify" that need spec work
  specflow status --json | jq -r '
    .features[]
    | select(.status == "pending" and (.phase == "none" or .phase == null))
    | "\(.id)\t\(.name)"
  ' | sort -t'-' -k2 -n
  ```

  **If features found:** Continue to Task 2

  **If NO features need specs:**
  - Write to `{{AUTORUN_FOLDER}}/CURRENT_FEATURE.md`:
    ```markdown
    # Current Feature
    ALL_SPECS_COMPLETE

    All features have specifications. Run SpecFlow_2_Plan next.
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
  - **Phase:** none
  - **Action:** Create specification
  - **Started:** {{DATE}}
  ```

### Task 3: Load Feature Context

- [ ] **Get feature details**:
  ```bash
  specflow status <feature-id> --json
  ```

  Record:
  - Feature description
  - Any existing notes or requirements
  - Related features (dependencies)

---

**Next:** Step 2 (SPECIFY) - Run the specification interview
