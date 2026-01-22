# Maestro PAI Playbooks

Maestro Auto Run playbooks for **spec-driven development** using **SpecFlow Bundle**.

## Philosophy

**"If you can't specify it, you can't test it. If you can't test it, you can't trust it."**

These playbooks orchestrate spec-driven development with human approval gates. By imposing structure, validation gates, and human approval points, we transform AI from a code generator into an engineering partner.

## The SpecFlow Workflow

```
┌─────────────────────────────────────────────────────────────┐
│  PHASE 1: SPECIFY                                           │
│  → specflow specify <feature>                               │
│  → Human approves spec                                      │
└─────────────────────────────────────────────────────────────┘
                        ↓ [Human Review]
┌─────────────────────────────────────────────────────────────┐
│  PHASE 2: PLAN                                              │
│  → specflow plan <feature>                                  │
│  → Human approves design                                    │
└─────────────────────────────────────────────────────────────┘
                        ↓ [Human Review]
┌─────────────────────────────────────────────────────────────┐
│  PHASE 3: TASKS                                             │
│  → specflow tasks <feature>                                 │
│  → Human approves breakdown                                 │
└─────────────────────────────────────────────────────────────┘
                        ↓ [Human Review]
┌─────────────────────────────────────────────────────────────┐
│  PHASE 4: IMPLEMENT (Loop)                                  │
│  → specflow implement                                       │
│  → TDD: Write tests, implement, verify                      │
└─────────────────────────────────────────────────────────────┘
                        ↓ (all tasks complete)
┌─────────────────────────────────────────────────────────────┐
│  COMPLETE                                                   │
│  → specflow complete <feature>                              │
└─────────────────────────────────────────────────────────────┘
```

## Playbooks

| Playbook | Purpose | Type |
|----------|---------|------|
| `SpecFlow_Development` | Generic spec-driven development for ANY project | Loop-based |
| `PR_Review` | Comprehensive pull request review | Linear |

## Foundation Documents (Constitutional Gates)

| Document | Purpose |
|----------|---------|
| `docs/PAI-PRINCIPLES.md` | Founding design principles |
| `docs/SKILL-BEST-PRACTICES.md` | Skill activation and structure guidelines |
| `docs/TDD-EVALS.md` | Test-driven development with eval criteria |
| `docs/RELEASE-FRAMEWORK.md` | Release discipline and checklists |

## Quick Start

### 1. Install SpecFlow Bundle

```bash
git clone --recursive https://github.com/jcfischer/specflow-bundle.git
cd specflow-bundle && bun run install.ts
```

### 2. Clone This Repo

```bash
git clone https://github.com/mellanon/maestro-pai-playbooks
```

### 3. Run with Maestro

1. Copy the playbook to your Maestro Auto Run directory
2. Configure Auto Run to point to `playbooks/SpecFlow_Development`
3. Provide your feature requirements
4. Playbook orchestrates SpecFlow commands

## How It Works

The playbook is **generic** - works for any project:
- PAI skills
- CLI tools
- Web applications
- Libraries
- Any codebase

Each step invokes SpecFlow Bundle commands:

| Phase | Command | What It Does |
|-------|---------|--------------|
| SPECIFY | `specflow specify <feature>` | Creates detailed specification |
| PLAN | `specflow plan <feature>` | Creates technical architecture |
| TASKS | `specflow tasks <feature>` | Breaks into implementation units |
| IMPLEMENT | `specflow implement` | TDD execution with verification |
| COMPLETE | `specflow complete <feature>` | Validates and marks done |

## Dependencies

| Dependency | Purpose | URL |
|------------|---------|-----|
| **SpecFlow Bundle** | CLI-first spec-driven development | [jcfischer/specflow-bundle](https://github.com/jcfischer/specflow-bundle) |
| **Maestro** | Multi-agent orchestration | [runmaestro.ai](https://runmaestro.ai/) |

## Attribution

### SpecFlow Bundle
The core spec-driven development tooling by [J.C. Fischer](https://github.com/jcfischer):
- [jcfischer/specflow-bundle](https://github.com/jcfischer/specflow-bundle)

### Maestro Playbooks
Playbook design patterns from [Kayvan Sylvan](https://github.com/ksylvan):

| Resource | URL |
|----------|-----|
| **Maestro** | https://runmaestro.ai/ |
| **Kayvan's Maestro fork** | [ksylvan/Maestro](https://github.com/ksylvan/Maestro) |
| **LaunchMaestroDev** | [ksylvan/LaunchMaestroDev](https://github.com/ksylvan/LaunchMaestroDev) |
| **Custom playbooks** | [ksylvan/maestro-playbooks-custom](https://github.com/ksylvan/maestro-playbooks-custom) |

### PAI (Personal AI Infrastructure)
Constitutional gates and founding principles:
- [danielmiessler/PAI](https://github.com/danielmiessler/PAI)

## License

MIT
