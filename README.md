# Maestro PAI Playbooks

Maestro Auto Run playbooks for developing PAI features using the **SpecFirst** methodology.

## Philosophy

**"If you can't specify it, you can't test it. If you can't test it, you can't trust it."**

These playbooks enforce a gated workflow that prevents AI agents from generating code without architectural constraints. By imposing structure, validation gates, and human approval points, we transform AI from a code generator into an engineering partner.

## The SpecFirst Flow

```
┌─────────────────────────────────────────────────────────┐
│  SPECIFY (Gate: 100%)                                    │
│  Define requirements in SHALL/MUST language              │
│  → Creates spec.md with testable requirements            │
└─────────────────────────────────────────────────────────┘
                        ↓ [Human Review]
┌─────────────────────────────────────────────────────────┐
│  PLAN (Gate: 80%)                                        │
│  Design architecture, data models, interfaces            │
│  → Creates plan.md with technical decisions              │
└─────────────────────────────────────────────────────────┘
                        ↓ [Human Review]
┌─────────────────────────────────────────────────────────┐
│  TASKS (Gate: 80%)                                       │
│  Break work into reviewable units with dependencies      │
│  → Creates tasks.md with dependency graph                │
└─────────────────────────────────────────────────────────┘
                        ↓ [Human Review]
┌─────────────────────────────────────────────────────────┐
│  IMPLEMENT (Loop until complete)                         │
│  TDD: Write tests, implement, verify                     │
│  → Produces working, tested code                         │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│  FINALIZE                                                │
│  Archive specs, update changelog, prepare release        │
│  → Ready for PR                                          │
└─────────────────────────────────────────────────────────┘
```

## Playbooks

| Playbook | Purpose |
|----------|---------|
| `PAI_Observability_Spec` | Develop PAI observability features using SpecFirst |

## Skills

| Skill | Purpose |
|-------|---------|
| `specfirst` | SpecFirst methodology, quality gates, state management |

## Quick Start

1. Clone this repo
2. Copy the playbook to your Maestro Auto Run directory
3. Configure the playbook with your feature requirements
4. Run with Maestro

```bash
# Clone
git clone https://github.com/mellanon/maestro-pai-playbooks
cd maestro-pai-playbooks

# Start Maestro with a playbook
# (Configure Auto Run to point to playbooks/PAI_Observability_Spec)
```

## Requirements

- [Maestro](https://runmaestro.ai/) - Multi-agent orchestration
- [PAI](https://github.com/danielmiessler/PAI) - Personal AI Infrastructure
- Git for version control

## Related Projects

- [ksylvan/maestro-playbooks-custom](https://github.com/ksylvan/maestro-playbooks-custom) - Custom Maestro playbooks
- [jcfischer/specflow-bundle](https://github.com/jcfischer/specflow-bundle) - SpecFlow methodology
- [danielmiessler/PAI](https://github.com/danielmiessler/PAI) - Personal AI Infrastructure

## License

MIT
