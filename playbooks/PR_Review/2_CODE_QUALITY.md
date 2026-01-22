# Step 2: CODE QUALITY - Review Code Standards

**Phase**: Code Review | **Output**: CODE_ISSUES.md

---

## Context
- **Playbook:** PR Review
- **Agent:** {{AGENT_NAME}}
- **Project:** {{AGENT_PATH}}
- **Auto Run Folder:** {{AUTORUN_FOLDER}}

## Objective

Review code changes for quality, idioms, error handling, and best practices.

## Instructions

### 1. Review Each Changed Source File

- [ ] **Read each file** from REVIEW_SCOPE.md source code list
- [ ] **Check against language idioms**

### 2. Common Quality Checks

| Check | Pass? | Notes |
|-------|-------|-------|
| **Error Handling** | | Errors properly caught/propagated |
| **Naming** | | Clear, descriptive names |
| **Comments** | | Complex logic explained |
| **DRY** | | No unnecessary duplication |
| **Single Responsibility** | | Functions do one thing |
| **Type Safety** | | Proper types, no `any` abuse |

### 3. Language-Specific Checks

#### TypeScript/JavaScript
- [ ] No `any` types without justification
- [ ] Proper async/await usage
- [ ] No floating promises
- [ ] Proper null/undefined handling

#### Go
- [ ] Errors wrapped with context
- [ ] Context propagation for cancellation
- [ ] No panics in library code
- [ ] Interfaces defined where consumed

#### Python
- [ ] Type hints on function signatures
- [ ] PEP8 compliance
- [ ] No bare except clauses
- [ ] Context managers for resources

### 4. Architecture Review

- [ ] **Check against PAI-PRINCIPLES.md** (if applicable):
  - [ ] CLI-first interfaces
  - [ ] Deterministic where possible
  - [ ] UNIX philosophy (single purpose)

### 5. Document Issues

- [ ] **Create `{{AUTORUN_FOLDER}}/CODE_ISSUES.md`**:

```markdown
# Code Quality Issues

## Summary
- **Issues Found**: X
- **Severity**: [critical/major/minor]

## Issues

### Issue 1: [title]
- **File**: [path]
- **Line**: [number]
- **Severity**: [critical/major/minor]
- **Description**: [what's wrong]
- **Suggestion**: [how to fix]

### Issue 2: [title]
...

## Positive Observations
- [good patterns observed]
```

## Output

- `{{AUTORUN_FOLDER}}/CODE_ISSUES.md`

## Next

→ Proceed to Step 3 (SECURITY_REVIEW)
