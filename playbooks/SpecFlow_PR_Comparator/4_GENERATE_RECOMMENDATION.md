# Document 4: Generate Recommendation

## Context

- **Playbook**: SpecFlow PR Comparator
- **Agent**: {{AGENT_NAME}}
- **Project**: {{AGENT_PATH}}
- **Date**: {{DATE}}
- **Working Folder**: {{AUTORUN_FOLDER}}

## Purpose

Produce final recommendation based on combined quantitative metrics and qualitative review. Determine merge strategy and create actionable next steps.

## Prerequisites

- `{{AUTORUN_FOLDER}}/PR_DISCOVERY.md` exists
- `{{AUTORUN_FOLDER}}/METRICS_ANALYSIS.md` exists
- `{{AUTORUN_FOLDER}}/SCORES.json` exists
- `{{AUTORUN_FOLDER}}/IMPLEMENTATION_REVIEW.md` exists

## Tasks

### Task 1: Calculate Final Scores

- [ ] **Combine quantitative and qualitative**:

  Final score = (Composite metric score * 0.6) + (Qualitative rating * 0.4)

- [ ] **Create final ranking**

### Task 2: Determine Merge Strategy

- [ ] **Evaluate strategy options**:

  **Winner Take All** when:
  - One PR scores >10 points higher than second place
  - Winner has all unique contributions
  - No significant cherry-pick candidates from others

  **Cherry-Pick** when:
  - Scores are within 5 points
  - Multiple PRs have unique valuable contributions
  - Can cleanly extract commits without conflicts

  **Hybrid** when:
  - Clear winner but other PRs have 2+ cherry-pick candidates
  - Cherry-picks don't conflict with winner's implementation

  **Defer** when:
  - Scores are tied
  - Significant trade-offs between approaches
  - Human judgment needed on architectural direction

### Task 3: Create Comparison Report

- [ ] **Write COMPARISON_REPORT.md**: Create `{{AUTORUN_FOLDER}}/COMPARISON_REPORT.md`:

```markdown
# PR Comparison Report

Generated: {{DATE}}
Agent: {{AGENT_NAME}}
PRs Compared: [N]

---

## Executive Summary

**Recommendation:** [WINNER TAKE ALL / CHERRY-PICK / HYBRID / DEFER]

**Winner:** PR #[N] ([agent-name])

**Rationale:**
[2-3 sentences explaining why]

---

## Final Rankings

| Rank | PR # | Agent | Metric Score | Qual Score | Final Score |
|------|------|-------|--------------|------------|-------------|
| 1 | 123 | agent-1 | 87.5 | 78 | 83.7 |
| 2 | 124 | agent-2 | 82.3 | 75 | 79.4 |
| 3 | 125 | agent-3 | 71.0 | 55 | 64.6 |

**Scoring Method:**
- Metric Score (60%): Automated metrics from manifests
- Qual Score (40%): Code review assessment
- Final = (Metric * 0.6) + (Qual * 0.4)

---

## Comparison Matrix

| Criterion | PR #123 | PR #124 | PR #125 | Winner |
|-----------|---------|---------|---------|--------|
| Features complete | 3/3 | 2/3 | 3/3 | 123/125 |
| Total tests | 200 | 150 | 100 | 123 |
| Spec quality | 87% | 90% | N/A | 124 |
| Plan quality | 92% | 88% | N/A | 123 |
| Code efficiency | 1400 | 1120 | 1600 | 124 |
| Doctorow Gate | ✓ | ✓ | ? | Tie |
| Architecture | 8/10 | 8/10 | 6/10 | Tie |
| Test quality | 9/10 | 7/10 | 5/10 | 123 |
| **Overall** | **1st** | **2nd** | **3rd** | **123** |

---

## Recommended Merge Strategy

### [WINNER TAKE ALL / CHERRY-PICK / HYBRID / DEFER]

#### If WINNER TAKE ALL:

**Merge PR #[N]**

```bash
gh pr merge [PR_NUMBER] --merge
```

No cherry-picks needed; winner contains all valuable contributions.

#### If CHERRY-PICK:

**Base:** PR #[N] (highest score)

**Cherry-picks:**
1. From PR #[M]: `[commit hash]` - [description]
2. From PR #[O]: `[commit hash]` - [description]

```bash
# Merge base PR
gh pr merge [BASE_PR] --merge

# Cherry-pick from others
git cherry-pick [COMMIT_1]
git cherry-pick [COMMIT_2]
```

#### If HYBRID:

**Merge:** PR #[N] (winner)

**Then cherry-pick:**
1. `[commit]` from PR #[M] - [reason]

**Verification:**
After cherry-picking, run tests to ensure compatibility:
```bash
bun test
```

#### If DEFER:

**Recommendation:** Human review required

**Trade-offs to consider:**
- [Trade-off 1]
- [Trade-off 2]

**Questions for human:**
1. [Question about architectural preference]
2. [Question about priority]

---

## Cherry-Pick Details

### Recommended Cherry-Picks

| Source PR | Commit | Files | Reason | Priority |
|-----------|--------|-------|--------|----------|
| 124 | abc123 | types.ts | Mapped type pattern | High |
| 124 | def456 | logger.ts | Perf optimization | Medium |

### Cherry-Pick Conflicts

| Commit | Conflicts With | Resolution |
|--------|----------------|------------|
| abc123 | Winner's types.ts | Manual merge needed |

---

## Action Items

### Immediate

- [ ] Merge PR #[N] to develop
- [ ] Cherry-pick [N] commits (if applicable)
- [ ] Run full test suite
- [ ] Close remaining PRs with note

### Follow-Up

- [ ] Consider refactoring [X] based on PR #[Y]'s approach
- [ ] Add tests from PR #[Z] for [feature]

---

## Appendix: Full Scoring Details

### PR #123 Detailed Breakdown

**Quantitative (60%):**
| Metric | Value | Normalized | Weight | Score |
|--------|-------|------------|--------|-------|
| Tests | 200 | 100 | 25% | 25.0 |
| Quality | 89.5% | 89.5 | 25% | 22.4 |
| Efficiency | 1400 | 80 | 20% | 16.0 |
| Doctorow | Pass | 100 | 15% | 15.0 |
| Complete | 3/3 | 100 | 15% | 15.0 |
| **Subtotal** | | | | **87.5** |

**Qualitative (40%):**
| Aspect | Rating | Weight | Score |
|--------|--------|--------|-------|
| Architecture | 8 | 30% | 24 |
| Code quality | 7.5 | 30% | 22.5 |
| Tests quality | 9 | 20% | 18 |
| Documentation | 6 | 20% | 12 |
| **Subtotal** | | | **76.5** |

**Final:** (87.5 * 0.6) + (76.5 * 0.4) = **83.1**

[Repeat for other PRs...]

---

*Report generated by SpecFlow PR Comparator Playbook*
```

### Task 4: Create Machine-Readable Output

- [ ] **Write COMPARISON_MATRIX.json**: Create `{{AUTORUN_FOLDER}}/COMPARISON_MATRIX.json`:

```json
{
  "generated": "{{DATE}}",
  "recommendation": {
    "strategy": "WINNER_TAKE_ALL | CHERRY_PICK | HYBRID | DEFER",
    "winner": {
      "prNumber": 123,
      "agent": "agent-1",
      "finalScore": 83.7
    },
    "cherryPicks": [
      {
        "fromPr": 124,
        "commit": "abc123",
        "files": ["types.ts"],
        "reason": "Mapped type pattern",
        "priority": "high"
      }
    ],
    "actions": [
      {
        "type": "merge",
        "prNumber": 123,
        "command": "gh pr merge 123 --merge"
      },
      {
        "type": "cherry-pick",
        "commit": "abc123",
        "command": "git cherry-pick abc123"
      }
    ]
  },
  "rankings": [
    {
      "rank": 1,
      "prNumber": 123,
      "agent": "agent-1",
      "metricScore": 87.5,
      "qualScore": 78,
      "finalScore": 83.7
    }
  ],
  "comparison": {
    "metrics": {
      "tests": { "123": 200, "124": 150, "125": 100, "winner": 123 },
      "quality": { "123": 89.5, "124": 89, "125": null, "winner": 123 },
      "efficiency": { "123": 1400, "124": 1120, "125": 1600, "winner": 124 }
    }
  }
}
```

### Task 5: Final Verification

- [ ] **Confirm all artifacts exist**:
  - [ ] `PR_DISCOVERY.md`
  - [ ] `MANIFESTS.json`
  - [ ] `METRICS_ANALYSIS.md`
  - [ ] `SCORES.json`
  - [ ] `IMPLEMENTATION_REVIEW.md`
  - [ ] `COMPARISON_REPORT.md`
  - [ ] `COMPARISON_MATRIX.json`

- [ ] **Output final recommendation**:

```
═══════════════════════════════════════════════════════════════
SpecFlow PR Comparator - COMPLETE
═══════════════════════════════════════════════════════════════
PRs Compared:     [N]
Strategy:         [WINNER TAKE ALL / CHERRY-PICK / etc.]
Winner:           PR #[N] ([agent]) - [score]
Cherry-picks:     [N] commits from [N] PRs

Report: {{AUTORUN_FOLDER}}/COMPARISON_REPORT.md
═══════════════════════════════════════════════════════════════
```

## Success Criteria

- Final ranking calculated
- Merge strategy determined
- COMPARISON_REPORT.md created with actionable steps
- COMPARISON_MATRIX.json created for automation
- All artifacts verified

## Status

Mark complete when recommendation is generated.

---

## Playbook Complete

The SpecFlow PR Comparator playbook has finished. Review COMPARISON_REPORT.md for the recommended merge strategy.
