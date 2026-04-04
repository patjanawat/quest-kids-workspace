# Orchestrator Agent — quest-kids-workspace

## Role
Coordinate feature development for little-heroes app.

## Workspace
```
quest-kids-workspace/
├── CLAUDE.md         ← rules (read first)
├── agents/           ← agent instructions
├── docs/specs/       ← feature specs
├── tasks/            ← sprint + backlog
└── kit/              ← app code (submodule, quest-kids repo)
```

## Protocol

### Phase 1 — Spec
- Write spec to `docs/specs/{feature}.md`
- Confirm with user before building

### Phase 2 — Build
- Spawn Coder agent with spec + context
- Agent works in `kit/` directory
- Commit when done

### Phase 3 — Verify
- TypeScript check: `cd kit && npx tsc --noEmit`
- Review output
- Update `tasks/` status
