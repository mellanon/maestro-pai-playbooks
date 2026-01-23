# Step 4: IMPLEMENT - TDD Execution

**Phase**: Implementation | **Loop**: Repeats until all tasks complete

---

## Context
- **Playbook:** SpecFlow Development
- **Agent:** Signal-1
- **Project:** /Users/andreas/Developer/pai/versions/worktrees/signal-agent-1
- **Auto Run Folder:** /Users/andreas/Developer/maestro-pai-playbooks/playbooks/SpecFlow_Development
- **Loop:** 00001

## Objective

Implement the next task using TDD (Test-Driven Development).

## Prerequisites

- Tasks approved (tasks.md exists and human approved)
- Feature phase is "tasks" or "implement"
- **Outputs directory exists** (create if needed):
  ```bash
  mkdir -p /Users/andreas/Developer/maestro-pai-playbooks/playbooks/SpecFlow_Development/outputs
  ```

## Instructions

### 1. Get Implementation Context

- [x] **Get next task context**: (2026-01-23 Signal-2: Read spec, plan, tasks from .specify/specs/f-1-event-schema-and-types/)
  ```bash
  specflow implement
  ```

  Or for a specific feature:
  ```bash
  specflow implement --feature <feature-id>
  ```

  This outputs:
  - Feature spec
  - Technical plan
  - Tasks with current progress
  - Implementation prompt for AI

### 2. TDD Cycle: RED

- [x] **Write failing test first**: (2026-01-23 Signal-2: Created guards.test.ts with 49 tests for T-2.1)
  - Create test file if doesn't exist
  - Write test for the current task's acceptance criteria
  - Run tests to confirm failure (RED state)

- [x] **Record test creation** in `/Users/andreas/Developer/maestro-pai-playbooks/playbooks/SpecFlow_Development/outputs/LOOP_00001_PROGRESS.md`: (2026-01-23 Signal-2)
  ```markdown
  ## Task: [task name]
  ### Tests Written
  - [ ] test_xxx: [description]
  ```

### 3. TDD Cycle: GREEN

- [x] **Implement minimal code** to make tests pass: (2026-01-23 Signal-2: Created guards.ts with 9 type guard functions)
  - Focus only on current task
  - Don't over-engineer
  - Follow the plan.md architecture

- [x] **Run tests** to confirm passing (GREEN state): (2026-01-23 Signal-2: 91 tests passing across 2 files)
  ```bash
  bun test
  # or project-specific test command
  ```

### 4. TDD Cycle: REFACTOR (if needed)

- [x] **Clean up code** while keeping tests green: (2026-01-23 Signal-2: Code already clean, no refactoring needed)
  - Remove duplication
  - Improve naming
  - Keep it simple

### 5. Update Progress

- [x] **Update progress file**: (2026-01-23 Signal-2: Updated LOOP_00001_PROGRESS.md with T-2.1 details)
  ```markdown
  ### Implementation
  - Files created: [list]
  - Files modified: [list]
  - Tests passing: [count]
  ```

## Output

- New/modified source files
- New/modified test files
- `/Users/andreas/Developer/maestro-pai-playbooks/playbooks/SpecFlow_Development/outputs/LOOP_00001_PROGRESS.md` updated

## Next

→ Proceed to Step 5 (VERIFY) to check progress and loop control
