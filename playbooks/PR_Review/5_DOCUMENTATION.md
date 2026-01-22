# Step 5: DOCUMENTATION - Check Completeness

**Phase**: Documentation | **Output**: DOC_ISSUES.md

---

## Context
- **Playbook:** PR Review
- **Agent:** {{AGENT_NAME}}
- **Project:** {{AGENT_PATH}}
- **Auto Run Folder:** {{AUTORUN_FOLDER}}

## Objective

Verify documentation is complete and accurate for the changes made.

## Instructions

### 1. Check README Updates

- [ ] **If public API changed**:
  - [ ] README updated with new usage
  - [ ] Examples updated
  - [ ] Breaking changes documented

- [ ] **If new feature added**:
  - [ ] Feature documented in README
  - [ ] Installation/setup updated if needed

### 2. Check Inline Documentation

- [ ] **For new functions/methods**:
  - [ ] JSDoc/docstrings present
  - [ ] Parameters documented
  - [ ] Return values documented
  - [ ] Examples for complex usage

- [ ] **For complex logic**:
  - [ ] Comments explain "why" not "what"
  - [ ] Algorithms explained
  - [ ] Edge cases noted

### 3. Check CHANGELOG

- [ ] **CHANGELOG.md updated** (if exists):
  - [ ] Entry added under Unreleased or version
  - [ ] Follows Keep a Changelog format
  - [ ] Links to PR/issue

### 4. Check Type Definitions (TypeScript)

- [ ] **Exported types documented**:
  - [ ] JSDoc on interfaces
  - [ ] Property descriptions
  - [ ] Generic parameters explained

### 5. Check CLI Help (if applicable)

- [ ] **CLI commands documented**:
  - [ ] --help output accurate
  - [ ] Examples in help text
  - [ ] All options documented

### 6. Document Issues

- [ ] **Create `{{AUTORUN_FOLDER}}/DOC_ISSUES.md`**:

```markdown
# Documentation Review

## Summary
- **Issues Found**: X
- **Severity**: [missing/incomplete/outdated]

## Issues

### Issue 1: [title]
- **Location**: [file/section]
- **Type**: [missing/incomplete/outdated]
- **Description**: [what's missing]
- **Suggestion**: [what to add]

## Checklist
- [ ] README updated
- [ ] CHANGELOG updated
- [ ] Inline docs complete
- [ ] Types documented
- [ ] CLI help accurate

## Positive Observations
- [good documentation practices observed]
```

## Output

- `{{AUTORUN_FOLDER}}/DOC_ISSUES.md`

## Next

→ Proceed to Step 6 (SUMMARIZE)
