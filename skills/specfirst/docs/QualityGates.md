# Quality Gates

Quality gates enforce thresholds before phase advancement. Inspired by SpecFlow Bundle's 80% threshold approach.

## Overview

Each phase has a quality score that must meet its threshold before advancing:

| Phase | Default Threshold | What's Measured |
|-------|-------------------|-----------------|
| SPECIFY | 100% | Spec completeness (all requirements defined) |
| TEST | 80% | Test coverage of requirements |
| IMPLEMENT | 80% | Task completion percentage |
| ARCHIVE | 100% | All tests pass, specs merged |
| RELEASE | 100% | File inventory complete, validation passed |

## How Gates Work

### Score Calculation

**TEST phase**: Pass rate of tests
```
score = (passed_tests / total_tests) * 100
```

**IMPLEMENT phase**: Task completion
```
score = (completed_tasks / total_tasks) * 100
```

**SPECIFY/ARCHIVE/RELEASE**: Binary (100% or 0%)

### Gate Check

```bash
specfirst gate          # Check all gates
specfirst gate TEST     # Check specific phase
```

Output:
```
📊 Quality Gate Status
═══════════════════════════════════════

  ✅ SPECIFY    100% (threshold: 100%) - PASSED
  ✅ TEST        85% (threshold: 80%) - PASSED
  ❌ IMPLEMENT   70% (threshold: 80%) - BLOCKED
  ⏳ ARCHIVE      -- (threshold: 100%) - PENDING
  ⏳ RELEASE      -- (threshold: 100%) - PENDING
```

### Phase Advancement

Phases advance automatically when gates pass:

```
SPECIFY (100%) → TEST (80%) → IMPLEMENT (80%) → ARCHIVE (100%) → RELEASE (100%)
```

If blocked:
```
⚠️  Blocked by IMPLEMENT gate
   Need 10% more to pass (currently 70%, threshold 80%)
```

## Configuration

Override thresholds in `.specfirst/config.json`:

```json
{
  "gates": {
    "default_threshold": 80,
    "phase_thresholds": {
      "SPECIFY": 100,
      "TEST": 90,        // Stricter for critical projects
      "IMPLEMENT": 80,
      "ARCHIVE": 100,
      "RELEASE": 100
    }
  }
}
```

## Gate Overrides

In exceptional cases, gates can be overridden with justification:

```typescript
// Framework API
stateManager.updateGate("my-change", "TEST", 75, 80, {
  reason: "Legacy tests excluded - manual verification done",
  by: "andreas"
});
```

Override is recorded in state:
```json
{
  "gates": {
    "test": {
      "passed": true,
      "score": 75,
      "threshold": 80,
      "override": {
        "reason": "Legacy tests excluded - manual verification done",
        "by": "andreas",
        "at": "2025-01-20T15:00:00Z"
      }
    }
  }
}
```

## Integration with Test Runs

Test results automatically update gate status:

```typescript
// After running tests
stateManager.recordTestRun("my-change", "unit", {
  total: 50,
  passed: 45,
  failed: 3,
  skipped: 2
});

// Gate status updated automatically
// TEST gate: 90% (45/50)
```

## Best Practices

### 1. Start Strict, Loosen If Needed

Begin with default thresholds. Only lower if genuinely blocking progress without adding value.

### 2. Don't Game the Metrics

80% test coverage is meaningless if tests don't validate requirements. Focus on:
- Tests reference requirement IDs
- Tests verify SHALL/MUST behaviors
- Tests catch real bugs

### 3. Use Gates as Checkpoints

Gates are not punishment—they're checkpoints that ensure quality before moving on. Better to fix issues early than discover them in production.

### 4. Override Sparingly

Overrides create technical debt. Document why and plan to address.

## CLI Reference

```bash
# Check all gates
specfirst gate

# Check specific phase
specfirst gate TEST
specfirst gate IMPLEMENT

# See gate status in context
specfirst status

# Get suggested next action when blocked
specfirst resume
```

## Framework API

For programmatic access:

```typescript
import { GateChecker, createGateChecker } from "pai-tooling/ingest-test/framework/gate-checker";

const gc = createGateChecker("/path/to/project");

// Check a gate
const result = gc.checkGate("TEST", 85);
// { phase: "TEST", passed: true, score: 85, threshold: 80, message: "...", canAdvance: true }

// Get threshold
const threshold = gc.getThreshold("IMPLEMENT"); // 80

// Format for display
const display = gc.formatGateStatus(gates);
```

## Related

- [StatePersistence.md](StatePersistence.md) - State file structure
- [TestPyramid.md](TestPyramid.md) - Test layer approach
- [SKILL.md](../SKILL.md) - Command reference
