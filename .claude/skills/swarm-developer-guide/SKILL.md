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
3. Implement EVERYTHING in Scope
4. Build: `make build`
5. Test: `make test`
6. Self-review (see checklist below)
7. Create PR
8. Report status

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
- [ ] **Discovery done** — I read `docs/DECISIONS.md` and used existing names, types, patterns
- [ ] **Naming consistency** — names match the shared registry
- [ ] **Frontend↔Backend contract** — if frontend calls an endpoint, it exists and accepts the exact format
- [ ] **Registry updated** — all new endpoints, types, conventions recorded in `docs/DECISIONS.md`
- [ ] `make build` passes
- [ ] `make test` passes

## Handover (after coding, before PR)

Record what you created for the next phase. Update `docs/DECISIONS.md` — add rows to the appropriate category:

| If you created... | Add row to section | Format |
|---|---|---|
| API endpoint | **Endpoints** | `METHOD /path → auth \| request → response \| note` |
| Constant/convention | **Naming conventions** | `area \| canonical \| aliases \| where defined` |
| Route registration | **Auth boundary** | Add to before/after list |
| Shared type | **Shared types** | `Type \| definition \| where \| consumers` |
| Reusable pattern | **Design patterns** | `pattern \| description \| impact` |
| Bug lesson | **Known pitfalls** | `#N \| what happened \| prevention` |

**You MUST record if you:**
- Created or changed an API endpoint
- Defined constants, enums, or naming conventions
- Changed auth/middleware boundary
- Created a pattern others should follow
- Discovered a constraint the plan doesn't mention

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

## DECISIONS.md changes
<what you added to the shared registry>

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
4. Read `docs/DECISIONS.md` — the fix may involve cross-phase consistency issues
5. Fix ALL issues — do not leave any unresolved
6. Run `make build` and `make test`
7. Update `docs/DECISIONS.md` if the fix changes any shared interface
8. For each fix, commit with message referencing the issue: `fix: <description> (closes #<issue-number>)`
9. Push and create PR: `git push -u origin HEAD && gh pr create --base <base-branch> --title "Phase N: <name> (fix attempt M/3)" --body "Fixes #<issue-number>"`
10. Hand-over: `python3 ~/.claude/skills/swarm/scripts/swarm.py update <plan-file> --phase <N> --status FOR_REVIEW --pr "<#N>"`
