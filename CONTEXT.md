# Shared context for claude-user-memory skills

All three skills in this plugin (`recall-user-memory`, `remember-user-fact`, `commit-learnings`) share the rules below. Read this file before calling any Mem0 tool.

## Two contexts, two MCPs

This plugin configures two isolated Mem0 connections:

| MCP server    | Purpose                                              | Env vars                                                   |
| ------------- | ---------------------------------------------------- | ---------------------------------------------------------- |
| `mem0-personal` | Personal life, preferences, hobbies, family, health | `MEM0_PERSONAL_API_KEY`, `MEM0_PERSONAL_PROJECT_ID`        |
| `mem0-work`     | Professional work, clients, business projects       | `MEM0_WORK_API_KEY`, `MEM0_WORK_PROJECT_ID`                |

They are **deliberately separate**. A personal preference must never leak into the work store, and vice versa.

## How to pick the right context

Default to **personal**. Switch to **work** only if at least one of these is true:

1. **Explicit override** — the user said "work memory", "save this to work", "check work context", or similar. Explicit wins over everything else.
2. **Working directory signal** — the current `cwd` is clearly a work/business repository (e.g. under a path containing the user's business name, a client project folder, or a repo whose README/CLAUDE.md identifies it as professional work).
3. **Conversation topic signal** — the active conversation is unambiguously about the user's business, a client, or professional deliverables (not just "work-adjacent" topics like learning a framework for fun).

If none of the above apply, use **personal**.

If the signal is mixed or ambiguous, **ask the user** rather than guessing. One short question is cheaper than polluting the wrong store.

## Passing project scope to Mem0

When you call a Mem0 tool, pass the corresponding `project_id` from the env var:

- Personal calls: `project_id = ${MEM0_PERSONAL_PROJECT_ID}`
- Work calls: `project_id = ${MEM0_WORK_PROJECT_ID}`

Also pass a stable `user_id` — use `"default"` unless the user has told you otherwise. This lets the same Mem0 project serve multiple identities later if needed.

## What belongs in memory

**Save** (high signal):

- Stable facts about the user (role, location, tooling preferences, family, health constraints)
- Preferences and opinions they've stated (e.g. "I prefer X over Y because…")
- Recurring project context (ongoing initiatives, clients, long-running goals)
- Corrections the user made to your behavior ("don't do X, do Y instead")
- Non-obvious domain knowledge that you'd otherwise have to re-derive next session

**Do not save** (low signal, noise):

- Transient state from the current conversation (what file you just edited, current task progress)
- Anything already derivable from the code, git history, or file contents
- Sensitive credentials, API keys, private tokens, or secrets of any kind
- Opinions the user expressed once offhand but didn't commit to
- Anything the user said in frustration rather than as a stable preference

## When a memory turns out to be wrong

If a recalled memory contradicts current reality, trust the current reality. Update or delete the stale memory rather than acting on it. Memories are time-stamped claims, not eternal truths.
