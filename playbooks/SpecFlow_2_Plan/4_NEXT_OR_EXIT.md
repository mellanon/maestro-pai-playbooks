# Step 4: Next Feature or Exit

**Phase**: Loop Control | **Purpose**: Continue to next feature or exit playbook

---

## Context
- **Playbook:** SpecFlow 2: Plan
- **Agent:** {{AGENT_NAME}}
- **Project:** {{AGENT_PATH}}
- **Auto Run Folder:** {{AUTORUN_FOLDER}}
- **Date:** {{DATE}}

## Objective

Check if more features need plans. If yes, loop. If no, exit.

## Tasks

### Task 1: Check Current State

- [ ] **Read current feature file**:
  ```bash
  cat {{AUTORUN_FOLDER}}/CURRENT_FEATURE.md
  ```

  **If contains `ALL_PLANS_COMPLETE`:**
  - Log final summary
  - EXIT playbook (do not loop)

### Task 2: Count Remaining Features

- [ ] **Query for features still needing plans**:
  ```bash
  REMAINING=$(specflow status --json | jq '[.features[] | select(.status == "pending" and .phase == "specify")] | length')
  echo "Features still needing plans: $REMAINING"
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
    ALL_PLANS_COMPLETE

    All features have plans and task breakdowns.
    Total planned: [count]
    Date: {{DATE}}

    ## Next Steps
    1. Review all plan.md files in .specflow/specs/*/
    2. Review all tasks.md files
    3. Adjust task breakdown if needed
    4. Run SpecFlow_3_Implement playbook
    ```
  - **EXIT** playbook

### Task 4: Final Summary (Exit Only)

- [ ] **If exiting, list all plans and create summary**:

  First, get the actual plan files:
  ```bash
  echo "=== PLANS READY FOR REVIEW ==="
  specflow status --json | jq -r '.features[] | select(.phase == "tasks") | "- \(.id): \(.name)\n  Plan: .specflow/specs/\(.id | ascii_downcase)-*/plan.md\n  Tasks: .specflow/specs/\(.id | ascii_downcase)-*/tasks.md"'
  echo ""
  echo "=== TASK COUNTS ==="
  for dir in .specflow/specs/*/; do
    if [ -f "$dir/tasks.md" ]; then
      count=$(grep -c "^- \[ \]" "$dir/tasks.md" 2>/dev/null || echo "0")
      echo "$(basename $dir): $count tasks"
    fi
  done
  ```

  Then write to `{{AUTORUN_FOLDER}}/PLANNING_SUMMARY.md`:
  ```markdown
  # Planning Phase Complete

  **Date:** {{DATE}}
  **Playbook:** SpecFlow 2: Plan

  ## Features Planned

  [INSERT TABLE FROM COMMAND OUTPUT ABOVE]

  ## Quick Review

  Open all plans:
  ```bash
  # VS Code
  code .specflow/specs/*/plan.md .specflow/specs/*/tasks.md

  # Cursor
  cursor .specflow/specs/*/plan.md .specflow/specs/*/tasks.md
  ```

  ## Review Checklist

  Before running SpecFlow_3_Implement:
  - [ ] Read each plan.md - verify architecture
  - [ ] Read each tasks.md - verify breakdown
  - [ ] Check task dependencies are correct
  - [ ] Confirm parallel groups make sense
  - [ ] Verify no missing tasks

  ## Next Playbook

  When ready: Run **SpecFlow_3_Implement**
  ```

- [ ] **Print summary to console for immediate visibility**:
  ```bash
  echo ""
  echo "============================================"
  echo "  PLANNING PHASE COMPLETE"
  echo "============================================"
  echo ""
  echo "Plans ready for your review:"
  specflow status --json | jq -r '.features[] | select(.phase == "tasks") | "  ✓ \(.id): \(.name)"'
  echo ""
  echo "Task breakdown:"
  for dir in .specflow/specs/*/; do
    if [ -f "$dir/tasks.md" ]; then
      count=$(grep -c "^- \[ \]" "$dir/tasks.md" 2>/dev/null || echo "0")
      echo "    $(basename $dir): $count tasks"
    fi
  done
  echo ""
  echo "Open all plans:"
  echo "  code .specflow/specs/*/plan.md .specflow/specs/*/tasks.md"
  echo ""
  echo "When ready, run: SpecFlow_3_Implement"
  echo "============================================"
  ```

---

## Loop Control

| Condition | Action |
|-----------|--------|
| Features need plans | Loop to Step 1 |
| All features planned | Exit playbook |

**This playbook does NOT loop eternally.** It exits when all features have plans and tasks.

---

**After Exit:** Human reviews plans, then runs SpecFlow_3_Implement
