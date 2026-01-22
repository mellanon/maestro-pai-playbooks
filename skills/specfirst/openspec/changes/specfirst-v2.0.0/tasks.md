# Tasks: SpecFirst v2.0.0

## Implementation Checklist

### Phase 0: TDD Framework (extend pai-tooling)
**Location:** `/Users/andreas/Documents/src/pai/tooling`

- [x] Create `specfirst-test/` directory structure (unit/, integration/, specs/, fixtures/)
- [x] Add `gate-checker.ts` to framework/ (quality gate calculation)
- [x] Add `state-integration.ts` to framework/ (.specfirst/state.json updates)
- [x] Add `coverage-matrix.ts` to framework/ (requirement traceability)
- [x] Update `run-all-tests.sh` to include specfirst-test
- [ ] Add gate status reporting to test output

**Test Results:** 53 passing, 17 todo, 0 failing

### Phase 1: State Management Foundation
- [x] Create `bin/specfirst/specfirst.ts` CLI tool (uses shared framework)
- [x] Define state schema (JSON structure) - in `state-integration.ts`
- [x] Implement state read/write operations - via `StateManager` class
- [x] Add `.specfirst/` to standard project structure - via `specfirst init`
- [x] Write tests: `pai-tooling/specfirst-test/specs/cli.spec.ts`

### Phase 2: Quality Gates
- [x] Define quality metrics structure - in `gate-checker.ts`
- [x] Implement threshold checking logic - `GateChecker` class
- [x] Add gate enforcement to phase transitions - `StateManager.advancePhase()`
- [x] Create `specfirst gate` command for manual checks
- [x] Write tests: `pai-tooling/specfirst-test/integration/gate-checker.test.ts`

### Phase 3: Progress Tracking
- [x] Implement `specfirst status` command
- [x] Add phase tracking (SPECIFY → TEST → IMPLEMENT → ARCHIVE → RELEASE)
- [x] Track task completion percentages - `StateManager.updateTaskProgress()`
- [x] Add time tracking (created_at, updated_at) - in Change interface
- [x] Implement `specfirst resume` command

### Phase 4: Workflow Integration
- [x] Update `SKILL.md` with v2 state management section
- [x] Update CLI help with comprehensive v2 documentation
- [x] Update `workflows/ProposeChange.md` with state commands
- [ ] Update `/openspec-apply` to track progress automatically
- [ ] Update `/openspec-archive` to record completion
- [ ] Update `/specfirst-release` to verify all gates passed

### Phase 5: Documentation
- [x] Create `docs/StatePersistence.md`
- [x] Create `docs/QualityGates.md`
- [x] Update `SKILL.md` with new commands
- [x] Update `workflows/*.md` with gate integration
- [x] Add v2 examples to `docs/WorkedExample.md`

### Phase 6: Acceptance Testing
- [x] Run specfirst-test suite - 53 passing, 17 todo, 0 failing
- [ ] Run coverage matrix - verify all requirements covered
- [ ] Manual acceptance testing (session resume across restarts)
- [x] Update `ACCEPTANCE_TESTS.md` with v2 tests

## Dependencies

```
Phase 0: TDD Framework
    │
    ▼
Phase 1: State Management ──────┐
    │                           │
    ▼                           │
Phase 2: Quality Gates ─────────┼──▶ Phase 4: Workflow Integration
    │                           │
    ▼                           │
Phase 3: Progress Tracking ─────┘
    │
    ▼
Phase 5: Documentation
    │
    ▼
Phase 6: Acceptance Testing
```

## Requirement Coverage Matrix

| Spec File | Requirements | Tests |
|-----------|--------------|-------|
| session-persistence.md | STATE-001 to STATE-032 | state.spec.ts |
| quality-gates.md | GATE-001 to GATE-032 | gates.spec.ts |
| progress-tracking.md | PROG-001 to PROG-042 | progress.spec.ts |
| tdd-framework.md | TDD-001 to TDD-062 | (framework tests) |

## Acceptance Tests

See `ACCEPTANCE_TESTS.md` (to be updated with v2 test cases).
