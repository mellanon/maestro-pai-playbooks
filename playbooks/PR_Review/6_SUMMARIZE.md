# Step 6: SUMMARIZE - Compile Review

**Phase**: Summary | **Output**: REVIEW_SUMMARY.md, PR_COMMENT.md

---

## Context
- **Playbook:** PR Review
- **Agent:** {{AGENT_NAME}}
- **Project:** {{AGENT_PATH}}
- **Auto Run Folder:** {{AUTORUN_FOLDER}}

## Objective

Compile all review findings into a summary and generate a ready-to-post PR comment.

## Instructions

### 1. Gather All Findings

- [ ] **Read all review documents**:
  - `REVIEW_SCOPE.md` - PR context
  - `CODE_ISSUES.md` - Code quality
  - `SECURITY_ISSUES.md` - Security findings
  - `TEST_RESULTS.md` - Test validation
  - `DOC_ISSUES.md` - Documentation gaps

### 2. Calculate Overall Assessment

- [ ] **Count issues by severity**:

| Severity | Code | Security | Tests | Docs | Total |
|----------|------|----------|-------|------|-------|
| Critical | | | | | |
| Major | | | | | |
| Minor | | | | | |

- [ ] **Determine recommendation**:
  - **Approve**: No critical/major issues, minor feedback only
  - **Request Changes**: Critical or multiple major issues
  - **Comment**: Questions or suggestions, not blocking

### 3. Create Summary

- [ ] **Create `{{AUTORUN_FOLDER}}/REVIEW_SUMMARY.md`**:

```markdown
# PR Review Summary

## Overview
- **PR**: [title]
- **Reviewer**: AI Agent ({{AGENT_NAME}})
- **Date**: [date]
- **Recommendation**: [Approve/Request Changes/Comment]

## Findings Summary

| Area | Issues | Severity |
|------|--------|----------|
| Code Quality | X | [max severity] |
| Security | X | [max severity] |
| Tests | X | [max severity] |
| Documentation | X | [max severity] |

## Critical Issues (Must Fix)
1. [issue description] - [file:line]
2. ...

## Major Issues (Should Fix)
1. [issue description] - [file:line]
2. ...

## Minor Issues (Consider)
1. [issue description] - [file:line]
2. ...

## Positive Observations
- [good things in this PR]

## Test Results
- Tests: X passing, X failing
- Coverage: X%
```

### 4. Generate PR Comment

- [ ] **Create `{{AUTORUN_FOLDER}}/PR_COMMENT.md`**:

```markdown
## Code Review Summary

**Recommendation**: 🟢 Approve / 🟡 Comment / 🔴 Request Changes

### Overview
[Brief summary of what this PR does and overall quality]

### Findings

#### Critical Issues
[List or "None found"]

#### Suggestions
- [ ] [actionable suggestion 1]
- [ ] [actionable suggestion 2]

### Test Results
✅ X tests passing | ❌ X tests failing | Coverage: X%

### Security
[Security findings or "No security issues found"]

---
*Reviewed by AI Agent using PR Review Playbook*
```

### 5. Post Comment (if requested)

- [ ] **Optionally post to PR**:
  ```bash
  gh pr comment --body-file {{AUTORUN_FOLDER}}/PR_COMMENT.md
  ```

## Output

- `{{AUTORUN_FOLDER}}/REVIEW_SUMMARY.md`
- `{{AUTORUN_FOLDER}}/PR_COMMENT.md`

## Playbook Complete

Review finished. Human can now:
1. Read REVIEW_SUMMARY.md for detailed findings
2. Post PR_COMMENT.md to GitHub
3. Request changes or approve the PR
