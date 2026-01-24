# Document 2: Analyze Metrics

## Context

- **Playbook**: SpecFlow PR Comparator
- **Agent**: {{AGENT_NAME}}
- **Project**: {{AGENT_PATH}}
- **Date**: {{DATE}}
- **Working Folder**: {{AUTORUN_FOLDER}}

## Purpose

Compare quantitative metrics across all discovered PRs to establish baseline rankings.

## Prerequisites

- `{{AUTORUN_FOLDER}}/PR_DISCOVERY.md` exists
- `{{AUTORUN_FOLDER}}/MANIFESTS.json` exists

## Tasks

### Task 1: Load Manifest Data

- [ ] **Read MANIFESTS.json**: Load all PR manifest data

### Task 2: Compare Quality Metrics

For each PR with a manifest:

- [ ] **Extract quality scores**:
  - Spec quality average
  - Plan quality average
  - Doctorow Gate pass/fail

- [ ] **Build quality comparison table**:

| PR # | Spec Quality | Plan Quality | Doctorow | Quality Score |
|------|--------------|--------------|----------|---------------|
| 123 | 87% | 92% | ✓ | 89.5% |
| 124 | 90% | 88% | ✓ | 89.0% |

### Task 3: Compare Test Metrics

- [ ] **Extract test counts**:
  - Total tests
  - Tests per feature
  - Test/code ratio (if available)

- [ ] **Build test comparison table**:

| PR # | Total Tests | Tests/Feature | Test Ratio |
|------|-------------|---------------|------------|
| 123 | 200 | 67 | 0.4 |
| 124 | 150 | 75 | 0.35 |

### Task 4: Compare Code Metrics

- [ ] **Extract code metrics**:
  - Lines added
  - Lines removed
  - Files changed
  - Net lines (added - removed)

- [ ] **Build code comparison table**:

| PR # | Lines Added | Lines Removed | Files | Net Lines |
|------|-------------|---------------|-------|-----------|
| 123 | 1500 | 100 | 25 | 1400 |
| 124 | 1200 | 80 | 20 | 1120 |

### Task 5: Calculate Composite Scores

- [ ] **Apply scoring weights**:

| Criterion | Weight |
|-----------|--------|
| Test count | 25% |
| Quality scores | 25% |
| Code efficiency | 20% |
| Doctorow Gate | 15% |
| Completeness | 15% |

- [ ] **Normalize and score**:

For each metric, normalize to 0-100 scale:
- Tests: (PR_tests / max_tests) * 100
- Quality: Direct percentage
- Efficiency: (min_lines / PR_lines) * 100 (lower is better)
- Doctorow: 100 if passed, 0 if failed
- Completeness: (completed_features / total_features) * 100

- [ ] **Calculate weighted composite**:

```
Score = (tests * 0.25) + (quality * 0.25) + (efficiency * 0.20) + (doctorow * 0.15) + (completeness * 0.15)
```

### Task 6: Create Metrics Report

- [ ] **Write METRICS_ANALYSIS.md**: Create `{{AUTORUN_FOLDER}}/METRICS_ANALYSIS.md`:

```markdown
# PR Metrics Analysis

Generated: {{DATE}}
Agent: {{AGENT_NAME}}

## Summary

| Rank | PR # | Agent | Composite Score |
|------|------|-------|-----------------|
| 1 | 123 | agent-1 | 87.5 |
| 2 | 124 | agent-2 | 82.3 |
| 3 | 125 | agent-3 | 71.0 |

## Quality Metrics

| PR # | Spec Quality | Plan Quality | Doctorow | Avg Quality |
|------|--------------|--------------|----------|-------------|
| 123 | 87% | 92% | ✓ | 89.5% |
| 124 | 90% | 88% | ✓ | 89.0% |
| 125 | N/A | N/A | ? | - |

**Analysis:**
- PR #124 has highest spec quality
- PR #123 has highest plan quality
- Both pass Doctorow Gate

## Test Metrics

| PR # | Total Tests | Tests/Feature | Best For |
|------|-------------|---------------|----------|
| 123 | 200 | 67 | Overall coverage |
| 124 | 150 | 75 | Tests per feature |

**Analysis:**
- PR #123 has more total tests
- PR #124 has better test density

## Code Metrics

| PR # | Lines Added | Lines Removed | Net | Files |
|------|-------------|---------------|-----|-------|
| 123 | 1500 | 100 | 1400 | 25 |
| 124 | 1200 | 80 | 1120 | 20 |

**Analysis:**
- PR #124 is more efficient (fewer lines)
- PR #123 touches more files

## Composite Score Breakdown

### PR #123
| Criterion | Raw | Normalized | Weight | Weighted |
|-----------|-----|------------|--------|----------|
| Tests | 200 | 100 | 25% | 25.0 |
| Quality | 89.5% | 89.5 | 25% | 22.4 |
| Efficiency | 1400 | 80 | 20% | 16.0 |
| Doctorow | Pass | 100 | 15% | 15.0 |
| Complete | 3/3 | 100 | 15% | 15.0 |
| **Total** | | | | **87.5** |

### PR #124
[Same structure...]

## Metric Highlights

### Best Quality: PR #[N]
[Why this PR has the best quality metrics]

### Most Tests: PR #[N]
[Why this PR has the most comprehensive tests]

### Most Efficient: PR #[N]
[Why this PR uses the least code]

## Preliminary Ranking

Based on quantitative metrics alone:

1. **PR #123** (87.5) - Best overall balance
2. **PR #124** (82.3) - More efficient but fewer tests
3. **PR #125** (71.0) - Limited data, lower confidence

## Next Steps

Proceed to Document 3 (Review Implementations) for qualitative analysis.
```

### Task 7: Store Scores

- [ ] **Write SCORES.json**: Create `{{AUTORUN_FOLDER}}/SCORES.json`:

```json
{
  "scoredAt": "{{DATE}}",
  "weights": {
    "tests": 0.25,
    "quality": 0.25,
    "efficiency": 0.20,
    "doctorow": 0.15,
    "completeness": 0.15
  },
  "prs": [
    {
      "number": 123,
      "scores": {
        "tests": 100,
        "quality": 89.5,
        "efficiency": 80,
        "doctorow": 100,
        "completeness": 100
      },
      "composite": 87.5,
      "rank": 1
    }
  ]
}
```

## Success Criteria

- All PRs scored on all available metrics
- Composite scores calculated with proper weights
- Preliminary ranking established
- Scores saved for recommendation

## Status

Mark complete when metrics analysis is done.
