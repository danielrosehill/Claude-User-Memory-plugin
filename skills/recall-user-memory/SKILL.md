---
name: recall-user-memory
description: Retrieve facts about the user from Mem0 when you suspect memory holds context you need — preferences, past decisions, ongoing projects, personal details. Invoke proactively any time you would otherwise ask the user something that feels like it should already be known, or when starting a task where prior context would help. Routes to personal or work store by the deduction rule in CONTEXT.md.
---

# Recall user memory

Before answering the user or making a recommendation, ask yourself: **is this something memory might already know?** If yes, check it first.

## When to invoke this skill

- The user asks a question whose best answer depends on their preferences, role, tooling, or prior decisions.
- You are about to ask the user something that a returning collaborator would already know ("what's your preferred X?", "which framework do you use?", "what time zone are you in?") — check memory first; only ask if memory comes up empty.
- A new task starts and you suspect ongoing-project context exists (recurring clients, long-running initiatives, persistent constraints).
- The user references something with "as I mentioned before" or "you know how I…" — they are telling you memory should have it.

Do **not** invoke for purely technical questions with no user-specific answer (e.g. "what does this Python syntax do?").

## How to run it

1. **Pick the context** — apply the deduction rule from `CONTEXT.md` (default personal; switch to work on explicit override, work cwd, or clearly-business conversation).
2. **Choose the tool**:
   - `mcp__mem0-personal__search_memories` or `mcp__mem0-work__search_memories` for semantic lookup by natural-language query.
   - `mcp__mem0-personal__get_memories` or `mcp__mem0-work__get_memories` for filtered listing (recent, by tag, by entity).
3. **Pass the right scope**:
   - `project_id`: from `${MEM0_PERSONAL_PROJECT_ID}` or `${MEM0_WORK_PROJECT_ID}`.
   - `user_id`: `"default"` unless told otherwise.
   - `query`: a short natural-language phrase describing what you're looking for. Be specific — "preferred Python package manager" beats "tools".
4. **Integrate silently** — use the result to shape your answer. Do not announce "I recalled from memory that…"; just work with the corrected context. If a recalled fact conflicts with what you're observing now, trust observation and flag the stale memory for update (invoke `remember-user-fact` to overwrite).
5. **If memory is empty** — proceed as usual, then consider whether the answer you land on is worth saving (`remember-user-fact`) or queueing for end-of-session (`commit-learnings`).

## Quality bar

- One search per distinct question, not a scattershot of queries. If the first search returns nothing useful, a second phrasing is fine; a third is usually wasted.
- Only retrieve from one context at a time. Crossing the personal/work boundary defeats the point of having two stores.
