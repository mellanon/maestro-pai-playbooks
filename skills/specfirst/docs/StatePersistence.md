# State Persistence

SpecFirst v2.0 adds session persistence for cross-session workflow tracking.

## Overview

State is stored in `.specfirst/state.json` at the project root. This enables:

- **Session Resume**: Pick up where you left off across Claude Code sessions
- **Progress Tracking**: See completion percentages for tasks and phases
- **Quality Gates**: Enforce thresholds before phase advancement

## State File Structure

```json
{
  "version": "2.0.0",
  "project": "my-project",
  "current_phase": "IMPLEMENT",
  "changes": [
    {
      "name": "add-feature",
      "created_at": "2025-01-20T10:00:00Z",
      "updated_at": "2025-01-20T14:30:00Z",
      "status": "in_progress",
      "phase": "IMPLEMENT",
      "tasks": {
        "total": 10,
        "completed": 7,
        "percentage": 70
      },
      "gates": {
        "specify": { "passed": true, "score": 100, "threshold": 100 },
        "test": { "passed": true, "score": 85, "threshold": 80 },
        "implement": { "passed": false, "score": 70, "threshold": 80 }
      }
    }
  ],
  "last_test_run": "2025-01-20T14:25:00Z"
}
```

## CLI Commands

### Initialize State

```bash
specfirst init [project-name]
```

Creates `.specfirst/` directory with default state.

### Check Status

```bash
specfirst status
```

Shows current phase, active changes, and progress.

### Resume Work

```bash
specfirst resume
```

Shows what to work on next based on current state.

### Query State

```bash
# Get a value
specfirst state get current_phase
specfirst state get changes

# Set a value (JSON format)
specfirst state set current_phase '"TEST"'
```

## State Schema

### Top-Level Fields

| Field | Type | Description |
|-------|------|-------------|
| `version` | string | State file version (e.g., "2.0.0") |
| `project` | string | Project identifier |
| `current_phase` | Phase | Current workflow phase |
| `changes` | Change[] | Active change proposals |
| `last_test_run` | string? | ISO timestamp of last test run |

### Change Object

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Change identifier (e.g., "add-feature") |
| `created_at` | string | ISO timestamp |
| `updated_at` | string | ISO timestamp |
| `status` | Status | "pending" | "in_progress" | "completed" |
| `phase` | Phase | Current phase for this change |
| `tasks` | TaskProgress | Task completion tracking |
| `gates` | Record<Phase, GateStatus> | Quality gate status per phase |

### Phases

```
SPECIFY → TEST → IMPLEMENT → ARCHIVE → RELEASE
```

## Integration with Workflows

### /openspec-proposal

When creating a new change proposal, state is initialized:

1. Creates change entry with `status: "pending"`
2. Sets phase to `SPECIFY`
3. Initializes task counters

### /openspec-apply

During implementation:

1. Updates `status` to `"in_progress"`
2. Tracks task completion in `tasks`
3. Records test runs
4. Updates gate status

### /openspec-archive

When archiving completed work:

1. Verifies all gates pass
2. Updates `status` to `"completed"`
3. Records completion timestamp

## Configuration

Optional configuration in `.specfirst/config.json`:

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

## Framework Integration

State management is implemented in `pai-tooling/ingest-test/framework/`:

- `state-integration.ts` - StateManager class
- `gate-checker.ts` - GateChecker class
- `coverage-matrix.ts` - Requirement traceability

These modules are shared across PAI skills for consistent behavior.

## Related

- [QualityGates.md](QualityGates.md) - Gate enforcement details
- [SKILL.md](../SKILL.md) - Command reference
- [ProposeChange.md](../workflows/ProposeChange.md) - Workflow integration
