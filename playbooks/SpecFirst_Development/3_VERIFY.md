# Step 3: VERIFY - Quality Gates and Loop Control

**Phase**: Verification | **Gate**: 80%+ | **Loop Control**

---

## Context
- **Playbook:** SpecFirst Development
- **Agent:** {{AGENT_NAME}}
- **Project:** {{AGENT_PATH}}
- **Auto Run Folder:** {{AUTORUN_FOLDER}}
- **Loop:** {{LOOP_NUMBER}}

## Objective

Verify implementation progress and determine whether to continue looping or proceed to archive.

## Instructions

### 1. Run Test Suite

- [ ] **Execute all tests** for the project
- [ ] **Record results** in `{{AUTORUN_FOLDER}}/LOOP_{{LOOP_NUMBER}}_TEST_RESULTS.md`:

```markdown
# Test Results - Loop {{LOOP_NUMBER}}

## Summary
- **Tests Run**: XX
- **Tests Passed**: XX
- **Tests Failed**: XX
- **Test Command**: [command used]

## Output
[Paste test output]

## Failures (if any)
[Details of any failures]
```

### 2. Check Quality Gate

- [ ] **Run `specfirst gate`** to check phase threshold:
  ```bash
  specfirst gate
  ```

- [ ] **Record gate status**:
  - Current phase: ___
  - Gate percentage: ___% (target: 80%)
  - Pass/Fail: ___

### 3. Check Task Progress

- [ ] **Run `specfirst status`** to see task breakdown:
  ```bash
  specfirst status
  ```

- [ ] **Count tasks by status**:
  - COMPLETE: ___
  - PENDING: ___
  - BLOCKED: ___

### 4. Validate Against TDD-EVALS

- [ ] **Check eval criteria** from `docs/TDD-EVALS.md`:

| Criterion | Pass? |
|-----------|-------|
| Tests are deterministic | Run 3 times, all same result |
| Outcomes verifiable | State changes can be checked |
| Regression suite growing | New tests added this loop |

## Loop Decision

| Condition | Action |
|-----------|--------|
| Tests fail | → **Loop back to Step 2** (fix) |
| Tasks remain PENDING with satisfied deps | → **Loop back to Step 2** (next task) |
| All tasks COMPLETE, gate ≥80% | → **Proceed to Step 4** |
| All tasks COMPLETE, gate <80% | → **Review specs** (may need more tasks) |

### Decision Logic

```
IF tests_failing:
    → Reset to Step 2, continue loop
ELIF pending_tasks_with_satisfied_deps > 0:
    → Reset to Step 2, continue loop
ELIF all_tasks_complete AND gate >= 80%:
    → Do NOT reset, proceed to Step 4
ELSE:
    → STOP for human review
```

## Output

- `{{AUTORUN_FOLDER}}/LOOP_{{LOOP_NUMBER}}_TEST_RESULTS.md`
- Updated `.specfirst/state.json`

## Loop Control Settings

For Maestro Auto Run:
- **This document (3_VERIFY.md)**: `Reset: ON`
- **Documents 1-2**: `Reset: OFF` (get reset by this document's logic)

When this document completes:
- If loop continues: Reset tasks in Steps 2-3
- If done: Do NOT reset, allow Step 4 to run
