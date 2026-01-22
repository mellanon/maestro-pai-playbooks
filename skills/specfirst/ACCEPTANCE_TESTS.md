# SpecFirst Skill Acceptance Tests

Manual skill-level tests to verify the SpecFirst skill works correctly before PR submission.

## Prerequisites

1. **Skill loaded:**
   - Claude Code should have loaded `.claude/skills/SpecFirst/SKILL.md`
   - Or manually: `read .claude/skills/SpecFirst/SKILL.md`

2. **Slash commands available:**
   - `/openspec-proposal`, `/openspec-apply`, `/openspec-archive`
   - `/specfirst-propose`, `/specfirst-release`, `/specfirst-validate`

---

## Test 1: Skill Routing Recognition

**Goal:** Verify Claude recognizes when to use SpecFirst.

**User Query:**
```
"I want to develop a JIRA skill"
```

**Expected Claude Response (similar to):**
```
I'll help you develop a JIRA skill. Let me start with a spec proposal
so we define requirements before implementation.

/openspec-proposal

What feature or change are you proposing?
> jira-skill

Which skill/project should contain the openspec directory?
> .claude/skills/Jira
```

**Success Criteria:**
- ✅ SpecFirst skill activated
- ✅ Spec creation offered BEFORE implementation talk
- ✅ `/openspec-proposal` invoked or offered
- ✅ Asks clarifying questions about scope

---

## Test 1a: Full JIRA Skill Development (End-to-End Acceptance Test)

**Goal:** Complete end-to-end test of SpecFirst workflow by developing a real skill.

### Context Sources

The spec should be derived from these external requirements:

| Source | Path/URL | What to Extract |
|--------|----------|-----------------|
| **mcp-atlassian** | `[local-path]/mcp-atlassian/README.md` | Available Jira tools (`jira_search`, `jira_get_issue`, `jira_create_issue`, etc.) |
| **PAI Principles** | PAI README or local notes on founding principles | Especially #5 (Spec/Test First), #8 (CLI as Interface), #9 (Goal→Code→CLI→Prompts→Agents) |
| **OpenSpec Workflow** | OpenSpec documentation or local notes | Spec format, SHALL/MUST language, workflow phases |

### Key PAI Principles to Apply

```
#5: Spec / Test / Evals First
   "Define expected behavior before writing implementation.
    If you can't specify it, you can't test it.
    If you can't test it, you can't trust it."

#8: CLI as Interface
   "Every operation should be accessible via command line.
    If there's no CLI command for it, you can't script it."

#9: Goal → Code → CLI → Prompts → Agents
   "The proper development pipeline for any new feature.
    Each layer builds on the previous."
```

### Design Pattern: Two-Phase Workflow

Reference `.claude/skills/Context/SKILL.md` for the two-phase pattern:

```
READ OPERATIONS (Search → Load):
1. SEARCH: User asks query → Show discovery results (table) → WAIT
2. LOAD: User selects items → Load full details into context

WRITE OPERATIONS (Direct):
- CREATE: Gather required fields → Execute
- UPDATE: Identify target → Modify
- DELETE: Confirm → Execute
```

The JIRA skill spec SHOULD follow this same pattern:
- `jira search` returns summary → User selects → `jira load` loads details
- `jira create` gathers fields → Executes directly
- `jira update` identifies target → Modifies directly

**Architecture (CLI-First per PAI Principles #4, #8, #9):**
```
bin/jira/jira.ts          ← TypeScript CLI (deterministic)
    ↓
.claude/skills/Jira/      ← Skill layer (prompts orchestrate CLI)
    ↓
CORE routing              ← Triggers skill for "jira", "issues", etc.
```

### User Query

```
I want to develop a JIRA skill for PAI following the deterministic-first architecture.

Source Requirements From:
- mcp-atlassian README — Reference for Jira API capabilities only (NOT as implementation dependency)
- PAI founding principles documentation — PAI 13 Founding Principles

Follow:
- OpenSpec workflow documentation — Spec format and workflow phases

Critical Principles (from PAI):
1. #4 Code Before Prompts — Write TypeScript CLI to handle Jira operations; prompts orchestrate the CLI
2. #3 Deterministic — CLI produces same output for same input, testable without AI
3. #8 CLI as Interface — Every Jira operation accessible via `jira <command>`
4. #9 Goal→Code→CLI→Prompts→Agents — Build layers in order
5. Stack: Bun + TypeScript — NOT MCP, NOT Python

Architecture Pattern (from Context skill):
Goal: Jira integration
    ↓
Code: bin/jira/jira.ts (TypeScript CLI, direct Jira REST API)
    ↓
CLI: jira search|create|update|transition|comment
    ↓
Prompts: SKILL.md translates "create issue for this bug" → jira create ...
    ↓
Agents: Claude executes workflows

NOT Using:
- ❌ mcp-atlassian MCP server
- ❌ Python or pip
- ❌ External dependencies beyond Jira REST API
```

### Expected Claude Behavior

**Phase 1: PROPOSAL (Claude MUST invoke `/openspec-proposal`)**

```
I'll help you develop a JIRA skill using SpecFirst methodology and CLI-first architecture.

First, let me read the source documents to understand the requirements.
[Reads mcp-atlassian README - for API reference only]
[Reads PAI principles - especially #3, #4, #8, #9]
[Reads OpenSpec workflow]

Now I'll create a spec proposal.

/openspec-proposal

Feature name: jira-skill
Skill/project: .claude/skills/Jira

[Creates directory structure]
.claude/skills/Jira/openspec/changes/jira-skill/
├── proposal.md
├── tasks.md
└── specs/
    └── jira-skill.md

[Shows proposal.md content]
# Proposal: JIRA Skill

## Summary
Create TypeScript CLI for Jira operations following PAI deterministic-first architecture.

## Impact
- **Scope:** New skill + CLI at bin/jira/
- **Breaking changes:** No (new skill)
- **Files affected:** .claude/skills/Jira/, bin/jira/, CORE routing

## Rationale
Per PAI Principle #4 (Code Before Prompts), we build a deterministic CLI first.
Per Principle #9 (Goal→Code→CLI→Prompts→Agents), layers build in order.
The CLI wraps Jira REST API directly - no MCP, no Python.

[Shows specs/jira-skill.md content]
# JIRA Skill Specification

## Requirements

### Requirement: Search Issues
The system SHALL provide issue search via `jira search` CLI command.

#### Scenario: Text Search
- **GIVEN** user wants to find issues
- **WHEN** user says "search jira for authentication"
- **THEN** skill SHALL execute `jira search "authentication" --format index`

### Requirement: Two-Phase Retrieval
The system SHALL follow SEARCH → LOAD pattern (like Context skill).

#### Scenario: Search then Load
- **GIVEN** search returns results
- **WHEN** results displayed
- **THEN** system SHALL wait for user selection before loading details

### Requirement: CLI Architecture
The CLI SHALL be implemented as:
- bin/jira/jira.ts — Main entry point
- bin/jira/lib/*.ts — Core functions
- Bun + TypeScript only
- Direct Jira REST API calls
...

[Shows tasks.md content]
# Tasks: JIRA Skill

## 1. CLI Foundation (Code layer)
- [ ] 1.1 Create bin/jira/jira.ts entry point
- [ ] 1.2 Implement lib/api.ts for Jira REST
- [ ] 1.3 Implement lib/config.ts for credentials
- [ ] 1.4 Add search command
- [ ] 1.5 Add create command

## 2. Skill Layer (Prompts layer)
- [ ] 2.1 Create .claude/skills/Jira/SKILL.md
- [ ] 2.2 Create workflow files
- [ ] 2.3 Update CORE routing

## 3. Testing
- [ ] 3.1 CLI unit tests (deterministic)
- [ ] 3.2 Integration tests
- [ ] 3.3 Validate against PAI principles

---

Here's my proposal. Please review:
- Does this match your intent?
- Any requirements to add or modify?
- Ready to implement, or need changes?
```

**CRITICAL: Claude MUST WAIT here for user review. DO NOT auto-implement.**

### Success Criteria

**Fundamental (MUST pass):**
- ✅ **Claude reads source documents BEFORE creating spec**
- ✅ **Claude invokes `/openspec-proposal` (not manual file creation)**
- ✅ **Claude WAITS for review (doesn't auto-implement)**
- ✅ **Spec references PAI principles by number (#3, #4, #8, #9)**
- ✅ **Uses SHALL/MUST language in requirements**

**Structural:**
- ✅ Directory created: `openspec/changes/[name]/`
- ✅ proposal.md has Summary, Impact, Rationale
- ✅ specs/*.md has Requirements with Scenarios
- ✅ tasks.md has checklist format

**Architecture (CLI-First):**
- ✅ **Spec describes TypeScript CLI at bin/jira/**
- ✅ **Spec does NOT use MCP server**
- ✅ **Tasks order: CLI first (Code layer) → Skill second (Prompts layer)**
- ✅ Tasks include CORE routing update
- ✅ Tasks include deterministic CLI tests

### Test Execution Notes

**If Claude manually creates files without invoking `/openspec-proposal`:**
- **TEST FAILS** - Claude should use the slash commands
- This indicates the skill routing isn't working
- Check: Is SpecFirst SKILL.md loaded?

**If Claude starts implementing before spec review:**
- **TEST FAILS** - Violates spec-first methodology
- Claude should ask "Ready to implement?" and WAIT

**If spec doesn't reference source documents:**
- **TEST FAILS** - Requirements should be traceable
- Spec should cite mcp-atlassian features and PAI principles

### Clean State Reset

Before running this test, ensure clean state:

```bash
# Remove any existing openspec changes
rm -rf .claude/skills/Jira/openspec/changes/

# Verify clean
ls .claude/skills/Jira/openspec/ 2>/dev/null || echo "No openspec dir (OK)"
```

---

## Test 2: /openspec-proposal Command

**Goal:** Verify proposal command creates correct structure and waits for review.

**User Query:**
```
/openspec-proposal
```

**When prompted:**
```
> Feature name: add-search
> Skill/project: .claude/skills/Context
```

**Expected Claude Response:**
```
Created:
.claude/skills/Context/openspec/changes/add-search/
├── proposal.md
├── tasks.md
└── specs/
    └── add-search.md

--- proposal.md ---
# Proposal: Add Search

## Summary
[Description of what add-search does]

## Impact
- **Scope:** Context skill
- **Breaking changes:** No
- **Files affected:** [list]

## Rationale
[Why this change is needed]

--- specs/add-search.md ---
# Add Search Specification

## Requirements

### Requirement: [Name]
The system SHALL [behavior].

#### Scenario: [Use case]
- **WHEN** [trigger]
- **THEN** [outcome]

--- tasks.md ---
# Tasks: Add Search

- [ ] 1.1 [First task]
- [ ] 1.2 [Second task]

---

Here's my proposal. Please review:
- Does this match your intent?
- Any requirements to add or modify?
- Ready to implement, or need changes?
```

**Success Criteria:**
- ✅ Directory structure created (verify with `ls`)
- ✅ proposal.md has Summary, Impact, Rationale sections
- ✅ tasks.md has checklist format `- [ ]`
- ✅ specs/ has file with SHALL/MUST language
- ✅ **Claude explicitly asks for review/approval**
- ✅ **Claude waits (doesn't auto-implement)**

---

## Test 3: Iterative Review (Feedback Loop)

**Goal:** Verify skill handles spec refinement iterations.

**Setup:** Run Test 2 first to create proposal.

**User Query:**
```
"Add a requirement for filtering by date"
```

**Expected Claude Response:**
```
I'll add that requirement to the spec.

[Updates specs/add-search.md]

### Requirement: Date Filter
The system SHALL allow filtering search results by date range.

#### Scenario: Filter by date
- **WHEN** user provides --from and --to flags
- **THEN** results SHALL only include items within date range

[Updates tasks.md]

- [ ] 2.1 Implement --from flag
- [ ] 2.2 Implement --to flag
- [ ] 2.3 Test date filtering

Updated. Anything else to add, or ready to implement?
```

**Then User Says:**
```
"Also add pagination"
```

**Expected Claude Response:**
```
[Updates specs/add-search.md]

### Requirement: Pagination
The system SHALL paginate results when exceeding limit.

[Updates tasks.md]

- [ ] 3.1 Implement --limit flag
- [ ] 3.2 Implement --page flag

Updated. Anything else?
```

**Success Criteria:**
- ✅ Claude updates spec (doesn't create new proposal)
- ✅ Claude updates tasks.md too
- ✅ Claude asks "anything else?" after each update
- ✅ **Claude stays in review phase until user says "implement"**

---

## Test 4: /openspec-apply Command

**Goal:** Verify apply command implements from specs with progress tracking.

**Setup:** Run Tests 2-3 first, then user says "implement".

**User Query:**
```
"Ready to implement"
```

**Expected Claude Response:**
```
/openspec-apply

I'll implement based on these requirements:

**From specs/add-search.md:**
1. Search - Allow searching vault content
2. Date Filter - Filter by date range
3. Pagination - Paginate results

**From tasks.md:**
- 1.1 Create search function
- 1.2 Add CLI command
- 2.1-2.3 Date filtering
- 3.1-3.2 Pagination

Proceeding with implementation...

✓ 1.1 Created search function in lib/search.ts
✓ 1.2 Added `obs search` command
Working on 2.1...
✓ 2.1 Implemented --from flag
✓ 2.2 Implemented --to flag
✓ 2.3 Added date filter tests (3 tests pass)
✓ 3.1 Implemented --limit flag
✓ 3.2 Implemented --page flag

All tasks complete. Running tests...
All 8 tests pass.

Implementation complete. Ready to archive?
```

**Success Criteria:**
- ✅ Lists requirements from specs BEFORE implementing
- ✅ Shows progress as tasks complete
- ✅ Updates tasks.md (`- [x]` format)
- ✅ Runs tests and reports results
- ✅ **Asks "ready to archive?" when done**

---

## Test 5: /openspec-archive Command

**Goal:** Verify archive command merges specs and updates CHANGELOG.

**Setup:** Run Test 4 first (implementation complete, tests pass).

**User Query:**
```
"Yes, archive it"
```

**Expected Claude Response:**
```
/openspec-archive

Archiving add-search...

1. Verifying completion:
   ✓ All 7 tasks complete in tasks.md
   ✓ All tests pass

2. Merging specs:
   ✓ changes/add-search/specs/add-search.md → openspec/specs/add-search.md

3. Updating CHANGELOG.md:

   ## [Unreleased]

   ### Added
   - Search functionality for vault content
   - Date range filtering (--from, --to flags)
   - Pagination support (--limit, --page flags)

4. Moving to archive:
   ✓ openspec/changes/add-search/ → openspec/archive/2025-12-12-add-search/

Archived successfully!
- Specs merged to openspec/specs/
- CHANGELOG.md updated
- Original proposal preserved in archive/

Ready for next feature? Or proceed to /specfirst-release?
```

**Verify with:**
```bash
ls .claude/skills/Context/openspec/specs/       # Should have add-search.md
ls .claude/skills/Context/openspec/changes/     # Should be empty
ls .claude/skills/Context/openspec/archive/     # Should have dated folder
cat .claude/skills/Context/CHANGELOG.md         # Should have new entry
```

**Success Criteria:**
- ✅ Specs merged to `openspec/specs/`
- ✅ CHANGELOG.md has new entry under [Unreleased]
- ✅ Change folder moved to `archive/YYYY-MM-DD-name/`
- ✅ Archive contains original proposal.md and tasks.md
- ✅ `changes/` folder is now empty

---

## Test 6: /specfirst-release Command

**Goal:** Verify release command creates file inventory.

**User Query:**
```
/specfirst-release
```

**Expected Behavior:**
- Claude creates RELEASE-vX.Y.Z.md (or uses template)
- Claude lists files to include
- Claude lists files to exclude
- Claude requires human approval gates

**Success Criteria:**
- ✅ File inventory created
- ✅ Include list populated
- ✅ Exclude list populated
- ✅ Human approval required at each gate

---

## Test 7: Two-Phase Enforcement (Spec Before Code)

**Goal:** Verify skill enforces spec-first, not code-first.

**User Query:**
```
"Just implement a quick --verbose flag"
```

**Expected Behavior:**
- Claude should redirect to spec creation
- Claude should NOT jump straight to coding
- Claude should explain why specs first

**Success Criteria:**
- ✅ Implementation deferred
- ✅ Spec creation suggested
- ✅ Rationale provided ("If you can't specify it...")

---

## Test 8: Universal Applicability

**Goal:** Verify skill works for non-PAI projects.

**User Query:**
```
"I'm working on a non-PAI project. How do I use SpecFirst?"
```

**Expected Behavior:**
- Claude explains standard development path
- Claude does NOT require contrib branch
- Claude offers simplified workflow

**Success Criteria:**
- ✅ Standard path explained
- ✅ PAI-specific steps marked optional
- ✅ Core spec-first flow preserved

---

## Test 9: Cross-Reference Validation

**Goal:** Verify documentation links work.

**Manual Checks:**
1. SKILL.md references all workflow files - they exist
2. Examples in SKILL.md use actual commands
3. Template references in docs point to real files

**Success Criteria:**
- ✅ All `workflows/*.md` files exist
- ✅ All `templates/*.md` files exist
- ✅ All `docs/*.md` files exist
- ✅ Commands referenced in SKILL.md match actual commands

---

## Test 10: Dogfooding Verification

**Goal:** Verify SpecFirst uses itself.

**Manual Checks:**
1. `openspec/` directory exists in SpecFirst skill
2. Specs exist for SpecFirst requirements
3. CHANGELOG.md documents releases

**Commands:**
```bash
ls .claude/skills/SpecFirst/openspec/
ls .claude/skills/SpecFirst/openspec/changes/
cat .claude/skills/SpecFirst/CHANGELOG.md
```

**Success Criteria:**
- ✅ openspec/ structure exists
- ✅ Specs define SpecFirst requirements
- ✅ CHANGELOG tracks changes
- ✅ Skill follows its own methodology

---

---

## v2.0 Tests: State Management & Quality Gates

### Test v2.1: State Initialization

**Goal:** Verify `specfirst init` creates state file.

**Commands:**
```bash
cd /tmp && mkdir test-project && cd test-project
specfirst init my-project
```

**Expected:**
```
✅ Initialized SpecFirst for project: my-project
   State: /tmp/test-project/.specfirst/state.json
```

**Verify:**
```bash
cat .specfirst/state.json
```

**Success Criteria:**
- ✅ `.specfirst/` directory created
- ✅ `state.json` exists with version, project, current_phase
- ✅ `current_phase` is "SPECIFY"

---

### Test v2.2: Status Display

**Goal:** Verify `specfirst status` shows workflow state.

**Commands:**
```bash
specfirst status
```

**Expected:**
```
📋 SpecFirst Status
═══════════════════

Project: my-project
Phase: SPECIFY
Active Changes: 0

Pipeline:
  🔄 SPECIFY ──▶ ⏳ TEST ──▶ ⏳ IMPLEMENT ──▶ ⏳ ARCHIVE ──▶ ⏳ RELEASE
```

**Success Criteria:**
- ✅ Project name displayed
- ✅ Current phase displayed
- ✅ Pipeline visualization shown

---

### Test v2.3: Resume Point

**Goal:** Verify `specfirst resume` shows next steps.

**Commands:**
```bash
specfirst resume
```

**Expected:**
```
🔄 Resume Point
═══════════════════════════════════════

No active changes found.

➡️  Next: Create a change proposal with /openspec-proposal
```

**Success Criteria:**
- ✅ Shows resume information
- ✅ Suggests next action

---

### Test v2.4: Gate Status

**Goal:** Verify `specfirst gate` shows quality gates.

**Commands:**
```bash
specfirst gate
specfirst gate TEST
```

**Expected:**
```
📊 Quality Gate Status
═══════════════════════════════════════

No active change. Gates will be checked when a change is in progress.
```

**Success Criteria:**
- ✅ Gate status displayed
- ✅ Thresholds shown
- ✅ Specific phase query works

---

### Test v2.5: State Query

**Goal:** Verify `specfirst state get/set` works.

**Commands:**
```bash
specfirst state get current_phase
specfirst state set current_phase '"TEST"'
specfirst state get current_phase
```

**Expected:**
```
"SPECIFY"
✅ Set current_phase
"TEST"
```

**Success Criteria:**
- ✅ Get returns JSON value
- ✅ Set updates value
- ✅ Changes persist

---

### Test v2.6: Session Persistence

**Goal:** Verify state persists across sessions.

**Steps:**
1. Initialize state: `specfirst init test`
2. Set a value: `specfirst state set current_phase '"IMPLEMENT"'`
3. Exit terminal
4. Open new terminal
5. Check state: `specfirst state get current_phase`

**Success Criteria:**
- ✅ State persists after terminal close
- ✅ Value matches what was set

---

### Test v2.7: CLI Help

**Goal:** Verify `specfirst --help` is comprehensive.

**Commands:**
```bash
specfirst --help
```

**Success Criteria:**
- ✅ All commands documented
- ✅ Quality gates section present
- ✅ Phases section present
- ✅ Examples provided

---

## Quick Test Checklist

Run these 5 essential tests before PR:

- [ ] **Test 1:** Skill routing recognized
- [ ] **Test 2:** /openspec-proposal creates structure
- [ ] **Test 5:** /openspec-archive merges correctly
- [ ] **Test 7:** Spec-before-code enforced
- [ ] **Test 10:** Dogfooding in place

---

## Automated Validation (Future)

These tests could be automated:

| Test | Automation |
|------|------------|
| File existence | `ls` + check exit code |
| Command availability | Check `.claude/commands/*.md` |
| Cross-references | Parse markdown links, verify targets |
| Structure creation | Run command, check directory |
| CHANGELOG format | Regex validation |

For now, run manually in Claude Code before PR.

---

## Notes

- These are **manual acceptance tests** - run them in Claude Code
- Focus on **skill behavior** (does Claude understand the methodology)
- If any test fails, document the issue before PR submission
- All essential tests should pass before creating PR
