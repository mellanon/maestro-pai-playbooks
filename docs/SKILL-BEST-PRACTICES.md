# Skill Development Best Practices

**Purpose**: Quality criteria for PAI skill development. Use these to validate skill structure and activation reliability.

---

## Skill Activation Statistics

Real-world testing across 200+ prompts reveals significant variation in activation reliability:

| Approach | Success Rate | Key Characteristic |
|----------|-------------|-------------------|
| No optimization | ~20% | Baseline/default behavior |
| Simple description | 20% | Vague trigger language |
| **Optimized description** | **50%** | Specific USE WHEN patterns |
| LLM pre-eval hook | 80% | API-based pre-screening |
| Forced eval hook | 84% | Explicit evaluation requirement |

**Target**: 50%+ activation through optimized descriptions.

---

## SKILL.md Structure

```yaml
---
name: your-skill-name
description: Brief explanation of what the skill does and when to use it
allowed-tools: [Optional list of permitted tools]
---

# Skill Documentation

## Instructions
Step-by-step guidance for Claude

## Examples
Concrete usage scenarios
```

### Naming Conventions

**Use gerund form (verb + -ing):**
- `processing-pdfs`
- `analyzing-spreadsheets`
- `managing-databases`
- `observing-events`

### Description Requirements

| Field | Required | Constraints |
|-------|----------|-------------|
| `name` | Yes | Max 64 chars, lowercase letters/numbers/hyphens only |
| `description` | Yes | Max 1024 chars, contains WHAT and WHEN |
| `allowed-tools` | No | Array of tool names |

---

## Description Writing Checklist

The description is **critical for skill discovery**. Claude uses it to choose the right skill.

### The Two-Part Structure

Every description must answer:
1. **What does it do?** (Capability statement)
2. **When to use it?** (Trigger conditions)

**Effective Pattern:**
```yaml
description: Extract text and tables from PDF files, fill forms, merge documents.
  Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
```

### USE WHEN Patterns

Include "Use when..." language with explicit triggers:

```yaml
description: |
  Knowledge Management for Obsidian vault. USE WHEN user asks
  "what do I know about X", "find notes about", "load context
  for project", "save to vault", "capture this", "validate tags".
```

### Optimization Checklist

- [ ] Written in **third person** (no "I" or "you")
- [ ] Under **1024 characters**
- [ ] Contains both **WHAT and WHEN**
- [ ] Includes **5+ specific trigger keywords**
- [ ] Uses **exact terms** from actual user requests
- [ ] Mentions **file types, formats, or domains**
- [ ] Tested with **3+ real scenarios**

---

## Progressive Disclosure

Skills use a three-level loading mechanism:

| Level | What Loads | When | Token Budget |
|-------|------------|------|--------------|
| Level 1 | Metadata (name + description) | Always at startup | ~100 tokens |
| Level 2 | Main SKILL.md body | When skill is triggered | Under 5k tokens |
| Level 3 | Referenced files (docs/, scripts/) | On-demand via Read tool | As needed |

**Key Rule**: Keep SKILL.md under **500 lines** and reference additional files for detailed documentation.

---

## Recommended Folder Structure

```
skill-name/
├── SKILL.md              # Entry point (required, <500 lines)
├── docs/                 # Reference documentation
│   ├── CLI-REFERENCE.md
│   ├── CONCEPTS.md
│   └── EXAMPLES.md
├── workflows/            # Operational procedures
│   ├── workflow-a.md
│   └── workflow-b.md
├── scripts/              # Executable helpers
│   └── helper.py
└── templates/            # Reusable templates
    └── template.txt
```

---

## The Five Fixes That Work

Based on analysis of 40+ skill failures:

### Fix #1: Write Specific Activation Triggers

Include exact keywords from your actual workflow:

```yaml
# Instead of:
description: Code review tool

# Use:
description: Review code for best practices, potential bugs, and maintainability.
  Use when reviewing pull requests, checking code quality, analyzing diffs,
  or when user mentions "review", "PR", "code quality", or "best practices".
```

### Fix #2: Show Real Examples (Not Descriptions)

Examples should be **longer than your rules section**:

```markdown
## Event Capture Format

**Example 1:**
Input: Tool call completed
Output:
{
  "type": "tool_result",
  "timestamp": "2026-01-22T12:00:00Z",
  "tool": "Edit",
  "success": true,
  "duration_ms": 150
}
```

### Fix #3: Progressive Disclosure

Keep SKILL.md under 500 lines:

```markdown
# SKILL.md
## Quick start
[Essential info here]

## Advanced features
**Event schemas**: See [EVENTS.md](docs/EVENTS.md)
**API reference**: See [REFERENCE.md](docs/REFERENCE.md)
```

### Fix #4: Set Explicit Boundaries

Define what the skill **doesn't** do:

```markdown
## Out of Scope
This skill does NOT:
- Handle external monitoring systems (use integration skill)
- Process historical data (use analytics skill)
- Modify hook execution (use hook management skill)
```

### Fix #5: Test With Real Work

**Evaluation-Driven Development:**
1. Identify gaps - Run Claude without skill, document failures
2. Create evaluations - Build 3+ test scenarios
3. Establish baseline - Measure performance without skill
4. Write minimal instructions - Just enough to pass evaluations
5. Iterate - Test, compare, refine

---

## Anti-Patterns to Avoid

| Anti-Pattern | Problem |
|--------------|---------|
| Vague triggers ("Helps with files") | Low activation rate |
| Too many options ("use pypdf, or pdfplumber, or...") | Decision paralysis |
| Deeply nested references | Context loss |
| Time-sensitive info | Stale instructions |
| First/second person descriptions | Discovery problems |

---

## Validation Gate

Before marking a skill complete, verify:

- [ ] SKILL.md under 500 lines
- [ ] Description under 1024 chars with USE WHEN patterns
- [ ] Examples longer than rules
- [ ] 3+ real scenarios tested
- [ ] Out of scope section defined
- [ ] No time-sensitive information

---

## Source

Extracted from: [Claude Code Skills Structure and Usage Guide](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
