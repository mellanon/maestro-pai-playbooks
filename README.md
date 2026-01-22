# Maestro PAI Playbooks

Maestro Auto Run playbooks for **spec-driven development** using the **SpecFirst** methodology.

## Philosophy

**"If you can't specify it, you can't test it. If you can't test it, you can't trust it."**

These playbooks orchestrate the SpecFirst skill to enforce gated, spec-driven development. By imposing structure, validation gates, and human approval points, we transform AI from a code generator into an engineering partner.

## The SpecFirst Flow

```
┌─────────────────────────────────────────────────────────┐
│  1. SPECIFY                                             │
│  Create change proposal with specs                      │
│  → /openspec-proposal                                   │
└─────────────────────────────────────────────────────────┘
                        ↓ [Human Review]
┌─────────────────────────────────────────────────────────┐
│  2. IMPLEMENT (Loop)                                    │
│  TDD: Write tests, implement, verify                    │
│  → /openspec-apply                                      │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│  3. VERIFY (Loop Control)                               │
│  Quality gates and test validation                      │
│  → specfirst gate                                       │
└─────────────────────────────────────────────────────────┘
                        ↓ (loop until complete)
┌─────────────────────────────────────────────────────────┐
│  4. ARCHIVE                                             │
│  Merge specs, update CHANGELOG                          │
│  → /openspec-archive                                    │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│  5. RELEASE                                             │
│  File inventory, validation, PR                         │
│  → /specfirst-release                                   │
└─────────────────────────────────────────────────────────┘
```

## Playbooks

| Playbook | Purpose |
|----------|---------|
| `SpecFirst_Development` | Generic spec-driven development for ANY project |

## Foundation Documents

| Document | Purpose |
|----------|---------|
| `docs/PAI-PRINCIPLES.md` | Founding design principles (constitutional gates) |
| `docs/SKILL-BEST-PRACTICES.md` | Skill activation and structure guidelines |
| `docs/TDD-EVALS.md` | Test-driven development with eval criteria |
| `docs/RELEASE-FRAMEWORK.md` | Release discipline and checklists |

## Skills

| Skill | Purpose |
|-------|---------|
| `specfirst` | SpecFirst methodology, slash commands, state management |

## Quick Start

1. Clone this repo
2. Copy the playbook to your Maestro Auto Run directory
3. Run with Maestro - provide your feature requirements
4. Playbook orchestrates SpecFirst commands

```bash
# Clone
git clone https://github.com/mellanon/maestro-pai-playbooks
cd maestro-pai-playbooks

# Start Maestro with the playbook
# Configure Auto Run to point to playbooks/SpecFirst_Development
```

## How It Works

The playbook is **generic** - it works for any project:
- PAI skills
- CLI tools
- Web applications
- Libraries
- Any codebase

Each step invokes SpecFirst skill commands:

| Step | Command | What It Does |
|------|---------|--------------|
| SPECIFY | `/openspec-proposal` | Creates specs folder with requirements |
| IMPLEMENT | `/openspec-apply` | Implements one task with TDD |
| VERIFY | `specfirst gate` | Checks quality thresholds |
| ARCHIVE | `/openspec-archive` | Merges specs, updates CHANGELOG |
| RELEASE | `/specfirst-release` | Creates file inventory, prepares PR |

## Requirements

- [Maestro](https://runmaestro.ai/) - Multi-agent orchestration
- [PAI](https://github.com/danielmiessler/PAI) - Personal AI Infrastructure (for SpecFirst skill)
- Git for version control

## Related Projects

- [danielmiessler/PAI](https://github.com/danielmiessler/PAI) - Personal AI Infrastructure
- [Maestro-Playbooks](https://github.com/maestro-org/Maestro-Playbooks) - Official Maestro playbooks

## License

MIT
