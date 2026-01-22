# Step 1: ANALYZE PR - Understand Scope

**Phase**: Analysis | **Output**: REVIEW_SCOPE.md

---

## Context
- **Playbook:** PR Review
- **Agent:** {{AGENT_NAME}}
- **Project:** {{AGENT_PATH}}
- **Auto Run Folder:** {{AUTORUN_FOLDER}}

## Objective

Analyze the pull request to understand its scope, categorize changed files, and prepare for detailed review.

## Instructions

### 1. Fetch PR Information

- [ ] **Get PR details**:
  ```bash
  gh pr view --json title,body,files,additions,deletions,changedFiles
  ```

- [ ] **Get the diff**:
  ```bash
  gh pr diff
  ```

### 2. Categorize Changed Files

- [ ] **Group files by type**:

| Category | Files |
|----------|-------|
| Source Code | .ts, .js, .go, .py, etc. |
| Tests | *.test.*, *.spec.*, *_test.* |
| Configuration | package.json, tsconfig.json, etc. |
| Documentation | *.md, docs/* |
| Other | Everything else |

### 3. Assess Scope

- [ ] **Determine PR size**:
  - Small: <100 lines changed
  - Medium: 100-500 lines
  - Large: 500+ lines

- [ ] **Identify risk areas**:
  - [ ] Security-sensitive files (auth, crypto, API keys)
  - [ ] Core business logic
  - [ ] Public API changes
  - [ ] Database/schema changes

### 4. Document Scope

- [ ] **Create `{{AUTORUN_FOLDER}}/REVIEW_SCOPE.md`**:

```markdown
# PR Review Scope

## PR Information
- **Title**: [title]
- **URL**: [url]
- **Author**: [author]
- **Size**: [small/medium/large]

## Changed Files Summary
- **Total Files**: X
- **Additions**: +X lines
- **Deletions**: -X lines

## File Categories

### Source Code
- [list files]

### Tests
- [list files]

### Configuration
- [list files]

### Documentation
- [list files]

## Risk Assessment
- [ ] Security-sensitive changes: [yes/no]
- [ ] Core logic changes: [yes/no]
- [ ] API changes: [yes/no]
- [ ] Breaking changes: [yes/no]

## Review Focus Areas
1. [area 1]
2. [area 2]
```

## Output

- `{{AUTORUN_FOLDER}}/REVIEW_SCOPE.md`

## Next

→ Proceed to Step 2 (CODE_QUALITY)
