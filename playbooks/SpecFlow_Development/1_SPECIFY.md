# Step 1: SPECIFY - Create Feature Specification

**Phase**: Specification | **Gate**: Human Approval Required

---

## Context
- **Playbook:** SpecFlow Development
- **Agent:** {{AGENT_NAME}}
- **Project:** {{AGENT_PATH}}
- **Auto Run Folder:** {{AUTORUN_FOLDER}}
- **Loop:** {{LOOP_NUMBER}}

## Input Specification

**Read and analyze the requirements specification:**
```
{{AUTORUN_FOLDER}}/assets/SPEC.md
```

This file contains the feature requirements to implement.

## Objective

Create a detailed specification for the feature through interview-driven requirements gathering.

## Instructions

### 1. Initialize Project (if needed)

- [ ] **Check if SpecFlow initialized**:
  ```bash
  specflow status
  ```

- [ ] **If not initialized**, load from requirements spec:
  ```bash
  specflow init --from-spec {{AUTORUN_FOLDER}}/assets/SPEC.md
  ```

  Or describe the project:
  ```bash
  specflow init "Description of the application or project"
  ```

  This decomposes the requirements into features in the queue.

- [ ] **Read constitutional gates** for this phase:
  - `docs/PAI-PRINCIPLES.md` - Design must follow founding principles

### 2. Check Current Status

- [ ] **View feature queue**:
  ```bash
  specflow status
  ```

- [ ] **Identify the next feature** to work on (highest priority pending)

### 3. Create Specification

- [ ] **Run the specify phase** for the target feature:
  ```bash
  specflow specify <feature-id>
  ```

  Example: `specflow specify F-001`

  This creates:
  ```
  specflow/<feature-id>/
  └── spec.md     ← Detailed requirements
  ```

### 4. Validate Spec Quality

- [ ] **Check spec format** against `docs/TDD-EVALS.md`:

| Criterion | Check |
|-----------|-------|
| Requirements are testable | Each can be verified by a test |
| Requirements are atomic | One requirement per statement |
| No ambiguity | Clear, unambiguous language |
| Dependencies identified | External dependencies noted |

- [ ] **Check design against `docs/PAI-PRINCIPLES.md`**:

| Principle | Applies? | Validated? |
|-----------|----------|------------|
| Deterministic code preferred | | |
| CLI interfaces where applicable | | |
| UNIX philosophy (single purpose) | | |

### 5. Verify Phase Complete

- [ ] **Check feature status**:
  ```bash
  specflow status --json | jq '.features[] | select(.id=="<feature-id>")'
  ```

- [ ] **Confirm phase is "specify"**

## Output

- `specflow/<feature-id>/spec.md` with detailed requirements
- Feature status updated in database

## Human Gate

**STOP** - Present spec to human for review before proceeding.

Show:
1. `spec.md` requirements
2. Feature description and rationale

**Only proceed to Step 2 after human approval.**
