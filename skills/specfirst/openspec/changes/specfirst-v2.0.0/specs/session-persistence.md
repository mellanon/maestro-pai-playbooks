# Spec: Session Persistence

## Overview

State persistence enables SpecFirst workflows to resume across Claude Code sessions.

## Requirements

### State Storage

- STATE-001: The system SHALL store workflow state in `.specfirst/state.json` at project root
- STATE-002: The system SHALL create `.specfirst/` directory automatically on first use
- STATE-003: The state file SHALL be human-readable JSON
- STATE-004: The system SHALL preserve state across Claude Code session restarts

### State Schema

- STATE-010: State SHALL include `version` field for schema migrations
- STATE-011: State SHALL include `project` identifier
- STATE-012: State SHALL include `current_phase` enum (SPECIFY, TEST, IMPLEMENT, ARCHIVE, RELEASE)
- STATE-013: State SHALL include `changes` array of active change proposals
- STATE-014: Each change SHALL track `name`, `created_at`, `updated_at`, `status`
- STATE-015: Each change SHALL track `tasks` with completion status

### State Operations

- STATE-020: The system SHALL provide `specfirst-state init` to initialize state
- STATE-021: The system SHALL provide `specfirst-state get <key>` to read values
- STATE-022: The system SHALL provide `specfirst-state set <key> <value>` to write values
- STATE-023: The system SHALL provide `specfirst-state reset` to clear state (with confirmation)

### Git Integration

- STATE-030: State files SHOULD be committed to git (tracks workflow progress)
- STATE-031: The system SHALL NOT store sensitive data in state files
- STATE-032: State files SHALL use consistent formatting for clean diffs

## Example State

```json
{
  "version": "2.0.0",
  "project": "specfirst",
  "current_phase": "IMPLEMENT",
  "changes": [
    {
      "name": "specfirst-v2.0.0",
      "created_at": "2026-01-20T00:00:00Z",
      "updated_at": "2026-01-20T12:00:00Z",
      "status": "in_progress",
      "phase": "IMPLEMENT",
      "tasks": {
        "total": 24,
        "completed": 8,
        "percentage": 33
      },
      "gates": {
        "specify": { "passed": true, "score": 100 },
        "test": { "passed": true, "score": 85 },
        "implement": { "passed": false, "score": 33 }
      }
    }
  ]
}
```

## Test Cases

- TEST-STATE-001: Init creates `.specfirst/state.json` with valid schema
- TEST-STATE-002: State persists after Claude Code restart
- TEST-STATE-003: Multiple changes tracked independently
- TEST-STATE-004: Reset clears state with confirmation prompt
