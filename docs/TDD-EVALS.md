# TDD and Evals Methodology

**Purpose**: Anthropic's guidance on evaluations for AI agents, adapted for PAI development. Use this for test-driven development and quality verification.

---

## Why Build Evaluations?

Good evaluations help teams ship AI agents more confidently. Without them, it's easy to get stuck in reactive loops—catching issues only in production.

**The Breaking Point**: When users report the agent feels worse after changes, and the team has no way to verify except to guess and check.

**With Evals**:
- Distinguish real regressions from noise
- Automatically test changes against hundreds of scenarios before shipping
- Measure improvements objectively

---

## Evaluation Structure

### Core Definitions

| Term | Definition |
|------|------------|
| **Task** | A single test with defined inputs and success criteria |
| **Trial** | Each attempt at a task (run multiple for consistency) |
| **Grader** | Logic that scores some aspect of agent performance |
| **Transcript** | Complete record of a trial (outputs, tool calls, reasoning) |
| **Outcome** | Final state in the environment after the trial |

### Grader Types

| Type | Methods | Strengths | Weaknesses |
|------|---------|-----------|------------|
| **Code-based** | String match, binary tests, static analysis, outcome verification | Fast, cheap, objective, reproducible | Brittle to valid variations |
| **Model-based** | Rubric scoring, natural language assertions, pairwise comparison | Flexible, captures nuance, handles open-ended tasks | Non-deterministic, more expensive |
| **Human** | SME review, crowdsourced judgment, spot-check sampling | Gold standard quality, matches expert judgment | Expensive, slow |

---

## Capability vs. Regression Evals

### Capability Evals ("What can this do well?")
- Start at **low pass rate**
- Target tasks the agent struggles with
- Give teams a hill to climb

### Regression Evals ("Does it still work?")
- Should have **~100% pass rate**
- Protect against backsliding
- Run continuously to catch drift

**Graduation**: High-performing capability evals become regression tests.

---

## PAI-Specific Eval Patterns

### For Observability Features

```yaml
task:
  id: "event-capture_1"
  desc: "Verify SessionStart event is captured with correct schema"
  graders:
    - type: deterministic_tests
      required: [test_session_start_captured.ts, test_event_schema.ts]
    - type: state_check
      expect:
        event_log: contains_event("SessionStart")
        event_schema: valid_against("schemas/SessionStart.json")
    - type: llm_rubric
      rubric: prompts/event_quality.md
  tracked_metrics:
    - type: transcript
      metrics: [n_events_captured, n_schema_errors]
```

### For Hook Integration

```yaml
task:
  id: "hook-integration_1"
  desc: "Verify hook fires on tool completion"
  graders:
    - type: deterministic_tests
      required: [test_hook_fires.ts]
    - type: state_check
      expect:
        hook_log: contains("ToolResult")
        timing: within_ms(100)
```

---

## Writing Good Tasks

### Task Quality Checklist

- [ ] **Unambiguous** - Two experts would reach same pass/fail verdict
- [ ] **Passable** - An agent following instructions correctly can pass
- [ ] **Complete** - Everything grader checks is clear from description
- [ ] **Reference solution** - Known-working output that passes all graders

### Balanced Problem Sets

Test both directions:
- Cases where behavior **should** occur
- Cases where behavior **shouldn't** occur

**Example**: If testing "agent searches when it should", also test "agent doesn't search when it shouldn't".

---

## The pass@k and pass^k Metrics

### pass@k
Likelihood of at least **one** correct solution in k attempts.
- As k increases, pass@k rises
- Use when **one success matters**

### pass^k
Probability that **all k** trials succeed.
- As k increases, pass^k falls
- Use when **consistency is essential**

| k | pass@k (75% per-trial) | pass^k (75% per-trial) |
|---|------------------------|------------------------|
| 1 | 75% | 75% |
| 3 | 98% | 42% |
| 10 | ~100% | 6% |

**For production agents**: Use pass^k for reliability.

---

## Eval-Driven Development

### The Process

1. **Identify gaps** - Run Claude without the feature, document failures
2. **Create evaluations** - Build 3+ test scenarios
3. **Establish baseline** - Measure performance without feature
4. **Write minimal implementation** - Just enough to pass evaluations
5. **Iterate** - Test, compare, refine

### When to Build Evals

| Stage | What to Build |
|-------|---------------|
| **Pre-launch** | Capability evals for planned features |
| **Launch** | Regression suite for core behaviors |
| **Post-launch** | Production monitoring + eval expansion |

---

## Grader Design Best Practices

### 1. Prefer Determinism Where Possible

```typescript
// Good: Deterministic
expect(event.type).toBe("SessionStart");
expect(event.timestamp).toMatch(/^\d{4}-\d{2}-\d{2}T/);

// Avoid: Non-deterministic
expect(event.description).toContain("session");  // LLM might word differently
```

### 2. Grade Outcomes, Not Paths

```typescript
// Good: Grade the result
expect(eventLog).toContainEvent({ type: "SessionStart" });

// Avoid: Grade the exact path
expect(toolCalls[0].name).toBe("Write");  // Agent might use different approach
```

### 3. Build in Partial Credit

```typescript
const score = {
  eventCaptured: eventLog.has("SessionStart") ? 50 : 0,
  schemaValid: validateSchema(event) ? 30 : 0,
  timingCorrect: event.timing < 100 ? 20 : 0,
};
return score.eventCaptured + score.schemaValid + score.timingCorrect;
```

### 4. Make Graders Cheat-Resistant

The agent shouldn't be able to "game" the eval.

```typescript
// Bad: Agent could just echo the expected output
expect(output).toContain("Event captured successfully");

// Good: Verify actual state change
expect(fs.existsSync(eventLogPath)).toBe(true);
expect(JSON.parse(fs.readFileSync(eventLogPath))).toHaveLength(1);
```

---

## Validation Gate

Before marking implementation complete:

- [ ] **Deterministic tests pass** (unit, integration)
- [ ] **Capability eval** shows improvement over baseline
- [ ] **Regression tests** still pass
- [ ] **Transcripts reviewed** - failures seem fair
- [ ] **Multiple trials run** - results are consistent

---

## Real Test Pattern Examples

The following patterns are extracted from PAI's production test suite.

### Unit Test: Date Parsing with Mocked Time

Deterministic tests require controlled time. Mock `Date.now()` for predictable results.

```typescript
import { describe, it, expect, beforeEach, afterEach } from "bun:test";
import { parseSince } from "../lib/search";

describe("parseSince", () => {
  // Use a fixed "now" for predictable tests
  let originalDate: typeof Date;
  const MOCK_NOW = new Date("2025-12-08T12:00:00.000Z");

  beforeEach(() => {
    // Mock Date.now() for predictable tests
    originalDate = global.Date;
    const MockDate = class extends Date {
      constructor(...args: any[]) {
        if (args.length === 0) {
          super(MOCK_NOW.getTime());
        } else {
          // @ts-ignore
          super(...args);
        }
      }
      static now() {
        return MOCK_NOW.getTime();
      }
    } as typeof Date;
    global.Date = MockDate;
  });

  afterEach(() => {
    global.Date = originalDate;
  });

  it("parses '7d' as 7 days ago", () => {
    const result = parseSince("7d");
    expect(result).not.toBeNull();
    expect(result!.toISOString().split("T")[0]).toBe("2025-12-01");
  });

  it("returns null for invalid format", () => {
    expect(parseSince("7x")).toBeNull();
    expect(parseSince("abc")).toBeNull();
    expect(parseSince("")).toBeNull();
  });
});
```

**Key patterns**:
- Mock time for determinism
- Test both valid and invalid inputs
- Use `beforeEach`/`afterEach` for setup/teardown

### CLI Integration Test: Spec-Driven Approach

Define test specs declaratively, run them with a generic runner.

```typescript
// spec file: taxonomy-cli.spec.ts
export interface CLITestSpec {
  id: string;
  name: string;
  command: string;
  expected: {
    contains?: string[];
    notContains?: string[];
    exitCode?: number;
  };
  skip?: boolean;
}

export const taxonomyCLITestSpecs: CLITestSpec[] = [
  {
    id: "TEST-TAX-CLI-001",
    name: "Taxonomy list shows available taxonomies",
    command: "bun run ctx.ts taxonomy list",
    expected: {
      contains: ["Available taxonomies", "default"],
      exitCode: 0,
    },
  },
  {
    id: "TEST-TAX-CLI-005",
    name: "Taxonomy validate rejects invalid tags",
    command: 'bun run ctx.ts taxonomy validate "INVALID,type/fleeting"',
    expected: {
      contains: ["Invalid tags", "INVALID"],
      exitCode: 1,
    },
  },
];
```

```typescript
// runner file: cli-specs.test.ts
import { describe, it, expect } from "bun:test";
import { exec } from "child_process";
import { promisify } from "util";
import { taxonomyCLITestSpecs, CLITestSpec } from "../specs/taxonomy-cli.spec";

const execAsync = promisify(exec);

async function runCommand(command: string) {
  try {
    const { stdout, stderr } = await execAsync(command, { timeout: 60000 });
    return { stdout, stderr, exitCode: 0 };
  } catch (error: any) {
    return { stdout: error.stdout || "", stderr: error.stderr || "", exitCode: error.code || 1 };
  }
}

function runSpec(spec: CLITestSpec) {
  const testFn = spec.skip ? it.skip : it;
  testFn(spec.name, async () => {
    const result = await runCommand(spec.command);
    const output = result.stdout + result.stderr;

    if (spec.expected.contains) {
      for (const text of spec.expected.contains) {
        expect(output).toContain(text);
      }
    }
    if (spec.expected.notContains) {
      for (const text of spec.expected.notContains) {
        expect(output).not.toContain(text);
      }
    }
    if (spec.expected.exitCode !== undefined) {
      expect(result.exitCode).toBe(spec.expected.exitCode);
    }
  });
}

describe("ctx CLI Integration Tests", () => {
  for (const spec of taxonomyCLITestSpecs) {
    runSpec(spec);
  }
});
```

**Key patterns**:
- Separate spec definitions from test runner
- Specs reference requirement IDs (TEST-TAX-CLI-001)
- Test both success and failure cases (exitCode)
- Skip flag for slow or flaky tests

### Test Organization

```
project/
├── test/
│   ├── unit/           # Fast, isolated tests
│   │   ├── parseSince.test.ts
│   │   └── schema.test.ts
│   ├── integration/    # CLI and system tests
│   │   └── cli-specs.test.ts
│   ├── specs/          # Declarative test specifications
│   │   ├── taxonomy-cli.spec.ts
│   │   └── rename-cli.spec.ts
│   └── fixtures/       # Test data and mocks
│       └── sample-event.json
```

### Checklist: Test Quality

| Criterion | Check |
|-----------|-------|
| **Deterministic** | Run 3 times, all same result |
| **Fast** | Unit tests complete in <100ms |
| **Isolated** | No test depends on another's state |
| **Readable** | Test name describes what's tested |
| **Verifiable** | Outcome is binary pass/fail |

---

## Source

Extracted from: [Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)
