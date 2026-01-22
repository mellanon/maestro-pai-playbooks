# SpecFlow Development Playbook

**A Maestro Auto Run playbook for spec-driven development using SpecFlow Bundle.**

## What This Playbook Does

Orchestrates the SpecFlow methodology for ANY software project. Takes a requirement, decomposes into features, and implements with TDD through four sequential phases with human approval gates.

### Key Features

- **Spec-Driven**: Requirements become specifications before code
- **Feature-Centric**: Work organized around discrete features with SQLite tracking
- **Four-Phase Workflow**: SPECIFY → PLAN → TASKS → IMPLEMENT
- **Quality Gates**: Each phase requires human approval before advancing
- **Progress Dashboard**: Web UI for tracking all features
- **Human Gates**: No code written until spec, plan, and tasks are approved

## The Four-Phase Workflow

```
┌─────────────────────────────────────────────────────────────┐
│  PHASE 1: SPECIFY                                           │
├─────────────────────────────────────────────────────────────┤
│  1_SPECIFY.md        → Interview-driven requirements         │
│                        → specflow specify <feature>          │
│                        → Human approves spec                 │
└─────────────────────────────────────────────────────────────┘
                        ↓ [Human Review]
┌─────────────────────────────────────────────────────────────┐
│  PHASE 2: PLAN                                              │
├─────────────────────────────────────────────────────────────┤
│  2_PLAN.md           → Architecture, data models, failure   │
│                        → specflow plan <feature>             │
│                        → Human approves design               │
└─────────────────────────────────────────────────────────────┘
                        ↓ [Human Review]
┌─────────────────────────────────────────────────────────────┐
│  PHASE 3: TASKS                                             │
├─────────────────────────────────────────────────────────────┤
│  3_TASKS.md          → Break into reviewable units          │
│                        → specflow tasks <feature>            │
│                        → Human approves breakdown            │
└─────────────────────────────────────────────────────────────┘
                        ↓ [Human Review]
┌─────────────────────────────────────────────────────────────┐
│  PHASE 4: IMPLEMENT (Loop)                                  │
├─────────────────────────────────────────────────────────────┤
│  4_IMPLEMENT.md      → TDD execution with verification      │
│                        → specflow implement                  │
│  5_VERIFY.md         → Test & check progress (LOOP GATE)    │
│         ↑                       │                           │
│         └── more tasks ─────────┘                           │
└─────────────────────────────────────────────────────────────┘
                        ↓ (all tasks complete)
┌─────────────────────────────────────────────────────────────┐
│  COMPLETE                                                   │
├─────────────────────────────────────────────────────────────┤
│  6_COMPLETE.md       → Validation and marking complete      │
│                        → specflow complete <feature>         │
└─────────────────────────────────────────────────────────────┘
```

## Prerequisites

- Maestro installed and configured
- **SpecFlow Bundle** installed:
  ```bash
  git clone --recursive https://github.com/jcfischer/specflow-bundle.git
  cd specflow-bundle && bun run install.ts
  ```
- **Recommended**: Feature branch (not `main`)

## Input: Requirements Specification

The playbook takes a **requirements specification** as input. Place your spec in `docs/`:

```
docs/
├── [Your-Requirements].md    ← INPUT: What to build
├── PAI-PRINCIPLES.md         ← GATE: Design validation
├── TDD-EVALS.md              ← GATE: Test quality criteria
├── SKILL-BEST-PRACTICES.md   ← GATE: Structure validation
└── RELEASE-FRAMEWORK.md      ← GATE: Release checklist
```

**Initialize from spec:**
```bash
specflow init --from-spec docs/Your-Requirements.md
```

This decomposes the requirements into features in the queue.

## Setup in Maestro

1. Clone/checkout the project repository
2. Create a feature branch:
   ```bash
   git checkout -b feature/your-feature-name
   ```
3. Place this playbook folder in your Maestro Auto Run directory
4. Provide requirements as input (issue link, description, etc.)
5. Configure Auto Run settings:
   - **Loop Mode**: ON (for implementation iteration)
   - **Max Loops**: 20 (reasonable limit for complex features)
6. Run the playbook

## Working Files Generated

| File | Purpose | Persists Across Loops |
|------|---------|----------------------|
| `.specflow/specflow.db` | SQLite database tracking all features | Yes |
| `specflow/[feature]/spec.md` | Detailed specification | Yes |
| `specflow/[feature]/plan.md` | Technical architecture plan | Yes |
| `specflow/[feature]/tasks.md` | Implementation task breakdown | Yes (updated) |
| `LOOP_{{N}}_PROGRESS.md` | Implementation progress per loop | Yes |
| `LOOP_{{N}}_TEST_RESULTS.md` | Test execution results | Updated each loop |

## SpecFlow Commands Used

| Phase | Command | Purpose |
|-------|---------|---------|
| INIT | `specflow init [description]` | Initialize project with feature decomposition |
| ADD | `specflow add <name> <description>` | Add a feature to the queue |
| STATUS | `specflow status` | Show feature queue and progress |
| SPECIFY | `specflow specify <feature-id>` | Create detailed specification |
| PLAN | `specflow plan <feature-id>` | Create technical architecture plan |
| TASKS | `specflow tasks <feature-id>` | Break into implementation tasks |
| IMPLEMENT | `specflow implement` | Generate implementation prompt |
| VALIDATE | `specflow validate <feature-id>` | Validate all phases complete |
| COMPLETE | `specflow complete <feature-id>` | Mark feature as complete |
| UI | `specflow ui` | Start web dashboard |

## Loop Behavior

The playbook loops through the IMPLEMENT phase until:
- All tasks in `tasks.md` are marked complete
- Or max loops reached
- Or a task fails repeatedly

Each loop iteration:
1. Runs `specflow implement` to get next implementation context
2. Writes tests (RED)
3. Implements code (GREEN)
4. Runs all tests
5. Updates task status

## Exit Conditions

- **Success**: All tasks complete, tests pass → runs `specflow complete <feature>`
- **Max Loops**: Reached loop limit → exits with partial progress
- **Validation Failure**: `specflow validate` fails → stops for review
- **Blocked**: Task has unsatisfied dependencies → waits or halts

## Constitutional Gates (Reference Documents)

These documents act as **validation gates** - each step must pass validation against the relevant principles before proceeding.

| Document | Purpose | Validates | Key Checks |
|----------|---------|-----------|------------|
| `docs/PAI-PRINCIPLES.md` | Founding design principles | Steps 1-3 | CLI-first, deterministic, UNIX philosophy |
| `docs/SKILL-BEST-PRACTICES.md` | Skill structure guidelines | Step 2 | Under 500 lines, USE WHEN triggers |
| `docs/TDD-EVALS.md` | Test patterns and eval criteria | Steps 4-5 | Deterministic tests, pass@k metrics |
| `docs/RELEASE-FRAMEWORK.md` | Release discipline | Step 6 | File inventory, sanitization, no secrets |

**How gates work:**
- Each playbook step includes a validation checklist
- AI agent checks work against the relevant gate document
- Human reviews gate compliance before approving phase

## Customization

The playbook is generic - provide requirements via:

1. **Input text**: Describe the feature/fix
2. **Issue link**: GitHub/Jira issue URL
3. **Spec document**: Existing requirements markdown

The SpecFirst skill handles all project types:
- PAI skills
- CLI tools
- Web applications
- Libraries
- Any codebase

## Advanced: Parallel Development with Git Worktrees

For teams or power users running multiple agents concurrently, use **git worktrees** to enable safe parallel execution.

### Setup

```bash
# Create worktrees for parallel agents
git worktree add ../project-agent-1 -b feature/agent-1
git worktree add ../project-agent-2 -b feature/agent-2

# Each agent works in its own directory
# Agent 1 → ~/src/worktrees/project/agent-1
# Agent 2 → ~/src/worktrees/project/agent-2
```

### Workflow

1. **Launch Maestro** with agents assigned to separate worktree directories
2. **Load playbook** and provide requirements to each agent
3. **AutoRun** - agents work concurrently and safely (isolated directories)
4. **Compare outputs** - use a "Best PR" playbook to compare agent results
5. **Merge** the winning approach, incorporate gems from runner-up

### Benefits

- **Safe parallelism** - No conflicts between agents
- **Faster iteration** - Multiple approaches explored simultaneously
- **Quality through competition** - Compare agent outputs for best solution

### Model Mix (from Kayvan's workflow)

| Model | Usage |
|-------|-------|
| Claude Opus 4.5 | 80% - Primary development |
| Codex 5.2 | 15% - Code generation |
| Experimental | 5% - Testing new models |

> "I am literally like a small programming team by myself." - Kayvan

### Resources

- [ksylvan/maestro-playbooks-custom](https://github.com/ksylvan/maestro-playbooks-custom) - Custom playbooks including "Fabric Issue Solver"
- [ksylvan/LaunchMaestroDev](https://github.com/ksylvan/LaunchMaestroDev) - Development launcher

---

## Resources

- [SpecFlow Bundle](https://github.com/jcfischer/specflow-bundle) - Complete spec-driven development system
- [PAI](https://github.com/danielmiessler/PAI) - Personal AI Infrastructure
- [Maestro](https://runmaestro.ai/) - Multi-agent orchestration

## Philosophy

> **"The AI is the same. The architecture around it isn't."**

The breakthrough isn't a better model — it's better scaffolding. When you give AI agents structure, constraints, and verification gates, they stop hallucinating and start engineering.

> **"Code is a liability, not an asset."** — [Cory Doctorow](https://pluralistic.net/2026/01/06/1000x-liability/)

Every line requires maintenance. Every feature adds complexity. SpecFlow embraces this truth by forcing engineering discipline before code generation.
