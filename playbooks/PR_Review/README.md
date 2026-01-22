# PR Review Playbook

**A Maestro Auto Run playbook for comprehensive pull request reviews.**

## What This Playbook Does

Performs a thorough, multi-pass code review examining code quality, security, test coverage, and documentation. Works with any codebase.

### Review Areas

1. **PR Analysis** - Understand scope, changed files, categorization
2. **Code Quality** - Language idioms, error handling, patterns
3. **Security Review** - OWASP vulnerabilities, secret handling, injection risks
4. **Test Validation** - Test execution, coverage gaps, TDD compliance
5. **Documentation** - README updates, inline comments, changelog
6. **Summary** - Consolidated findings and PR comment

## Playbook Structure

This playbook follows a **linear 6-document pattern** (no looping):

```
1_ANALYZE_PR.md        → Understand PR scope, changed files
2_CODE_QUALITY.md      → Language-specific code review
3_SECURITY_REVIEW.md   → Security-focused analysis
4_TEST_VALIDATION.md   → Execute tests, check coverage
5_DOCUMENTATION.md     → Documentation completeness
6_SUMMARIZE.md         → Compile findings, create PR comment
```

## Prerequisites

- Maestro installed and configured
- Git repository with the PR branch checked out
- GitHub CLI (`gh`) installed for PR fetching
- Test runner available for the project

## Setup in Maestro

1. Place this folder in your Maestro Auto Run directory
2. Check out the PR branch:
   ```bash
   gh pr checkout <PR-number>
   ```
3. Configure Auto Run settings:
   - **Loop Mode**: OFF (sequential execution)
   - **Max Loops**: N/A (no looping)
4. Run the playbook

## Working Files Generated

| File | Purpose |
|------|---------|
| `REVIEW_SCOPE.md` | PR context and changed file categorization |
| `CODE_ISSUES.md` | Code quality findings |
| `SECURITY_ISSUES.md` | Security vulnerabilities found |
| `TEST_RESULTS.md` | Test execution results |
| `DOC_ISSUES.md` | Documentation gaps |
| `REVIEW_SUMMARY.md` | Final consolidated review |
| `PR_COMMENT.md` | Ready-to-post GitHub PR comment |

## Integration with SpecFlow_Development

This playbook is designed to run **after** the SpecFlow_Development playbook creates a PR:

```
SpecFlow_Development (creates PR)
            ↓
        PR Created
            ↓
PR_Review (this playbook)
            ↓
        Merge
```

Reference from Step 6 of SpecFlow_Development:
```
After creating PR, optionally run PR_Review playbook for self-review
```

## Customization

### For TypeScript/JavaScript Projects
- Update 2_CODE_QUALITY.md checks for ESLint rules, type safety
- Update 4_TEST_VALIDATION.md for bun/jest/vitest

### For Go Projects
- Update 2_CODE_QUALITY.md for Go idioms, error wrapping
- Update 4_TEST_VALIDATION.md for `go test ./...`

### For Python Projects
- Update 2_CODE_QUALITY.md for PEP8, type hints
- Update 4_TEST_VALIDATION.md for pytest

## Constitutional Gates

This playbook validates against:

| Document | Purpose |
|----------|---------|
| `docs/TDD-EVALS.md` | Test quality criteria |
| `docs/RELEASE-FRAMEWORK.md` | Release checklist compliance |

## Exit Conditions

Playbook completes when all 6 documents report `COMPLETED` status.

## Attribution

This playbook is adapted from the **Fabric PR Review** playbook by [Kayvan Sylvan](https://github.com/ksylvan), originally designed for reviewing PRs to [danielmiessler/fabric](https://github.com/danielmiessler/fabric).

### Key Resources

| Resource | URL |
|----------|-----|
| **Maestro** | https://runmaestro.ai/ |
| **Kayvan's Maestro fork** | [ksylvan/Maestro](https://github.com/ksylvan/Maestro) (preview branch) |
| **LaunchMaestroDev** | [ksylvan/LaunchMaestroDev](https://github.com/ksylvan/LaunchMaestroDev) |
| **Custom playbooks** | [ksylvan/maestro-playbooks-custom](https://github.com/ksylvan/maestro-playbooks-custom) |

### SpecFlow Bundle

This playbook integrates with the **SpecFlow Bundle** by [J.C. Fischer](https://github.com/jcfischer):
- [jcfischer/specflow-bundle](https://github.com/jcfischer/specflow-bundle)

---

> **"I am literally like a small programming team by myself."** — Kayvan Sylvan
