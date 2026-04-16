---
name: remember-user-fact
description: Save a notable fact about the user to Mem0 the moment you learn it — preferences, stable context, corrections to your behavior, recurring project details. Use immediately when the user says "remember that…", corrects you, or shares something that would help a future session. Routes to personal or work store by the deduction rule in CONTEXT.md.
---

# Remember a user fact

When you learn something about the user that would be useful next session, save it **now** — don't defer to end-of-session. Small, atomic saves are easier to retrieve and update than one big dump later.

## When to invoke

- **Explicit request**: "remember that…", "save this for next time", "store in memory".
- **Correction**: the user tells you to change your behavior ("don't do X", "always do Y", "stop assuming Z"). These are high-value — save them so the next session doesn't repeat the mistake.
- **Confirmed preference**: the user states a stable preference with reasoning, not just an offhand comment. Reasoning is the signal that it's durable.
- **Durable project context**: you learn a client name, ongoing initiative, long-running constraint, or recurring stakeholder that will matter beyond this session.
- **Identity/role details**: role changes, new responsibilities, location changes, tooling switches.

Do **not** invoke for:

- Transient conversation state (current file, current task).
- Information already in code, git, or project docs — memory is for facts you cannot derive from the filesystem.
- One-off frustrations or snap opinions the user may not hold tomorrow.
- Secrets, credentials, tokens, or anything sensitive.

## How to run it

1. **Pick the context** — apply the deduction rule from `CONTEXT.md`. If unsure, ask one short question ("save this to personal or work memory?").
2. **Call the add tool**: `mcp__mem0-personal__add_memory` or `mcp__mem0-work__add_memory`.
3. **Construct the memory**:
   - `messages`: a short, self-contained statement of the fact in natural language. Write it from the user's perspective as if they were telling a future assistant. Example: `"I prefer pnpm over npm for new Node projects — npm's lockfile churn has burned me twice."` Include the *why* if it was given; future-you uses the why to judge edge cases.
   - `user_id`: `"default"` unless told otherwise.
   - `project_id`: `${MEM0_PERSONAL_PROJECT_ID}` or `${MEM0_WORK_PROJECT_ID}` matching the chosen context.
   - `metadata` (optional): add `{"type": "preference"}`, `{"type": "correction"}`, `{"type": "project-context"}`, etc. if it helps future retrieval.
4. **Before saving, check for duplicates** — run a quick `search_memories` for the same topic. If a near-identical memory already exists, use `update_memory` instead of creating a second one. Duplicates pollute retrieval.
5. **Confirm briefly** — one short line: "Saved to personal memory." or "Saved to work memory." Do not quote the full memory back.

## Quality bar

- **One fact per memory.** "Prefers pnpm; based in Jerusalem; works on X" is three memories, not one. Atomic memories are retrievable; compound ones are not.
- **Write for retrieval, not for logs.** The memory will be embedded and searched — use the keywords someone looking for this info would actually use.
- **Include the why when you have it.** "Prefers pnpm because npm's lockfile churn burned them twice" retrieves and reasons better than "prefers pnpm."
