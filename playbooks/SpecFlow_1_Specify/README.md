# SpecFlow 1: Specify

**Phase 1 of the SpecFlow trilogy.** Creates specifications for all pending features, then exits for human review.

## What This Playbook Does

1. Finds all features needing specification (phase = `none`)
2. Runs `specflow specify` interview for each
3. Creates `spec.md` for each feature
4. **Exits** when all features have specs

## Artifacts Produced

```
.specflow/specs/
├── f-1-feature-name/
│   └── spec.md          ← Review this
├── f-2-feature-name/
│   └── spec.md          ← Review this
└── f-3-feature-name/
    └── spec.md          ← Review this
```

## Workflow

```
┌─────────────────────────────────────────────────────────────┐
│  1_SELECT_FEATURE.md                                        │
│  → Find next feature needing spec (phase = none)            │
│  → Write CURRENT_FEATURE.md                                 │
└─────────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│  2_SPECIFY.md                                               │
│  → Run specflow specify interview                           │
│  → Create spec.md                                           │
│  → Mark feature phase = specify                             │
└─────────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│  3_NEXT_OR_EXIT.md                                          │
│  → Check for more features needing spec                     │
│  → If yes: loop to 1_SELECT_FEATURE                         │
│  → If no: EXIT playbook                                     │
└─────────────────────────────────────────────────────────────┘
```

## After This Playbook

1. **Review all spec.md files** in `.specflow/specs/*/`
2. Edit specs if needed
3. Run **SpecFlow_2_Plan** to create architecture plans

## No Eternal Loops

This playbook exits when:
- All features have specs (phase >= `specify`)
- No features exist

It does NOT loop forever waiting for work.
