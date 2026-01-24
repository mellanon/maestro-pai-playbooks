# RCA: Maestro Auto Run Does Not Respect ALL_FEATURES_COMPLETE Exit Condition

**Date:** 2026-01-24
**Severity:** Medium
**Status:** Open
**Related Ticket:** SMS-454

---

## Summary

Maestro Auto Run continues cycling through playbook documents (Loop 16, 17, 18...) indefinitely, even after all features are complete and `.maestro/CURRENT_FEATURE.md` contains the documented exit condition `ALL_FEATURES_COMPLETE`.

## Timeline

| Time | Event |
|------|-------|
| ~12h ago | signal-agent-1 and signal-agent-2 started Auto Run with SpecFlow_Development playbook |
| ~6h ago | Both agents completed all 16 features |
| ~6h ago | `.maestro/CURRENT_FEATURE.md` updated with `ALL_FEATURES_COMPLETE` |
| Loop 16-17+ | Agents continue cycling through documents, checking tasks but finding nothing to do |
| Present | Agents still running at Loop 16 (signal-2) and Loop 17 (signal-1) |

## Symptoms

1. **Task counts increasing without progress**: 1019/1083 and 957/1021 tasks shown, but no real work being done
2. **Document cycling continues**: Documents 5_VERIFY.md and 6_COMPLETE.md keep resetting
3. **Loop counter incrementing**: Loop 16 → 17 → 18... despite completion state
4. **ALL_FEATURES_COMPLETE ignored**: The documented exit condition in CURRENT_FEATURE.md has no effect

## Evidence

### Signal-Agent-1 CURRENT_FEATURE.md
```markdown
# Current Feature

ALL_FEATURES_COMPLETE

All features have been implemented. Playbook complete.

## Summary
- **Total Features:** 16
- **Completed:** 15
- **Skipped:** 1 (F-12 Vector Configuration)
- **Tests:** 394 passing
```

### Signal-Agent-2 CURRENT_FEATURE.md
```markdown
# Current Feature
ALL_FEATURES_COMPLETE

All features have been implemented. Playbook complete.
Date: 2026-01-24

## Summary
- **Total Features:** 16
- **Completed:** 16 (100%)
```

### Loop 17 6_COMPLETE.md (signal-agent-1)
Shows agent recognizing completion state but continuing anyway:
```markdown
### 11. Check for Next Feature

- [x] **List remaining pending features (by ID order)**: No features remaining

  `.maestro/CURRENT_FEATURE.md` contains `ALL_FEATURES_COMPLETE` - playbook exits
```

## Root Cause Analysis

### The Disconnect: Playbook Documentation vs App Implementation

The playbook documents an exit condition:
```markdown
## Playbook Complete (Final Exit)

The playbook only exits when `.maestro/CURRENT_FEATURE.md` contains `ALL_FEATURES_COMPLETE`.
```

**However**, this is documentation of *intent*, not implementation. The Maestro app does not:
1. Parse `.maestro/CURRENT_FEATURE.md` for exit conditions
2. Understand `ALL_FEATURES_COMPLETE` as a signal to stop
3. Have any mechanism to exit Auto Run based on playbook state

### Why Maestro Keeps Running

Maestro Auto Run operates on a simple loop model:
1. Process Document 0 → 1 → 2 → 3 → 4 → 5 → 6
2. After Document 6 (or 7), reset to Document 0 and increment loop counter
3. Continue until: **Max Loops reached** OR **Manual Stop**

There is no mechanism for the playbook to signal "I'm done, stop running".

### The Agent's Dilemma

The Claude agent correctly:
- Detects ALL_FEATURES_COMPLETE
- Notes "playbook exits" in the checklist
- Completes all tasks as "done"

But it cannot:
- Actually exit the Maestro Auto Run session
- Signal to Maestro that the playbook is complete
- Prevent Maestro from advancing to the next document

## Contributing Factors

1. **Misleading Documentation**: The playbook documents say "exits when ALL_FEATURES_COMPLETE" but this is aspirational, not functional
2. **No Playbook Exit Signal**: Maestro has no API or mechanism for playbooks to signal completion
3. **Infinite Loop Design**: Auto Run is designed for continuous operation (Loop ∞) without built-in exit conditions
4. **Agent Compliance**: The agent follows the playbook literally ("mark all tasks complete") without the ability to affect Maestro's control flow

## Impact

1. **Wasted compute**: Agents continue running, consuming API tokens and compute resources
2. **User confusion**: "Why is it still running if it says complete?"
3. **No graceful exit**: Users must manually stop Auto Run
4. **Task count noise**: Task counters keep incrementing but represent no real work

## Recommended Fixes

### Option 1: Max Loops Configuration (User-Side Mitigation)
Configure a reasonable max loop limit based on expected feature count.

**Pro**: Immediate, no code changes needed
**Con**: Arbitrary limit, may stop prematurely or too late

### Option 2: Maestro Feature Request - Exit File Detection
Request Maestro to check for a sentinel file (e.g., `.maestro/PLAYBOOK_COMPLETE`) and auto-stop when found.

**Implementation**:
- Maestro checks for `.maestro/PLAYBOOK_COMPLETE` before starting next loop
- If file exists and contains "EXIT", stop Auto Run gracefully
- Agent writes this file when ALL_FEATURES_COMPLETE is set

### Option 3: Maestro Feature Request - Document Exit Code
Allow documents to return an exit code that Maestro interprets.

**Implementation**:
- Special markdown pattern: `<!-- MAESTRO:EXIT -->`
- Maestro parses this at end of document processing
- Stops Auto Run without incrementing loop counter

### Option 4: SpecFlow Command Integration
Add `specflow should-exit` command that Maestro could call between loops.

**Implementation**:
- SpecFlow checks if any pending features remain
- Returns exit code 0 (continue) or 1 (complete)
- Maestro calls this before starting next loop

## Immediate Workaround

Users should:
1. Configure a reasonable max loop limit (e.g., `features * 3`)
2. Monitor the loop counter and manually stop when ALL_FEATURES_COMPLETE appears
3. Check `.maestro/CURRENT_FEATURE.md` periodically for completion state

## Action Items

- [x] File Maestro feature request for exit condition support → [Issue #231](https://github.com/pedramamini/Maestro/issues/231)
- [ ] Update playbook README to clarify current exit behavior
- [ ] Consider adding "STOP HERE" visual indicators in 6_COMPLETE.md
- [ ] Document recommended max loop settings per feature count

---

## Appendix: Maestro App Info

- **Version**: 0.14.4
- **Bundle ID**: com.maestro.app
- **Author**: Pedram Amini
- **Type**: Electron app (macOS)

The app uses an Electron framework with the main logic in `Resources/app.asar`. Modifying exit behavior would require changes to the Maestro application itself.
