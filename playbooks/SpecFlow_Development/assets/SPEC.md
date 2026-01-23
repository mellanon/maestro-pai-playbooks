---
generation_date: 2026-01-22 14:45
updated: 2026-01-23 15:00
tags:
  - type/requirements
  - para/project
  - topic/observability
  - topic/pai
  - topic/architecture
  - lifeos/awareness
  - project/lifeos
  - git/track
  - scope/personal
source: research-session
codename: Signal
---

# PAI Signal Architecture

> *Codename: **Signal** — Advanced observability stack built on existing JSONL infrastructure.*

---

## Relationship to PAI Observability Server

> **Important:** The PAI Observability Server (basic, out of box) reads JSONL Events directly. The PAI Signal Stack is a separate, more advanced pipeline using Vector Collector to feed pluggable backends.

| System | PAI Observability Server | PAI Signal Stack (This Spec) |
|--------|--------------------------|------------------------------|
| **Fed From** | Direct read from JSONL Events | Vector Collector |
| **Focus** | Current session visibility | Cross-session analysis & alerting |
| **Analogy** | Browser DevTools | Datadog/Grafana |
| **Scope** | Single PAI instance | Multi-system (PAI + home automation + off-grid) |
| **Storage** | In-memory (1000 events) | Persistent (weeks/months) |
| **Use Case** | "What's happening now?" | "What patterns emerge over time?" |
| **Complexity** | Basic, out of box | Advanced, pluggable backends |

### How They Work Together

```
Event Sources ──► JSONL Events
                       │
                       ├────► PAI Observability Server (direct read)
                       │      (basic, out of box, current session)
                       │
                       ▼
              ┌─────────────────────────────────────────┐
              │  PAI SIGNAL STACK                       │
              │                                         │
              │  Vector Collector                       │
              │        │                                │
              │        ▼                                │
              │  Observability Stack (pluggable)        │
              │  • VictoriaMetrics / LGTM / Cloud       │
              │        │                                │
              │        ▼                                │
              │     Grafana (dashboards + alerting)     │
              └─────────────────────────────────────────┘
```

**Both paths are near real-time.** The hooks write JSONL files that:
1. PAI Observability Server reads directly (basic current session visibility)
2. Vector Collector tails and transforms for the Signal Stack (advanced cross-session analysis)

**Open question:** JSONL format may need enhancement for Signal Stack requirements (richer metadata, correlation IDs, etc.).

---

## Core Building Blocks for Trusted AI Agents

Based on AWS Bedrock Agent Core patterns, here are the foundational capabilities for autonomous AI systems:

| Building Block | Purpose | Signal Coverage |
|---------------|---------|-----------------|
| **Policy** | Enforceable boundaries, not suggestions | Via hook validation |
| **Identity** | Clear agency & delegation chains | session_id, agent_instance_id |
| **Memory** | Context without poisoning risk | JSONL audit trail |
| **Observability** | Full audit trail of decisions and actions | ✅ This spec |
| **Evaluations** | Continuous quality assurance | Quality gates in SpecFirst |
| **Runtime** | Isolation prevents cross-contamination | Container-based stack |
| **Gateway** | Controlled blast radius for tools | Hook-level gating |

---

## Executive Summary

**What:** The PAI Signal Stack is an advanced observability pipeline—Vector Collector consuming JSONL Events and forwarding to pluggable backends (VictoriaMetrics, Grafana LGTM, or cloud services). It complements the basic PAI Observability Server which reads JSONL directly.

**Why:** PAI agents run autonomously—you need to know what's happening, what's failing, and what it costs. The basic Observability Server gives you current session visibility. The Signal Stack adds cross-session analysis, alerting, and multi-system aggregation.

**How:** Three-layer architecture (from nothing to full stack):

| Layer | What You Get | Complexity |
|-------|--------------|------------|
| **Stage 0: CLI Only** | JSONL files + grep/jq queries | Zero setup |
| **Stage 1: Local Stack** | Grafana dashboards, historical queries | `docker compose up` |
| **Stage 2+: Distributed** | Multi-machine, alerting, cloud backends | Production-ready |

**Quick Start (Stage 0):**
```bash
# Real-time event stream (no stack needed)
tail -f $PAI_HOME/MEMORY/Events/$(date +%Y-%m-%d).jsonl | jq

# Find all errors today
grep '"event_type":"skill.error"' $PAI_HOME/MEMORY/Events/$(date +%Y-%m-%d).jsonl | jq
```

**Quick Start (Stage 1):**
```bash
# Start observability stack
cd ~/observability && docker compose up -d

# Open Grafana
open http://localhost:3000
```

---

## Architecture Diagram

![PAI Signal Architecture](assets/pai-signal-architecture-v5.png)

*PAI Signal Architecture: JSONL Events feed both the basic PAI Observability Server (direct read, current session) and the PAI Signal Stack (Vector Collector → pluggable Observability Stack for cross-session analysis).*

---

## Infrastructure Choices

Before diving into architecture, here are the key infrastructure decisions and their rationale.

### Container Runtime: OrbStack vs Docker Desktop

| Factor | OrbStack | Docker Desktop |
|--------|----------|----------------|
| **RAM usage** | ~300MB total | ~2GB |
| **Battery impact** | Low | High |
| **Startup time** | ~2 seconds | ~10-15 seconds |
| **License** | Free for personal use | Free tier with limits |
| **Docker compatibility** | Full | Full |

**Decision:** OrbStack is the recommended runtime. It's a drop-in replacement—same `docker` and `docker compose` commands work identically, but with dramatically lower resource usage.

```bash
# Install OrbStack (one-time)
brew install --cask orbstack

# Open to initialize
open -a OrbStack
```

### Observability Stack: VictoriaMetrics vs Grafana LGTM

Both can run locally in Docker. Here's why we chose VictoriaMetrics:

| Factor | VictoriaMetrics Stack | Grafana LGTM Stack |
|--------|----------------------|-------------------|
| **Components** | VM + VictoriaLogs + VictoriaTraces + Grafana | Loki + Mimir + Tempo + Grafana |
| **Total RAM** | ~350-450MB | ~800MB-1.2GB |
| **Disk efficiency** | 15x better compression (VictoriaLogs vs Loki) | Standard |
| **Query speed** | Optimized for high-cardinality | Good |
| **Configuration** | Minimal (sane defaults) | More knobs |
| **Community** | Growing | Large, mature |

**Grafana LGTM Stack** (Loki + Grafana + Tempo + Mimir) is the "native" Grafana experience. If you're already invested in the Grafana ecosystem or prefer it:

```yaml
# Alternative: Grafana LGTM docker-compose.yml
services:
  loki:
    image: grafana/loki:latest
    ports: ["3100:3100"]

  tempo:
    image: grafana/tempo:latest
    ports: ["3200:3200", "4317:4317"]

  mimir:
    image: grafana/mimir:latest
    ports: ["9009:9009"]

  grafana:
    image: grafana/grafana:latest
    ports: ["3000:3000"]
    depends_on: [loki, tempo, mimir]
```

**Decision:** VictoriaMetrics stack is the default because:
- Lower resource footprint (PAI prioritizes battery life)
- Single-binary simplicity (easier to troubleshoot)
- Prometheus-compatible (easy migration path)
- All components from same vendor (consistent APIs)

Both work with the same Grafana frontend—dashboards are portable.

### Collector: curl/launchd vs Vector vs OTel Collector

| Option | Dependencies | Visibility | Footprint | Best For |
|--------|--------------|------------|-----------|----------|
| **launchd + curl** | None (macOS built-in) | Low | ~0MB | Minimal, "it just works" |
| **Vector** | `brew install` | High (`vector top`) | ~50MB | Debugging, production |
| **OTel Collector** | Binary or container | High | ~100MB | OTLP ecosystem, enterprise |

**launchd + curl** is invisible—it runs, it works, you don't think about it. Failures are silent unless you check logs. Good for "set and forget."

**Vector** provides excellent observability of itself—`vector top` shows real-time throughput, `vector tap` lets you debug the pipeline. Better for troubleshooting.

**OTel Collector** makes sense if you're standardizing on OpenTelemetry across your stack, or need OTLP protocol support.

**Decision:** Start with launchd + curl (zero dependencies). Upgrade to Vector if you need better visibility into the collector itself.

### Understanding the Data Types: Logs/Traces vs Metrics

Both are timestamped, but the **data type** is fundamentally different:

- **Logs & Traces** = **Textual** (strings, JSON, structured events)
- **Metrics** = **Numeric** (integers, floats, counters, gauges)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    OBSERVABILITY DATA TYPES                                  │
│                                                                              │
│   LOGS & TRACES (Textual Events)          METRICS (Numeric Values)          │
│   ──────────────────────────────          ────────────────────────          │
│   "What happened?"                        "How much/how many?"              │
│                                                                              │
│   • Textual: JSON, strings, context       • Numeric: integers, floats       │
│   • Individual events with rich detail    • Aggregated numbers over time    │
│   • High cardinality (unique IDs)         • Low cardinality (labels)        │
│   • Variable size (bytes to KB)           • Fixed size (~8 bytes/point)     │
│   • Query: "Show me this session"         • Query: "Avg duration last hour" │
│   • Filter: regex, LogsQL                 • Aggregate: sum, avg, percentiles│
│                                                                              │
│   Examples:                               Examples:                          │
│   ├── session.start (session_id=abc)      ├── pai_sessions_total: 47        │
│   ├── skill.error (stack trace, context)  ├── pai_session_duration_avg: 12.3│
│   ├── tool.result (input, output)         ├── pai_errors_total: 3           │
│   └── api.call (request, response)        └── pai_api_cost_usd: 4.82        │
│                                                                              │
│   Storage:                                Storage:                           │
│   └── VictoriaLogs (logs)                 └── VictoriaMetrics (Prometheus)  │
│   └── VictoriaTraces (distributed)        └── PromQL queries                │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**When to use each:**

| Use Case | Data Type | Why |
|----------|-----------|-----|
| "What happened in session X?" | Logs | Need full event detail |
| "Why did the _JIRA skill fail?" | Logs + Traces | Need context, stack trace |
| "How many sessions today?" | Metrics | Aggregate count |
| "What's the p95 session duration?" | Metrics | Statistical analysis |
| "Track API spend over time" | Metrics | Trend analysis, alerting |
| "Find all rate limit errors" | Logs | Search by content |

**The flow:**

```
Events (JSONL)                    Metrics (derived)
─────────────                     ─────────────────
{"event_type":"session.start"}
{"event_type":"session.end",      ──► pai_session_duration_seconds
 "data":{"duration_ms":12345}}        histogram

{"event_type":"skill.error"}      ──► pai_skill_errors_total
{"event_type":"skill.error"}          counter (labels: skill, code)

{"event_type":"api.cost",         ──► pai_api_cost_usd_total
 "data":{"cost_usd":0.42}}            counter (labels: model, skill)
```

**PAI's approach:**
1. **Capture events first** — JSONL files are the source of truth
2. **Derive metrics when needed** — Collector transforms events into metrics
3. **Query both** — Grafana queries VictoriaLogs (events) and VictoriaMetrics (metrics)

This means you don't have to instrument twice. Write events, get metrics for free (via transformation).

### Event-to-Metric Transformation

The collector layer is responsible for turning textual events into numeric metrics. This happens at collection time, not at query time:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    EVENT → METRIC TRANSFORMATION                            │
│                                                                              │
│   JSONL Event (textual)                   Prometheus Metric (numeric)       │
│   ─────────────────────                   ───────────────────────────       │
│                                                                              │
│   {"event": "session.end",          →     pai_session_duration_seconds{     │
│    "session_id": "abc123",                  skill="Browser"                 │
│    "skill": "Browser",                    } 847                             │
│    "duration_seconds": 847}                                                 │
│                                                                              │
│   {"event": "skill.error",          →     pai_skill_errors_total{           │
│    "skill": "_JIRA",                        skill="_JIRA",                  │
│    "error_code": "AUTH_FAILED"}             error_code="AUTH_FAILED"        │
│                                           } 1  (counter increment)          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**How it works:**

| Component | Role | Transformation |
|-----------|------|----------------|
| **Collector (launchd/Vector)** | Reads JSONL events | Extracts numeric fields (duration_seconds, cost_usd) |
| **Vector `log_to_metric` transform** | Converts events to Prometheus format | Maps JSON fields → metric labels + values |
| **VictoriaMetrics** | Stores numeric time series | Receives Prometheus exposition format |

**Transformation rules:**
- **Counters**: Increment on each event occurrence (`pai_sessions_total`, `pai_errors_total`)
- **Gauges**: Use the numeric value directly (`pai_session_duration_seconds`)
- **Labels**: Extract from event fields (`skill`, `error_code`, `model`)

→ *Full Vector transform configuration: See Appendix A*

### PAI Integration Architecture: How Observability is Packaged

Observability in PAI follows a three-tier distribution model:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    PAI OBSERVABILITY DISTRIBUTION                            │
│                                                                              │
│   TIER 1: Core PAI (Always Present)                                         │
│   ─────────────────────────────────                                         │
│   Built into PAI itself, zero configuration needed.                          │
│                                                                              │
│   Instrumentation Library:                                                   │
│   hooks/lib/events.ts          ← logEvent() function                        │
│   hooks/lib/skill-registry.ts  ← Skill registry for pattern matching        │
│   MEMORY/Events/               ← Event storage directory                    │
│                                                                              │
│   Core Hooks (instrumented):                                                 │
│   hooks/SessionStart.hook.ts   ← session.start event                        │
│   hooks/SessionStop.hook.ts    ← session.end event                          │
│   hooks/SkillEnforcer.hook.ts  ← skill.matched event (REQUIRED)             │
│   hooks/PreToolUse.hook.ts     ← tool.use event                             │
│   hooks/PostToolUse.hook.ts    ← tool.result event                          │
│                                                                              │
│   This tier works out of the box: tail -f Events/*.jsonl | jq               │
│                                                                              │
│   TIER 2: Observability Skill (Optional, Installable)                       │
│   ────────────────────────────────────────────────────                      │
│   Provides enhanced querying, stack management, dashboard deployment.        │
│                                                                              │
│   skills/Observability/SKILL.md                                             │
│   ├── USE WHEN: "show sessions", "query events", "observability status"     │
│   ├── Provides: pre-built jq queries, dashboard templates                   │
│   ├── Commands: "start stack", "stop stack", "show errors today"            │
│   └── References: this spec, docker-compose, collector configs              │
│                                                                              │
│   User installs this skill when they want the full observability workflow.   │
│                                                                              │
│   TIER 3: External Infrastructure (User-Managed)                            │
│   ──────────────────────────────────────────────                            │
│   The actual backends - managed outside PAI.                                 │
│                                                                              │
│   ~/observability/docker-compose.yml    ← VictoriaMetrics stack             │
│   ~/observability/grafana/              ← Grafana config and dashboards     │
│   ~/observability/.env                  ← Credentials                       │
│   OrbStack                              ← Container runtime                 │
│                                                                              │
│   PAI doesn't "own" this - it's infrastructure like any other service.      │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Why this separation?**

| Tier | Who | Maintenance | Dependencies |
|------|-----|-------------|--------------|
| **Core** | PAI | Updates with PAI | None |
| **Skill** | User | User installs/updates | Core PAI |
| **Infrastructure** | User | User deploys/scales | OrbStack, Docker |

**The Observability Skill:**

```yaml
---
name: Observability
description: PAI observability queries, stack management, and dashboards.
triggers:
  - keywords: [observability, events, sessions, errors, metrics, traces]
  - commands: ["/obs", "/events", "/sessions"]
---
```

**What the skill provides:**

1. **Query helpers:**
   - "Show errors today" → runs pre-built jq query
   - "Sessions this week" → aggregates and displays
   - "Slow hooks" → identifies performance issues

2. **Stack management:**
   - "Start observability stack" → `docker compose up -d`
   - "Stop stack" → `docker compose down`
   - "Stack status" → checks containers, heartbeat

3. **Dashboard deployment:**
   - "Deploy sessions dashboard" → imports JSON to Grafana
   - "Update dashboards" → syncs latest from PAI repo

4. **Troubleshooting:**
   - "Why is my terminal slow?" → queries hook timing events
   - "Debug _JIRA errors" → filters and displays relevant traces

**Example skill invocations:**

```
User: "Show me session errors from today"
→ Observability skill runs:
  grep '"event_type":"skill.error"' $PAI_HOME/MEMORY/Events/$(date +%Y-%m-%d).jsonl | jq

User: "Start the observability stack"
→ Observability skill runs:
  cd ~/observability && docker compose up -d

User: "What's consuming API cost?"
→ Observability skill runs:
  cat $PAI_HOME/MEMORY/Events/2026-01-*.jsonl | jq -s '
    [.[] | select(.event_type == "api.cost")] |
    group_by(.data.skill) |
    map({skill: .[0].data.skill, total: (map(.data.cost_usd) | add)}) |
    sort_by(.total) | reverse'
```

### Instrumentation Overhead

**The key insight:** Instrumentation code is always present in Tier 1, but **disabled by default**. This means:

- Zero performance impact when observability is off
- No configuration needed for users who don't want it
- Opt-in activation via `settings.json`

```typescript
// hooks/lib/events.ts
export function logEvent(event: PaiEvent): void {
  // Early exit when disabled - nanosecond cost (single boolean check)
  if (!config.observability?.enabled) return;

  // Only runs when enabled:
  // - JSON serialization: ~0.5-2ms
  // - File append: ~0.5-3ms
  // Total overhead: ~1-5ms per event
  const line = JSON.stringify({ ...event, ts: Date.now() }) + '\n';
  appendFileSync(`${EVENTS_DIR}/${today()}.jsonl`, line);
}
```

**Enable/disable in settings.json:**

```json
{
  "observability": {
    "enabled": false,         // Default: disabled (zero cost)
    "events_dir": "${PAI_HOME}/MEMORY/Events"
  }
}
```

**Performance characteristics:**

| State | Overhead | What happens |
|-------|----------|--------------|
| **Disabled** (default) | ~nanoseconds | Single `if` check, immediate return |
| **Enabled** | ~1-5ms/event | JSON serialize + file append |

With ~10-50 events per session, enabled overhead is ~10-250ms total—imperceptible in sessions that typically run minutes to hours.

### Log Rotation Strategy

The JSONL files are date-partitioned (`2026-01-22.jsonl`) but will grow indefinitely without cleanup. **Local retention** should match or exceed **backend retention** (90 days for VictoriaLogs).

**Recommended: launchd + find (macOS native)**

```bash
#!/bin/bash
# $PAI_HOME/Observability/rotate-events.sh
# Delete event files older than 90 days

EVENTS_DIR="${PAI_HOME}/MEMORY/Events"
RETENTION_DAYS=90

find "$EVENTS_DIR" -name "*.jsonl" -mtime +$RETENTION_DAYS -delete
find "$EVENTS_DIR/buffer" -name "*.jsonl" -mtime +$RETENTION_DAYS -delete
```

**launchd plist (runs daily at 3am):**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.pai.event-rotation</string>
    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>/Users/andreas/.claude/Observability/rotate-events.sh</string>
    </array>
    <key>StartCalendarInterval</key>
    <dict>
        <key>Hour</key>
        <integer>3</integer>
        <key>Minute</key>
        <integer>0</integer>
    </dict>
    <key>EnvironmentVariables</key>
    <dict>
        <key>PAI_HOME</key>
        <string>/Users/andreas/.claude</string>
    </dict>
</dict>
</plist>
```

**Why not newsyslog/logrotate?**

| Tool | Issue |
|------|-------|
| newsyslog | Designed for system logs, rotates by size not date |
| logrotate | Linux tool, not native to macOS |
| find + launchd | Simple, date-based, matches our file naming |

**Storage estimate:**

| Events/day | File size | 90-day total |
|------------|-----------|--------------|
| 100 | ~50KB | ~4.5MB |
| 500 | ~250KB | ~22MB |
| 1000 | ~500KB | ~45MB |

Negligible disk usage even at high activity levels.

---

**Distribution in PAI packs:**

For PAI packs (bundled skill collections), observability components distribute as:

```
pai-core-pack/
├── hooks/                         ← Tier 1: always included
│   ├── lib/
│   │   ├── events.ts              ← logEvent() function
│   │   └── skill-registry.ts      ← Skill pattern matching
│   ├── SessionStart.hook.ts       ← Instrumented
│   ├── SessionStop.hook.ts        ← Instrumented
│   ├── SkillEnforcer.hook.ts      ← Instrumented (REQUIRED)
│   ├── PreToolUse.hook.ts         ← Instrumented
│   └── PostToolUse.hook.ts        ← Instrumented
├── MEMORY/Events/                 ← Event storage
├── skills/Observability/          ← Tier 2: optional skill
│   ├── SKILL.md
│   └── queries/                   ← Pre-built query library
└── Observability/                 ← Tier 3: reference configs
    ├── docker-compose.yml
    ├── grafana/
    └── README.md                  ← "How to deploy"
```

Users can:
1. **Use Tier 1 only** — Events captured, query with CLI
2. **Add Tier 2** — Install skill for convenience
3. **Deploy Tier 3** — Full Grafana stack for historical analysis

This keeps the core lightweight while allowing progressive enhancement.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    PAI OBSERVABILITY ARCHITECTURE                            │
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                         PAI COMPONENTS                               │   │
│   │                                                                      │   │
│   │   Claude Code    Skills (28)    CLI Tools    Hooks (15)             │   │
│   │       │              │              │            │                   │   │
│   │       └──────────────┴──────────────┴────────────┘                   │   │
│   │                           │                                          │   │
│   │                    logEvent(event)                                   │   │
│   │                    (simple append)                                   │   │
│   │                           │                                          │   │
│   └───────────────────────────┼──────────────────────────────────────────┘   │
│                               ▼                                              │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                      LOCAL STORAGE                                   │   │
│   │                                                                      │   │
│   │   $PAI_HOME/MEMORY/Events/                                          │   │
│   │   ├── 2026-01-22.jsonl     ← Append-only, one file per day          │   │
│   │   ├── 2026-01-21.jsonl                                              │   │
│   │   └── ...                                                            │   │
│   │                                                                      │   │
│   │   Queryable immediately:                                             │   │
│   │   • tail -f 2026-01-22.jsonl | jq                                   │   │
│   │   • grep "skill.error" *.jsonl | jq                                 │   │
│   │   • cat *.jsonl | jq 'select(.event_type=="session.start")'         │   │
│   │                                                                      │   │
│   └───────────────────────────┬──────────────────────────────────────────┘   │
│                               │                                              │
│                        (optional forwarding)                                 │
│                               │                                              │
│   ┌───────────────────────────▼──────────────────────────────────────────┐   │
│   │                    DISTRIBUTED COLLECTOR                             │   │
│   │                    (runs alongside PAI)                              │   │
│   │                                                                      │   │
│   │   • Tails local files (no inbound connections)                      │   │
│   │   • Batches and forwards to observability stack                     │   │
│   │   • PUSHES out, nothing PULLS in (security)                         │   │
│   │   • Can run as launchd/systemd daemon                               │   │
│   │                                                                      │   │
│   └───────────────────────────┬──────────────────────────────────────────┘   │
│                               │                                              │
│                        (HTTPS POST outbound)                                 │
│                               │                                              │
│   ┌───────────────────────────▼──────────────────────────────────────────┐   │
│   │                    OBSERVABILITY STACK                               │   │
│   │                    (optional, decoupled)                             │   │
│   │                                                                      │   │
│   │   ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐   │   │
│   │   │VictoriaLogs │ │VictoriaMetr.│ │VictoriaTrace│ │   Grafana   │   │   │
│   │   │  (logs)     │ │  (metrics)  │ │  (traces)   │ │ (dashboards)│   │   │
│   │   │  :9428      │ │   :8428     │ │   :9411     │ │   :3000     │   │   │
│   │   └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘   │   │
│   │                                                                      │   │
│   │   • Historical queries     • Distributed tracing                     │   │
│   │   • Cross-session analysis • Alerting → Notifications                │   │
│   │   • Alerting → Notifications                                         │   │
│   │                                                                      │   │
│   └──────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Dashboard Examples

What you'll see in Grafana once the stack is running.

### PAI Sessions Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  PAI SESSIONS OVERVIEW                                    Last 24 hours ▼  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │   Sessions   │  │  Avg Length  │  │    Errors    │  │   API Cost   │    │
│  │      47      │  │   12.3 min   │  │      3       │  │    $4.82     │    │
│  │   ▲ +12%     │  │   ▼ -8%      │  │   ▲ +2       │  │   ▼ -15%     │    │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘    │
│                                                                             │
│  Sessions Over Time                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │    ▄                                                                │   │
│  │ ▄▄▄█▄  ▄▄    ▄                                                     │   │
│  │▄█████▄▄██▄▄▄▄█▄▄  ▄▄                                               │   │
│  │████████████████████████▄▄▄▄▄▄  ▄▄▄▄                                │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│   00:00    04:00    08:00    12:00    16:00    20:00                       │
│                                                                             │
│  Recent Sessions                                                            │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ Session ID      │ Duration │ Skills Used        │ Errors │ Cost    │   │
│  ├─────────────────┼──────────┼────────────────────┼────────┼─────────┤   │
│  │ ses_abc123      │ 23m 14s  │ _JIRA, Browser     │ 0      │ $0.42   │   │
│  │ ses_def456      │ 8m 02s   │ CORE               │ 0      │ $0.15   │   │
│  │ ses_ghi789      │ 45m 33s  │ _JIRA, Agents, PAI │ 1      │ $1.23   │   │
│  └─────────────────┴──────────┴────────────────────┴────────┴─────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Skill Usage & Errors

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  SKILL USAGE & ERRORS                                     Last 7 days ▼    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Top Skills by Invocation                    Error Rate by Skill           │
│  ┌────────────────────────────────┐         ┌────────────────────────────┐ │
│  │ _JIRA          ████████████ 89 │         │ _COUPA         ██ 12%      │ │
│  │ Browser        ████████     67 │         │ _JIRA          █  4%       │ │
│  │ CORE           ███████      52 │         │ Browser        ░  1%       │ │
│  │ Agents         █████        34 │         │ CORE           ░  0%       │ │
│  │ System         ████         28 │         │ Agents         ░  0%       │ │
│  └────────────────────────────────┘         └────────────────────────────┘ │
│                                                                             │
│  Recent Errors                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ Time          │ Skill   │ Error                                     │   │
│  ├───────────────┼─────────┼───────────────────────────────────────────┤   │
│  │ 14:23:45      │ _COUPA  │ Browser timeout waiting for element       │   │
│  │ 12:01:12      │ _JIRA   │ API rate limit exceeded                   │   │
│  │ 09:45:33      │ _COUPA  │ Authentication session expired            │   │
│  └───────────────┴─────────┴───────────────────────────────────────────┘   │
│                                                                             │
│  SkillEnforcer Matches (from hook instrumentation)                          │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ Prompt Preview                  │ Matched Skill │ Match Type        │   │
│  ├─────────────────────────────────┼───────────────┼───────────────────┤   │
│  │ "create issue for..."           │ _JIRA         │ keyword           │   │
│  │ "take a screenshot of..."       │ Browser       │ command           │   │
│  │ "what did we work on..."        │ System        │ fuzzy             │   │
│  └─────────────────────────────────┴───────────────┴───────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Autonomous Agent Monitoring

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  AUTONOMOUS AGENT MONITORING                               Live ●          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Active Agents                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ Agent ID        │ Type     │ Status      │ Runtime   │ Parent      │   │
│  ├─────────────────┼──────────┼─────────────┼───────────┼─────────────┤   │
│  │ agent_001       │ Intern   │ ● Running   │ 4m 23s    │ main        │   │
│  │ agent_002       │ Intern   │ ● Running   │ 4m 21s    │ main        │   │
│  │ agent_003       │ Engineer │ ○ Complete  │ 12m 45s   │ main        │   │
│  └─────────────────┴──────────┴─────────────┴───────────┴─────────────┘   │
│                                                                             │
│  Agent Timeline                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ main     ═══════════════════════════════════════════════════►       │   │
│  │                  │                                                   │   │
│  │ agent_001        ├──────────────────────────────────────────►       │   │
│  │ agent_002        ├──────────────────────────────────────────►       │   │
│  │ agent_003        └─────────────────────────█ (complete)             │   │
│  │                                                                      │   │
│  │           14:00      14:05      14:10      14:15      14:20         │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ⚠️ ALERT: Agent idle > 10 minutes (agent_001)                              │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### API Cost Tracking

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  API COST TRACKING                                       This month ▼      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │  Total Cost  │  │  Daily Avg   │  │  Token In    │  │  Token Out   │    │
│  │   $127.43    │  │    $5.79     │  │   2.4M       │  │   890K       │    │
│  │   Budget: $200│  │              │  │              │  │              │    │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘    │
│                                                                             │
│  Daily Spend                                                                │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  $12 ┤     ▄                                                        │   │
│  │  $10 ┤   ▄▄█▄                                                       │   │
│  │   $8 ┤ ▄▄████▄   ▄▄                                                 │   │
│  │   $6 ┼▄████████▄▄██▄▄▄▄   ▄▄▄▄▄                                     │   │
│  │   $4 ┤██████████████████▄▄█████▄▄▄▄                                 │   │
│  │   $2 ┤██████████████████████████████                                │   │
│  │   $0 ┴─────────────────────────────────────────────────────────     │   │
│  │       1    5    10   15   20   22 (today)                           │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  Cost by Model                           Cost by Skill                      │
│  ┌────────────────────────────┐         ┌────────────────────────────┐     │
│  │ claude-opus-4   ████ $89   │         │ _JIRA        ████████ $67  │     │
│  │ claude-sonnet   ██   $32   │         │ Agents       ███      $28  │     │
│  │ claude-haiku    ░    $6    │         │ CORE         ██       $18  │     │
│  └────────────────────────────┘         │ Other        █        $14  │     │
│                                          └────────────────────────────┘     │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Tracing & Troubleshooting Examples

Understanding where errors occur and how to trace them is the core value of observability.

### Error Taxonomy: Where Things Go Wrong

Errors can occur at multiple levels of the PAI stack:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    PAI ERROR TAXONOMY                                        │
│                                                                              │
│   LAYER 1: User Input                                                       │
│   ├── Ambiguous prompts (skill mismatch)                                    │
│   ├── Missing context (file paths, project refs)                            │
│   └── Conflicting instructions                                              │
│                                                                              │
│   LAYER 2: Skill Routing                                                    │
│   ├── SkillEnforcer pattern miss (no skill matched)                         │
│   ├── Wrong skill matched (fuzzy match too loose)                           │
│   └── Skill not loaded (registry stale)                                     │
│                                                                              │
│   LAYER 3: Tool Execution                                                   │
│   ├── Bash: Command not found, permission denied, timeout                   │
│   ├── Read/Write: File not found, permission denied                         │
│   ├── Browser: Element not found, timeout, selector changed                 │
│   └── WebFetch: Network error, rate limit, auth failure                     │
│                                                                              │
│   LAYER 4: External Services                                                │
│   ├── API rate limits (Jira, GitHub, inference)                             │
│   ├── Authentication expired (OAuth tokens, session cookies)                │
│   ├── Network timeouts (VPN, firewall, DNS)                                 │
│   └── Service unavailable (503, maintenance)                                │
│                                                                              │
│   LAYER 5: Infrastructure                                                   │
│   ├── OrbStack not running (containers down)                                │
│   ├── Disk full (JSONL files, Docker volumes)                               │
│   ├── Memory pressure (too many parallel agents)                            │
│   └── Hook failures (TypeScript errors, missing deps)                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Example: Complete Session Trace

Here's what a real session looks like in the event log, showing the journey from user prompt through skill execution to completion:

```jsonl
{"ts":1737574800000,"event_type":"session.start","source":"hook.SessionStart","session_id":"ses_abc123","data":{"model":"claude-opus-4-5","working_dir":"/Users/andreas/project"}}
{"ts":1737574800100,"event_type":"skill.matched","source":"hook.SkillEnforcer","session_id":"ses_abc123","data":{"prompt_preview":"create a jira issue for the login bug","matched_skills":["_JIRA"],"match_types":["keyword"]}}
{"ts":1737574801000,"event_type":"skill.invoke","source":"skill._JIRA","session_id":"ses_abc123","data":{"skill":"_JIRA","trigger":"user_request"}}
{"ts":1737574802000,"event_type":"tool.use","source":"hook.PreToolUse","session_id":"ses_abc123","data":{"tool":"Bash","input":{"command":"jira issue create --project HD --type Bug"}}}
{"ts":1737574805000,"event_type":"tool.result","source":"hook.PostToolUse","session_id":"ses_abc123","data":{"tool":"Bash","duration_ms":3000,"exit_code":0,"output_preview":"HD-1234"}}
{"ts":1737574806000,"event_type":"skill.complete","source":"skill._JIRA","session_id":"ses_abc123","data":{"duration_ms":5000,"success":true,"issue_key":"HD-1234"}}
{"ts":1737574900000,"event_type":"session.end","source":"hook.SessionStop","session_id":"ses_abc123","data":{"duration_ms":100000,"tool_calls":3,"tokens_in":1500,"tokens_out":800}}
```

**Reading the trace:**
1. Session starts at `ses_abc123`
2. SkillEnforcer matches "_JIRA" via keyword pattern
3. Skill is invoked
4. Tool call to Bash (jira CLI)
5. Tool returns success with issue key
6. Skill completes successfully
7. Session ends with summary metrics

### Example: Tracing an Error Through Multiple Layers

Errors can occur at different layers of the same operation. Here's a _JIRA skill failure that demonstrates how the trace reveals the actual layer where things went wrong:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    _JIRA SKILL LAYERS                                        │
│                                                                              │
│   Layer 1: Skill                                                            │
│   └── _JIRA skill invoked                                                   │
│                                                                              │
│   Layer 2: CLI Tool                                                         │
│   └── Bash: jira issue create --project HD --type Bug                       │
│                                                                              │
│   Layer 3: Configuration                                                    │
│   └── Profile loaded from ~/.config/.jira/.config.yaml                      │
│   └── Project key: HD, Server: jira.company.com                             │
│                                                                              │
│   Layer 4: API Call                                                         │
│   └── POST https://jira.company.com/rest/api/3/issue                        │
│   └── Auth: OAuth token from keychain                                       │
│                                                                              │
│   Layer 5: External Service                                                 │
│   └── Jira Cloud response                                                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Trace showing failure at API layer (rate limit):**
```jsonl
{"ts":1737574800000,"event_type":"session.start","source":"hook.SessionStart","session_id":"ses_def456","data":{"model":"claude-opus-4-5","working_dir":"/Users/andreas/project"}}
{"ts":1737574800100,"event_type":"skill.matched","source":"hook.SkillEnforcer","session_id":"ses_def456","data":{"prompt_preview":"create issue for login bug","matched_skills":["_JIRA"],"match_types":["keyword"]}}
{"ts":1737574801000,"event_type":"skill.invoke","source":"skill._JIRA","session_id":"ses_def456","data":{"skill":"_JIRA","trigger":"user_request"}}
{"ts":1737574802000,"event_type":"tool.use","source":"hook.PreToolUse","session_id":"ses_def456","data":{"tool":"Bash","input":{"command":"jira issue create --project HD --type Bug --summary 'Login button unresponsive'"}}}
{"ts":1737574802100,"event_type":"cli.config","source":"skill._JIRA","session_id":"ses_def456","data":{"config_path":"~/.config/.jira/.config.yaml","profile":"work","server":"jira.company.com"}}
{"ts":1737574802500,"event_type":"api.call","source":"skill._JIRA","session_id":"ses_def456","data":{"method":"POST","url":"https://jira.company.com/rest/api/3/issue","auth_type":"oauth"}}
{"ts":1737574803000,"event_type":"api.response","source":"skill._JIRA","session_id":"ses_def456","data":{"status":429,"error":"Rate limit exceeded","retry_after":60,"headers":{"x-ratelimit-remaining":"0"}}}
{"ts":1737574803100,"event_type":"tool.result","source":"hook.PostToolUse","session_id":"ses_def456","data":{"tool":"Bash","duration_ms":1100,"exit_code":1,"stderr":"Error: 429 Too Many Requests"}}
{"ts":1737574803200,"event_type":"skill.error","source":"skill._JIRA","session_id":"ses_def456","error":{"message":"Jira API rate limit exceeded","code":"API_RATE_LIMIT","context":{"layer":"api","retry_after_seconds":60}}}
```

**Reading the layered trace:**
1. Session starts, _JIRA skill matched (Layer 1: Skill)
2. CLI command invoked (Layer 2: CLI Tool)
3. Config loaded, profile "work" selected (Layer 3: Configuration)
4. API call made to Jira (Layer 4: API)
5. 429 response - rate limit hit (Layer 5: External Service)
6. Error propagates back through layers

**Alternative failure: Config layer (wrong profile):**
```jsonl
{"ts":1737574802100,"event_type":"cli.config","source":"skill._JIRA","session_id":"ses_xyz","data":{"config_path":"~/.config/.jira/.config.yaml","profile":"personal","server":"personal.atlassian.net"}}
{"ts":1737574802500,"event_type":"api.call","source":"skill._JIRA","session_id":"ses_xyz","data":{"method":"POST","url":"https://personal.atlassian.net/rest/api/3/issue"}}
{"ts":1737574803000,"event_type":"api.response","source":"skill._JIRA","session_id":"ses_xyz","data":{"status":404,"error":"Project 'HD' not found"}}
{"ts":1737574803200,"event_type":"skill.error","source":"skill._JIRA","session_id":"ses_xyz","error":{"message":"Project HD not found - wrong Jira instance","code":"PROJECT_NOT_FOUND","context":{"layer":"config","expected_server":"jira.company.com","actual_server":"personal.atlassian.net"}}}
```

**Root cause:** CLI picked wrong profile, sent request to personal Jira instead of work Jira.

**Alternative failure: Auth layer (token expired):**
```jsonl
{"ts":1737574802500,"event_type":"api.call","source":"skill._JIRA","session_id":"ses_auth","data":{"method":"POST","url":"https://jira.company.com/rest/api/3/issue","auth_type":"oauth"}}
{"ts":1737574803000,"event_type":"api.response","source":"skill._JIRA","session_id":"ses_auth","data":{"status":401,"error":"OAuth token expired","www_authenticate":"Bearer realm=\"Jira\""}}
{"ts":1737574803200,"event_type":"skill.error","source":"skill._JIRA","session_id":"ses_auth","error":{"message":"Jira authentication failed - token expired","code":"AUTH_EXPIRED","context":{"layer":"auth","action_required":"re-authenticate via jira auth"}}}
```

**Query to find errors by layer:**
```bash
# Group errors by which layer failed
grep '"event_type":"skill.error"' $PAI_HOME/MEMORY/Events/$(date +%Y-%m-%d).jsonl | \
  jq -s 'group_by(.error.context.layer) | map({layer: .[0].error.context.layer, count: length, errors: [.[].error.code] | unique})'
```

**Output:**
```json
[
  {"layer": "api", "count": 5, "errors": ["API_RATE_LIMIT", "API_TIMEOUT"]},
  {"layer": "auth", "count": 2, "errors": ["AUTH_EXPIRED"]},
  {"layer": "config", "count": 1, "errors": ["PROJECT_NOT_FOUND"]}
]
```

This tells you: most errors are at the API layer (rate limits), followed by auth issues. Config problems are rare.

### Troubleshooting Queries

**Find all errors today:**
```bash
grep '"event_type":"skill.error"' $PAI_HOME/MEMORY/Events/$(date +%Y-%m-%d).jsonl | jq
```

**Find API rate limit errors and when they occurred:**
```bash
grep '"code":"API_RATE_LIMIT"' $PAI_HOME/MEMORY/Events/2026-01-*.jsonl | \
  jq '{time: (.ts / 1000 | strftime("%Y-%m-%d %H:%M")), skill: .source, retry_after: .error.context.retry_after_seconds}'
```

**Find auth failures by service:**
```bash
grep '"code":"AUTH_EXPIRED"' $PAI_HOME/MEMORY/Events/2026-01-*.jsonl | \
  jq -s 'group_by(.source) | map({skill: .[0].source, count: length})'
```

**Trace a specific session:**
```bash
grep '"session_id":"ses_def456"' $PAI_HOME/MEMORY/Events/*.jsonl | jq -s 'sort_by(.ts)'
```

**Find sessions with errors and their duration:**
```bash
cat $PAI_HOME/MEMORY/Events/$(date +%Y-%m-%d).jsonl | jq -s '
  group_by(.session_id) |
  map(select(any(.event_type == "skill.error"))) |
  map({
    session_id: .[0].session_id,
    errors: [.[] | select(.event_type == "skill.error") | .error.message],
    duration_ms: ([.[] | select(.event_type == "session.end")][0].data.duration_ms // "ongoing")
  })
'
```

**Find slow tool calls (>5 seconds):**
```bash
grep '"event_type":"tool.result"' $PAI_HOME/MEMORY/Events/$(date +%Y-%m-%d).jsonl | \
  jq 'select(.data.duration_ms > 5000) | {tool: .data.tool, duration_ms: .data.duration_ms, session: .session_id}'
```

**API cost by skill this week:**
```bash
cat $PAI_HOME/MEMORY/Events/2026-01-*.jsonl | jq -s '
  [.[] | select(.event_type == "api.cost")] |
  group_by(.data.skill) |
  map({skill: .[0].data.skill, total_cost: (map(.data.cost_usd) | add)}) |
  sort_by(.total_cost) | reverse
'
```

### Correlated Trace: Multi-Agent Debugging

When parallel agents are involved, tracing becomes more complex. PAI uses a hierarchical identification scheme to correlate events across agent boundaries.

**Session & Agent Identification:**

| Field | Source | Purpose |
|-------|--------|---------|
| `session_id` | Claude Code (via stdin to hooks) | Identifies the top-level conversation session (UUID) |
| `agent_instance_id` | PAI metadata extraction | Unique agent identifier: `{agent_type}-{N}` (e.g., `perplexity-researcher-1`) |
| `agent_type` | Task tool `subagent_type` | Base agent type (e.g., `Intern`, `Explore`, `Engineer`) |
| `parent_session_id` | Prompt convention | Session that spawned this agent |
| `parent_task_id` | Prompt convention | Task ID that spawned this agent |

**How Agent Identification Works:**

Claude Code provides `session_id` to every hook via stdin. For subagents spawned via the Task tool, PAI extracts additional metadata using conventions embedded in prompts:

```typescript
// Pattern: [AGENT_INSTANCE: perplexity-researcher-1]
const promptMatch = toolInput.prompt.match(/\[AGENT_INSTANCE:\s*([^\]]+)\]/);

// Pattern: [PARENT_SESSION: b7062b5a-03d3-4168-9555-a748e0b2efa3]
const parentMatch = toolInput.prompt.match(/\[PARENT_SESSION:\s*([^\]]+)\]/);
```

This enables tracing a request from the parent session through all spawned agents, even when running in parallel across different processes.

**Example: Follow the Execution Tree**

Use `agent_id` and `parent_agent_id` to follow the execution tree:

```jsonl
{"ts":1737574800000,"event_type":"session.start","source":"hook.SessionStart","session_id":"ses_ghi789","data":{"model":"claude-opus-4-5"}}
{"ts":1737574801000,"event_type":"agent.spawn","source":"Task","session_id":"ses_ghi789","agent_id":"agent_001","data":{"agent_type":"Intern","parent":"main","task":"search for auth files"}}
{"ts":1737574801100,"event_type":"agent.spawn","source":"Task","session_id":"ses_ghi789","agent_id":"agent_002","data":{"agent_type":"Intern","parent":"main","task":"search for config files"}}
{"ts":1737574805000,"event_type":"tool.use","source":"hook.PreToolUse","session_id":"ses_ghi789","agent_id":"agent_001","data":{"tool":"Grep","input":{"pattern":"authentication"}}}
{"ts":1737574805500,"event_type":"tool.use","source":"hook.PreToolUse","session_id":"ses_ghi789","agent_id":"agent_002","data":{"tool":"Glob","input":{"pattern":"**/*.config.*"}}}
{"ts":1737574808000,"event_type":"tool.result","source":"hook.PostToolUse","session_id":"ses_ghi789","agent_id":"agent_002","data":{"tool":"Glob","duration_ms":2500,"success":true}}
{"ts":1737574810000,"event_type":"skill.error","source":"hook.PostToolUse","session_id":"ses_ghi789","agent_id":"agent_001","error":{"message":"Grep pattern too broad, 10000+ matches","code":"RESULT_TOO_LARGE"}}
{"ts":1737574811000,"event_type":"agent.complete","source":"hook.SubagentStop","session_id":"ses_ghi789","agent_id":"agent_002","data":{"duration_ms":10000,"status":"success"}}
{"ts":1737574815000,"event_type":"agent.complete","source":"hook.SubagentStop","session_id":"ses_ghi789","agent_id":"agent_001","data":{"duration_ms":14000,"status":"error"}}
```

**Query to visualize agent execution timeline:**
```bash
grep '"session_id":"ses_ghi789"' $PAI_HOME/MEMORY/Events/*.jsonl | jq -s '
  sort_by(.ts) |
  map({
    time: (.ts - .[0].ts) / 1000,  # seconds since session start
    agent: (.agent_id // "main"),
    event: .event_type,
    detail: (.data.tool // .data.agent_type // .error.code // null)
  })
'
```

**Output:**
```json
[
  {"time": 0, "agent": "main", "event": "session.start", "detail": null},
  {"time": 1, "agent": "main", "event": "agent.spawn", "detail": "Intern"},
  {"time": 1.1, "agent": "main", "event": "agent.spawn", "detail": "Intern"},
  {"time": 5, "agent": "agent_001", "event": "tool.use", "detail": "Grep"},
  {"time": 5.5, "agent": "agent_002", "event": "tool.use", "detail": "Glob"},
  {"time": 8, "agent": "agent_002", "event": "tool.result", "detail": "Glob"},
  {"time": 10, "agent": "agent_001", "event": "skill.error", "detail": "RESULT_TOO_LARGE"},
  {"time": 11, "agent": "agent_002", "event": "agent.complete", "detail": null},
  {"time": 15, "agent": "agent_001", "event": "agent.complete", "detail": null}
]
```

This shows agent_001 failed at t=10s due to an overly broad grep, while agent_002 completed successfully at t=11s.

**Current Limitations:**

| Gap | Impact | Future Improvement |
|-----|--------|-------------------|
| No automatic parent context injection | Requires explicit `[PARENT_SESSION: ...]` tags in Task prompts | Add PreToolUse hook that auto-injects parent metadata |
| No `SubagentStart` hook | Can only trace agent completion, not spawn | Add lifecycle event for agent initialization |
| Task ID format not standardized | Inconsistent `parent_task_id` values | Use UUID v4 for all task identifiers |
| Subagent context isolation | Each subagent gets fresh environment | Intentional for security, but limits shared state |

**Implementation Note:** Parent-child tracking depends on the calling agent including metadata tags in the Task tool prompt. If these tags are absent, the parent relationship is lost in observability data.

### Grafana: Trace View

In Grafana, the same data visualizes as a trace waterfall:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  SESSION TRACE: ses_ghi789                                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  main     ════════════════════════════════════════════════════════►         │
│               │                                                             │
│  agent_001    ├────────[Grep]──────────────────────✗ ERROR                  │
│               │                                    │                        │
│  agent_002    └────[Glob]────────────────█ OK      │                        │
│                                                    │                        │
│           0s     2s     4s     6s     8s    10s   12s   14s   16s          │
│                                                                             │
│  ─────────────────────────────────────────────────────────────────────────  │
│  Error Detail:                                                              │
│  agent_001 @ 10s: RESULT_TOO_LARGE - Grep pattern too broad, 10000+ matches │
│  Suggestion: Narrow pattern or add path filter                              │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Performance Profiling: Hook Execution Timing

**Real Problem:** After completing a request, the terminal locks up for 60+ seconds. Where's the bottleneck?

Without observability, you'd add `console.log` timestamps throughout hook code. With the event system, you capture timing data automatically and analyze it properly.

**Hook timing events:**
```jsonl
{"ts":1737574900000,"event_type":"hook.start","source":"hook.SessionStop","session_id":"ses_abc123","data":{"hook":"SessionStop"}}
{"ts":1737574900050,"event_type":"hook.phase","source":"hook.SessionStop","session_id":"ses_abc123","data":{"hook":"SessionStop","phase":"load_transcript","duration_ms":50}}
{"ts":1737574900150,"event_type":"hook.phase","source":"hook.SessionStop","session_id":"ses_abc123","data":{"hook":"SessionStop","phase":"calculate_metrics","duration_ms":100}}
{"ts":1737574960000,"event_type":"hook.phase","source":"hook.SessionStop","session_id":"ses_abc123","data":{"hook":"SessionStop","phase":"voice_notification","duration_ms":59850}}
{"ts":1737574960100,"event_type":"hook.complete","source":"hook.SessionStop","session_id":"ses_abc123","data":{"hook":"SessionStop","total_duration_ms":60100}}
```

**Instrumentation in hook code:**
```typescript
// In SessionStop.hook.ts
const hookStart = Date.now();
logEvent({ event_type: 'hook.start', source: 'hook.SessionStop', data: { hook: 'SessionStop' } });

// Phase 1: Load transcript
const phase1Start = Date.now();
const transcript = await loadTranscript(input.transcript_path);
logEvent({
  event_type: 'hook.phase',
  source: 'hook.SessionStop',
  data: { hook: 'SessionStop', phase: 'load_transcript', duration_ms: Date.now() - phase1Start }
});

// Phase 2: Calculate metrics
const phase2Start = Date.now();
const metrics = calculateSessionMetrics(transcript);
logEvent({
  event_type: 'hook.phase',
  source: 'hook.SessionStop',
  data: { hook: 'SessionStop', phase: 'calculate_metrics', duration_ms: Date.now() - phase2Start }
});

// Phase 3: Voice notification (THE BOTTLENECK!)
const phase3Start = Date.now();
await speakNotification(`Session complete. ${metrics.duration} minutes.`);
logEvent({
  event_type: 'hook.phase',
  source: 'hook.SessionStop',
  data: { hook: 'SessionStop', phase: 'voice_notification', duration_ms: Date.now() - phase3Start }
});

logEvent({
  event_type: 'hook.complete',
  source: 'hook.SessionStop',
  data: { hook: 'SessionStop', total_duration_ms: Date.now() - hookStart }
});
```

**Find the slow hooks:**
```bash
# Hooks taking >1 second
grep '"event_type":"hook.complete"' $PAI_HOME/MEMORY/Events/$(date +%Y-%m-%d).jsonl | \
  jq 'select(.data.total_duration_ms > 1000) | {hook: .data.hook, duration_ms: .data.total_duration_ms}'
```

**Drill into phases of slow hooks:**
```bash
# Find which phase is slow in SessionStop
grep '"hook":"SessionStop"' $PAI_HOME/MEMORY/Events/$(date +%Y-%m-%d).jsonl | \
  jq 'select(.event_type == "hook.phase") | {phase: .data.phase, duration_ms: .data.duration_ms}' | \
  jq -s 'sort_by(.duration_ms) | reverse'
```

**Output reveals the bottleneck:**
```json
[
  {"phase": "voice_notification", "duration_ms": 59850},
  {"phase": "calculate_metrics", "duration_ms": 100},
  {"phase": "load_transcript", "duration_ms": 50}
]
```

**Diagnosis:** The voice notification is taking ~60 seconds. Root causes:
- ElevenLabs API slow/timeout
- Audio playback blocking
- Large text being synthesized

**Fix verification:** After fixing, run the same query to confirm improvement.

**Grafana: Hook Performance Dashboard**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  HOOK PERFORMANCE                                          Last 24 hours ▼  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Hook Execution Time (p95)                                                  │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ SessionStop       ████████████████████████████████████████ 62.1s    │   │
│  │ PreToolUse        ██ 0.8s                                           │   │
│  │ PostToolUse       █ 0.3s                                            │   │
│  │ SessionStart      █ 0.2s                                            │   │
│  │ SkillEnforcer     ░ 0.05s                                           │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  SessionStop Phase Breakdown (today)                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                                                                      │   │
│  │  load_transcript     ██ 2%                                          │   │
│  │  calculate_metrics   ██ 2%                                          │   │
│  │  voice_notification  ████████████████████████████████████████ 96%   │   │
│  │                                                                      │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  Hook Execution Over Time                                                   │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  70s ┤                        ▄▄▄                                    │   │
│  │  60s ┤    ▄▄▄   ▄▄▄   ▄▄▄   ▄███▄      ← voice_notification        │   │
│  │  50s ┤   ████▄ ████▄ ████▄ ██████▄                                  │   │
│  │   1s ┼───────────────────────────────────── other phases ──────────│   │
│  │   0s ┴─────────────────────────────────────────────────────────────│   │
│  │       00:00    04:00    08:00    12:00    16:00    20:00           │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ⚠️ ALERT: SessionStop hook p95 > 10s (currently 62.1s)                     │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Drill-down capability:** In Grafana, you don't just see "SessionStop is slow"—you click to drill into the trace:

1. Click the slow bar → opens trace detail view
2. See the span waterfall with each phase as a child span
3. Click `voice_notification` span → see the actual API call, timeout, response
4. Correlate with logs: "ElevenLabs returned 503 at 14:32:05"

This is the same data available via `jq` on the CLI, but Grafana's visual waterfall makes the investigation 10x faster.

**Comparison: console.log vs Observability**

| Approach | Effort | Persistence | Analysis | Production-Safe |
|----------|--------|-------------|----------|-----------------|
| **console.log** | Add/remove manually | Lost on scroll | Manual eyeball | Noisy, may leak |
| **Observability** | Instrument once | Queryable forever | jq/Grafana/alerts | Structured, filterable |

The observability approach turns "why is this slow?" from a 30-minute console.log debugging session into a 30-second query.

---

## Scaling Path

The architecture supports progressive scaling. Start with just files and CLI tools, add infrastructure only when needed.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    SCALING PATH                                              │
│                                                                              │
│   STAGE 0: No Stack (CLI Only)                                              │
│   ┌─────────────────┐     ┌─────────────────┐                               │
│   │  PAI Agent      │────►│ JSONL Files     │  Query with: tail, grep, jq   │
│   │  (laptop)       │     │ (local)         │  No collector, no containers  │
│   └─────────────────┘     └─────────────────┘                               │
│                                                                              │
│   STAGE 1: Local Stack                                                      │
│   ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐       │
│   │  PAI Agent      │────►│  Collector      │────►│ Local Stack     │       │
│   └─────────────────┘     └─────────────────┘     └─────────────────┘       │
│                                                                              │
│   STAGE 2: Central Stack                                                    │
│   ┌─────────────────┐                                                       │
│   │  PAI Agent      │──┐  ┌─────────────────┐     ┌─────────────────┐       │
│   │  (laptop)       │  ├─►│  Collector      │────►│   Central       │       │
│   └─────────────────┘  │  └─────────────────┘     │    Stack        │       │
│   ┌─────────────────┐  │                          └─────────────────┘       │
│   │  PAI Agent      │──┘                                                    │
│   │  (desktop)      │                                                       │
│   └─────────────────┘                                                       │
│                                                                              │
│   STAGE 3: Cloud                                                            │
│   ┌─────────────────┐                                                       │
│   │  User A         │──┐  ┌─────────────────┐     ┌─────────────────┐       │
│   │  (3 agents)     │  ├─►│  Collector      │────►│    Cloud        │       │
│   └─────────────────┘  │  └─────────────────┘     │   Backend       │       │
│   ┌─────────────────┐  │                          └─────────────────┘       │
│   │  User B         │──┘                                                    │
│   │  (2 agents)     │                                                       │
│   └─────────────────┘                                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**What changes at each stage:**

| Stage | What You Add | Collector Config |
|-------|--------------|------------------|
| **0** | Nothing — just JSONL files | No collector |
| **0 → 1** | OrbStack + VM stack | `http://localhost:9428` |
| **1 → 2** | Move stack to NAS | `http://nas.local:9428` |
| **2 → 3** | Use cloud backend | Depends on provider (see below) |

**What stays the same:**
- Event schema (PAIEvent interface)
- Local JSONL capture (always works offline)
- Hook instrumentation
- Watchdog self-monitoring

---

## Design Principles

This design follows the PAI Founding Principles:

| Requirement | PAI Principle | How It Aligns |
|-------------|---------------|---------------|
| File-based capture | **#6 Code Before Prompts** | Simple append to file, no HTTP overhead |
| Text/JSONL format | **#8 UNIX Philosophy** | Text interfaces, composable with grep/jq/tail |
| Local-first operation | **#5 Deterministic Infrastructure** | Works without stack, no network dependencies |
| Distributed collector (push) | **#9 ENG/SRE Principles** | Security: agents push out, nothing reaches in |
| Feeds The Algorithm | **#2 Continuously Upgrading** | Events → Signals → Learning → Algorithm improvement |
| CLI queryable | **#10 CLI as Interface** | `tail -f events.jsonl | jq` for real-time debugging |
| Captured to MEMORY | **#13 Memory System** | Everything worth knowing gets captured |

---

## Key Design Decisions

### 1. File-Based Capture (Not HTTP)

**Why:** PAI is built on files and markdown. HTTP calls add latency, failure modes, and complexity.

```typescript
// Capture is ONE LINE - dead simple
function logEvent(event: PAIEvent): void {
  const line = JSON.stringify({ ...event, ts: Date.now() }) + '\n';
  appendFileSync(`${PAI_HOME}/MEMORY/Events/${today()}.jsonl`, line);
}
```

**Trade-offs:**

| Approach | Latency | Failure Mode | Complexity |
|----------|---------|--------------|------------|
| HTTP POST | ~10-100ms | Collector down = lost/retry | Error handling, retries |
| File append | ~1ms | Always succeeds | One line |

### 2. Distributed Collector (Push, Not Pull)

**Why:** Security. Nothing reaches into the PAI environment. The collector sits alongside and pushes outbound.

```
WRONG (security risk):
  Observability Stack ──► reaches into ──► PAI Environment
                         (inbound connection)

RIGHT (secure):
  PAI Environment ──► writes files ──► Collector ──► pushes to ──► Stack
                     (local only)                    (outbound only)
```

**The collector:**
- Runs as a daemon (launchd/systemd) on the same machine
- Tails JSONL files, batches events, pushes to stack
- No inbound network listeners
- If stack is unreachable, buffers locally

### 3. Works Without Stack (Local-First)

**Why:** Deterministic. No external dependencies for basic observability.

```bash
# Real-time event stream (no stack needed)
tail -f $PAI_HOME/MEMORY/Events/$(date +%Y-%m-%d).jsonl | jq

# Find all skill errors today
grep '"event_type":"skill.error"' $PAI_HOME/MEMORY/Events/$(date +%Y-%m-%d).jsonl | jq

# Count sessions this week
cat $PAI_HOME/MEMORY/Events/2026-01-*.jsonl | jq -s '[.[] | select(.event_type=="session.start")] | length'
```

### 4. Concurrent Write Handling

**Problem:** Multiple parallel agents may write to the same daily JSONL file simultaneously. Without coordination, lines could interleave mid-write, corrupting the JSON.

**Options:**

| Approach | Complexity | Guarantees |
|----------|------------|------------|
| **Rename-based lock** | Low | Atomic acquisition, one writer at a time |
| **Per-agent files** | Lowest | No coordination needed, collector merges |
| **flock()** | Medium | POSIX advisory locking |

**Recommended: Rename-based lock**

```bash
# Atomic lock acquisition with retry
LOCK_FILE="${EVENTS_DIR}/.write.lock"
TEMP_LOCK="${EVENTS_DIR}/.write.lock.$$"
MAX_RETRIES=50
RETRY_DELAY=0.01  # 10ms

acquire_lock() {
    local retries=0
    echo $$ > "$TEMP_LOCK"

    while [ $retries -lt $MAX_RETRIES ]; do
        if mv -n "$TEMP_LOCK" "$LOCK_FILE" 2>/dev/null; then
            return 0  # Got lock
        fi

        # Check if lock is stale (holder died)
        if [ -f "$LOCK_FILE" ]; then
            local holder=$(cat "$LOCK_FILE" 2>/dev/null)
            if [ -n "$holder" ] && ! kill -0 "$holder" 2>/dev/null; then
                # Holder process is dead, remove stale lock
                rm -f "$LOCK_FILE"
            fi
        fi

        sleep $RETRY_DELAY
        retries=$((retries + 1))
    done

    rm -f "$TEMP_LOCK"
    return 1  # Failed after max retries
}

release_lock() {
    rm -f "$LOCK_FILE"
}

# Usage in logEvent()
if acquire_lock; then
    echo "$JSON_LINE" >> "$EVENT_FILE"
    release_lock
else
    # Fallback: write to per-agent file
    echo "$JSON_LINE" >> "${EVENT_FILE%.jsonl}.${AGENT_ID}.jsonl"
fi
```

**Why rename works:**
- `mv -n` (no-clobber) is atomic on POSIX filesystems
- If two processes race, exactly one succeeds
- No external dependencies (works with bash built-ins)

**Alternative: Per-agent files**

If lock contention becomes an issue, each agent writes to its own file:

```
$PAI_HOME/MEMORY/Events/
├── 2026-01-22.jsonl           # Main session
├── 2026-01-22.agent_001.jsonl # Parallel agent 1
├── 2026-01-22.agent_002.jsonl # Parallel agent 2
```

Collector merges all `*.jsonl` files when forwarding to stack.

### 5. Autonomous Agent Notifications

**Why:** When you're not at the terminal, you need to know when things go wrong.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    NOTIFICATION FLOW                                         │
│                                                                              │
│   PAI Agent (autonomous)                                                     │
│        │                                                                     │
│        ▼                                                                     │
│   Writes event: { event_type: "skill.error", skill: "_JIRA", ... }          │
│        │                                                                     │
│        ▼                                                                     │
│   Collector detects pattern: "skill.error count > 3 in 5min"                │
│        │                                                                     │
│        ▼                                                                     │
│   Pushes to stack → Grafana alert triggers                                  │
│        │                                                                     │
│        ▼                                                                     │
│   PAI Notify skill → Voice / Push / Telegram                                │
│        │                                                                     │
│        ▼                                                                     │
│   Human (on the go) receives: "PAI: _JIRA skill failing repeatedly"         │
│        │                                                                     │
│        ▼                                                                     │
│   Human investigates via Grafana dashboard (decoupled from agent)           │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 6. Security as a Design Principle

**Why:** Observability data is sensitive. Session transcripts, skill invocations, and error contexts can expose confidential information.

**Architectural security decisions:**

| Decision | Rationale |
|----------|-----------|
| **Outbound-only** | Collector pushes to stack; nothing pulls from PAI environment |
| **No inbound listeners** | Collector has no network ports open—eliminates attack surface |
| **Credentials in .env** | Never hardcoded; `.env` excluded from version control |
| **PII scrubbing** | Events must not contain passwords, API keys, or personal data |
| **Local-first storage** | Events land in local JSONL before any network transmission |

**Data sensitivity tiers:**

| Tier | Data | Example | Handling |
|------|------|---------|----------|
| **Safe** | Event types, timestamps, durations | `skill.invoke`, `hook.complete` | Log freely |
| **Caution** | Skill names, error codes | `_JIRA`, `AUTH_EXPIRED` | Log, but review patterns |
| **Sensitive** | Full prompts, outputs, credentials | API keys, user data | NEVER log |

**The scrubbing pattern:**

```typescript
function sanitizeEvent(event: PAIEvent): PAIEvent {
  const scrubbed = { ...event };
  // Remove sensitive fields
  delete scrubbed.data?.prompt;
  delete scrubbed.data?.response;
  delete scrubbed.data?.api_key;
  // Mask potential secrets in error messages
  if (scrubbed.data?.error_message) {
    scrubbed.data.error_message = scrubbed.data.error_message
      .replace(/sk-[a-zA-Z0-9]+/g, 'sk-***')
      .replace(/Bearer [a-zA-Z0-9]+/g, 'Bearer ***');
  }
  return scrubbed;
}
```

→ *Security mitigations summary: See Security Considerations section*

---

## Why Not OpenTelemetry (OTLP)?

**Decision:** Use direct HTTP ingestion to VictoriaLogs rather than OTLP protocol.

| Factor | OTLP | Direct HTTP |
|--------|------|-------------|
| **Complexity** | OTel Collector + SDK instrumentation | Simple curl POST |
| **Dependencies** | OTel Collector binary or container | None (curl is built-in) |
| **Vendor lock-in** | Low (standard protocol) | Medium (VictoriaLogs API) |
| **Best for** | Multi-service distributed systems | Single-tenant, single-destination |

**When OTLP makes sense:**
- You have multiple applications instrumented with OTel SDKs
- You need to switch observability backends without changing instrumentation
- You're in an enterprise environment with OTLP standardization
- You need automatic instrumentation of HTTP/database calls

**Why we chose direct ingestion (for now):**
- Single destination (one observability stack)
- We control the event schema (no need for OTel SDK auto-instrumentation)
- Simpler initial implementation (one less layer)

**Important caveat:** PAI can become complex—multiple parallel agents, background processes, CLI tools, and external API calls (Jira, inference APIs) all executing concurrently. This IS distributed tracing territory. As PAI complexity grows, OTLP instrumentation may become valuable for correlating spans across agents.

**What "migration to OTLP" actually means:**

| Layer | Current | With OTLP |
|-------|---------|-----------|
| **Backend** | VictoriaLogs/Traces | Same (already supports OTLP) |
| **Collector** | curl/Vector → HTTP POST | OTel Collector → OTLP |
| **Instrumentation** | `logEvent()` → JSONL | OTel SDK → spans/traces |

**Backend support is easy** — VictoriaTraces accepts OTLP on ports 4317/4318.

**Instrumentation is the rearchitecture** — Converting hooks from `logEvent(event)` to OTel SDK spans (`tracer.startSpan()`) requires rewriting the instrumentation layer. This is not a config change; it's real work.

---

## Alternative Backends

The collector is backend-agnostic. The JSONL event format stays the same—only the sink configuration changes.

| Backend | Type | Hosting |
|---------|------|---------|
| **VictoriaMetrics Stack** | VM/VictoriaLogs | Self-hosted (OrbStack, NAS, EC2) |
| **Grafana Cloud** | Loki/Mimir/Tempo | Managed SaaS |
| **InfluxDB Cloud** | InfluxDB | Managed SaaS |
| **AWS CloudWatch** | CloudWatch Logs | AWS managed |
| **Datadog** | Datadog | Managed SaaS |

→ *Full sink configurations for each backend: See Appendix C*

---

## Event Schema

### Core Event Structure

```typescript
interface PAIEvent {
  // Required
  ts: number;                    // Unix timestamp (ms)
  event_type: string;            // Hierarchical: "session.start", "skill.invoke"
  source: string;                // Component: "hook.SessionStart", "skill._JIRA"

  // Session context (when applicable)
  session_id?: string;           // Links events to session
  agent_id?: string;             // For parallel agents
  parent_agent_id?: string;      // Hierarchy tracking

  // Event-specific payload
  data?: Record<string, any>;    // Flexible payload

  // Error context (when applicable)
  error?: {
    message: string;
    stack?: string;
    code?: string;
  };
}
```

### Event Types

| Event Type | Source | When | Data |
|------------|--------|------|------|
| `session.start` | hook.SessionStart | Session begins | `{ model, working_dir }` |
| `session.end` | hook.SessionStop | Session ends | `{ duration_ms, tool_calls }` |
| `session.idle` | collector | 10min no activity | `{ idle_since }` |
| `agent.spawn` | Task tool | Parallel agent created | `{ agent_type, parent }` |
| `agent.complete` | hook.SubagentStop | Agent finishes | `{ duration_ms, status }` |
| `skill.matched` | hook.SkillEnforcer | Skill pattern matched | `{ matched_skills, match_types }` |
| `skill.invoke` | skill routing | Skill triggered | `{ skill, trigger }` |
| `skill.complete` | skill | Skill finishes | `{ duration_ms, success }` |
| `skill.error` | skill | Skill fails | `{ error, context }` |
| `tool.use` | hook.PreToolUse | Tool about to run | `{ tool, input }` |
| `tool.result` | hook.PostToolUse | Tool finished | `{ tool, duration_ms }` |
| `api.call` | inference tool | API request | `{ model, tokens_in, tokens_out }` |
| `api.cost` | inference tool | Cost incurred | `{ cost_usd, model }` |

### Example Events

```jsonl
{"ts":1737574800000,"event_type":"session.start","source":"hook.SessionStart","session_id":"ses_abc123","data":{"model":"claude-opus-4-5","working_dir":"/Users/andreas/project"}}
{"ts":1737574800500,"event_type":"skill.matched","source":"hook.SkillEnforcer","session_id":"ses_abc123","data":{"prompt_preview":"create issue for the bug...","matched_skills":["_JIRA"],"match_types":["keyword"]}}
{"ts":1737574801000,"event_type":"skill.invoke","source":"skill._JIRA","session_id":"ses_abc123","data":{"skill":"_JIRA","trigger":"user_request"}}
{"ts":1737574805000,"event_type":"agent.spawn","source":"Task","session_id":"ses_abc123","agent_id":"agent_001","data":{"agent_type":"Intern","parent":"main"}}
{"ts":1737574810000,"event_type":"skill.error","source":"skill._JIRA","session_id":"ses_abc123","error":{"message":"API authentication failed","code":"AUTH_ERROR"}}
```

---

## Instrumentation Points (Claude Code Hooks)

Claude Code provides 11 lifecycle hooks for instrumentation. **Note:** Skills are markdown files loaded as context—they cannot be instrumented directly. All instrumentation happens via hooks.

### Available Hooks for Observability

| Hook | Blocks? | Best Instrumentation Use |
|------|---------|--------------------------|
| `SessionStart` | No | Initialize session metrics, log session start |
| `SessionEnd` | No | Finalize metrics, session summary, cleanup |
| `PreToolUse` | Yes | Tool timing start, audit logging, command tracking |
| `PostToolUse` | Yes | Tool timing end, result logging, error capture |
| `UserPromptSubmit` | Yes | Detect skill triggers via pattern matching |
| `Stop` | Yes | Session completion metrics |
| `SubagentStop` | Yes | Agent completion, duration tracking |
| `PreCompact` | No | Context window metrics before compaction |

### Hook Input Data

Every hook receives JSON via stdin:

```json
{
  "session_id": "abc123",
  "transcript_path": "/path/to/conversation.jsonl",
  "cwd": "/current/working/directory",
  "hook_event_name": "PreToolUse",
  "tool_name": "Bash",
  "tool_input": { "command": "ls -la" }
}
```

### Skill Tracking via PreToolUse Hook

**Recommended approach:** Use a `PreToolUse` hook on the `Skill` tool rather than pattern matching at `UserPromptSubmit`. This provides **definitive** skill invocation logging.

**Why PreToolUse over UserPromptSubmit pattern matching:**

| Aspect | UserPromptSubmit (Pattern) | PreToolUse (Skill) |
|--------|---------------------------|-------------------|
| Accuracy | Speculative (pattern matched) | Definitive (skill IS firing) |
| Skill name | Inferred from pattern | Exact from TOOL_INPUT |
| False positives | Yes (pattern matches, skill doesn't fire) | No |
| False negatives | Yes (synonym used, pattern misses) | No |

This approach logs what *actually* happens rather than what pattern matching *predicts* will happen.

```typescript
// hooks/SkillInvocation.hook.ts
// PreToolUse matcher: "Skill"

import { appendFileSync } from "fs";

const toolInput = JSON.parse(process.env.TOOL_INPUT || "{}");
const sessionId = process.env.SESSION_ID || "unknown";
const skillName = toolInput.skill;

if (!skillName) process.exit(0);

const event = {
  event_type: "skill.invoked",
  ts: Date.now(),
  session_id: sessionId,
  skill: skillName,
  args: toolInput.args || null,
};

const today = new Date().toISOString().split("T")[0];
const eventsDir = `${process.env.PAI_DIR}/MEMORY/Events`;
appendFileSync(`${eventsDir}/${today}.jsonl`, JSON.stringify(event) + "\n");
```

**settings.json configuration:**

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Skill",
        "hooks": [{
          "type": "command",
          "command": "${PAI_DIR}/hooks/SkillInvocation.hook.ts"
        }]
      }
    ]
  }
}
```

See [pai-skill-enforcer#1](https://github.com/jcfischer/pai-skill-enforcer/issues/1) for architectural discussion on deterministic matching for enforcement vs discovery.

### Existing PAI Hook Infrastructure

PAI already has an observability foundation in `hooks/lib/observability.ts`:
- `sendEventToObservability()` — Posts events to localhost:4000
- Graceful degradation (fails silently if dashboard offline)
- Can be adapted to write JSONL files instead of HTTP

---

## Dependencies

### SkillEnforcer Hook (External) — Optional Discovery Boost

**Source:** https://github.com/jcfischer/pai-skill-enforcer

The SkillEnforcer hook provides deterministic skill surfacing via pattern matching on `UserPromptSubmit`.

**Important:** This is a **discovery boost**, not the primary logging mechanism. Pattern matching at UserPromptSubmit is speculative—it logs what *might* happen based on keyword matches, not what *actually* happens. For accurate skill invocation logging, use the PreToolUse approach above.

**Best use cases for SkillEnforcer:**
- Boosting activation for skills with explicit `/command` triggers
- Guaranteeing specific patterns always surface a skill hint
- Supplementing native LLM routing (not replacing it)

**Limitation:** Narrow matching misses synonyms and paraphrases:
- User says "capture the webpage" → pattern "screenshot" doesn't match
- User says "show me my tickets" → pattern "jira" doesn't match

**Recommended hybrid approach:**
- **Discovery:** Native LLM + USE WHEN descriptions → Semantic matching (handles synonyms)
- **Logging:** PreToolUse hook on Skill tool → Definitive invocation capture
- **Enforcement:** PreToolUse hook also guarantees customisation loading (solves known PAI issue where USER extensions don't reliably load)

#### Community Analysis: Deterministic Matching — Better for Enforcement Than Discovery

*From [GitHub Issue #1](https://github.com/jcfischer/pai-skill-enforcer/issues/1) — Analysis against skill best practices compiled from official docs and community research.*

**The Concern: Matching Might Be Too Narrow for Discovery**

The official Claude Code skill routing uses pure LLM reasoning—semantic understanding rather than pattern matching. The enforcer uses `startsWith`, `includes`, and "all words present" checks. This means:

- User says "capture the webpage" → pattern "screenshot" doesn't match
- User says "show me my tickets" → pattern "jira" doesn't match
- Synonyms, paraphrases, contextual references all miss

The best practices explicitly state: "You're writing for Claude's language understanding, not keyword matching."

**Where This Pattern Shines: Post-Activation Enforcement**

There's a known problem in PAI where skill customisations (USER extensions, EXTEND.yaml files) don't reliably load when a skill activates. Claude inconsistently checks for them—sometimes it does, sometimes it doesn't. Users end up putting manual reminders in CLAUDE.md as a workaround.

Deterministic pattern matching is actually perfect for this—but as a **PreToolUse hook on the Skill tool**, not UserPromptSubmit:

```
User: "help with jira"
    ↓
[Native LLM routing handles discovery - semantic, catches synonyms]
    ↓
Claude decides to call Skill({ skill: "_JIRA" })
    ↓
[PreToolUse hook fires - TOOL_INPUT contains exact skill name]
    ↓
Hook does TWO things:
  1. Logs skill.invoked event (definitive observability capture)
  2. Injects: "Load USER/_JIRA/EXTEND.yaml before proceeding"
    ↓
[100% reliable - no pattern matching needed, we KNOW which skill]
```

This is the critical insight: **one hook handles both observability AND enforcement**. The PreToolUse hook on the Skill tool:
- Knows exactly which skill is being invoked (from TOOL_INPUT)
- Can log the `skill.invoked` event with certainty (not speculation)
- Can inject USER extension loading instructions
- Runs at exactly the right time (after LLM decided, before execution)

**Suggested Hybrid Architecture:**

- **Discovery**: Native LLM + USE WHEN descriptions → Semantic matching (handles synonyms)
- **Enforcement + Logging**: PreToolUse hook on Skill tool → Guarantees customisations load AND logs definitive skill invocations

The enforcer's deterministic approach is valuable—just positioned differently. For discovery, it's a boost/hint. For customization loading, it's the solution.

### skill-registry Library Module

**Source:** [pai-skill-enforcer/src/lib/skill-registry.ts](https://github.com/jcfischer/pai-skill-enforcer/blob/main/src/lib/skill-registry.ts)

The SkillEnforcer hook uses a skill-registry module for pattern matching. Key exports:

```typescript
// Types
export type TriggerType = 'command' | 'keyword' | 'fuzzy';

export interface SkillTrigger {
  pattern: string;                    // The pattern to match against
  type: TriggerType;                  // command (startsWith), keyword (includes), fuzzy (all words)
  priority: number;                   // Higher = matched first (default: command=100, keyword=50, fuzzy=30)
  skillName: string;                  // Name of the skill this trigger belongs to
  skillPath: string;                  // Full path to the skill's SKILL.md file
}

export interface SkillRegistry {
  triggers: SkillTrigger[];           // All triggers, sorted by priority descending
  skillPaths: Map<string, string>;    // Map of skill names to their SKILL.md paths
  loadedAt: string;                   // ISO timestamp when registry was built
  version: number;                    // Schema version for migrations
}

// Constants
export const REGISTRY_VERSION = 1;
export const DEFAULT_PRIORITIES: Record<TriggerType, number> = {
  command: 100,
  keyword: 50,
  fuzzy: 30,
};

// Core Functions
export function matchTriggers(
  prompt: string,
  registry: SkillRegistry,
  options?: { dedupe?: boolean; limit?: number }
): SkillTrigger[];

export function parseSkillTriggers(
  yamlContent: string,
  skillName: string,
  skillPath: string
): SkillTrigger[];

export function extractUseWhenClauses(content: string): string[];
export function generateTriggerYaml(skillName: string, useWhenClauses: string[]): string;
export function serializeRegistry(registry: SkillRegistry): object;
export function deserializeRegistry(data: unknown): SkillRegistry;
export function createEmptyRegistry(): SkillRegistry;

// Path Utilities
export function getPaiDir(): string;                          // Returns PAI_DIR or ~/.claude
export function getRegistryPath(config?: RegistryConfig): string;  // Default: $PAI_HOME/MEMORY/STATE/skill-registry.json
export function getSkillsDir(config?: RegistryConfig): string;
```

**Registry building:**
- Scans `$PAI_HOME/skills/*/SKILL.md` for trigger patterns
- Parses YAML frontmatter `triggers:` field
- Also extracts patterns from `USE WHEN` sections in descriptions
- Caches to `$PAI_HOME/MEMORY/STATE/skill-registry.json`
- Rebuilt by `LoadContext` hook at session start

**SKILL.md Trigger Format:**
```yaml
---
name: Browser
description: Web automation and screenshots
triggers:
  - pattern: "/browser"
    type: command
    priority: 100
  - pattern: "screenshot"
    type: keyword
    priority: 50
  - pattern: "browser automation"
    type: fuzzy
    priority: 30
---
```

---

## Collector Architecture

The collector forwards events from local JSONL files to the observability stack. Two options are available:

| Option | Pros | Cons | Best For |
|--------|------|------|----------|
| **launchd + curl** | Zero dependencies, macOS built-in | Silent failures | Simple setups |
| **Vector** | `vector top` visibility, auto-retry, buffering | Requires install | Production |

**Recommendation:** Start with launchd + curl. Upgrade to Vector if you need visibility into the collector itself.

**How it works:**
1. Collector tails `$PAI_HOME/MEMORY/Events/*.jsonl`
2. Batches new lines, POSTs to VictoriaLogs
3. On success, updates heartbeat file
4. On failure, buffers locally for retry

**Self-monitoring:** A local watchdog (separate launchd job) checks the heartbeat file. If stale > 3 minutes, it alerts via configured channels (voice, Telegram, ntfy) — independent of the stack.

→ *Full scripts, configs, and launchd plists: See Appendix A*

---

## Implementation Phases

### Phase 0: Local Capture (No Stack)

**Goal:** Capture PAI events to queryable local files.

- [ ] Create `$PAI_HOME/MEMORY/Events/` directory structure
- [ ] Implement `logEvent()` helper in TypeScript
- [ ] Instrument SessionStart hook
- [ ] Instrument SessionStop hook
- [ ] Instrument SkillEnforcer hook (skill.matched events)
- [ ] Instrument PreToolUse hook (optional, high volume)
- [ ] Instrument PostToolUse hook (optional, high volume)
- [ ] Document CLI queries (tail, grep, jq patterns)
- [ ] Test: Run session, verify events captured

**Deliverable:** PAI events captured locally, queryable with Unix tools.

### Phase 1: Collector & Watchdog

**Goal:** Background process that forwards events to stack with self-monitoring.

**Choose collector approach:**
- [ ] Option A: Native macOS (launchd + curl) — zero dependencies
- [ ] Option B: Vector — production-grade with automatic retry/buffering

**Collector tasks:**
- [ ] Implement collector script/config (see Collector Architecture)
- [ ] Create launchd plist for auto-start
- [ ] Test: Forward events to VictoriaLogs
- [ ] Test: Kill stack, verify local buffering, restore, verify catch-up

**Watchdog tasks:**
- [ ] Implement heartbeat file write in collector
- [ ] Create watchdog script with configurable notification channels
- [ ] Configure channels in settings.json (voice, Telegram, ntfy, etc.)
- [ ] Create launchd plist for watchdog (2-minute interval)
- [ ] Test: Stop collector, verify watchdog alerts via configured channels

**Deliverable:** Events flow to stack automatically; alerts if link breaks.

### Phase 2: Observability Stack

**Goal:** Deploy VictoriaMetrics stack (containerized) for historical analysis.

**Note:** "VM" throughout this document refers to **VictoriaMetrics** (the time-series database), not virtual machines. The stack runs as Docker containers on OrbStack.

- [ ] Install OrbStack container runtime
- [ ] Deploy VictoriaLogs (Docker container)
- [ ] Deploy VictoriaMetrics (Docker container)
- [ ] Deploy Grafana (Docker container)
- [ ] Configure Grafana datasources
- [ ] Build PAI Sessions dashboard
- [ ] Build PAI Errors dashboard
- [ ] Build PAI API Costs dashboard
- [ ] Test: Query last week's sessions, skill errors, API costs

**Deliverable:** Historical PAI observability in Grafana.

### Phase 3: Alerting & Notifications

**Goal:** Get notified when things go wrong (autonomous agent support).

- [ ] Configure Grafana alerting rules
- [ ] Alert: Skill error rate > threshold
- [ ] Alert: Session idle > 30min (autonomous agent stalled)
- [ ] Alert: API cost > daily budget
- [ ] Route alerts to PAI Notify skill
- [ ] Test: Trigger alert, verify notification received

**Deliverable:** Proactive notifications for autonomous agent issues.

---

## Grafana: The Unified Front-End

Grafana is the single interface for all observability data. Everything happens here.

### Why Grafana Over CLI?

**The aggregation advantage:** When you monitor via CLI on a single machine, you see one agent stack. When you monitor via Grafana at the central observability layer, you see:

- **Agent swarms**: Multiple parallel agents working on a task
- **Cross-machine activity**: If PAI runs on multiple devices (laptop, server, cloud)
- **Historical correlation**: Trends across days/weeks, not just the current session
- **System-wide health**: All agents, all skills, all hooks—one dashboard

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    CLI vs GRAFANA: VISIBILITY SCOPE                          │
│                                                                              │
│   CLI (tail -f | jq)                  GRAFANA (Central Stack)               │
│   ─────────────────                   ───────────────────────               │
│                                                                              │
│   ┌─────────────┐                     ┌─────────────────────────────────┐   │
│   │ This Agent  │                     │ Agent 1 │ Agent 2 │ Agent 3     │   │
│   │ This Machine│                     │ Laptop  │ Server  │ Cloud       │   │
│   │ Right Now   │                     │ Historical + Real-time          │   │
│   └─────────────┘                     └─────────────────────────────────┘   │
│                                                                              │
│   Good for: Quick debugging           Good for: Fleet monitoring,           │
│             Single session                      Swarm debugging,            │
│                                                 Trend analysis              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### CLI vs Grafana: Same Data, Better UX

You can do everything on the command line—but Grafana provides visual tooling that makes analysis faster and more intuitive:

| Task | CLI (Stage 0) | Grafana (Stage 2) |
|------|---------------|-------------------|
| **Filter by session** | `grep "ses_abc123" \| jq` | Type in search box, click session |
| **View trace timeline** | `jq -s 'sort_by(.ts)'` + mental parsing | Visual waterfall, click to expand spans |
| **Find errors** | `grep "skill.error" \| jq` | Click "Errors" tab, see highlighted rows |
| **Track metrics over time** | `cat *.jsonl \| jq -s 'group_by(.date)'` | Line graph, drag to zoom, hover for values |
| **Set up alerts** | Write bash script, manage cron | Click "Alert" → define rule → done |
| **Correlate logs + metrics** | Multiple terminal windows, mental correlation | Split view, click log → see related metrics |

**The CLI approach works** — and it's valuable for Stage 0 when you have no stack. But Grafana turns "possible" into "pleasant."

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    GRAFANA: UNIFIED OBSERVABILITY                            │
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                         GRAFANA (localhost:3000)                     │   │
│   │                                                                      │   │
│   │   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │   │
│   │   │    LOGS      │  │   TRACES     │  │   METRICS    │              │   │
│   │   │  Explore     │  │   Explore    │  │  Dashboards  │              │   │
│   │   │  + Filtering │  │  + Waterfall │  │  + Graphs    │              │   │
│   │   └──────────────┘  └──────────────┘  └──────────────┘              │   │
│   │         │                  │                  │                      │   │
│   │         │                  │                  │                      │   │
│   │   ┌─────▼─────┐      ┌─────▼─────┐      ┌─────▼─────┐               │   │
│   │   │VictoriaLogs│      │VictoriaTrace│    │VictoriaMetrics│           │   │
│   │   │  :9428    │      │   :9411   │      │   :8428   │               │   │
│   │   └───────────┘      └───────────┘      └───────────┘               │   │
│   │                                                                      │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Logs: Filtering with Regex in Grafana Explore

In Grafana's Explore view, use LogsQL to filter logs by session, skill, CLI command, or any pattern:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  GRAFANA EXPLORE: Logs                                       VictoriaLogs ▼ │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Query: {app="pai"} |~ "session_id.*ses_abc123"                    [Run]    │
│                                                                             │
│  ─────────────────────────────────────────────────────────────────────────  │
│                                                                             │
│  Filter Examples:                                                           │
│                                                                             │
│  # By session ID                                                            │
│  {app="pai"} |~ "session_id.*ses_abc123"                                   │
│                                                                             │
│  # By skill name                                                            │
│  {app="pai"} |~ "source.*skill._JIRA"                                      │
│                                                                             │
│  # By CLI tool                                                              │
│  {app="pai"} |~ "tool.*Bash"                                               │
│                                                                             │
│  # Errors only                                                              │
│  {app="pai"} |= "skill.error" OR |= "hook.error"                           │
│                                                                             │
│  # Auth failures (regex)                                                    │
│  {app="pai"} |~ "AUTH_EXPIRED|status.*401|token.*expired"                  │
│                                                                             │
│  # Rate limits                                                              │
│  {app="pai"} |~ "status.*429|RATE_LIMIT"                                   │
│                                                                             │
│  # Slow operations (parse JSON, filter by field)                            │
│  {app="pai"} | json | duration_ms > 5000                                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

**LogsQL quick reference:**

| Pattern | Meaning |
|---------|---------|
| `{app="pai"}` | All PAI logs |
| `\|= "text"` | Contains exact text |
| `\|~ "regex"` | Matches regex pattern |
| `\| json` | Parse as JSON |
| `\| field > value` | Filter parsed field |

### Traces: Viewing Execution Flow

In Grafana's Explore view with the VictoriaTraces datasource, view session execution as a trace waterfall:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  GRAFANA EXPLORE: Traces                                   VictoriaTraces ▼ │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Search: session_id = "ses_abc123"                                 [Search] │
│                                                                             │
│  ─────────────────────────────────────────────────────────────────────────  │
│                                                                             │
│  Trace: ses_abc123 | Duration: 45s | 12 spans                               │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ session.start                                                        │   │
│  │ ├── skill.matched (_JIRA)                                            │   │
│  │ │   └── skill.invoke                                                 │   │
│  │ │       ├── tool.use (Bash: jira issue create)                       │   │
│  │ │       │   └── cli.config (profile: work)                           │   │
│  │ │       │       └── api.call (POST /rest/api/3/issue)                │   │
│  │ │       │           └── api.response (201 Created) ✓                 │   │
│  │ │       └── skill.complete (5.2s)                                    │   │
│  │ └── session.end                                                      │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  Span Details:                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ api.call                                                             │   │
│  │ ├── method: POST                                                     │   │
│  │ ├── url: https://jira.company.com/rest/api/3/issue                  │   │
│  │ ├── auth_type: oauth                                                 │   │
│  │ └── duration: 2.3s                                                   │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Metrics: PromQL Dashboards

VictoriaMetrics is Prometheus-compatible. Use PromQL in Grafana dashboards:

```promql
# Total sessions today
increase(pai_sessions_total[24h])

# Average session duration
avg(pai_session_duration_seconds)

# Error rate by skill
sum(rate(pai_skill_errors_total[5m])) by (skill) /
sum(rate(pai_skill_invocations_total[5m])) by (skill)

# API cost by model (last 24h)
sum(increase(pai_api_cost_usd_total[24h])) by (model)

# Hook execution time p95
histogram_quantile(0.95, sum(rate(pai_hook_duration_seconds_bucket[5m])) by (le, hook))

# Active sessions
pai_active_sessions
```

### Alerting: Grafana Alert Rules

Define all alerting in Grafana. Alerts can be based on logs (LogsQL) or metrics (PromQL):

```yaml
# ~/observability/grafana/provisioning/alerting/pai-alerts.yaml
apiVersion: 1
groups:
  - name: pai_alerts
    folder: PAI
    interval: 1m
    rules:
      # ─────────────────────────────────────────────
      # LOG-BASED ALERTS (LogsQL)
      # ─────────────────────────────────────────────

      - name: SkillErrorSpike
        condition: count
        data:
          - refId: A
            datasourceUid: victorialogs
            model:
              expr: '{app="pai"} |= "skill.error"'
        for: 2m
        threshold: 3
        annotations:
          summary: "Skill errors spiking"
          description: "More than 3 skill errors in 5 minutes"

      - name: AuthenticationFailures
        condition: count
        data:
          - refId: A
            datasourceUid: victorialogs
            model:
              expr: '{app="pai"} |~ "AUTH_EXPIRED|status.*401"'
        for: 1m
        threshold: 2
        annotations:
          summary: "Authentication failures detected"

      - name: APIRateLimits
        condition: count
        data:
          - refId: A
            datasourceUid: victorialogs
            model:
              expr: '{app="pai"} |~ "status.*429|RATE_LIMIT"'
        for: 5m
        threshold: 3
        annotations:
          summary: "API rate limits being hit"

      # ─────────────────────────────────────────────
      # METRIC-BASED ALERTS (PromQL)
      # ─────────────────────────────────────────────

      - name: HighAPICost
        condition: gt
        data:
          - refId: A
            datasourceUid: victoriametrics
            model:
              expr: sum(increase(pai_api_cost_usd_total[24h]))
        for: 5m
        threshold: 15
        annotations:
          summary: "API cost exceeds $15/day"

      - name: SlowHooks
        condition: gt
        data:
          - refId: A
            datasourceUid: victoriametrics
            model:
              expr: histogram_quantile(0.95, sum(rate(pai_hook_duration_seconds_bucket[5m])) by (le))
        for: 5m
        threshold: 30
        annotations:
          summary: "Hook execution p95 > 30 seconds"

      - name: SessionIdle
        condition: gt
        data:
          - refId: A
            datasourceUid: victoriametrics
            model:
              expr: time() - pai_last_event_timestamp
        for: 5m
        threshold: 1800  # 30 minutes
        annotations:
          summary: "Autonomous session idle > 30 minutes"
```

**Alert notification channels:**

Configure in Grafana UI (Alerting → Contact points):
- **Email** — SMTP configuration
- **Slack/Discord** — Webhook URL
- **PagerDuty** — Integration key
- **Webhook** — POST to PAI voice server, ntfy, etc.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  Grafana Alert → Contact Point → Notification                               │
│                                                                              │
│  Example: Route to PAI voice server                                         │
│                                                                              │
│  Contact Point: pai-voice                                                   │
│  Type: Webhook                                                               │
│  URL: http://localhost:4123/speak                                           │
│  Method: POST                                                                │
│  Body: {"text": "{{ .CommonAnnotations.summary }}"}                         │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Observability Stack Infrastructure

Four containers running on OrbStack:

| Service | Port | Purpose | Retention |
|---------|------|---------|-----------|
| **VictoriaMetrics** | 8428 | Time series metrics (Prometheus-compatible) | 12 months |
| **VictoriaLogs** | 9428 | Log aggregation | 90 days |
| **VictoriaTraces** | 9411 | Distributed tracing (Zipkin-compatible) | 30 days |
| **Grafana** | 3000 | Visualization & alerting | — |

**Resource usage:** ~350-450MB RAM total, <4% CPU idle.

**Quick start:**
```bash
cd ~/observability && docker compose up -d
open http://localhost:3000  # Grafana
```

**Auto-start:** OrbStack starts on Mac login. Containers with `restart: unless-stopped` auto-start when OrbStack runs.

**Persistent storage:** Named Docker volumes (`observability-vm-data`, `observability-vmlogs-data`, etc.) persist across container restarts. Your monitoring data survives `docker compose down` and Mac reboots. Only `docker compose down -v` destroys data.

→ *Full docker-compose.yml, datasource configs, and commands: See Appendix B*

---

## Directory Structure

```
$PAI_HOME/
├── MEMORY/
│   ├── Events/                    # NEW: Event capture
│   │   ├── 2026-01-22.jsonl      # Today's events
│   │   ├── 2026-01-21.jsonl      # Yesterday's events
│   │   └── ...
│   ├── Signals/                   # Existing: ratings, failures, loopbacks
│   ├── Learning/                  # Existing: phase-based learnings
│   ├── History/                   # Existing: session summaries
│   └── Work/                      # Existing: per-task traces
├── Observability/                 # NEW: Collector and tooling
│   ├── collector/
│   │   ├── index.ts              # Collector daemon
│   │   ├── config.yaml           # Stack endpoints, buffer settings
│   │   └── com.pai.collector.plist  # launchd auto-start
│   └── queries/                   # Saved jq/grep patterns
│       ├── errors-today.sh
│       ├── session-duration.sh
│       └── api-costs.sh
└── ...
```

---

## Integration with PAI Memory System

Events feed into the existing PAI learning loop:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    THE ALGORITHM FEEDBACK LOOP                               │
│                                                                              │
│   Events (new)              Signals (existing)        Learning (existing)   │
│   ───────────               ────────────────          ──────────────────    │
│   session.start             ratings.jsonl             OBSERVE/              │
│   skill.matched      →      failures.jsonl      →     THINK/                │
│   skill.error               loopbacks.jsonl           PLAN/                 │
│   agent.complete            patterns.jsonl            BUILD/                │
│   api.cost                                            EXECUTE/              │
│                                                       VERIFY/               │
│                                                       ALGORITHM/            │
│                                                                              │
│   Raw event stream          Aggregated signals        Curated learnings     │
│   (high volume)             (processed)               (algorithm upgrades)  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Events → Signals:** Collector can detect patterns (e.g., "skill.error 3x in 5min") and write to Signals.

**Signals → Learning:** Existing PAI learning loop reviews Signals and extracts learnings.

---

## Security Considerations

| Concern | Mitigation |
|---------|------------|
| Sensitive data in events | Scrub PII before logging (no passwords, keys, personal data) |
| Collector as attack surface | No inbound listeners, outbound-only HTTPS |
| Stack credentials | Stored in `$PAI_HOME/.env`, not in code |
| Log retention | Auto-rotate: 30 days local, 90 days stack |
| Multi-agent isolation | Each agent gets unique agent_id, session_id links them |

---

## Success Criteria

| Criterion | Metric | Phase |
|-----------|--------|-------|
| Events captured | 100% of sessions have start/end events | 0 |
| Local queryable | Can answer "errors today?" with grep/jq | 0 |
| Forwarding works | Events appear in VictoriaLogs < 60s | 1 |
| Offline resilience | No lost events during 1hr stack outage | 1 |
| Historical queries | Can query 30 days of sessions in Grafana | 2 |
| Alerting works | Skill error alert fires within 5min | 3 |
| Notification received | Alert → Voice/Push notification | 3 |

---

## References

### Stack Components
- [VictoriaMetrics](https://victoriametrics.com/) — Time series metrics (30x less RAM than Prometheus)
- [VictoriaLogs](https://docs.victoriametrics.com/victorialogs/) — Log aggregation (15x less disk than Loki)
- [VictoriaTraces](https://docs.victoriametrics.com/victorialogs/) — Distributed tracing (Zipkin-compatible)
- [Vector](https://vector.dev/) — Observability data pipeline (optional collector)
- [OrbStack](https://orbstack.dev/) — Lightweight Docker runtime for macOS

### Dependencies
- [SkillEnforcer Hook](https://github.com/jcfischer/pai-skill-enforcer) — Deterministic skill surfacing via pattern matching
- [SkillEnforcer Issue #1](https://github.com/jcfischer/pai-skill-enforcer/issues/1) — Analysis: deterministic matching better for enforcement than discovery

---

## Contributors & Influences

This design was shaped by community input:

| Contributor | Contribution |
|-------------|--------------|
| **Jens-Christian Fischer** | [SkillEnforcer hook](https://github.com/jcfischer/pai-skill-enforcer) — deterministic skill surfacing via pattern matching; [Issue #1](https://github.com/jcfischer/pai-skill-enforcer/issues/1) analysis on discovery vs enforcement |
| **Rudy Ruiz** | [Argus](https://github.com/nickpending/argus) — synchronous event capture pattern, session/agent tracking |
| **Kayvan** | Maestro + runbooks workflow — implementation methodology |

---

# Appendix

Detailed configurations for implementers. Readers reviewing the design can skip this section.

---

## Appendix A: Collector Scripts & Configs

### A.1 Native macOS Collector (launchd + curl)

```bash
#!/bin/bash
# $PAI_HOME/Observability/collector.sh
# Zero dependencies - uses only macOS built-ins

EVENTS_DIR="${PAI_HOME}/MEMORY/Events"
HEARTBEAT_FILE="${PAI_HOME}/MEMORY/.collector-heartbeat"
VICTORIA_LOGS="http://localhost:9428/insert/jsonline"
BUFFER_DIR="${EVENTS_DIR}/buffer"

mkdir -p "$BUFFER_DIR"

TODAY=$(date +%Y-%m-%d)
EVENT_FILE="${EVENTS_DIR}/${TODAY}.jsonl"
POSITION_FILE="${EVENTS_DIR}/.position"
LAST_POS=$(cat "$POSITION_FILE" 2>/dev/null || echo 0)

if [ -f "$EVENT_FILE" ]; then
  NEW_LINES=$(tail -c +$((LAST_POS + 1)) "$EVENT_FILE")
  if [ -n "$NEW_LINES" ]; then
    if echo "$NEW_LINES" | curl -s -X POST \
        --data-binary @- \
        -H "Content-Type: application/json" \
        "$VICTORIA_LOGS" > /dev/null 2>&1; then
      date +%s > "$HEARTBEAT_FILE"
      wc -c < "$EVENT_FILE" > "$POSITION_FILE"
    else
      echo "$NEW_LINES" >> "${BUFFER_DIR}/${TODAY}.jsonl"
    fi
  fi
fi
```

### A.2 launchd plist (runs every 30 seconds)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.pai.collector</string>
    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>/Users/andreas/.claude/Observability/collector.sh</string>
    </array>
    <key>StartInterval</key>
    <integer>30</integer>
    <key>RunAtLoad</key>
    <true/>
    <key>EnvironmentVariables</key>
    <dict>
        <key>PAI_HOME</key>
        <string>/Users/andreas/.claude</string>
    </dict>
</dict>
</plist>
```

### A.3 Vector Configuration (Logs Only)

```yaml
# $PAI_HOME/Observability/vector.yaml
# Minimal config: forwards events to VictoriaLogs only

sources:
  pai_events:
    type: file
    include:
      - ${PAI_HOME}/MEMORY/Events/*.jsonl
    read_from: beginning

transforms:
  parse_json:
    type: remap
    inputs: [pai_events]
    source: |
      . = parse_json!(.message)

sinks:
  victoria_logs:
    type: http
    inputs: [parse_json]
    uri: http://localhost:9428/insert/jsonline
    encoding:
      codec: json
    healthcheck:
      enabled: true

  local_backup:
    type: file
    inputs: [parse_json]
    path: ${PAI_HOME}/MEMORY/Events/buffer/%Y-%m-%d.jsonl
    encoding:
      codec: json
```

### A.3.1 Vector Configuration (Logs + Derived Metrics)

```yaml
# $PAI_HOME/Observability/vector-with-metrics.yaml
# Full config: forwards events to VictoriaLogs AND derives metrics for VictoriaMetrics

sources:
  pai_events:
    type: file
    include:
      - ${PAI_HOME}/MEMORY/Events/*.jsonl
    read_from: beginning

transforms:
  parse_json:
    type: remap
    inputs: [pai_events]
    source: |
      . = parse_json!(.message)

  # Event-to-Metric Transformation
  # Extracts numeric values from textual events and emits Prometheus metrics
  events_to_metrics:
    type: log_to_metric
    inputs: [parse_json]
    metrics:
      # Counter: increment on each event occurrence
      - type: counter
        field: event_type
        name: pai_events_total
        tags:
          event_type: "{{event_type}}"
          skill: "{{data.skill}}"

      # Counter: track errors by skill and code
      - type: counter
        field: event_type
        name: pai_skill_errors_total
        tags:
          skill: "{{data.skill}}"
          error_code: "{{data.error_code}}"
        # Only emit for error events
        # (Vector filters on field presence)

      # Gauge: session duration from numeric field
      - type: gauge
        field: data.duration_seconds
        name: pai_session_duration_seconds
        tags:
          skill: "{{data.skill}}"

      # Counter: API cost tracking
      - type: counter
        field: data.cost_usd
        name: pai_api_cost_usd_total
        increment_by_value: true
        tags:
          model: "{{data.model}}"
          skill: "{{data.skill}}"

sinks:
  # Forward textual events to VictoriaLogs
  victoria_logs:
    type: http
    inputs: [parse_json]
    uri: http://localhost:9428/insert/jsonline
    encoding:
      codec: json
    healthcheck:
      enabled: true

  # Forward derived metrics to VictoriaMetrics (Prometheus remote write)
  victoria_metrics:
    type: prometheus_remote_write
    inputs: [events_to_metrics]
    endpoint: http://localhost:8428/api/v1/write
    healthcheck:
      enabled: true

  local_backup:
    type: file
    inputs: [parse_json]
    path: ${PAI_HOME}/MEMORY/Events/buffer/%Y-%m-%d.jsonl
    encoding:
      codec: json
```

### A.4 Watchdog Script

```bash
#!/bin/bash
# $PAI_HOME/Observability/watchdog.sh

HEARTBEAT_FILE="${PAI_HOME}/MEMORY/.collector-heartbeat"
STALE_THRESHOLD=180  # 3 minutes

if [ ! -f "$HEARTBEAT_FILE" ]; then
  echo "Heartbeat file missing"
  exit 1
fi

HEARTBEAT=$(cat "$HEARTBEAT_FILE")
NOW=$(date +%s)
AGE=$((NOW - HEARTBEAT))

if [ $AGE -gt $STALE_THRESHOLD ]; then
  bun "${PAI_HOME}/Observability/notify-watchdog.ts" \
    --message "Collector heartbeat stale: ${AGE}s (threshold: ${STALE_THRESHOLD}s)" \
    --severity warning
fi
```

### A.5 Watchdog Configuration (settings.json)

```json
{
  "observability": {
    "watchdog": {
      "enabled": true,
      "heartbeat_path": "${PAI_HOME}/MEMORY/.collector-heartbeat",
      "stale_threshold_seconds": 180,
      "check_interval_seconds": 120,
      "channels": [
        { "type": "voice", "enabled": true },
        { "type": "telegram", "enabled": false },
        { "type": "ntfy", "enabled": false },
        { "type": "macos_notification", "enabled": true }
      ]
    }
  }
}
```

---

## Appendix B: Observability Stack Infrastructure

### B.1 docker-compose.yml

```yaml
# ~/observability/docker-compose.yml
services:
  victoriametrics:
    image: victoriametrics/victoria-metrics:latest
    container_name: vm
    ports:
      - "8428:8428"
    volumes:
      - vm-data:/storage
    command:
      - "-storageDataPath=/storage"
      - "-retentionPeriod=12"
      - "-selfScrapeInterval=10s"
      - "-httpListenAddr=:8428"
    restart: unless-stopped
    labels:
      - "pai.system=observability"

  victorialogs:
    image: victoriametrics/victoria-logs:latest
    container_name: vmlogs
    ports:
      - "9428:9428"
    volumes:
      - vmlogs-data:/vlogs
    command:
      - "-storageDataPath=/vlogs"
      - "-retentionPeriod=90d"
      - "-httpListenAddr=:9428"
    restart: unless-stopped
    labels:
      - "pai.system=observability"

  victoriatraces:
    image: victoriametrics/victoria-traces:latest
    container_name: vmtraces
    ports:
      - "9411:9411"
    volumes:
      - vmtraces-data:/vtraces
    command:
      - "-storageDataPath=/vtraces"
      - "-retentionPeriod=30d"
      - "-httpListenAddr=:9411"
    restart: unless-stopped
    labels:
      - "pai.system=observability"

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana-data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD:-changeme}
      - GF_USERS_ALLOW_SIGN_UP=false
      - GF_SERVER_ROOT_URL=http://localhost:3000
      - GF_INSTALL_PLUGINS=victoriametrics-logs-datasource
    restart: unless-stopped
    depends_on:
      - victoriametrics
      - victorialogs
      - victoriatraces
    labels:
      - "pai.system=observability"

volumes:
  vm-data:
    name: observability-vm-data
  vmlogs-data:
    name: observability-vmlogs-data
  vmtraces-data:
    name: observability-vmtraces-data
  grafana-data:
    name: observability-grafana-data
```

### B.2 Grafana Datasource Provisioning

```yaml
# ~/observability/grafana/provisioning/datasources/datasources.yml
apiVersion: 1

datasources:
  - name: VictoriaMetrics
    type: prometheus
    access: proxy
    url: http://victoriametrics:8428
    isDefault: true
    editable: false

  - name: VictoriaLogs
    type: victoriametrics-logs-datasource
    access: proxy
    url: http://victorialogs:9428
    editable: false

  - name: VictoriaTraces
    type: jaeger
    access: proxy
    url: http://victoriatraces:9411
    editable: false
```

### B.3 Stack Commands

```bash
# Start stack
cd ~/observability && docker compose up -d

# View logs
docker compose logs -f

# Stop stack (preserves data)
docker compose down

# Stop and remove all data (destructive!)
docker compose down -v

# Check status
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

### B.4 Access URLs

| Service | URL | Credentials |
|---------|-----|-------------|
| **Grafana** | http://localhost:3000 | admin / (from .env) |
| **VictoriaMetrics UI** | http://localhost:8428/vmui | — |
| **VictoriaLogs** | http://localhost:9428 | — |
| **VictoriaTraces** | http://localhost:9411 | — |

---

## Appendix C: Alternative Backend Configurations

The JSONL event format is the same for all backends. Only the collector sink changes.

### C.1 VictoriaLogs (Default)

```bash
# collector.sh - VictoriaLogs
ENDPOINT="http://localhost:9428/insert/jsonline"
curl -X POST --data-binary @- "$ENDPOINT"
```

### C.2 Grafana Cloud (Loki)

```bash
# collector.sh - Grafana Cloud
ENDPOINT="https://logs-prod-us-central1.grafana.net/loki/api/v1/push"
AUTH="${GRAFANA_USER}:${GRAFANA_API_KEY}"
# Note: Loki expects different format - needs label wrapper
```

```yaml
# vector.yaml - Grafana Cloud
sinks:
  grafana_cloud:
    type: loki
    inputs: [parse_json]
    endpoint: https://logs-prod-us-central1.grafana.net
    auth:
      strategy: basic
      user: "${GRAFANA_CLOUD_USER}"
      password: "${GRAFANA_CLOUD_API_KEY}"
    labels:
      app: pai
      host: "{{ host }}"
```

### C.3 InfluxDB Cloud

```yaml
# vector.yaml - InfluxDB Cloud
sinks:
  influxdb_cloud:
    type: influxdb_logs
    inputs: [parse_json]
    endpoint: https://us-east-1-1.aws.cloud2.influxdata.com
    org: "${INFLUX_ORG}"
    bucket: pai-events
    token: "${INFLUX_TOKEN}"
```

### C.4 Self-Hosted on AWS EC2

Same as local VictoriaMetrics stack, but:
- Run `docker-compose up` on EC2 instance
- Configure security group for ports 8428, 9428, 3000
- Collector points to EC2 public/private IP

```bash
# collector.sh - EC2 hosted stack
ENDPOINT="http://ec2-xx-xx-xx-xx.compute.amazonaws.com:9428/insert/jsonline"
```

---

*PAI Observability Requirements - Simplified, file-based, PAI-first design. January 2026.*
