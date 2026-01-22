# Spec: TDD Framework Integration (pai-tooling v2)

## Overview

Extend the existing pai-tooling framework to support SpecFirst v2 features: state persistence, quality gates, and progress tracking.

**Note:** pai-tooling already exists at `/Users/andreas/Documents/src/pai/tooling` with mature test infrastructure. This spec defines additions, not replacements.

## Existing Infrastructure

```
pai-tooling/                          ← EXISTING
├── ingest-test/framework/            ← Mature framework
│   ├── cli-runner.ts                 ✓ EXISTS
│   ├── integration-runner.ts         ✓ EXISTS
│   ├── acceptance-runner.ts          ✓ EXISTS
│   ├── llm-judge.ts                  ✓ EXISTS
│   ├── report.ts                     ✓ EXISTS
│   └── types.ts                      ✓ EXISTS
├── ctx-test/                         ✓ 100+ tests
├── coupa-test/                       ✓ EXISTS
├── jira-test/                        ✓ EXISTS
└── run-all-tests.sh                  ✓ EXISTS
```

## Requirements

### New Directory: specfirst-test/

- TDD-001: pai-tooling SHALL include `specfirst-test/` directory for SpecFirst-specific tests
- TDD-002: specfirst-test SHALL follow the same structure as ctx-test (unit/, integration/, specs/)
- TDD-003: specfirst-test SHALL reuse framework/ from ingest-test via symlink or import

### Quality Gate Integration (NEW)

- TDD-010: Framework SHALL add `gate-checker.ts` to calculate quality scores
- TDD-011: Gate checker SHALL read test results and calculate pass percentage
- TDD-012: Gate checker SHALL integrate with `.specfirst/state.json`
- TDD-013: Gate checker SHALL support configurable thresholds per-phase
- TDD-014: `run-all-tests.sh` SHALL report gate status after test run

### State Integration (NEW)

- TDD-020: Test results SHALL update `.specfirst/state.json` automatically
- TDD-021: State SHALL record last test run timestamp
- TDD-022: State SHALL record test coverage percentage
- TDD-023: State SHALL record per-requirement pass/fail status

### Requirement Traceability (ENHANCE)

- TDD-030: Tests SHALL reference requirement IDs (STATE-001, GATE-001, etc.)
- TDD-031: Framework SHALL generate coverage matrix (requirements → tests)
- TDD-032: `pai-test coverage` command SHALL show unmapped requirements
- TDD-033: Report SHALL include requirement coverage summary

### CLI Enhancements (NEW)

- TDD-040: Add `pai-test gate` command to check quality thresholds
- TDD-041: Add `pai-test status` command to show test state
- TDD-042: Add `pai-test coverage <skill>` to show requirement mapping
- TDD-043: Integrate with existing `run-all-tests.sh`

## Directory Structure (Additions)

```
pai-tooling/
├── framework/                        ← NEW: Shared framework (extract from ingest-test)
│   ├── gate-checker.ts               ← NEW: Quality gate calculation
│   ├── state-integration.ts          ← NEW: .specfirst/state.json updates
│   └── coverage-matrix.ts            ← NEW: Requirement traceability
├── specfirst-test/                   ← NEW: SpecFirst-specific tests
│   ├── unit/
│   │   ├── state.test.ts             ← Tests for state persistence
│   │   ├── gates.test.ts             ← Tests for quality gates
│   │   └── progress.test.ts          ← Tests for progress tracking
│   ├── integration/
│   │   └── workflow-resume.test.ts   ← Session resume tests
│   ├── specs/
│   │   ├── state.spec.ts             ← STATE-* requirement tests
│   │   ├── gates.spec.ts             ← GATE-* requirement tests
│   │   └── progress.spec.ts          ← PROG-* requirement tests
│   └── fixtures/
│       └── sample-state.json
└── bin/
    └── pai-test.ts                   ← NEW: Unified CLI (optional)
```

## Test Specification Format

```typescript
// Example: specfirst-test/specs/state.spec.ts
import { describe, test, expect } from 'bun:test';
import { runCli } from '../../ingest-test/framework/cli-runner';

describe('STATE-001: Store workflow state', () => {
  test('creates .specfirst/state.json on init', async () => {
    const result = await runCli('specfirst-state', ['init']);
    expect(result.exitCode).toBe(0);
    expect(Bun.file('.specfirst/state.json').exists()).toBe(true);
  });
});

// Requirement mapping for coverage report
export const requirements = ['STATE-001'];
```

## Quality Gate Calculation

```
Gate Score = (passing tests / total tests for phase) × 100

Example for IMPLEMENT phase:
- Total tests in specfirst-test/: 45
- Passing tests: 38
- IMPLEMENT gate score = 38/45 = 84.4%
- Threshold: 80%
- Result: PASS ✅
```

## Migration Path

1. **Phase 1:** Create `specfirst-test/` directory
2. **Phase 2:** Add gate-checker.ts to framework
3. **Phase 3:** Add state integration
4. **Phase 4:** Extract shared framework to pai-tooling/framework/

## Test Cases

- TEST-TDD-001: specfirst-test/ follows ctx-test structure
- TEST-TDD-002: gate-checker calculates correct percentages
- TEST-TDD-003: State file updates after test run
- TEST-TDD-004: Coverage matrix shows all requirements
- TEST-TDD-005: pai-test gate integrates with run-all-tests.sh
