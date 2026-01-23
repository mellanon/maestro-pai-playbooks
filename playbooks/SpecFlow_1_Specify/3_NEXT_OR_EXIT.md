# Step 3: Next Feature or Exit

**Phase**: Loop Control | **Purpose**: Continue to next feature or exit playbook

---

## Context
- **Playbook:** SpecFlow 1: Specify
- **Agent:** {{AGENT_NAME}}
- **Project:** {{AGENT_PATH}}
- **Auto Run Folder:** {{AUTORUN_FOLDER}}
- **Date:** {{DATE}}

## Objective

Check if more features need specifications. If yes, loop. If no, exit.

## Tasks

### Task 1: Check Current State

- [ ] **Read current feature file**:
  ```bash
  cat {{AUTORUN_FOLDER}}/CURRENT_FEATURE.md
  ```

  **If contains `ALL_SPECS_COMPLETE`:**
  - Log final summary
  - EXIT playbook (do not loop)

### Task 2: Count Remaining Features

- [ ] **Query for features still needing specs**:
  ```bash
  REMAINING=$(specflow status --json | jq '[.features[] | select(.status == "pending" and (.phase == "none" or .phase == null))] | length')
  echo "Features still needing specs: $REMAINING"
  ```

### Task 3: Decision Point

- [ ] **Evaluate and act**:

  **If REMAINING > 0:**
  - Clear `CURRENT_FEATURE.md` for next iteration
  - Log: "X features remaining. Continuing to next."
  - **LOOP** back to Step 1

  **If REMAINING = 0:**
  - Write to `{{AUTORUN_FOLDER}}/CURRENT_FEATURE.md`:
    ```markdown
    # Current Feature
    ALL_SPECS_COMPLETE

    All features have specifications.
    Total specified: [count]
    Date: {{DATE}}

    ## Next Steps
    1. Review all spec.md files in .specflow/specs/*/
    2. Edit specifications as needed
    3. Run SpecFlow_2_Plan playbook
    ```
  - **EXIT** playbook

### Task 4: Final Summary (Exit Only)

- [ ] **If exiting, list all specs and create summary**:

  First, get the actual spec files:
  ```bash
  echo "=== SPECS READY FOR REVIEW ==="
  specflow status --json | jq -r '.features[] | select(.phase == "specify") | "- \(.id): \(.name)\n  Path: .specflow/specs/\(.id | ascii_downcase)-*/spec.md"'
  echo ""
  echo "=== QUICK REVIEW COMMANDS ==="
  specflow status --json | jq -r '.features[] | select(.phase == "specify") | "cat .specflow/specs/\(.id | ascii_downcase)-*/spec.md"'
  ```

  Then write to `{{AUTORUN_FOLDER}}/SPECIFICATION_SUMMARY.md`:
  ```markdown
  # Specification Phase Complete

  **Date:** {{DATE}}
  **Playbook:** SpecFlow 1: Specify

  ## Features Specified

  [INSERT TABLE FROM COMMAND OUTPUT ABOVE]

  ## Quick Review

  Run these commands to read each spec:
  [INSERT COMMANDS FROM OUTPUT ABOVE]

  Or open all specs in your editor:
  ```bash
  # VS Code
  code .specflow/specs/*/spec.md

  # Cursor
  cursor .specflow/specs/*/spec.md
  ```

  ## Review Checklist

  Before running SpecFlow_2_Plan:
  - [ ] Read each spec.md
  - [ ] Verify requirements are complete
  - [ ] Check acceptance criteria are testable
  - [ ] Confirm dependencies are identified

  ## Next Playbook

  When ready: Run **SpecFlow_2_Plan**
  ```

- [ ] **Print summary to console for immediate visibility**:
  ```bash
  echo ""
  echo "============================================"
  echo "  SPECIFICATION PHASE COMPLETE"
  echo "============================================"
  echo ""
  echo "Specs ready for your review:"
  specflow status --json | jq -r '.features[] | select(.phase == "specify") | "  ✓ \(.id): \(.name)"'
  echo ""
  echo "Open all specs:"
  echo "  code .specflow/specs/*/spec.md"
  echo ""
  echo "When ready, run: SpecFlow_2_Plan"
  echo "============================================"
  ```

---

## Loop Control

| Condition | Action |
|-----------|--------|
| Features need specs | Loop to Step 1 |
| All features specified | Exit playbook |

**This playbook does NOT loop eternally.** It exits when all features have specifications.

---

**After Exit:** Human reviews specs, then runs SpecFlow_2_Plan
