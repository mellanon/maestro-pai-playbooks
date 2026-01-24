# Acceptance Tests for PAI Signal Observability Stack

**Purpose:** Manual validation checklist for testing the observability stack implementation before creating a PR. Tests are organized by stack level, from foundation to full integration.

**Usage:** Switch to the feature branch using `pai-switch`, then work through each level.

---

## Test Prerequisites

```bash
# Switch to the feature branch
pai-switch signal-1  # or signal-2

# Verify you're in the right branch
git branch --show-current
# Expected: feature/signal-agent-1 (or signal-agent-2)

# Source the environment if needed
source ~/.pai-env
```

---

## Level 1: Foundation (F-1, F-2)

### L1.1: Event Schema Types

**Goal:** Verify TypeScript types compile and exports work.

```bash
cd $PAI_DIR

# Check types compile
bun -e "import { PAIEvent, EventType, SESSION_START, SKILL_INVOKE } from './Observability/lib/events'; console.log('EventType constants:', { SESSION_START, SKILL_INVOKE })"
```

**Expected:**
- No compile errors
- Outputs: `EventType constants: { SESSION_START: 'session.start', SKILL_INVOKE: 'skill.invoke' }`

- [ ] **PASS** - Types compile and export correctly

### L1.2: Type Guards

**Goal:** Verify type guards work at runtime.

```bash
bun -e "
import { isPAIEvent, createSessionStartEvent } from './Observability/lib/events';

const validEvent = createSessionStartEvent('test', 'session-123', { model: 'opus', working_dir: '/tmp' });
const invalidEvent = { foo: 'bar' };

console.log('Valid event check:', isPAIEvent(validEvent));
console.log('Invalid event check:', isPAIEvent(invalidEvent));
"
```

**Expected:**
- `Valid event check: true`
- `Invalid event check: false`

- [ ] **PASS** - Type guards validate correctly

### L1.3: Event Logging to JSONL

**Goal:** Verify logEvent() writes to correct file.

```bash
# Create a test event
bun -e "
import { logEvent, createSessionStartEvent } from './Observability/lib/events';

const event = createSessionStartEvent('test', 'test-session-$(date +%s)', { model: 'test', working_dir: '/tmp' });
logEvent(event);
console.log('Event logged');
"

# Verify file was created
TODAY=$(date +%Y-%m-%d)
ls -la $PAI_DIR/MEMORY/Events/$TODAY.jsonl

# Check last line is valid JSON
tail -1 $PAI_DIR/MEMORY/Events/$TODAY.jsonl | jq .
```

**Expected:**
- File exists at expected path
- Last line is valid JSON with `event_type: "session.start"`

- [ ] **PASS** - Events written to date-partitioned JSONL

### L1.4: Observability Toggle

**Goal:** Verify enable/disable toggle works.

```bash
# Check current setting
jq '.observability' $PAI_DIR/settings.json

# If enabled, events should be written
# If disabled (false), logEvent should be a no-op
```

**Expected:**
- Setting exists (or defaults to enabled)
- Behavior matches setting

- [ ] **PASS** - Toggle controls logging behavior

---

## Level 2: Hook Instrumentation (F-5, F-6, F-7, F-8, F-9)

### L2.1: SessionStart Event

**Goal:** Verify session.start event is emitted when Claude starts.

```bash
# Start a new Claude session (in a separate terminal)
claude

# After session starts, check for event
grep "session.start" $PAI_DIR/MEMORY/Events/$(date +%Y-%m-%d).jsonl | tail -1 | jq .
```

**Expected:**
- Event with `event_type: "session.start"`
- Has `session_id`, `data.model`, `data.working_dir`

- [ ] **PASS** - SessionStart hook emits event

### L2.2: Tool Use Events

**Goal:** Verify PreToolUse and PostToolUse events are captured.

```bash
# In a Claude session, run a simple command
# Then check for tool events
grep "tool\." $PAI_DIR/MEMORY/Events/$(date +%Y-%m-%d).jsonl | tail -5 | jq -s '.[] | {event_type, tool: .data.tool}'
```

**Expected:**
- `tool.use` event before tool execution
- `tool.result` event after tool execution
- Same tool name in both

- [ ] **PASS** - Tool use events captured

### L2.3: Hook Timing

**Goal:** Verify hook phase timing is captured.

```bash
grep "hook\." $PAI_DIR/MEMORY/Events/$(date +%Y-%m-%d).jsonl | jq -s '.[] | {event_type, phase: .data.phase, duration_ms: .data.duration_ms}'
```

**Expected:**
- Events with `event_type: "hook.phase"` or `"hook.complete"`
- `duration_ms` is a reasonable number (< 1000ms typically)

- [ ] **PASS** - Hook timing instrumentation works

### L2.4: Session Correlation

**Goal:** Verify all events share the same session_id.

```bash
# Get session_id from most recent session.start
SESSION_ID=$(grep "session.start" $PAI_DIR/MEMORY/Events/$(date +%Y-%m-%d).jsonl | tail -1 | jq -r .session_id)

# Count events with that session_id
grep "$SESSION_ID" $PAI_DIR/MEMORY/Events/$(date +%Y-%m-%d).jsonl | wc -l

# Show distinct event types in session
grep "$SESSION_ID" $PAI_DIR/MEMORY/Events/$(date +%Y-%m-%d).jsonl | jq -r .event_type | sort | uniq -c
```

**Expected:**
- Multiple events share the same session_id
- Various event types present (session.start, tool.use, tool.result, etc.)

- [ ] **PASS** - Session correlation working

---

## Level 3: Data Pipeline (F-3, F-4, F-14)

### L3.1: Concurrent Write Safety

**Goal:** Verify concurrent writes don't corrupt the file.

```bash
# Run parallel writes
for i in {1..10}; do
  bun -e "
import { logEvent, createSessionStartEvent } from './Observability/lib/events';
const event = createSessionStartEvent('concurrent-test', 'session-$i', { model: 'test', working_dir: '/tmp' });
logEvent(event);
" &
done
wait

# Verify all lines are valid JSON
wc -l $PAI_DIR/MEMORY/Events/$(date +%Y-%m-%d).jsonl
cat $PAI_DIR/MEMORY/Events/$(date +%Y-%m-%d).jsonl | while read line; do echo "$line" | jq . > /dev/null || echo "INVALID: $line"; done
```

**Expected:**
- No "INVALID" lines printed
- All lines parse as valid JSON

- [ ] **PASS** - Concurrent writes safe

### L3.2: PII Scrubbing (if implemented)

**Goal:** Verify sensitive data is scrubbed.

```bash
# Check for any events with potential PII
grep -i "password\|secret\|token\|api_key" $PAI_DIR/MEMORY/Events/$(date +%Y-%m-%d).jsonl || echo "No PII patterns found"
```

**Expected:**
- No sensitive patterns in event data
- Or patterns are redacted (e.g., `[REDACTED]`)

- [ ] **PASS** - PII scrubbing active (or N/A if not implemented)

### L3.3: Log Rotation

**Goal:** Verify daily rotation creates new files.

```bash
# List event files
ls -la $PAI_DIR/MEMORY/Events/*.jsonl | head -10
```

**Expected:**
- Files named YYYY-MM-DD.jsonl
- Multiple days if system has been running

- [ ] **PASS** - Date-partitioned files created

---

## Level 4: Stack Infrastructure (F-11, F-13, F-15)

### L4.1: Docker Compose Stack

**Goal:** Verify all services start.

```bash
cd ~/observability
docker compose up -d
docker compose ps
```

**Expected:**
- All services "Up" (vector, victoriametrics, victorialogs, grafana)
- No restart loops

- [ ] **PASS** - Docker stack starts

### L4.2: Service Health Checks

**Goal:** Verify each service responds to health endpoints.

```bash
# VictoriaMetrics
curl -s http://localhost:8428/-/healthy && echo " VM: OK"

# VictoriaLogs
curl -s http://localhost:9428/health && echo " VLogs: OK"

# Vector
curl -s http://localhost:8686/health && echo " Vector: OK"

# Grafana
curl -s http://localhost:3000/api/health && echo " Grafana: OK"
```

**Expected:**
- All return OK or healthy status

- [ ] **PASS** - All health checks pass

### L4.3: Vector Event Ingestion

**Goal:** Verify Vector is tailing JSONL files and shipping to VictoriaLogs.

```bash
# Create a test event
bun -e "
import { logEvent, createSessionStartEvent } from './Observability/lib/events';
logEvent(createSessionStartEvent('ingestion-test', 'ingest-$(date +%s)', { model: 'test', working_dir: '/tmp' }));
"

# Wait for ingestion
sleep 5

# Query VictoriaLogs
curl -s 'http://localhost:9428/select/logsql/query?query=event_type:session.start' | head -5
```

**Expected:**
- Test event appears in VictoriaLogs query results

- [ ] **PASS** - Vector ingestion working

### L4.4: Grafana Data Sources

**Goal:** Verify Grafana can query both data sources.

```bash
# Open Grafana
open http://localhost:3000

# Check: Configuration → Data Sources
# Both VictoriaMetrics and VictoriaLogs should show "Working"
```

**Expected:**
- VictoriaMetrics datasource: Working
- VictoriaLogs datasource: Working

- [ ] **PASS** - Grafana datasources configured

### L4.5: Data Persistence

**Goal:** Verify data survives container restarts.

```bash
# Write test data to VictoriaMetrics
curl -d 'test_persistence{label="test"} 123' http://localhost:8428/api/v1/import/prometheus

# Query to confirm
curl 'http://localhost:8428/api/v1/query?query=test_persistence'

# Restart stack
cd ~/observability
docker compose down
docker compose up -d

# Query again
sleep 10
curl 'http://localhost:8428/api/v1/query?query=test_persistence'
```

**Expected:**
- Data present before restart
- Same data present after restart

- [ ] **PASS** - Data persists across restarts

### L4.6: Resource Usage

**Goal:** Verify stack stays within resource limits.

```bash
docker stats --no-stream --format "table {{.Name}}\t{{.MemUsage}}\t{{.CPUPerc}}"
```

**Expected:**
- Total RAM < 600MB
- CPU usage minimal when idle

- [ ] **PASS** - Resource usage acceptable

---

## Level 5: End-to-End Integration

### L5.1: Full Session Trace

**Goal:** Trace a complete session from start to Grafana dashboard.

1. Start Claude session
2. Run a few commands (use tools)
3. End session
4. Query session in Grafana

```bash
# Get most recent session_id
SESSION_ID=$(grep "session.start" $PAI_DIR/MEMORY/Events/$(date +%Y-%m-%d).jsonl | tail -1 | jq -r .session_id)
echo "Session: $SESSION_ID"

# Query all events in VictoriaLogs
curl -s "http://localhost:9428/select/logsql/query?query=session_id:$SESSION_ID" | jq -r '.[] | "\(.ts) \(.event_type)"'
```

**Expected:**
- session.start at beginning
- tool.use / tool.result events
- hook.phase / hook.complete events
- Complete timeline visible

- [ ] **PASS** - Full session trace works

### L5.2: CLI Query Patterns

**Goal:** Verify CLI queries work as documented.

```bash
# Count events by type today
jq -r '.event_type' $PAI_DIR/MEMORY/Events/$(date +%Y-%m-%d).jsonl | sort | uniq -c | sort -rn

# Find slow hooks (> 100ms)
jq -r 'select(.event_type == "hook.complete" and .data.duration_ms > 100) | "\(.data.hook): \(.data.duration_ms)ms"' $PAI_DIR/MEMORY/Events/$(date +%Y-%m-%d).jsonl

# Count tool usage
jq -r 'select(.event_type == "tool.use") | .data.tool' $PAI_DIR/MEMORY/Events/$(date +%Y-%m-%d).jsonl | sort | uniq -c | sort -rn | head -10
```

**Expected:**
- Queries run without errors
- Results are meaningful

- [ ] **PASS** - CLI query patterns work

---

## Test Summary

| Level | Tests | Pass | Fail |
|-------|-------|------|------|
| L1: Foundation | 4 | | |
| L2: Hooks | 4 | | |
| L3: Pipeline | 3 | | |
| L4: Infrastructure | 6 | | |
| L5: Integration | 2 | | |
| **Total** | **19** | | |

## Notes

Record any issues, observations, or deviations here:

```
[Date] [Level] [Test] - [Observation]


```

---

## After Testing

If all tests pass:
1. Return to the worktree directory
2. Run the SpecFlow_PR_Creator playbook
3. Review generated artifacts
4. Execute PR_COMMAND.sh to create PR

If tests fail:
1. Document failures in Notes section
2. Fix issues before creating PR
3. Re-run failed tests

---

*Part of SpecFlow PR Creator Playbook*
