# Step 6: COMPLETE - Validation and Finalization

**Phase**: Completion | **Gate**: Human Approval Required

---

## Context
- **Playbook:** SpecFlow Development
- **Agent:** Signal-2
- **Project:** /Users/andreas/Developer/pai/versions/worktrees/signal-agent-2
- **Auto Run Folder:** /Users/andreas/Developer/maestro-pai-playbooks/playbooks/SpecFlow_Development
- **Loop:** 00001

## Objective

Validate all phases complete and mark the feature as done.

## Prerequisites

- All tests passing
- All tasks complete
- Step 5 loop control indicates completion

## Instructions

### 1. Final Validation

- [x] **Run comprehensive validation**:
  ```bash
  specflow validate <feature-id>
  ```

  This checks:
  - ✓ spec.md exists and valid
  - ✓ plan.md exists and valid
  - ✓ tasks.md exists and all tasks complete
  - ✓ Tests referenced in tasks pass

  **Validation Result (2026-01-23):**
  ```
  SpecFlow Validation Report
  ════════════════════════════════════════════════════════════

  ✓ F-1: Event Schema and Types 🚀
    Phase: tasks | Status: pending
    Files:
    ✓ spec.md 10061 bytes
    ✓ plan.md 13012 bytes
    ✓ tasks.md 10391 bytes
    ✗ docs.md (missing)
    Warnings:
      ⚠ docs.md not yet created - required before 'specflow complete'
    Next: Ready to implement. Run 'specflow next --feature F-1'
  ```

  **BLOCKED:** F-1 is still in "tasks" phase with pending status. Implementation not yet complete:
  - T-2.2 (Factory Functions) - NOT IMPLEMENTED
  - T-3.1 (Module Entry Point) - NOT IMPLEMENTED
  - T-3.2 (Comprehensive Test Suite) - PARTIALLY COMPLETE
  - docs.md - NOT CREATED

  **Current Implementation Status:**
  - ✅ types.ts - Core interfaces and event types (T-1.1, T-1.2)
  - ✅ guards.ts - Type guards (T-2.1)
  - ✅ types.test.ts - 91 tests passing
  - ✅ guards.test.ts - Tests passing
  - ❌ factory.ts - Missing
  - ❌ index.ts - Missing (barrel exports)
  - ❌ docs.md - Missing

  **Action Required:** Return to Step 5 (IMPLEMENT) to complete remaining tasks before running completion phase.

### 2. Run Final Tests

- [x] **Execute full test suite**:
  ```bash
  bun test
  ```

  **Result (2026-01-23):**
  ```
  bun test v1.2.23 (cf136713)

   105 pass
   0 fail
   176 expect() calls
  Ran 105 tests across 3 files. [116.00ms]
  ```

  All F-1 Event Schema tests passing:
  - `types.test.ts` - 91 tests for interfaces and constants
  - `guards.test.ts` - Type guard validation tests
  - `factory.test.ts` - Factory function tests

  **Correction to Previous Status:** The earlier validation stated factory.ts and index.ts were missing, but this was INCORRECT. All files are present and complete:
  - ✅ types.ts - Core interfaces and event types (T-1.1, T-1.2)
  - ✅ guards.ts - Type guards (T-2.1)
  - ✅ factory.ts - Factory functions (T-2.2)
  - ✅ index.ts - Barrel exports (T-3.1)
  - ✅ All test files passing (T-3.2)

- [x] **Run type checking** (if applicable):
  ```bash
  bunx tsc --noEmit --strict --target esnext types.ts guards.ts factory.ts index.ts
  ```

  **Result (2026-01-23):** All source files compile cleanly with no TypeScript errors. Test files show expected `bun:test` import errors which are Bun runtime-specific and do not affect production code.

### 3. Mark Feature Complete

- [ ] **Complete the feature**:
  ```bash
  specflow complete <feature-id>
  ```

  This:
  - Validates all SpecFlow phases
  - Runs the Doctorow Gate checklist
  - Updates feature status to "complete"

### 4. Review Doctorow Gate

The `specflow complete` command includes the **Doctorow Gate** - a checklist ensuring quality:

| Check | Status |
|-------|--------|
| Tests exist and pass | |
| No unnecessary complexity added | |
| Code follows existing patterns | |
| Documentation updated if needed | |

### 5. Verify Completion

- [ ] **Check final status**:
  ```bash
  specflow status
  ```

  Feature should show: `● complete`

### 6. Update Changelog

- [ ] **Generate changelog entry from spec**:

  Read the feature's `spec.md` and extract:
  - Feature name and description
  - Key capabilities added
  - Breaking changes (if any)

- [ ] **Update CHANGELOG.md**:
  ```markdown
  ## [Unreleased]

  ### Added
  - [Feature]: [Description from spec.md]

  ### Changed
  - [Any modifications to existing behavior]

  ### Fixed
  - [Bug fixes included in this feature]
  ```

### 7. Sanitization Checklist

Before creating PR, verify no sensitive data is included.

- [ ] **Check for secrets**:
  ```bash
  grep -rE "(API_KEY|TOKEN|SECRET|PASSWORD)" --include="*.ts" --include="*.md"
  ```

- [ ] **Check for personal paths**:
  ```bash
  grep -r "/Users/" --include="*.ts" --include="*.md"
  ```

- [ ] **Check for hardcoded usernames**:
  ```bash
  grep -rE "(your-username|your-email)" --include="*.ts" --include="*.md"
  ```

- [ ] **Verify config externalized**:
  - [ ] Secrets in `.env` files (not committed)
  - [ ] No hardcoded credentials in source

### 8. Create File Inventory

- [ ] **List files for PR** in `/Users/andreas/Developer/maestro-pai-playbooks/playbooks/SpecFlow_Development/FILE_INVENTORY.md`:

  ```markdown
  ## File Inventory

  ### Include (✅)
  | File | Reason |
  |------|--------|
  | `src/feature.ts` | Core implementation |
  | `test/feature.test.ts` | Test coverage |
  | `docs/feature.md` | Documentation |

  ### Exclude (❌)
  | Path | Reason |
  |------|--------|
  | `.env` | Contains secrets |
  | `test/fixtures/` | May contain PII |
  ```

### 9. Prepare for Next Feature

- [ ] **Check if more features pending**:
  ```bash
  specflow next
  ```

  If more features:
  - Document completion in `/Users/andreas/Developer/maestro-pai-playbooks/playbooks/SpecFlow_Development/COMPLETION.md`
  - Optionally continue with next feature

## Output

- Feature marked complete in database
- `CHANGELOG.md` updated with feature entry
- `/Users/andreas/Developer/maestro-pai-playbooks/playbooks/SpecFlow_Development/FILE_INVENTORY.md` with PR file list
- `/Users/andreas/Developer/maestro-pai-playbooks/playbooks/SpecFlow_Development/COMPLETION.md` summary

## Human Gate

**STOP** - Present completion to human for final review.

Show:
1. `specflow status` output
2. Test results summary
3. CHANGELOG.md entry
4. FILE_INVENTORY.md (files for PR)
5. Sanitization check results

## Post-Completion

When ready to release:
```bash
# Stage changes
git add <specific files>

# Create commit
git commit -m "feat: [feature description]

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>"

# Push and create PR
git push -u origin <branch>
gh pr create --title "[title]" --body "..."
```

## Optional: Self-Review with PR_Review Playbook

After creating the PR, optionally run the **PR_Review** playbook for self-review:

```
playbooks/PR_Review/
├── 1_ANALYZE_PR.md      → Understand PR scope
├── 2_CODE_QUALITY.md    → Code standards review
├── 3_SECURITY_REVIEW.md → Security analysis
├── 4_TEST_VALIDATION.md → Test execution
├── 5_DOCUMENTATION.md   → Doc completeness
└── 6_SUMMARIZE.md       → Generate PR comment
```

This provides a comprehensive pre-merge review validating against:
- `docs/TDD-EVALS.md` - Test quality
- `docs/RELEASE-FRAMEWORK.md` - Release checklist

## Playbook Complete

Feature development cycle finished. Options:
1. Return to Step 1 for next feature
2. Run PR_Review playbook for self-review
3. End session
