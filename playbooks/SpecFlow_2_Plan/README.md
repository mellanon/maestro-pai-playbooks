# SpecFlow 2: Plan

**Phase 2 of the SpecFlow trilogy.** Creates architecture plans and task breakdowns for all specified features, then exits for human review.

## Prerequisites

- Run **SpecFlow_1_Specify** first
- Review and approve all `spec.md` files

## What This Playbook Does

1. Finds all features with specs but no plans (phase = `specify`)
2. Runs `specflow plan` for architecture design
3. Runs `specflow tasks` for implementation breakdown
4. Creates `plan.md` and `tasks.md` for each feature
5. **Exits** when all features have plans

## Artifacts Produced

```
.specflow/specs/
├── f-1-feature-name/
│   ├── spec.md          ← Already exists (from Phase 1)
│   ├── plan.md          ← Review this
│   └── tasks.md         ← Review this
├── f-2-feature-name/
│   ├── spec.md
│   ├── plan.md          ← Review this
│   └── tasks.md         ← Review this
```

## Workflow

```
┌─────────────────────────────────────────────────────────────┐
│  1_SELECT_FEATURE.md                                        │
│  → Find next feature needing plan (phase = specify)         │
│  → Write CURRENT_FEATURE.md                                 │
└─────────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│  2_PLAN.md                                                  │
│  → Run specflow plan for architecture                       │
│  → Create plan.md                                           │
└─────────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│  3_TASKS.md                                                 │
│  → Run specflow tasks for breakdown                         │
│  → Create tasks.md                                          │
│  → Mark feature phase = tasks                               │
└─────────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│  4_NEXT_OR_EXIT.md                                          │
│  → Check for more features needing plans                    │
│  → If yes: loop to 1_SELECT_FEATURE                         │
│  → If no: EXIT playbook                                     │
└─────────────────────────────────────────────────────────────┘
```

## After This Playbook

1. **Review all plan.md files** - architecture decisions
2. **Review all tasks.md files** - implementation breakdown
3. Edit plans/tasks if needed
4. Run **SpecFlow_3_Implement** to build the features

## No Eternal Loops

This playbook exits when:
- All features have plans and tasks (phase >= `tasks`)
- No features with specs remain unplanned
