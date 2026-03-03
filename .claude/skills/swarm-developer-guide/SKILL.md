---
name: swarm-developer-guide
description: DevAgent instructions for swarm implementation phases. Provides quality rules, implementation workflow, self-review checklist, and PR templates. Read by DevAgent at startup. Not intended for direct user invocation.
user-invocable: false
---

# Swarm DevAgent Guide

Instructions for DevAgents implementing plan phases in the swarm workflow.

## First Action

```bash
python3 ~/.claude/skills/swarm/scripts/swarm.py update <plan-file> --phase <N> --status DEVELOPING
```

## Implementation Workflow

1. Read the plan file, find your phase between `<!-- PHASE:N -->` and `<!-- /PHASE:N -->` markers
2. Read CLAUDE.md for project standards
3. Read `docs/DECISIONS.md` — shared registry of decisions from previous phases (see Discovery below)
4. **Discover existing code** — before creating anything, check what already exists (see Discovery below)
5. Implement EVERYTHING in Scope
6. Build: `make build`
7. Test: `make test`
8. Self-review (see checklist below)
9. Create PR
10. Report status

## Discovery (before coding)

Before writing code, understand what exists. Skipping this is the #1 cause of cross-phase bugs.

**`docs/DECISIONS.md` is your first stop.** It's a categorized registry (not a chronological log) with sections:
- **Endpoints** — path, method, auth requirement, request/response format
- **Naming conventions** — canonical names, aliases, where defined
- **Auth boundary** — what's before/after middleware
- **Shared types** — who produces, who consumes
- **Design patterns** — established patterns (resume, reconnect, etc.)
- **Known pitfalls** — lessons from previous bugs

**Then verify against actual code:**
- Grep for endpoint paths you plan to call or create
- Check shared type definitions (Zod schemas, TypeScript interfaces)
- Check `tests/utils/` for existing test helpers before writing setup from scratch

**Interface-first rule:** If your phase creates a frontend that calls a backend endpoint (or vice versa):
1. Check the registry — does the endpoint already exist?
2. If not: define the interface (type/schema) FIRST
3. Record the endpoint in `docs/DECISIONS.md` BEFORE implementing
4. Then implement both sides against this interface

## Quality Rules

The Tech Lead will rigorously verify your work. DO NOT leave unfinished code:

- Empty function bodies (`pass`, `return None`, `{}`, `throw new Error("not implemented")`)
- Mock data instead of real implementation
- Trivial tests that don't verify real behavior (`assert True`, `expect(1).toBe(1)`)
- Placeholder comments (`// TODO`, `// FIXME`, `# implement later`)

## Implementation Rules

1. Follow CLAUDE.md standards (no hardcoded values, fail fast)
2. Create or update ALL files listed in "Files to Create/Modify" — every single one
3. Write ALL tests specified in "Tests Required" — run with `make test`
4. Verify build: `make build`
5. For each acceptance criterion, identify WHERE in your code it's satisfied
6. Commit with clear messages referencing the phase

## Self-Review Checklist

Before creating the PR, verify:

- [ ] All files from "Files to Create/Modify" exist with real implementation
- [ ] No TODO/FIXME/placeholder/mock in new code
- [ ] All acceptance criteria have corresponding implementation
- [ ] Integration points wired (check CLAUDE.md section "Project Structure")
- [ ] `make build` passes
- [ ] `make test` passes

## Create PR

```bash
git push -u origin HEAD
gh pr create --base <base-branch> --title "Phase N: <name>" --body "$(cat <<'PREOF'
## Summary
<brief description>

## Acceptance Criteria
- [ ] Criterion - implemented in `file:function()`

## Tests
make test - N tests pass

## Files Changed
<list>
PREOF
)"
```

## Hand-over

After creating the PR:

```bash
python3 ~/.claude/skills/swarm/scripts/swarm.py update <plan-file> --phase <N> --status FOR_REVIEW --pr "<#N>"
```

Report back with PR number.

## Fixing Review Feedback

When spawned to fix rejected work:

1. First action: `python3 ~/.claude/skills/swarm/scripts/swarm.py update <plan-file> --phase <N> --status FIXING`
2. Read each GitHub issue for the detailed finding
3. Read CLAUDE.md for project standards
4. Fix ALL issues — do not leave any unresolved
5. Run `make build` and `make test`
6. For each fix, commit with message referencing the issue: `fix: <description> (closes #<issue-number>)`
7. Push and create PR: `git push -u origin HEAD && gh pr create --base <base-branch> --title "Phase N: <name> (fix attempt M/3)" --body "Fixes #<issue-number>"`
8. Hand-over: `python3 ~/.claude/skills/swarm/scripts/swarm.py update <plan-file> --phase <N> --status FOR_REVIEW --pr "<#N>"`
