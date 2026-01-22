# Step 5: VERIFY GATE

**Phase**: Verification | **Gate**: 80%+ | **Loop Control**

---

## Objective

Verify implementation progress and determine loop continuation.

## Execute

### 1. Run Tests

```bash
bun test
```

### 2. Check Gate Status

```bash
specfirst gate
```

### 3. Check Task Progress

```bash
specfirst status
```

## Loop Decision

| Condition | Action |
|-----------|--------|
| Tests fail | → Back to Step 4 (fix) |
| Tasks remain PENDING | → Back to Step 4 (next task) |
| All tasks COMPLETE, gate passes | → Proceed to Step 6 |

## Output

- `TEST_RESULTS.md` with latest results
- Updated `.specfirst/state.json`
