# Step 2: IMPLEMENT - TDD Implementation

**Phase**: Implementation (Loop) | **TDD Required**

---

## Context
- **Playbook:** SpecFirst Development
- **Agent:** {{AGENT_NAME}}
- **Project:** {{AGENT_PATH}}
- **Auto Run Folder:** {{AUTORUN_FOLDER}}
- **Loop:** {{LOOP_NUMBER}}

## Objective

Implement ONE task using Test-Driven Development via the SpecFirst skill.

## Constitutional Gate

**Before implementing, review `docs/TDD-EVALS.md`:**

- [ ] Tests are written FIRST (tests fail initially)
- [ ] Tests are deterministic (pass@k = pass^k)
- [ ] No LLM inference in test assertions
- [ ] Outcomes are verifiable state changes

## Instructions

### 1. Check Current Status

- [ ] **Run `specfirst status`** to see pending tasks:
  ```bash
  specfirst status
  ```

- [ ] **Identify next task** with satisfied dependencies

### 2. Invoke SpecFirst Apply

- [ ] **Run `/openspec-apply`** to implement the next task:

```
/openspec-apply
> Which change? [change-name]
```

The SpecFirst skill handles:
1. Selecting next PENDING task with satisfied dependencies
2. Writing tests for the task requirements (RED - tests fail)
3. Implementing code to make tests pass (GREEN)
4. Refactoring if needed (REFACTOR)
5. Updating task status in `tasks.md`

### 3. TDD Flow (Executed by SpecFirst)

```
┌─────────────────────────────────────────────────────┐
│  1. READ: Task requirements from tasks.md           │
├─────────────────────────────────────────────────────┤
│  2. RED: Write failing tests                        │
│     - Test location follows project convention      │
│     - Run tests: expect FAIL                        │
├─────────────────────────────────────────────────────┤
│  3. GREEN: Implement code                           │
│     - Minimal code to pass tests                    │
│     - Run tests: expect PASS                        │
├─────────────────────────────────────────────────────┤
│  4. REFACTOR: Clean up if needed                    │
│     - Run tests: still PASS                         │
├─────────────────────────────────────────────────────┤
│  5. UPDATE: Mark task as COMPLETE in tasks.md       │
└─────────────────────────────────────────────────────┘
```

### 4. Verify Eval Criteria

After SpecFirst completes the task, verify against `docs/TDD-EVALS.md`:

| Criterion | Check |
|-----------|-------|
| Tests are deterministic | Run tests 3 times, all same result |
| Outcome verifiable | State change can be checked |
| No LLM calls in tests | Tests don't require AI inference |
| Partial credit possible | Multiple assertions per test |

### 5. Record Results

- [ ] **Update `{{AUTORUN_FOLDER}}/LOOP_{{LOOP_NUMBER}}_PROGRESS.md`**:

```markdown
# Progress - Loop {{LOOP_NUMBER}}

## Task Completed
- **Task ID**: T-XXX
- **Description**: [task description]
- **Tests Added**: N
- **Tests Passing**: N/N

## Test Output
[Paste test output]

## Files Modified
- [file path] - [what changed]

## Next Task
- T-YYY (if dependencies satisfied)
- OR "All tasks complete"
```

## Output

- Implementation code in project structure
- Test files in project test directory
- Updated `openspec/changes/[change-name]/tasks.md`
- `{{AUTORUN_FOLDER}}/LOOP_{{LOOP_NUMBER}}_PROGRESS.md`

## Next

→ Proceed to Step 3 (VERIFY)
