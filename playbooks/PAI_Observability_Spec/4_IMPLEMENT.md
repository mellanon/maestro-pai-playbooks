# Step 4: IMPLEMENT

**Phase**: Implementation (Loop) | **TDD Required**

**SpecFirst Skill**: `/openspec-apply`

---

## Objective

Implement tasks using TDD. SpecFirst handles test-first workflow.

## Prerequisites

- Tasks approved (Step 3 complete)
- At least one task PENDING with satisfied dependencies

## Execute

### 1. Invoke SpecFirst Apply

```
/openspec-apply
> Which change? pai-observability-stage1
```

SpecFirst handles:
- Reading specs and tasks
- Writing tests for requirements (tests fail initially)
- Implementing code to make tests pass
- Updating task status in `tasks.md`

### 2. Loop Control

After each task:
- More PENDING tasks with satisfied deps → Continue looping (Step 5)
- No more actionable tasks → Continue to Step 5 (detects completion)

## Output

- Implementation code
- Test files
- Updated `tasks.md` with completion status
