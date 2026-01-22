# Step 1: SPECIFY

**Phase**: Specification | **Gate**: 100% | **Human Approval**: Required

**SpecFirst Skill**: `/openspec-proposal`

---

## Objective

Create an OpenSpec-compliant change proposal for PAI Observability.

## Input

**Requirements Source**: `docs/PAI-Observability-Requirements.md`

**Scope**: [DEFINE SCOPE - e.g., "Stage 1: Local observability stack"]

## Execute

### 1. Invoke SpecFirst Proposal

```
/openspec-proposal
> What feature or change? pai-observability-stage1
> Which skill/project? PAI
```

The SpecFirst skill handles:
- Creating `openspec/changes/` structure
- Generating `proposal.md` template
- Creating `specs/` folder with requirement format
- Validating SHALL/MUST language
- Quality gate checking (100% threshold)

### 2. Provide Context

When prompted, reference the requirements document:

```
Requirements are in: docs/PAI-Observability-Requirements.md
Focus on: [Your defined scope]
```

### 3. Human Review

SpecFirst will pause for human approval before proceeding.

## Output

- OpenSpec proposal structure (created by SpecFirst skill)
- `.specfirst/state.json` updated

## Next

After human approval → Proceed to Step 2 (PLAN)
