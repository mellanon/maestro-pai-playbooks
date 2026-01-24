# Document 6: Finalize and Create Manifest

## Context

- **Playbook**: SpecFlow PR Creator
- **Agent**: {{AGENT_NAME}}
- **Project**: {{AGENT_PATH}}
- **Date**: {{DATE}}
- **Working Folder**: {{AUTORUN_FOLDER}}

## Purpose

Create the machine-readable PR manifest for the comparator agent and document the final results. This document handles both scenarios:
1. **PR already created** - Human ran PR_COMMAND.sh, PR_INFO.json exists
2. **PR not yet created** - Manifest prepared with placeholder, updated after human creates PR

## Prerequisites

- All previous documents complete
- Either:
  - `{{AUTORUN_FOLDER}}/PR_INFO.json` exists (PR was created)
  - OR PR artifacts ready for human to create PR

## Tasks

### Task 1: Check PR Status

- [ ] **Check if PR was created**:
  ```bash
  if [ -f "{{AUTORUN_FOLDER}}/PR_INFO.json" ]; then
    echo "PR exists"
    cat "{{AUTORUN_FOLDER}}/PR_INFO.json"
  else
    echo "PR not yet created"
  fi
  ```

- [ ] **If PR exists, load details**:
  ```bash
  gh pr view --json number,url,title,additions,deletions,changedFiles
  ```

### Task 2: Gather Final Metrics

- [ ] **Count total tests**:
  ```bash
  bun test --reporter summary 2>&1 | grep -E "^\d+ pass" || echo "0"
  ```

- [ ] **Get feature quality scores** from SpecFlow:
  ```bash
  specflow status --format json 2>/dev/null || echo '{"features":[]}'
  ```

- [ ] **Get git statistics**:
  ```bash
  git diff --stat $TARGET_BRANCH...HEAD | tail -1
  ```

### Task 3: Create PR Manifest

- [ ] **Write PR_MANIFEST.json**: Create `{{AUTORUN_FOLDER}}/PR_MANIFEST.json`:

```json
{
  "version": "1.0",
  "generated": "{{DATE}}",
  "agent": {
    "name": "{{AGENT_NAME}}",
    "workingDirectory": "{{AGENT_PATH}}"
  },
  "pr": {
    "number": null,
    "url": null,
    "title": "[Pending - run PR_COMMAND.sh to create]",
    "baseBranch": "$TARGET_BRANCH",
    "headBranch": "[current branch]",
    "state": "pending_creation"
  },
  "features": [
    {
      "id": "F-1",
      "name": "",
      "status": "complete",
      "specQuality": 0,
      "planQuality": 0,
      "taskCount": 0,
      "tasksCompleted": 0,
      "testCount": 0,
      "verificationPassed": true
    }
  ],
  "metrics": {
    "totalFeatures": 0,
    "totalTasks": 0,
    "totalTests": 0,
    "linesAdded": 0,
    "linesRemoved": 0,
    "filesChanged": 0
  },
  "qualityGates": {
    "allTestsPass": true,
    "specQualityAvg": 0,
    "planQualityAvg": 0,
    "doctorowGatePass": true,
    "noSecurityIssues": true
  },
  "artifacts": {
    "specflowContext": "{{AUTORUN_FOLDER}}/SPECFLOW_CONTEXT.md",
    "changelog": "{{AUTORUN_FOLDER}}/CHANGELOG.md",
    "branchComparison": "{{AUTORUN_FOLDER}}/BRANCH_COMPARISON.md",
    "installationGuide": "{{AUTORUN_FOLDER}}/INSTALLATION_GUIDE.md",
    "prDescription": "{{AUTORUN_FOLDER}}/PR_DESCRIPTION.md",
    "prCommand": "{{AUTORUN_FOLDER}}/PR_COMMAND.sh"
  },
  "humanActions": {
    "prCreated": false,
    "prCommandPath": "{{AUTORUN_FOLDER}}/PR_COMMAND.sh",
    "nextStepsPath": "{{AUTORUN_FOLDER}}/NEXT_STEPS.md"
  },
  "timestamps": {
    "playbookStarted": "",
    "playbookCompleted": "{{DATE}}"
  }
}
```

**If PR_INFO.json exists**, update the `pr` section:
```json
"pr": {
  "number": [from PR_INFO.json],
  "url": "[from PR_INFO.json]",
  "title": "[from gh pr view]",
  "baseBranch": "$TARGET_BRANCH",
  "headBranch": "[branch]",
  "state": "open"
},
"humanActions": {
  "prCreated": true
}
```

### Task 4: Create Completion Summary

- [ ] **Write COMPLETION_SUMMARY.md**: Create `{{AUTORUN_FOLDER}}/COMPLETION_SUMMARY.md`:

```markdown
# SpecFlow PR Creator - Completion Summary

## Agent Information
- **Agent**: {{AGENT_NAME}}
- **Date**: {{DATE}}
- **Working Directory**: {{AGENT_PATH}}

## PR Status

### If PR Created:
- **PR #**: [number]
- **URL**: [url]
- **Title**: [title]
- **Base**: $TARGET_BRANCH
- **Head**: [branch]

### If PR Not Yet Created:
- **Status**: Awaiting human action
- **Command**: `./{{AUTORUN_FOLDER}}/PR_COMMAND.sh`
- **Instructions**: See `NEXT_STEPS.md`

## Features Included

| ID | Name | Quality | Tests |
|----|------|---------|-------|
| F-1 | [name] | [score]% | [count] |
| F-2 | [name] | [score]% | [count] |

## Metrics

| Metric | Value |
|--------|-------|
| Features | [N] |
| Tasks completed | [N] |
| Tests | [N] |
| Files changed | [N] |
| Lines added | +[N] |
| Lines removed | -[N] |

## Quality Gates

| Gate | Status |
|------|--------|
| All tests pass | ✓ |
| Spec quality avg | [X]% |
| Plan quality avg | [X]% |
| Doctorow Gate | ✓ |
| No security issues | ✓ |

## Generated Artifacts

| Artifact | Purpose | Status |
|----------|---------|--------|
| `SPECFLOW_CONTEXT.md` | Feature list and status | ✓ |
| `CHANGELOG.md` | Detailed changelog | ✓ |
| `BRANCH_COMPARISON.md` | Diff analysis | ✓ |
| `INSTALLATION_GUIDE.md` | How to install/upgrade | ✓ |
| `PR_DESCRIPTION.md` | PR body content | ✓ |
| `PR_COMMAND.sh` | Human-triggered PR creation | ✓ |
| `NEXT_STEPS.md` | Human instructions | ✓ |
| `PR_MANIFEST.json` | Machine-readable summary | ✓ |
| `COMPLETION_SUMMARY.md` | This file | ✓ |

## For Comparator Agent

The `PR_MANIFEST.json` file contains all structured data needed for automated comparison across multiple agent PRs.

**Note:** If `pr.state` is `"pending_creation"`, the PR has not been created yet. The comparator should wait for human action or skip this agent.

Key comparison points:
- `metrics.totalTests` - More tests = more confidence
- `qualityGates.*` - All should be true
- `metrics.filesChanged` - Lower may indicate cleaner solution
- `features[].specQuality` - Higher = better specified

## Next Steps

### If PR Not Created:
1. Review artifacts in `{{AUTORUN_FOLDER}}/`
2. Run `./{{AUTORUN_FOLDER}}/PR_COMMAND.sh`
3. Update `PR_MANIFEST.json` with PR details

### If PR Created:
1. Human or comparator agent reviews this PR
2. If multiple agents, compare PR manifests
3. Merge best solution(s) to $TARGET_BRANCH
4. Follow INSTALLATION_GUIDE.md for deployment

---

## Playbook Complete

The SpecFlow PR Creator playbook has finished.

**Human Action Required:** Run `PR_COMMAND.sh` to create the PR.

*Generated by {{AGENT_NAME}}*
```

### Task 5: Verify All Artifacts

- [ ] **Confirm all files exist**:
  - [ ] `SPECFLOW_CONTEXT.md`
  - [ ] `CHANGELOG.md`
  - [ ] `BRANCH_COMPARISON.md`
  - [ ] `INSTALLATION_GUIDE.md`
  - [ ] `PR_DESCRIPTION.md`
  - [ ] `PR_COMMAND.sh` (executable)
  - [ ] `NEXT_STEPS.md`
  - [ ] `PR_MANIFEST.json`
  - [ ] `COMPLETION_SUMMARY.md`

- [ ] **Validate PR_MANIFEST.json is valid JSON**:
  ```bash
  cat "{{AUTORUN_FOLDER}}/PR_MANIFEST.json" | jq .
  ```

### Task 6: Final Status Report

- [ ] **Output final status**:

```
═══════════════════════════════════════════════════════════════
SpecFlow PR Creator - COMPLETE
═══════════════════════════════════════════════════════════════
Agent:     {{AGENT_NAME}}
Branch:    [branch-name]
Features:  [N] (F-1 through F-N)
Tests:     [count] passing
Quality:   [avg]% spec, [avg]% plan

PR Status: [CREATED: #N / PENDING: Run PR_COMMAND.sh]

Artifacts: {{AUTORUN_FOLDER}}/
  - PR_DESCRIPTION.md   (PR body)
  - PR_COMMAND.sh       (human-triggered creation)
  - PR_MANIFEST.json    (for comparator agent)
  - NEXT_STEPS.md       (human instructions)

═══════════════════════════════════════════════════════════════
```

## Success Criteria

- PR_MANIFEST.json created with all metrics populated
- COMPLETION_SUMMARY.md created
- All 9 artifacts exist and are valid
- Clear instructions for human to create PR

## Status

Mark complete when all artifacts are verified.

---

## Playbook Complete

The SpecFlow PR Creator playbook has finished.

**Human Action:** Run `./{{AUTORUN_FOLDER}}/PR_COMMAND.sh` to create the PR.
