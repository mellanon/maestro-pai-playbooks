# Step 5: VERIFY - Quality Gates and Loop Control

**Phase**: Verification | **Loop Control**: Determines next action

---

## Context
- **Playbook:** SpecFlow Development
- **Agent:** {{AGENT_NAME}}
- **Project:** {{AGENT_PATH}}
- **Auto Run Folder:** {{AUTORUN_FOLDER}}
- **Loop:** {{LOOP_NUMBER}}

## Objective

Verify implementation progress and determine whether to continue looping or proceed to completion.

## Instructions

### 1. Run Test Suite

- [ ] **Execute all tests**:
  ```bash
  bun test
  # or project-specific test command
  ```

- [ ] **Record results** in `{{AUTORUN_FOLDER}}/LOOP_{{LOOP_NUMBER}}_TEST_RESULTS.md`:
  ```markdown
  # Test Results - Loop {{LOOP_NUMBER}}

  ## Summary
  - **Tests Run**: XX
  - **Tests Passed**: XX
  - **Tests Failed**: XX

  ## Output
  [Paste test output]
  ```

### 2. Check Feature Progress

- [ ] **View current status**:
  ```bash
  specflow status
  ```

- [ ] **Validate feature phases complete**:
  ```bash
  specflow validate <feature-id>
  ```

  This checks:
  - spec.md exists and is valid
  - plan.md exists and is valid
  - tasks.md exists and tasks are progressing

### 3. Validate Against TDD-EVALS

- [ ] **Check eval criteria** from `docs/TDD-EVALS.md`:

| Criterion | Pass? |
|-----------|-------|
| Tests are deterministic | Run 3 times, all same result |
| Outcomes verifiable | State changes can be checked |
| Regression suite growing | New tests added this loop |

### 4. Loop Decision

| Condition | Action |
|-----------|--------|
| Tests fail | → **Loop back to Step 4** (fix) |
| Tasks remain incomplete | → **Loop back to Step 4** (next task) |
| All tasks complete, tests pass | → **Proceed to Step 6** (complete) |
| Validation fails | → **STOP for human review** |

### Decision Logic

```
IF tests_failing:
    → Reset to Step 4, continue loop
ELIF incomplete_tasks > 0:
    → Reset to Step 4, continue loop
ELIF all_tasks_complete AND tests_pass:
    → Do NOT reset, proceed to Step 6
ELSE:
    → STOP for human review
```

## Output

- `{{AUTORUN_FOLDER}}/LOOP_{{LOOP_NUMBER}}_TEST_RESULTS.md`

## Loop Control Settings

For Maestro Auto Run:
- **This document (5_VERIFY.md)**: `Reset: ON`
- **Step 4**: Gets reset by this document's logic
- **Steps 1-3**: `Reset: OFF` (run once, not in loop)

When this document completes:
- If loop continues: Reset to Step 4
- If done: Allow Step 6 to run
