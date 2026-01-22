# PAI Observability SpecFirst Playbook

A Maestro Auto Run playbook for developing PAI observability features using the **SpecFirst** methodology.

## What This Playbook Does

Takes a feature requirement, guides it through gated specification phases, and implements using TDD with quality gates at each transition.

### Key Features

- **Gated Workflow**: No code until specs are approved (SPECIFY → PLAN → TASKS → IMPLEMENT)
- **Quality Gates**: 80-100% thresholds before phase transitions
- **Human Checkpoints**: Explicit approval required at each gate
- **TDD Enforcement**: Tests written before implementation
- **State Persistence**: Progress tracked in `.specfirst/state.json`

## Playbook Structure

```
┌─────────────────────────────────────────────────────────────┐
│  SPECIFICATION PHASE (Human approval required)               │
├─────────────────────────────────────────────────────────────┤
│  1_SPECIFY.md      → Define SHALL/MUST requirements          │
│  2_PLAN.md         → Architecture and technical design       │
│  3_TASKS.md        → Break into reviewable units             │
└─────────────────────────────────────────────────────────────┘
                        ↓ [Gate Check: 80%+]
┌─────────────────────────────────────────────────────────────┐
│  IMPLEMENTATION LOOP (repeats until complete)                │
├─────────────────────────────────────────────────────────────┤
│  4_IMPLEMENT.md    → Execute one task (TDD)                  │
│  5_VERIFY_GATE.md  → Test & check quality gate               │
│         ↑                         │                          │
│         └── more tasks ───────────┘                          │
└─────────────────────────────────────────────────────────────┘
                        ↓ (all tasks complete, gates pass)
┌─────────────────────────────────────────────────────────────┐
│  FINALIZATION (runs once)                                    │
├─────────────────────────────────────────────────────────────┤
│  6_FINALIZE.md     → Archive specs, update CHANGELOG, PR     │
└─────────────────────────────────────────────────────────────┘
```

## Prerequisites

- Maestro installed and configured
- PAI repository cloned
- Checked out to a feature branch (NOT `main` or `develop`)
- SpecFirst CLI available (optional but recommended)

## Setup in Maestro

1. Clone the PAI repository
2. Create and checkout a feature branch:
   ```bash
   git checkout -b feature/observability-phase-1
   ```
3. Initialize SpecFirst state (optional):
   ```bash
   specfirst init
   ```
4. Configure Auto Run:
   - **Loop Mode**: ON
   - **Max Loops**: 15
5. Update `1_SPECIFY.md` with your requirements source
6. Run the playbook

## Working Files Generated

| File | Purpose | Phase |
|------|---------|-------|
| `.specfirst/state.json` | Workflow state and progress | All |
| `SPEC.md` | SHALL/MUST requirements | 1 |
| `PLAN.md` | Technical architecture | 2 |
| `TASKS.md` | Dependency-tracked tasks | 3 |
| `TEST_RESULTS.md` | Latest test output | 4-5 |
| `CHANGELOG.md` | Release notes entry | 6 |
| `PR_DESCRIPTION.md` | Ready-to-use PR body | 6 |

## Quality Gates

| Phase | Threshold | Criteria |
|-------|-----------|----------|
| SPECIFY | 100% | All requirements have SHALL/MUST language |
| PLAN | 80% | Architecture covers all requirements |
| TASKS | 80% | Tasks cover all spec items |
| IMPLEMENT | 80% | Tests pass, tasks complete |
| FINALIZE | 100% | All gates pass, changelog updated |

## Input: Requirements Source

Update `1_SPECIFY.md` to point to your requirements:

```markdown
**Requirements Document**: docs/PAI-Observability-Requirements.md
**Scope**: Stage 1 - Local observability stack
```

## Branch Safety

This playbook refuses to run on protected branches (`main`, `develop`, `master`).

## Support

- [Maestro](https://github.com/pedramamini/Maestro)
- [PAI](https://github.com/danielmiessler/PAI)
- [SpecFirst Methodology](../skills/specfirst/SKILL.md)
