# Document 1: Discover Pull Requests

## Context

- **Playbook**: SpecFlow PR Comparator
- **Agent**: {{AGENT_NAME}}
- **Project**: {{AGENT_PATH}}
- **Date**: {{DATE}}
- **Working Folder**: {{AUTORUN_FOLDER}}

## Purpose

Discover all open pull requests targeting the target branch (auto-detected or configured), load their PR_MANIFEST.json files, and validate they are comparable (implementing the same features).

## Configuration

**Target Branch:** Determined dynamically by detecting the main development branch:
- First checks for `.maestro/TARGET_BRANCH` file (override)
- Then checks if `develop` branch exists
- Falls back to `main` if no `develop`

```bash
# Auto-detect or read override
if [[ -f .maestro/TARGET_BRANCH ]]; then
    TARGET_BRANCH=$(cat .maestro/TARGET_BRANCH)
elif git rev-parse --verify develop >/dev/null 2>&1; then
    TARGET_BRANCH="develop"
else
    TARGET_BRANCH="main"
fi
echo "Target branch: $TARGET_BRANCH"
```

## Tasks

### Task 1: List Open PRs

- [ ] **Find all open PRs targeting $TARGET_BRANCH**:
  ```bash
  gh pr list --base $TARGET_BRANCH --state open --json number,title,headRefName,author,url,additions,deletions,changedFiles
  ```

- [ ] **Record PR list** for processing

### Task 2: Locate PR Manifests

For each PR found:

- [ ] **Check out PR branch** (read-only):
  ```bash
  gh pr checkout [PR_NUMBER] --detach
  ```

- [ ] **Look for PR_MANIFEST.json** in known locations:
  - Check `{{AUTORUN_FOLDER}}/../*/PR_MANIFEST.json`
  - Check any `.maestro/` folders
  - Check working directory for manifest files

- [ ] **If manifest not found**:
  - Mark PR as "no manifest"
  - Can still compare using git diff, but limited metrics

- [ ] **Return to original branch**:
  ```bash
  git checkout -
  ```

### Task 3: Load and Validate Manifests

- [ ] **Parse each PR_MANIFEST.json**:
  - Validate JSON structure
  - Extract feature list
  - Extract metrics

- [ ] **Check feature compatibility**:
  - All PRs should target same feature IDs (F-1, F-2, etc.)
  - Warn if feature sets differ
  - Note any partial implementations

### Task 4: Create Discovery Document

- [ ] **Write PR_DISCOVERY.md**: Create `{{AUTORUN_FOLDER}}/PR_DISCOVERY.md`:

```markdown
# PR Discovery Report

Generated: {{DATE}}
Agent: {{AGENT_NAME}}

## Open PRs Targeting $TARGET_BRANCH

| PR # | Title | Author | Branch | Manifest |
|------|-------|--------|--------|----------|
| 123 | feat: F-1 to F-3 | agent-1 | feature/signal-agent-1 | ✓ |
| 124 | feat: F-1 to F-3 | agent-2 | feature/signal-agent-2 | ✓ |
| 125 | feat: F-1 to F-3 | agent-3 | feature/signal-agent-3 | ✗ |

## Feature Coverage

| Feature | PR #123 | PR #124 | PR #125 |
|---------|---------|---------|---------|
| F-1 | ✓ | ✓ | ✓ |
| F-2 | ✓ | ✓ | ✓ |
| F-3 | ✓ | ✗ | ✓ |

## Manifest Summary

### PR #123 (agent-1)
- **Features**: 3 complete
- **Total tests**: 200
- **Quality avg**: 89%
- **Lines**: +1500/-100

### PR #124 (agent-2)
- **Features**: 2 complete
- **Total tests**: 150
- **Quality avg**: 92%
- **Lines**: +1200/-80

### PR #125 (agent-3)
- **Manifest**: Not found
- **Features**: Unknown (will analyze from diff)

## Validation

| Check | Status |
|-------|--------|
| PRs found | [N] |
| Manifests loaded | [N] |
| Feature sets compatible | ✓/✗ |
| Ready for comparison | ✓/✗ |

## Next Steps

Proceed to Document 2 (Analyze Metrics) with [N] comparable PRs.
```

### Task 5: Store Manifest Data

- [ ] **Write MANIFESTS.json**: Create `{{AUTORUN_FOLDER}}/MANIFESTS.json`:

```json
{
  "discoveredAt": "{{DATE}}",
  "prs": [
    {
      "number": 123,
      "url": "...",
      "branch": "feature/signal-agent-1",
      "hasManifest": true,
      "manifest": { /* full PR_MANIFEST.json content */ }
    },
    {
      "number": 124,
      "url": "...",
      "branch": "feature/signal-agent-2",
      "hasManifest": true,
      "manifest": { /* full PR_MANIFEST.json content */ }
    }
  ],
  "commonFeatures": ["F-1", "F-2"],
  "totalPRs": 3,
  "prsWithManifests": 2
}
```

## Success Criteria

- All open PRs discovered
- Manifests loaded where available
- Feature compatibility assessed
- Discovery data saved for analysis

## Status

Mark complete when all PRs are discovered and documented.
