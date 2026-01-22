# Step 4: TEST VALIDATION - Execute and Verify Tests

**Phase**: Testing | **Output**: TEST_RESULTS.md

---

## Context
- **Playbook:** PR Review
- **Agent:** {{AGENT_NAME}}
- **Project:** {{AGENT_PATH}}
- **Auto Run Folder:** {{AUTORUN_FOLDER}}

## Objective

Execute tests, verify coverage, and validate TDD compliance.

## Instructions

### 1. Run Test Suite

- [ ] **Execute all tests**:
  ```bash
  # TypeScript/JavaScript
  bun test
  # or: npm test, yarn test

  # Go
  go test ./...

  # Python
  pytest
  ```

- [ ] **Record results**:
  - Tests run: X
  - Tests passed: X
  - Tests failed: X
  - Tests skipped: X

### 2. Check Test Coverage (if available)

- [ ] **Run coverage report**:
  ```bash
  # TypeScript/JavaScript
  bun test --coverage

  # Go
  go test -cover ./...

  # Python
  pytest --cov
  ```

- [ ] **Evaluate coverage**:
  - [ ] New code has tests
  - [ ] Coverage not decreased
  - [ ] Critical paths covered

### 3. Validate Against TDD-EVALS.md

- [ ] **Check eval criteria**:

| Criterion | Pass? | Notes |
|-----------|-------|-------|
| Tests are deterministic | | Run 3 times, same result |
| Outcomes verifiable | | State changes checked |
| Tests are isolated | | No shared state |
| Tests are readable | | Names describe behavior |

### 4. Review Test Quality

- [ ] **Check new/modified tests**:
  - [ ] Tests follow existing patterns
  - [ ] Table-driven tests where appropriate
  - [ ] Edge cases covered
  - [ ] Error cases tested

### 5. Identify Coverage Gaps

- [ ] **For each changed source file**:
  - [ ] Corresponding test file exists
  - [ ] New functions have tests
  - [ ] Modified code paths tested

### 6. Document Results

- [ ] **Create `{{AUTORUN_FOLDER}}/TEST_RESULTS.md`**:

```markdown
# Test Validation Results

## Test Execution
- **Total Tests**: X
- **Passed**: X
- **Failed**: X
- **Skipped**: X
- **Duration**: Xs

## Test Output
```
[paste test output]
```

## Coverage Summary
- **Overall Coverage**: X%
- **Coverage Change**: +/-X%

## TDD Compliance
| Criterion | Pass |
|-----------|------|
| Deterministic | |
| Isolated | |
| Readable | |

## Coverage Gaps
- [file1.ts] - Missing tests for [function]
- [file2.ts] - Edge case not covered

## New Tests Added
- [list of new test files]

## Recommendations
- [suggestions for improving test coverage]
```

## Output

- `{{AUTORUN_FOLDER}}/TEST_RESULTS.md`

## Next

→ Proceed to Step 5 (DOCUMENTATION)
