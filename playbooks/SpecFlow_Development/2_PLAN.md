# Step 2: PLAN - Create Technical Architecture

**Phase**: Planning | **Gate**: Human Approval Required

---

## Context
- **Playbook:** SpecFlow Development
- **Agent:** {{AGENT_NAME}}
- **Project:** {{AGENT_PATH}}
- **Auto Run Folder:** {{AUTORUN_FOLDER}}
- **Loop:** {{LOOP_NUMBER}}

## Objective

Create a technical architecture plan including data models, failure modes, and design decisions.

## Prerequisites

- Specification approved (spec.md exists and human approved)
- Feature phase is "specify"

## Instructions

### 1. Check Current Status

- [ ] **Verify spec exists**:
  ```bash
  specflow status
  ```
  Feature should show phase: `① specify`

### 2. Create Technical Plan

- [ ] **Run the plan phase**:
  ```bash
  specflow plan <feature-id>
  ```

  Example: `specflow plan F-001`

  This creates:
  ```
  specflow/<feature-id>/
  ├── spec.md      ← (from step 1)
  └── plan.md      ← Architecture decisions
  ```

### 3. Validate Plan Quality

- [ ] **Check plan covers**:

| Section | Included? |
|---------|-----------|
| Data models | |
| API/Interface design | |
| Error handling strategy | |
| Failure modes | |
| Dependencies | |
| Testing strategy | |

- [ ] **Check design against `docs/PAI-PRINCIPLES.md`**:

| Principle | Applied? |
|-----------|----------|
| Code before prompts | |
| Deterministic where possible | |
| CLI-first interfaces | |

### 4. Verify Phase Complete

- [ ] **Check feature status**:
  ```bash
  specflow status
  ```
  Feature should now show phase: `② plan`

## Output

- `specflow/<feature-id>/plan.md` with technical architecture
- Feature phase updated to "plan"

## Human Gate

**STOP** - Present plan to human for review before proceeding.

Show:
1. `plan.md` architecture decisions
2. Data models and interfaces
3. Failure modes and mitigations

**Only proceed to Step 3 after human approval.**
