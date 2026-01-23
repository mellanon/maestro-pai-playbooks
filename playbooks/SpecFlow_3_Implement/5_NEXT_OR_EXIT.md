# Step 5: Next Feature or Exit

**Phase**: Loop Control | **Purpose**: Continue to next feature or exit playbook

---

## Context
- **Playbook:** SpecFlow 3: Implement
- **Agent:** {{AGENT_NAME}}
- **Project:** {{AGENT_PATH}}
- **Auto Run Folder:** {{AUTORUN_FOLDER}}
- **Date:** {{DATE}}

## Objective

Check if more features need implementation. If yes, loop. If no, exit.

## Tasks

### Task 1: Check Current State

- [ ] **Read current feature file**:
  ```bash
  cat {{AUTORUN_FOLDER}}/CURRENT_FEATURE.md
  ```

  **If contains `ALL_FEATURES_IMPLEMENTED`:**
  - Log final summary
  - EXIT playbook (do not loop)

### Task 2: Archive Completed Feature

- [ ] **Move completed feature to archive**:
  ```bash
  mkdir -p {{AUTORUN_FOLDER}}/COMPLETED_FEATURES
  mv {{AUTORUN_FOLDER}}/CURRENT_FEATURE.md {{AUTORUN_FOLDER}}/COMPLETED_FEATURES/<feature-id>_{{DATE}}.md
  ```

### Task 3: Count Remaining Features

- [ ] **Query for features still needing implementation**:
  ```bash
  REMAINING=$(specflow status --json | jq '[.features[] | select(.status == "pending" and .phase == "tasks")] | length')
  echo "Features still needing implementation: $REMAINING"
  ```

### Task 4: Decision Point

- [ ] **Evaluate and act**:

  **If REMAINING > 0:**
  - Log: "X features remaining. Continuing to next."
  - **LOOP** back to Step 1

  **If REMAINING = 0:**
  - Write to `{{AUTORUN_FOLDER}}/CURRENT_FEATURE.md`:
    ```markdown
    # Current Feature
    ALL_FEATURES_IMPLEMENTED

    All features have been implemented!
    Date: {{DATE}}

    ## Summary
    - Total features implemented: [count]
    - Total tests: [count]
    - Total files created: [count]

    ## Next Steps
    1. Review all implemented code
    2. Run full test suite
    3. Create PR for review
    4. Optionally run PR_Review playbook
    ```
  - **EXIT** playbook

### Task 5: Final Summary (Exit Only)

- [ ] **If exiting, gather implementation stats**:

  ```bash
  echo "=== IMPLEMENTATION COMPLETE ==="
  echo ""
  echo "Features implemented:"
  specflow status --json | jq -r '.features[] | select(.status == "complete") | "  ✓ \(.id): \(.name)"'
  echo ""
  echo "Test results:"
  # Run test suite and capture results
  npm test 2>&1 | tail -20 || bun test 2>&1 | tail -20 || echo "Run tests manually"
  echo ""
  echo "Files changed:"
  git diff --stat HEAD~$(specflow status --json | jq '[.features[] | select(.status == "complete")] | length') 2>/dev/null || git status --short
  ```

  Write to `{{AUTORUN_FOLDER}}/IMPLEMENTATION_SUMMARY.md`:
  ```markdown
  # Implementation Phase Complete

  **Date:** {{DATE}}
  **Playbook:** SpecFlow 3: Implement

  ## Features Implemented

  [INSERT FROM COMMAND OUTPUT]

  ## Statistics

  [INSERT TEST AND FILE STATS]

  ## Review Checklist

  Before creating PR:
  - [ ] Review all implemented code
  - [ ] Run full test suite: `npm test` or `bun test`
  - [ ] Check for code style issues
  - [ ] Verify no secrets committed
  - [ ] Update CHANGELOG if needed

  ## Create PR

  ```bash
  git add -A && git commit -m "feat: implement features [LIST]

  Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>"
  git push -u origin $(git branch --show-current)
  gh pr create --fill
  ```
  ```

- [ ] **Print summary to console for immediate visibility**:
  ```bash
  echo ""
  echo "============================================"
  echo "  IMPLEMENTATION COMPLETE!"
  echo "============================================"
  echo ""
  echo "Features implemented:"
  specflow status --json | jq -r '.features[] | select(.status == "complete") | "  ✓ \(.id): \(.name)"'
  echo ""
  echo "Quick commands:"
  echo "  npm test          # Run tests"
  echo "  git diff --stat   # See changes"
  echo "  gh pr create      # Create PR"
  echo ""
  echo "============================================"
  ```

---

## Loop Control

| Condition | Action |
|-----------|--------|
| Features need implementation | Loop to Step 1 |
| All features implemented | Exit playbook |

**This playbook does NOT loop eternally.** It exits when all features are implemented.

---

**After Exit:** Human reviews code and creates PR
