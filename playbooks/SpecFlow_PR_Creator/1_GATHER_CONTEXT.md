# Document 1: Gather SpecFlow Context

## Context

- **Playbook**: SpecFlow PR Creator
- **Agent**: {{AGENT_NAME}}
- **Project**: {{AGENT_PATH}}
- **Date**: {{DATE}}
- **Working Folder**: {{AUTORUN_FOLDER}}

## Purpose

Extract the current state of SpecFlow implementation, identify completed features, and verify the branch is ready for PR creation.

## Configuration

```yaml
targetBranch: develop
```

## Tasks

### Task 1: Verify SpecFlow State

- [ ] **Check for SpecFlow databases**:
  - Verify `.specflow/features.db` exists
  - Verify `.specify/` directory exists with spec artifacts
  - If SpecFlow is not initialized, STOP and report error

- [ ] **Get current feature status**:
  ```bash
  specflow status
  ```
  Record output for later processing.

### Task 2: Extract Completed Features

- [ ] **Query features database**: Extract all features with `status = 'complete'`

  Use the specflow CLI or query directly:
  ```bash
  specflow list --status complete
  ```

- [ ] **For each completed feature, verify artifacts exist**:
  - `.specify/specs/F-N-*/spec.md`
  - `.specify/specs/F-N-*/plan.md`
  - `.specify/specs/F-N-*/tasks.md`
  - `.specify/specs/F-N-*/docs.md`
  - `.specify/specs/F-N-*/verify.md`

- [ ] **Document any missing artifacts** as warnings

### Task 3: Verify Git Branch State

- [ ] **Confirm current branch**:
  ```bash
  git branch --show-current
  ```

- [ ] **Check if branch exists on remote**:
  ```bash
  git ls-remote --heads origin $(git branch --show-current)
  ```

- [ ] **Count commits ahead of target**:
  ```bash
  git rev-list --count develop..HEAD
  ```
  If 0, STOP - nothing to PR.

- [ ] **Check for uncommitted changes**:
  ```bash
  git status --porcelain
  ```
  If dirty, warn but continue (may need final commit).

### Task 4: Create Context Document

- [ ] **Write SPECFLOW_CONTEXT.md**: Create `{{AUTORUN_FOLDER}}/SPECFLOW_CONTEXT.md`:

```markdown
# SpecFlow Context for PR Creation

## Agent Information
- **Agent**: {{AGENT_NAME}}
- **Date**: {{DATE}}
- **Working Directory**: {{AGENT_PATH}}

## Branch Information
- **Current Branch**: [branch name]
- **Target Branch**: develop
- **Commits Ahead**: [count]
- **Branch Pushed**: [yes/no]

## SpecFlow Status

### Completed Features

| ID | Name | Spec Path | Tasks | Tests |
|----|------|-----------|-------|-------|
| F-1 | [name] | .specify/specs/f-1-.../ | 6/6 | 105 |
| F-2 | [name] | .specify/specs/f-2-.../ | 4/4 | 45 |

### Features In Progress (Not Included)

| ID | Name | Phase | Status |
|----|------|-------|--------|
| F-3 | [name] | implement | in_progress |

### Artifact Verification

| Feature | spec.md | plan.md | tasks.md | docs.md | verify.md |
|---------|---------|---------|----------|---------|-----------|
| F-1 | ✓ | ✓ | ✓ | ✓ | ✓ |
| F-2 | ✓ | ✓ | ✓ | ✓ | ✓ |

## Pre-PR Checklist

- [ ] At least one feature is COMPLETE
- [ ] All completed features have all 5 artifacts
- [ ] Branch has commits ahead of develop
- [ ] No critical uncommitted changes

## Warnings

[List any issues found but proceeding anyway]

## Next Steps

Proceed to Document 2 (Extract Changelog) with [N] completed features.
```

### Task 5: Push Branch if Needed

- [ ] **Check if remote tracking exists**:
  ```bash
  git rev-parse --abbrev-ref --symbolic-full-name @{u} 2>/dev/null || echo "no-upstream"
  ```

- [ ] **Push branch to origin if needed**:
  ```bash
  git push -u origin $(git branch --show-current)
  ```

## Success Criteria

- SPECFLOW_CONTEXT.md created with complete information
- At least one feature is COMPLETE with all artifacts
- Branch is pushed to remote and ahead of target

## Status

Mark complete when context is gathered and prerequisites verified.
