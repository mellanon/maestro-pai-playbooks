# Spec: Progress Tracking

## Overview

Progress tracking provides visibility into workflow status and completion metrics.

## Requirements

### Status Command

- PROG-001: `/specfirst-status` SHALL display current workflow state
- PROG-002: Status SHALL show current phase with visual indicator
- PROG-003: Status SHALL show all active changes with their status
- PROG-004: Status SHALL show task completion percentage per change
- PROG-005: Status SHALL show time since last update

### Phase Visualization

- PROG-010: The system SHALL display phases as a visual pipeline
- PROG-011: Completed phases SHALL show checkmark indicator
- PROG-012: Current phase SHALL be highlighted
- PROG-013: Pending phases SHALL show dimmed indicator

### Task Tracking

- PROG-020: The system SHALL parse `tasks.md` for checkbox items
- PROG-021: Checked items (`- [x]`) SHALL count as completed
- PROG-022: Unchecked items (`- [ ]`) SHALL count as pending
- PROG-023: Task sections SHALL be tracked separately (e.g., Phase 1, Phase 2)

### Resume Support

- PROG-030: `/specfirst-resume` SHALL continue from last known state
- PROG-031: Resume SHALL display what was completed and what remains
- PROG-032: Resume SHALL suggest next action based on current phase

### History

- PROG-040: The system SHALL track phase transitions with timestamps
- PROG-041: The system SHALL record gate pass/fail events
- PROG-042: History SHALL be viewable via `/specfirst-history`

## Status Output Example

```
📋 SpecFirst Status
═══════════════════

Project: specfirst
Active Changes: 1

───────────────────────────────────────
Change: specfirst-v2.0.0
───────────────────────────────────────

Pipeline:
  ✅ SPECIFY ──▶ ✅ TEST ──▶ 🔄 IMPLEMENT ──▶ ⏳ ARCHIVE ──▶ ⏳ RELEASE
                              ▲
                              └── You are here

Progress:
  Phase 1: State Management    [████████░░] 80% (4/5 tasks)
  Phase 2: Quality Gates       [██░░░░░░░░] 20% (1/5 tasks)
  Phase 3: Progress Tracking   [░░░░░░░░░░]  0% (0/5 tasks)
  Phase 4: Workflow Integration[░░░░░░░░░░]  0% (0/4 tasks)
  Phase 5: Documentation       [░░░░░░░░░░]  0% (0/4 tasks)
  Phase 6: Testing             [░░░░░░░░░░]  0% (0/4 tasks)
  ─────────────────────────────────────────
  Overall:                     [██░░░░░░░░] 19% (5/27 tasks)

Last Updated: 2 hours ago
Started: 2026-01-20

➡️ Next: Complete Phase 1 tasks (1 remaining)
```

## Test Cases

- TEST-PROG-001: Status displays current phase correctly
- TEST-PROG-002: Task percentages calculated from tasks.md
- TEST-PROG-003: Resume continues from correct state
- TEST-PROG-004: History tracks all phase transitions
