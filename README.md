# Maestro PAI Playbooks

Maestro Auto Run playbooks for **spec-driven development** using the **SpecFlow Skill**.

## Philosophy

**"If you can't specify it, you can't test it. If you can't test it, you can't trust it."**

These playbooks orchestrate spec-driven development with human approval gates. By imposing structure, validation gates, and human approval points, we transform AI from a code generator into an engineering partner.

## The SpecFlow Skill

The playbooks use the **SpecFlow Skill** from [jcfischer/specflow-bundle](https://github.com/jcfischer/specflow-bundle) which provides:

### Four-Phase Workflow
```
SPECIFY → PLAN → TASKS → IMPLEMENT
 (What)   (How)  (Work)   (Code)
    ↓        ↓       ↓        ↓
 spec.md  plan.md tasks.md  src/
```

### Key Capabilities

| Capability | Description |
|------------|-------------|
| **Gated Phases** | Cannot advance until current phase validates (≥80%) |
| **Interview-Driven Specs** | 8-phase structured requirements elicitation |
| **Quality Evals** | `spec-quality` and `plan-quality` rubrics |
| **TDD Enforcement** | RED → GREEN → BLUE for every task |
| **Doctorow Gate** | Post-implementation verification |
| **Constitutional Compliance** | CLI-first, library-first, test-first, deterministic |

### Workflows

| Workflow | Purpose |
|----------|---------|
| `sdd-workflow.md` | Complete Spec-Driven Development process |
| `specify-with-interview.md` | 8-phase interview protocol for requirements |

**Full skill documentation**: [SpecFlow SKILL.md](https://github.com/jcfischer/specflow-bundle/blob/main/packages/specflow/SKILL.md)

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
4. The SpecFlow Skill auto-loads and orchestrates the workflow

## How It Works

The playbook is **generic** - works for any project:
- PAI skills
- CLI tools
- Web applications
- Libraries
- Any codebase

The SpecFlow Skill **auto-loads** when:
- Project has `.specify/` or `.specflow/` directory
- User mentions "F-1", "F-2", "F-XXX" pattern
- User says "spec", "specify", "specflow", "new feature"

## Dependencies

| Dependency | Purpose | URL |
|------------|---------|-----|
| **SpecFlow Bundle** | Spec-driven development skill + CLI | [jcfischer/specflow-bundle](https://github.com/jcfischer/specflow-bundle) |
| **Maestro** | Multi-agent orchestration | [runmaestro.ai](https://runmaestro.ai/) |

## Attribution

### SpecFlow Bundle
The core spec-driven development skill by [J.C. Fischer](https://github.com/jcfischer):
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
