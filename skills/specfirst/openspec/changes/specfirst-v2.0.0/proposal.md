# Proposal: SpecFirst v2.0.0 - Session Persistence & Quality Gates

## Summary

Add session state persistence, quality threshold gates, and progress tracking inspired by SpecFlow Bundle. This enables resumable workflows across Claude Code sessions and enforces quality standards before phase transitions.

## Impact

- **Scope:** Enhancement to existing `_SPECFIRST` skill + new pai-tooling framework
- **Breaking changes:** No (additive features)
- **Files affected:**
  - `~/.claude/pai-tooling/` (new - TDD framework, Layer 2)
  - `~/.claude/pai-tooling/specfirst-test/` (new - SpecFirst-specific tests, Layer 3)
  - `tools/specfirst-state.ts` (new - state management)
  - `SKILL.md` (updated - new commands)
  - `workflows/*.md` (updated - gate integration)
  - `docs/StatePersistence.md` (new - documentation)
  - `docs/QualityGates.md` (new - documentation)

## Rationale

**Problem:** Current SpecFirst is stateless. Each Claude Code session starts fresh with no memory of:
- Which phase we're in (SPECIFY → TEST → IMPLEMENT → ARCHIVE → RELEASE)
- What tasks are complete vs pending
- Quality metrics from previous runs

**Inspiration:** SpecFlow Bundle uses SQLite for context persistence, enabling:
- Resumable workflows across sessions
- Progress dashboards
- Quality threshold enforcement

**Solution:** Add lightweight JSON-based state tracking (simpler than SQLite, fits PAI patterns) with:
1. Session persistence in `.specfirst/state.json`
2. Quality gates with configurable thresholds
3. Progress visibility via `/specfirst-status` command

## Goals

1. **Resume workflows** - Pick up where you left off across sessions
2. **Enforce quality** - Configurable pass threshold (default 80%) before advancing
3. **Track progress** - Clear visibility into current phase and completion status
4. **Stay lightweight** - JSON files, no database dependencies

## Non-Goals

- Full database (SQLite) - Too heavy for PAI's UNIX philosophy
- Web UI dashboard - Terminal-first approach
- Dependency graph analysis - Future consideration (v3)

## Architecture Decision

| Option | Pros | Cons | Decision |
|--------|------|------|----------|
| SQLite | Rich queries, SpecFlow pattern | Heavy dependency, overkill | No |
| JSON files | Simple, portable, readable | Manual querying | **Yes** |
| YAML | Human-friendly | Parsing complexity | No |

**Chosen:** JSON files in `.specfirst/` directory per project.

## Success Criteria

- [ ] `/specfirst-status` shows current phase and progress
- [ ] Workflows resume correctly after session restart
- [ ] Quality gates block advancement when threshold not met
- [ ] State files are human-readable and git-friendly
