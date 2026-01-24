# SpecFlow PR Comparator Playbook

## Purpose

Reviews and compares pull requests created by multiple agents, identifies the best implementation(s), and produces a recommendation for which PR(s) to merge or which parts to cherry-pick.

## Use Case

In a "border run" scenario with multiple parallel agents:
1. Multiple agents each implement the same feature specification
2. Each agent creates a PR using the **SpecFlow PR Creator** playbook
3. **This playbook** compares all PRs and recommends the best solution
4. Human makes final merge decision based on comparison report

## Workflow Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                   SpecFlow PR Comparator                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. DISCOVER_PRS                                                │
│     └─ Find all open PRs targeting develop                      │
│     └─ Load PR_MANIFEST.json from each agent's artifacts        │
│     └─ Validate manifests are comparable (same features)        │
│                                                                 │
│  2. ANALYZE_METRICS                                             │
│     └─ Compare quality scores (spec, plan)                      │
│     └─ Compare test counts and coverage                         │
│     └─ Compare code metrics (lines, files, complexity)          │
│                                                                 │
│  3. REVIEW_IMPLEMENTATIONS                                      │
│     └─ Diff each PR against common base                         │
│     └─ Identify unique approaches                               │
│     └─ Note strengths/weaknesses of each                        │
│                                                                 │
│  4. GENERATE_RECOMMENDATION                                     │
│     └─ Rank PRs by composite score                              │
│     └─ Identify cherry-pick candidates                          │
│     └─ Create merge strategy recommendation                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Documents

| # | Document | Purpose | resetOnCompletion |
|---|----------|---------|-------------------|
| 1 | DISCOVER_PRS | Find and load all agent PR manifests | false |
| 2 | ANALYZE_METRICS | Compare quantitative metrics | false |
| 3 | REVIEW_IMPLEMENTATIONS | Qualitative code review | false |
| 4 | GENERATE_RECOMMENDATION | Produce final recommendation | false |

## Input

This playbook expects:

1. **Multiple open PRs** targeting develop from feature branches
2. **PR_MANIFEST.json** files in each agent's working folder
3. **Same feature scope** - PRs should implement the same SpecFlow features

## Output

**COMPARISON_REPORT.md** containing:
- Side-by-side metric comparison
- Qualitative analysis of each implementation
- Ranked recommendations
- Merge strategy (winner-take-all, cherry-pick, or hybrid)

**COMPARISON_MATRIX.json** containing:
- Machine-readable comparison data
- Scores and rankings
- Recommended actions

## Scoring Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Test count | 25% | More tests = more confidence |
| Quality scores | 25% | Spec + plan quality from SpecFlow |
| Code efficiency | 20% | Fewer lines for same functionality |
| Doctorow Gate | 15% | All 4 checks passed |
| Completeness | 15% | All features implemented vs partial |

## Cherry-Pick Criteria

Consider cherry-picking when:
- One agent has better tests for feature X
- Another agent has cleaner implementation for feature Y
- Different agents excelled at different aspects

## Merge Strategies

1. **Winner Take All**: One PR clearly superior across all metrics
2. **Cherry-Pick**: Combine best parts from multiple PRs
3. **Hybrid**: Merge winner, then cherry-pick specific improvements
4. **Defer**: No clear winner, need human review

## Usage

1. Run SpecFlow PR Creator on all agent branches
2. Ensure all PRs are open and targeting develop
3. Run this playbook from any agent session
4. Review COMPARISON_REPORT.md for recommendation

---

*Part of the PAI Maestro Playbook Collection*
