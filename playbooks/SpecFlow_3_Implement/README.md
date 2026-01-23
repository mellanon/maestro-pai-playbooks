# SpecFlow 3: Implement

**Phase 3 of the SpecFlow trilogy.** Implements all features using TDD, looping through tasks until complete.

## Prerequisites

- Run **SpecFlow_1_Specify** first (creates specs)
- Run **SpecFlow_2_Plan** second (creates plans and tasks)
- Review and approve all specs, plans, and task breakdowns

## What This Playbook Does

1. Finds all features ready for implementation (phase = `tasks`)
2. For each feature, loops through TDD cycle:
   - Write failing test (RED)
   - Implement code (GREEN)
   - Refactor (BLUE)
   - Run all tests
3. Marks features complete when all tasks done
4. **Exits** when all features are implemented

## Artifacts Produced

```
# In your project (not .specflow):
src/
├── feature-1-code.ts    ← Implementation
├── feature-1.test.ts    ← Tests
├── feature-2-code.ts
└── feature-2.test.ts

.specflow/specs/
├── f-1-feature-name/
│   ├── spec.md
│   ├── plan.md
│   ├── tasks.md         ← Tasks marked complete
│   └── docs.md          ← Auto-generated docs
```

## Workflow

```
┌─────────────────────────────────────────────────────────────┐
│  1_SELECT_FEATURE.md                                        │
│  → Find next feature ready to implement (phase = tasks)     │
│  → Write CURRENT_FEATURE.md                                 │
└─────────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│  2_IMPLEMENT.md                                             │
│  → Get next incomplete task                                 │
│  → TDD cycle: RED → GREEN → BLUE                            │
│  → Mark task complete                                       │
└─────────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│  3_VERIFY.md                                                │
│  → Run all tests                                            │
│  → Check task completion                                    │
│  → If tasks remain: loop to 2_IMPLEMENT                     │
│  → If feature done: proceed to 4_COMPLETE                   │
└─────────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│  4_COMPLETE.md                                              │
│  → Run specflow complete                                    │
│  → Generate docs.md                                         │
│  → Mark feature phase = complete                            │
└─────────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│  5_NEXT_OR_EXIT.md                                          │
│  → Check for more features to implement                     │
│  → If yes: loop to 1_SELECT_FEATURE                         │
│  → If no: EXIT playbook                                     │
└─────────────────────────────────────────────────────────────┘
```

## After This Playbook

1. **Review implemented code** in your project
2. **Run full test suite** to verify
3. **Create PR** for review
4. Optionally run **PR_Review playbook** for automated review

## Loop Behavior

This playbook has TWO loop levels:

1. **Task Loop** (Steps 2-3): Repeats until all tasks in a feature are done
2. **Feature Loop** (Steps 1-5): Repeats until all features are implemented

Both loops have clear exit conditions - no eternal looping.
