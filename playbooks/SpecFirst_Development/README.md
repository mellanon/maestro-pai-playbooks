# SpecFirst Development Playbook

**Orchestrates spec-driven development using the SpecFirst skill.**

## Purpose

This playbook guides Maestro through spec-driven development for ANY project. It leverages the SpecFirst skill's slash commands to maintain discipline while being flexible on implementation.

## Flow

```
1_SPECIFY     → Create change proposal (specs, requirements)
      ↓
2_IMPLEMENT   → TDD implementation via /openspec-apply
      ↓
3_VERIFY      → Quality gates and test validation (LOOPS)
      ↓
4_ARCHIVE     → Merge specs, update CHANGELOG
      ↓
5_RELEASE     → File inventory, validation, PR
```

## Prerequisites

- **PAI installation** with SpecFirst skill available
- **Project initialized** with `specfirst init` or existing structure

## Loop Control

- **Step 3 (VERIFY)** has `Reset: ON` - drives the implementation loop
- When tasks remain, Step 3 resets Steps 2-3 for next iteration
- When all tasks complete and gates pass, continues to Step 4

## Foundation Documents

The playbook validates work against these principles:

| Document | Purpose |
|----------|---------|
| `docs/PAI-PRINCIPLES.md` | Founding design principles |
| `docs/SKILL-BEST-PRACTICES.md` | Skill activation and structure |
| `docs/TDD-EVALS.md` | Test-driven development with eval criteria |
| `docs/RELEASE-FRAMEWORK.md` | Release discipline and checklists |

## How to Use

1. **User provides a feature request or requirement**
2. **Maestro runs playbook steps sequentially**
3. **Steps 2-3 loop** until all tasks complete with passing tests
4. **Human reviews** at key gates (spec approval, release approval)
5. **Playbook completes** with PR-ready code

## SpecFirst Commands Used

| Step | Command | Purpose |
|------|---------|---------|
| 1 | `/openspec-proposal` | Create specs folder with requirements |
| 2 | `/openspec-apply` | Implement one task with TDD |
| 3 | `specfirst gate` | Check quality thresholds |
| 4 | `/openspec-archive` | Merge specs, update CHANGELOG |
| 5 | `/specfirst-release` | File inventory and release prep |
