# Step 5: RELEASE - File Inventory and PR Preparation

**Phase**: Release | **Gate**: Human Approval Required

---

## Context
- **Playbook:** SpecFirst Development
- **Agent:** {{AGENT_NAME}}
- **Project:** {{AGENT_PATH}}
- **Auto Run Folder:** {{AUTORUN_FOLDER}}
- **Loop:** {{LOOP_NUMBER}}

## Objective

Prepare release with explicit file inventory, validation checks, and PR creation.

## Constitutional Gate

**Before releasing, complete `docs/RELEASE-FRAMEWORK.md` checklist:**

- [ ] **File inventory** created (explicit include list)
- [ ] **Sanitization** passed (no PII, no secrets)
- [ ] **Tests** all passing
- [ ] **Documentation** complete

## Instructions

### 1. Invoke SpecFirst Release

- [ ] **Run `/specfirst-release`** to prepare release:

```
/specfirst-release
> Version? [vX.Y.Z or description]
> Project? [project-name]
```

The command creates a release plan with:
- File inventory (what's included)
- Exclude list (what NOT to commit)
- Pre-release checklist
- Human approval gates

### 2. Create File Inventory

- [ ] **Document all files for release** in `{{AUTORUN_FOLDER}}/RELEASE_INVENTORY.md`:

```markdown
# Release Inventory

## Include List
Files to be committed:

| Category | Files |
|----------|-------|
| Implementation | [list files] |
| Tests | [list test files] |
| Documentation | [list docs] |
| Configuration | [list configs] |

## Exclude List
Files NOT to commit (verify these are NOT staged):

| File | Reason |
|------|--------|
| .env* | Contains secrets |
| */credentials* | Contains secrets |
| node_modules/ | Dependencies |
| *.log | Temporary files |
```

### 3. Sanitization Check

- [ ] **Scan for secrets**:
  ```bash
  grep -rE "(API_KEY|TOKEN|SECRET|PASSWORD)" [implementation-dir]/
  ```

- [ ] **Scan for personal paths**:
  ```bash
  grep -r "/Users/" [implementation-dir]/
  ```

- [ ] **Verify no PII** in code or comments

### 4. Documentation Check

- [ ] README updated (if applicable)
- [ ] CHANGELOG updated (from Step 4)
- [ ] CLI help complete (if CLI tool)
- [ ] Skill SKILL.md complete (if skill)

### 5. Create PR Description

- [ ] **Generate `{{AUTORUN_FOLDER}}/PR_DESCRIPTION.md`**:

```markdown
## Summary

[Brief description of what this change adds/fixes]

## What This Adds

### [Category 1]
- [Item description]
- [Item description]

### [Category 2]
- [Item description]

## Test Results

- Tests: XX passing
- Coverage: XX% (if measured)

## Checklist

- [x] Tests passing
- [x] No secrets in code
- [x] Documentation complete
- [x] CHANGELOG updated
- [ ] Reviewer approval

---

Generated with SpecFirst Development Playbook
```

### 6. Prepare Commit

- [ ] **Stage files from include list only**:
  ```bash
  git add [specific files from inventory]
  ```

- [ ] **Verify no excluded files staged**:
  ```bash
  git status
  ```

- [ ] **Create commit** (if requested):
  ```bash
  git commit -m "[type]: [description]

  [Body describing changes]

  Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>"
  ```

## Output

- `{{AUTORUN_FOLDER}}/RELEASE_INVENTORY.md`
- `{{AUTORUN_FOLDER}}/PR_DESCRIPTION.md`
- Staged files ready for commit

## Human Gate

**STOP** - Present release inventory to human for final review.

Show:
1. File inventory (include list)
2. Sanitization results
3. PR description

**Only commit/push after human approval.**

## Complete

Playbook finished. When ready:
```bash
git push -u origin [branch-name]
gh pr create --title "[title]" --body-file {{AUTORUN_FOLDER}}/PR_DESCRIPTION.md
```
