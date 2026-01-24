# SpecFlow PR Creator Playbook

## Purpose

Creates pull requests from SpecFlow-completed feature branches with rich changelogs extracted from spec artifacts. Designed for multi-agent Maestro workflows where each agent completes a feature branch and creates a PR to the target branch (auto-detected or configurable).

## Use Case

In a "border run" scenario with multiple parallel agents:
1. Each agent works on their own feature branch (e.g., `feature/signal-agent-1`)
2. Each agent uses SpecFlow to implement features (F-1, F-2, etc.)
3. **This playbook** creates a PR from their feature branch to target (auto-detected)
4. A separate comparator agent reviews all PRs and cherry-picks the best

## Data Sources

This playbook extracts information from:

| Source | Location | Content |
|--------|----------|---------|
| SpecFlow DB | `.specflow/features.db` | Feature metadata, status, dates |
| Spec artifacts | `.specify/specs/F-N-*/` | spec.md, plan.md, tasks.md, docs.md, verify.md |
| Eval results | `.specflow/evals.db` | Quality scores, revision history |
| Git history | Feature branch | Commits, file changes |

## Workflow Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    SpecFlow PR Creator                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. GATHER_CONTEXT                                              │
│     └─ Extract completed features from SpecFlow                 │
│     └─ Identify target branch (auto-detect or override)         │
│     └─ Verify branch is ahead of target                         │
│                                                                 │
│  2. EXTRACT_CHANGELOG                                           │
│     └─ Read all spec artifacts for completed features           │
│     └─ Extract quality scores from evals.db                     │
│     └─ Build structured changelog                               │
│                                                                 │
│  3. COMPARE_BRANCHES                                            │
│     └─ git diff $TARGET_BRANCH...HEAD                           │
│     └─ Categorize changes (features, tests, config)             │
│     └─ Identify breaking changes                                │
│                                                                 │
│  4. PACKAGE_RELEASE                                             │
│     └─ Generate installation instructions                       │
│     └─ List new dependencies                                    │
│     └─ Create upgrade path documentation                        │
│                                                                 │
│  5. CREATE_PR                                                   │
│     └─ Generate PR description from artifacts                   │
│     └─ Create PR via gh CLI                                     │
│     └─ Add labels and request reviews                           │
│                                                                 │
│  6. FINALIZE                                                    │
│     └─ Document PR URL and summary                              │
│     └─ Create comparison manifest for aggregator                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Documents

| # | Document | Purpose | resetOnCompletion |
|---|----------|---------|-------------------|
| 1 | GATHER_CONTEXT | Extract SpecFlow state and verify branch | false |
| 2 | EXTRACT_CHANGELOG | Build changelog from spec artifacts | false |
| 3 | COMPARE_BRANCHES | Analyze what this branch adds vs target | false |
| 4 | PACKAGE_RELEASE | Generate installation/upgrade docs | false |
| 5 | CREATE_PR | Create the actual pull request | false |
| 6 | FINALIZE | Document results for comparator agent | false |

## Generated Artifacts

Working files created in `{{AUTORUN_FOLDER}}`:

- `SPECFLOW_CONTEXT.md` - Feature list with completion status
- `CHANGELOG.md` - Structured changelog from spec artifacts
- `BRANCH_COMPARISON.md` - Detailed diff analysis
- `INSTALLATION_GUIDE.md` - How to install/upgrade with this branch
- `PR_DESCRIPTION.md` - Ready-to-use PR body
- `PR_MANIFEST.json` - Machine-readable summary for comparator agent

## Integration with Comparator Agent

The `PR_MANIFEST.json` contains structured data for automated comparison:

```json
{
  "agent": "{{AGENT_NAME}}",
  "branch": "feature/signal-agent-1",
  "prNumber": 123,
  "prUrl": "https://github.com/...",
  "targetBranch": "develop",  // Auto-detected or from .maestro/TARGET_BRANCH
  "features": [
    {
      "id": "F-1",
      "name": "Event Schema and Types",
      "specQuality": 87,
      "planQuality": 92,
      "taskCount": 6,
      "testCount": 105
    }
  ],
  "metrics": {
    "totalFeatures": 3,
    "totalTests": 200,
    "linesAdded": 1500,
    "linesRemoved": 100,
    "filesChanged": 25
  },
  "qualityGates": {
    "allTestsPass": true,
    "doctorowGate": true,
    "noSecurityIssues": true
  }
}
```

## Prerequisites

- SpecFlow features are COMPLETE (not in_progress)
- Feature branch exists and has commits ahead of target
- `gh` CLI is authenticated
- Target branch exists (develop or main, auto-detected)

## Configuration

### Target Branch

The target branch is determined dynamically:
1. First checks for `.maestro/TARGET_BRANCH` file (override)
2. Then checks if `develop` branch exists
3. Falls back to `main` if no `develop`

To override:
```bash
echo "my-custom-branch" > .maestro/TARGET_BRANCH
```

The playbook uses `$TARGET_BRANCH` variable throughout all documents.

## Usage with Maestro

1. Agent completes SpecFlow implementation
2. Load this playbook in Maestro
3. Run Auto Run
4. PR is created automatically
5. Comparator agent reads PR_MANIFEST.json files

---

*Part of the PAI Maestro Playbook Collection*
