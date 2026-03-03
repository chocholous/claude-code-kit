# Project Decision Registry

Shared memory between phases. Each DevAgent **reads** at the start and **updates** at the end of their phase.

Organized by category — agent looks up "does endpoint X exist?" not "what happened in phase N?"

---

## Endpoints

Each row = one endpoint. Format: `METHOD path → auth | request → response | note`

```
(empty — add rows as you create endpoints)
```

**Rule:** New endpoint → add a row here. `no-auth` = before auth middleware.

---

## Naming conventions

Format: `area | canonical form | aliases | where defined`

```
(empty — add rows as you establish naming conventions)
```

**Rule:** New constant, enum, or naming convention → add a row. If something already has a name — use it.

---

## Auth boundary

```
BEFORE auth middleware:
  (list routes here)

auth middleware  ← BOUNDARY

AFTER auth middleware:
  (list routes here)
```

**Rule:** New route → add to the appropriate section.

---

## Shared types

Format: `type | definition | where | who consumes`

```
(empty — add rows as you create shared types)
```

**Rule:** New shared type → record who produces and who consumes it.

---

## Design patterns

Format: `pattern | description | impact on other code`

```
(empty — add rows as you establish patterns)
```

---

## Known pitfalls

Lessons from bugs, so they don't repeat.

Format: `#N | what happened | how to prevent`

```
(empty — add rows as you discover pitfalls)
```
