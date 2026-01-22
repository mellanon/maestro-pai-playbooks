# Spec: Quality Gates

## Overview

Quality gates enforce minimum standards before advancing to the next workflow phase.

## Requirements

### Gate Configuration

- GATE-001: The system SHALL support configurable quality thresholds
- GATE-002: The default threshold SHALL be 80% (matching SpecFlow Bundle)
- GATE-003: Thresholds SHALL be configurable per-project in `.specfirst/config.json`
- GATE-004: Thresholds SHALL be configurable per-phase (e.g., stricter for RELEASE)

### Gate Enforcement

- GATE-010: The system SHALL block phase advancement when threshold not met
- GATE-011: The system SHALL display clear failure reason when gate blocks
- GATE-012: The system SHALL allow manual override with explicit `--force` flag
- GATE-013: Manual overrides SHALL be logged in state with reason

### Gate Metrics

- GATE-020: SPECIFY gate SHALL measure spec completeness (all requirements have SHALL/MUST)
- GATE-021: TEST gate SHALL measure test coverage (tests exist for each requirement)
- GATE-022: IMPLEMENT gate SHALL measure task completion (percentage of tasks done)
- GATE-023: ARCHIVE gate SHALL verify all tests pass
- GATE-024: RELEASE gate SHALL verify all previous gates passed

### Gate Command

- GATE-030: `/specfirst-gate` SHALL display current gate status
- GATE-031: `/specfirst-gate check` SHALL evaluate all gates for current change
- GATE-032: `/specfirst-gate override <reason>` SHALL allow manual advancement with logged reason

## Configuration Example

```json
{
  "gates": {
    "default_threshold": 80,
    "phase_thresholds": {
      "SPECIFY": 100,
      "TEST": 80,
      "IMPLEMENT": 80,
      "ARCHIVE": 100,
      "RELEASE": 100
    }
  }
}
```

## Gate Status Output

```
📊 Quality Gate Status: specfirst-v2.0.0
═══════════════════════════════════════

Phase: IMPLEMENT (current)

Gates:
  ✅ SPECIFY    100% (threshold: 100%) - PASSED
  ✅ TEST        85% (threshold: 80%)  - PASSED
  🔄 IMPLEMENT   33% (threshold: 80%)  - IN PROGRESS
  ⏳ ARCHIVE     --  (threshold: 100%) - PENDING
  ⏳ RELEASE     --  (threshold: 100%) - PENDING

Next gate: IMPLEMENT requires 80% task completion (currently 33%)
```

## Test Cases

- TEST-GATE-001: Gate blocks advancement when threshold not met
- TEST-GATE-002: Gate allows advancement when threshold met
- TEST-GATE-003: Force flag overrides gate with logged reason
- TEST-GATE-004: Custom thresholds respected per-phase
