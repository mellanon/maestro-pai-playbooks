# SpecFirst Development Playbook

**A Maestro Auto Run playbook for spec-driven development using the SpecFirst skill.**

## What This Playbook Does

Orchestrates the SpecFirst methodology for ANY software project. Takes a requirement, creates specifications, implements with TDD, and prepares for release.

### Key Features

- **Spec-Driven**: Requirements become specifications before code
- **TDD Enforced**: Tests written before implementation
- **Quality Gates**: Progress validated at each phase
- **Loop Control**: Document 3 drives the implementation loop
- **Human Gates**: Spec approval and release approval require human sign-off

## Playbook Structure

This playbook uses a **hybrid pattern** with a fixed specification phase and iterative implementation:

```
┌─────────────────────────────────────────────────────────────┐
│  SPECIFICATION PHASE (runs once, persists via files)        │
├─────────────────────────────────────────────────────────────┤
│  1_SPECIFY.md        → Create change proposal with specs    │
│                        → Invokes /openspec-proposal         │
└─────────────────────────────────────────────────────────────┘
                        ↓ [Human Review]
┌─────────────────────────────────────────────────────────────┐
│  IMPLEMENTATION LOOP (repeats until all tasks complete)     │
├─────────────────────────────────────────────────────────────┤
│  2_IMPLEMENT.md      → Execute one task with TDD            │
│                        → Invokes /openspec-apply            │
│  3_VERIFY.md         → Test & check progress (LOOP GATE)    │
│         ↑                       │                           │
│         └── more tasks ─────────┘                           │
└─────────────────────────────────────────────────────────────┘
                        ↓ (all tasks complete)
┌─────────────────────────────────────────────────────────────┐
│  FINALIZATION PHASE (runs once)                             │
├─────────────────────────────────────────────────────────────┤
│  4_ARCHIVE.md        → Merge specs, update CHANGELOG        │
│                        → Invokes /openspec-archive          │
│  5_RELEASE.md        → File inventory, prepare PR           │
│                        → Invokes /specfirst-release         │
└─────────────────────────────────────────────────────────────┘
```

## Prerequisites

- Maestro installed and configured
- PAI installation with SpecFirst skill available
- Project initialized with `specfirst init`
- **Recommended**: Feature branch (not `main`)

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
| `openspec/changes/[name]/proposal.md` | Change summary and rationale | Yes |
| `openspec/changes/[name]/specs/*.md` | Requirements (SHALL/MUST) | Yes |
| `openspec/changes/[name]/tasks.md` | Task breakdown with status | Yes (updated) |
| `LOOP_{{N}}_REQUIREMENTS.md` | Initial requirements capture | Yes |
| `LOOP_{{N}}_PROGRESS.md` | Implementation progress per loop | Yes |
| `LOOP_{{N}}_TEST_RESULTS.md` | Test execution results | Updated each loop |
| `RELEASE_INVENTORY.md` | Files for release | Final |
| `PR_DESCRIPTION.md` | Ready-to-use PR description | Final |

## SpecFirst Commands Used

| Step | Command | Purpose |
|------|---------|---------|
| SPECIFY | `/openspec-proposal` | Creates specs folder with requirements |
| IMPLEMENT | `/openspec-apply` | Implements one task with TDD |
| VERIFY | `specfirst gate` | Checks quality thresholds |
| ARCHIVE | `/openspec-archive` | Merges specs, updates CHANGELOG |
| RELEASE | `/specfirst-release` | Creates file inventory, prepares PR |

## Loop Behavior

The playbook loops through documents 2-3 until:
- All tasks in `tasks.md` are marked `COMPLETE`
- Or max loops reached
- Or a task fails repeatedly

Each loop iteration:
1. Finds the next `PENDING` task with satisfied dependencies
2. Writes tests (RED)
3. Implements code (GREEN)
4. Runs all tests
5. Updates task status to `COMPLETE` or keeps `PENDING`

## Exit Conditions

- **Success**: All tasks complete, tests pass → proceeds to Document 4
- **Max Loops**: Reached loop limit → exits with partial progress
- **Gate Failure**: Quality gate < 80% → stops for review
- **Blocked**: Task has unsatisfied dependencies → waits or halts

## Foundation Documents

This playbook validates work against these principles:

| Document | Purpose | Used In |
|----------|---------|---------|
| `docs/PAI-PRINCIPLES.md` | Founding design principles | Steps 1, 4 |
| `docs/SKILL-BEST-PRACTICES.md` | Skill structure guidelines | Step 4 |
| `docs/TDD-EVALS.md` | Test patterns and eval criteria | Steps 2, 3 |
| `docs/RELEASE-FRAMEWORK.md` | Release discipline | Step 5 |

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

## Support

- [PAI](https://github.com/danielmiessler/PAI) - Personal AI Infrastructure
- [Maestro](https://runmaestro.ai/) - Multi-agent orchestration
