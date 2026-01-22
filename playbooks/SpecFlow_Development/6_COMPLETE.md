# Step 6: COMPLETE - Validation and Finalization

**Phase**: Completion | **Gate**: Human Approval Required

---

## Context
- **Playbook:** SpecFlow Development
- **Agent:** {{AGENT_NAME}}
- **Project:** {{AGENT_PATH}}
- **Auto Run Folder:** {{AUTORUN_FOLDER}}
- **Loop:** {{LOOP_NUMBER}}

## Objective

Validate all phases complete and mark the feature as done.

## Prerequisites

- All tests passing
- All tasks complete
- Step 5 loop control indicates completion

## Instructions

### 1. Final Validation

- [ ] **Run comprehensive validation**:
  ```bash
  specflow validate <feature-id>
  ```

  This checks:
  - ✓ spec.md exists and valid
  - ✓ plan.md exists and valid
  - ✓ tasks.md exists and all tasks complete
  - ✓ Tests referenced in tasks pass

### 2. Run Final Tests

- [ ] **Execute full test suite**:
  ```bash
  bun test
  ```

- [ ] **Run type checking** (if applicable):
  ```bash
  bun run typecheck
  ```

### 3. Mark Feature Complete

- [ ] **Complete the feature**:
  ```bash
  specflow complete <feature-id>
  ```

  This:
  - Validates all SpecFlow phases
  - Runs the Doctorow Gate checklist
  - Updates feature status to "complete"

### 4. Review Doctorow Gate

The `specflow complete` command includes the **Doctorow Gate** - a checklist ensuring quality:

| Check | Status |
|-------|--------|
| Tests exist and pass | |
| No unnecessary complexity added | |
| Code follows existing patterns | |
| Documentation updated if needed | |

### 5. Verify Completion

- [ ] **Check final status**:
  ```bash
  specflow status
  ```

  Feature should show: `● complete`

### 6. Prepare for Next Feature

- [ ] **Check if more features pending**:
  ```bash
  specflow next
  ```

  If more features:
  - Document completion in `{{AUTORUN_FOLDER}}/COMPLETION.md`
  - Optionally continue with next feature

## Output

- Feature marked complete in database
- `{{AUTORUN_FOLDER}}/COMPLETION.md` summary

## Human Gate

**STOP** - Present completion to human for final review.

Show:
1. `specflow status` output
2. Test results summary
3. Files created/modified

## Post-Completion

When ready to release:
```bash
# Stage changes
git add <specific files>

# Create commit
git commit -m "feat: [feature description]

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>"

# Push and create PR
git push -u origin <branch>
gh pr create --title "[title]" --body "..."
```

## Playbook Complete

Feature development cycle finished. Return to Step 1 for next feature or end session.
