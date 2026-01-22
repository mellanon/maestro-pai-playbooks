# PAI Founding Principles

**Purpose**: Constitutional gates that govern all PAI development. Specs and implementations MUST align with these principles.

---

## The 16 Principles

| # | Principle | Summary | Gate Application |
|---|-----------|---------|------------------|
| 1 | **User Centricity** | PAI is built around you, not tooling. Your goals, preferences, and context come first. | SPECIFY: Requirements must serve user goals, not technical convenience |
| 2 | **The Foundational Algorithm** | Observe → Think → Plan → Build → Execute → Verify → Learn. Define ideal state, iterate until reached. | ALL: Every phase follows this loop |
| 3 | **Clear Thinking First** | Good prompts come from clear thinking. Clarify the problem before writing the prompt. | SPECIFY: Requirements must be unambiguous |
| 4 | **Scaffolding > Model** | System architecture matters more than which model you use. | PLAN: Architecture decisions trump model selection |
| 5 | **Deterministic Infrastructure** | AI is probabilistic; your infrastructure shouldn't be. Use templates and patterns. | IMPLEMENT: Code must produce consistent results |
| 6 | **Code Before Prompts** | If you can solve it with a bash script, don't use AI. | IMPLEMENT: Prefer deterministic code over LLM calls |
| 7 | **Spec / Test / Evals First** | Write specifications and tests before building. Measure if the system works. | TASKS: Tests written before implementation |
| 8 | **UNIX Philosophy** | Do one thing well. Make tools composable. Use text interfaces. | PLAN: Components must be modular and composable |
| 9 | **ENG / SRE Principles** | Treat AI infrastructure like production software: version control, automation, monitoring. | FINALIZE: Observability, logging, error handling required |
| 10 | **CLI as Interface** | Command-line interfaces are faster, more scriptable, and more reliable than GUIs. | IMPLEMENT: CLIs over GUIs where applicable |
| 11 | **Goal → Code → CLI → Prompts → Agents** | The decision hierarchy: clarify goal, then code, then CLI, then prompts, then agents. | ALL: Follow this hierarchy |
| 12 | **Skill Management** | Modular capabilities that route intelligently based on context. | PLAN: New functionality should be skill-based |
| 13 | **Memory System** | Everything worth knowing gets captured. History feeds future context. | IMPLEMENT: Events must be capturable |
| 14 | **Agent Personalities** | Different work needs different approaches. Specialized agents with unique voices. | N/A for observability |
| 15 | **Science as Meta-Loop** | Hypothesis → Experiment → Measure → Iterate. | VERIFY: Test results inform next iteration |
| 16 | **Permission to Fail** | Explicit permission to say "I don't know" prevents hallucinations. | ALL: Graceful error handling required |

---

## Principle Validation Gates

### SPECIFY Phase Gate
- [ ] Requirements serve **user goals** (Principle 1)
- [ ] Each requirement is **unambiguous** (Principle 3)
- [ ] Uses **SHALL/MUST** language (Principle 7)

### PLAN Phase Gate
- [ ] Architecture is **modular and composable** (Principles 8, 12)
- [ ] **CLI interfaces** where applicable (Principle 10)
- [ ] **Deterministic code** preferred over LLM calls (Principles 5, 6)
- [ ] Follows **goal → code → CLI → prompts → agents** hierarchy (Principle 11)

### IMPLEMENT Phase Gate
- [ ] **Tests written first** (Principle 7)
- [ ] Code produces **consistent, deterministic results** (Principle 5)
- [ ] **Version controlled** with proper commits (Principle 9)
- [ ] Events are **capturable** by memory system (Principle 13)
- [ ] **Graceful error handling** implemented (Principle 16)

### FINALIZE Phase Gate
- [ ] **Observability** in place (Principle 9)
- [ ] **Documentation** complete (Principle 3)
- [ ] Follows **release framework** (Principle 9)

---

## Anti-Patterns to Reject

These violate PAI principles and should fail validation:

| Anti-Pattern | Violation | Principle |
|--------------|-----------|-----------|
| Building for AI, not user | User Centricity | 1 |
| Unclear requirements | Clear Thinking First | 3 |
| Monolithic components | UNIX Philosophy | 8 |
| No tests before code | Spec/Test/Evals First | 7 |
| Using LLM where script works | Code Before Prompts | 6 |
| No error handling | Permission to Fail | 16 |
| Non-deterministic outputs | Deterministic Infrastructure | 5 |
| GUI-only interfaces | CLI as Interface | 10 |

---

## Source

Extracted from: [PAI Founding Principles and Architecture](https://github.com/danielmiessler/PAI)

These principles are **constitutional**—they cannot be overridden by spec requirements.
