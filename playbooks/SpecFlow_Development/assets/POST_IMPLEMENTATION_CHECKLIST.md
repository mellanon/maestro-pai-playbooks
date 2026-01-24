# Post-Implementation Checklist

## Docker Compose Stack (F-15)

### Persistent Storage Validation

Before declaring F-15 complete, validate persistent storage works correctly:

```bash
# 1. Verify named volumes are created
docker volume ls | grep observability
# Expected: 4 volumes (vm-data, vmlogs-data, vmtraces-data, grafana-data)

# 2. Write test data to VictoriaMetrics
curl -d 'test_metric{label="test"} 123' http://localhost:8428/api/v1/import/prometheus

# 3. Query to confirm data written
curl 'http://localhost:8428/api/v1/query?query=test_metric'

# 4. Stop and remove containers (but NOT volumes)
cd ~/observability && docker compose down

# 5. Verify volumes still exist
docker volume ls | grep observability
# Expected: All 4 volumes still present

# 6. Restart stack
docker compose up -d

# 7. Query to confirm data persisted
curl 'http://localhost:8428/api/v1/query?query=test_metric'
# Expected: Same data as step 3
```

### Volume Backup Consideration

For long-term data protection:

```bash
# Export volumes for backup
docker run --rm -v observability-vm-data:/data -v $(pwd):/backup alpine tar cvf /backup/vm-data-backup.tar /data

# Restore from backup
docker run --rm -v observability-vm-data:/data -v $(pwd):/backup alpine tar xvf /backup/vm-data-backup.tar -C /
```

### OrbStack-Specific Notes

- OrbStack stores volumes in `~/.orbstack/data/volumes/`
- Volume data survives OrbStack restarts
- Use `orb` CLI for OrbStack-specific management if needed

---

## Vector Collector (F-11, F-12, F-13)

### Vector Service Health

```bash
# Check Vector is tailing files
curl http://localhost:8686/health

# Check Vector internal metrics
curl http://localhost:8686/api/v1/metrics

# Verify log ingestion to VictoriaLogs
curl 'http://localhost:9428/select/logsql/query?query=*'
```

### Watchdog Validation (F-13)

```bash
# Simulate Vector failure
docker stop vector

# Wait for watchdog alert (check configured notification channel)

# Restart and verify recovery
docker start vector
```

---

## General Post-Implementation Items

- [ ] All services respond to health checks
- [ ] Grafana datasources show "Working" status
- [ ] Data persists across `docker compose down/up` cycle
- [ ] RAM usage < 600MB total (`docker stats --no-stream`)
- [ ] Ports are accessible only on localhost (security)
- [ ] .env file created from .env.example with real password

---

## OTLP Hook Instrumentation (F-017)

### Trace Generation Validation

```bash
# 1. Verify VictoriaTraces OTLP ports are exposed
docker compose ps | grep victoriatraces
# Expected: Ports 4317 (gRPC), 4318 (HTTP), 10428 (Jaeger API)

# 2. Start a new Claude session to generate traces
claude --print "Hello, testing OTLP traces"
# Expected: Session creates root span

# 3. Query VictoriaTraces for traces via Jaeger API
curl 'http://localhost:10428/api/traces?service=pai-hooks&limit=5'
# Expected: Trace data with session spans

# 4. Check trace_id in JSONL events
cat ~/.claude/MEMORY/Events/$(date +%Y-%m-%d).jsonl | jq 'select(.trace_id != null)' | head -5
# Expected: Events with trace_id and span_id fields
```

### Trace Hierarchy Validation

```bash
# 1. Create session with tool calls (spawns child spans)
claude --print "Read the file README.md"
# Expected: session.start → tool.use spans in hierarchy

# 2. Verify parent-child span relationships in Grafana
# Open: http://localhost:3000/explore
# Select VictoriaTraces datasource
# Query recent traces
# Expected: Waterfall view shows session → tool hierarchy

# 3. Test agent spawn creates child trace
claude --print "Use the Explore agent to find TypeScript files"
# Expected: agent.spawn creates child span linked to parent
```

### Trace-Log Correlation

```bash
# 1. Find a trace_id from recent logs
TRACE_ID=$(cat ~/.claude/MEMORY/Events/$(date +%Y-%m-%d).jsonl | jq -r 'select(.trace_id != null) | .trace_id' | head -1)

# 2. Query VictoriaLogs with that trace_id
curl "http://localhost:9428/select/logsql/query?query=trace_id:$TRACE_ID"
# Expected: All events from that trace returned

# 3. Query VictoriaTraces with same trace_id
curl "http://localhost:10428/api/traces/$TRACE_ID"
# Expected: Full trace tree returned
```

### Acceptance Criteria Checklist (F-017)

- [ ] VictoriaTraces exposes OTLP ports (4317, 4318)
- [ ] session.start creates root span with trace_id
- [ ] tool.use creates child spans under session
- [ ] agent.spawn creates child spans linked to parent
- [ ] JSONL events include trace_id and span_id fields
- [ ] Grafana shows trace waterfall visualization
- [ ] Can jump from log event to full trace

---

## Session Environment Tracking (F-018)

### Environment Metadata Validation

```bash
# 1. Check current PAI environment
echo "PAI_DIR: $PAI_DIR"
cat ~/.pai-env
# Note the current PAI version

# 2. Start a session and check event metadata
claude --print "Test session environment"

# 3. Verify environment fields in session.start event
cat ~/.claude/MEMORY/Events/$(date +%Y-%m-%d).jsonl | \
  jq 'select(.event_type == "session.start") | .data | {pai_version, pai_dir, git_branch, git_commit}' | \
  tail -1
# Expected: All four fields populated
```

### Cross-Environment Query

```bash
# 1. Query sessions by PAI version
cat ~/.claude/MEMORY/Events/*.jsonl | \
  jq -r 'select(.event_type == "session.start" and .data.pai_version != null) |
         "\(.session_id) | \(.data.pai_version) | \(.data.git_branch)"' | \
  sort -u | head -10
# Expected: Sessions with version metadata

# 2. Find sessions from specific PAI version
PAI_VERSION="signal-agent-2"
cat ~/.claude/MEMORY/Events/*.jsonl | \
  jq "select(.event_type == \"session.start\" and .data.pai_version == \"$PAI_VERSION\")" | \
  jq -r '.session_id' | wc -l
# Expected: Count of sessions from that version
```

### pai-switch Test

```bash
# 1. Note current version
echo "Before: $PAI_DIR"

# 2. Start session, note session_id
claude --print "Session in $(basename $PAI_DIR)"
# Note the session_id from the event

# 3. Switch PAI version (if multiple versions available)
# pai-switch <other-version>
# echo "After: $PAI_DIR"

# 4. Start another session
# claude --print "Session in $(basename $PAI_DIR)"

# 5. Query both sessions show different pai_version
cat ~/.claude/MEMORY/Events/$(date +%Y-%m-%d).jsonl | \
  jq 'select(.event_type == "session.start") | {session_id, pai_version: .data.pai_version}' | \
  tail -4
# Expected: Different pai_version values
```

### Graceful Fallback Test

```bash
# 1. Test with git unavailable (in non-git directory)
cd /tmp
claude --print "Session outside git repo"

# 2. Check event still logs without crashing
cat ~/.claude/MEMORY/Events/$(date +%Y-%m-%d).jsonl | \
  jq 'select(.event_type == "session.start") | .data' | tail -1
# Expected: pai_dir populated, git_branch may be null, no crash
```

### Acceptance Criteria Checklist (F-018)

- [ ] session.start events include pai_version field
- [ ] session.start events include pai_dir field
- [ ] session.start events include git_branch field
- [ ] session.start events include git_commit field
- [ ] Works when PAI_DIR is a symlink (resolves correctly)
- [ ] Graceful fallback if git unavailable (no crash, null values)
- [ ] Cross-environment query finds sessions from all versions

---

## End-to-End Stack Validation (L4+)

### Full Pipeline Test

```bash
# 1. Restart stack fresh
cd ~/observability && docker compose down && docker compose up -d

# 2. Wait for services
sleep 10

# 3. Start instrumented session
claude --print "Full pipeline test with traces and environment tracking"

# 4. Verify all paths working:
# - JSONL event written
cat ~/.claude/MEMORY/Events/$(date +%Y-%m-%d).jsonl | tail -5

# - Vector ingested to VictoriaLogs
curl 'http://localhost:9428/select/logsql/query?query=*&limit=5'

# - Trace created in VictoriaTraces
curl 'http://localhost:10428/api/services'
# Expected: "pai-hooks" service listed

# - Environment metadata captured
cat ~/.claude/MEMORY/Events/$(date +%Y-%m-%d).jsonl | \
  jq 'select(.event_type == "session.start") | {trace_id, pai_version: .data.pai_version}' | \
  tail -1
# Expected: Both trace_id and pai_version present
```

---

Created: 2026-01-24
Updated: 2026-01-25 (Added F-017 OTLP and F-018 Environment Tracking tests)
Purpose: Manual validation checklist for observability stack features
