# Step 1: SPECIFY - Create Change Proposal

**Phase**: Specification | **Gate**: Human Approval Required

---

## Context
- **Playbook:** SpecFirst Development
- **Agent:** {{AGENT_NAME}}
- **Project:** {{AGENT_PATH}}
- **Auto Run Folder:** {{AUTORUN_FOLDER}}
- **Loop:** {{LOOP_NUMBER}}

## Objective

Create a structured change proposal with specifications that define what will be built.

## Instructions

### 1. Identify the Change

- [ ] **Read user requirements** from the task description or input files
- [ ] **Determine change name** (kebab-case: `add-feature`, `fix-bug-123`)
- [ ] **Document rationale** in `{{AUTORUN_FOLDER}}/LOOP_{{LOOP_NUMBER}}_REQUIREMENTS.md`

### 2. Invoke SpecFirst Proposal

- [ ] **Run `/openspec-proposal`** to create the spec structure:

```
/openspec-proposal
> What feature or change? [change-name]
> Which skill/project? [target-project]
```

The command creates:
```
openspec/changes/[change-name]/
├── proposal.md     ← Summary, impact, rationale
├── tasks.md        ← Implementation checklist
└── specs/
    └── feature.md  ← SHALL/MUST requirements
```

### 3. Validate Spec Quality

- [ ] **Check spec format** against `docs/TDD-EVALS.md`:

| Criterion | Check |
|-----------|-------|
| SHALL statements are testable | Each can be verified by a test |
| Requirements are atomic | One requirement per statement |
| No ambiguity | Clear, unambiguous language |
| Dependencies identified | External dependencies noted |

- [ ] **Check design against `docs/PAI-PRINCIPLES.md`**:

| Principle | Applies? | Validated? |
|-----------|----------|------------|
| Deterministic code preferred | | |
| CLI interfaces where applicable | | |
| UNIX philosophy (single purpose) | | |
| Skill-based organization | | |

### 4. Initialize State Tracking

- [ ] **Run `specfirst init`** if not already initialized:
  ```bash
  specfirst init
  ```

- [ ] **Verify state tracking**:
  ```bash
  specfirst status
  ```

## Output

- `openspec/changes/[change-name]/` folder structure
- `{{AUTORUN_FOLDER}}/LOOP_{{LOOP_NUMBER}}_REQUIREMENTS.md` with captured requirements
- `.specfirst/state.json` initialized

## Human Gate

**STOP** - Present specs to human for review before proceeding.

Show:
1. `proposal.md` summary
2. `specs/*.md` requirements
3. `tasks.md` breakdown

**Only proceed to Step 2 after human approval.**
