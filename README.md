# claude-user-memory

A Claude Code plugin that gives Claude a **persistent, cross-session memory layer about the user**, backed by [Mem0](https://mem0.ai). Two isolated contexts — personal and work — so Claude can remember your preferences, role, ongoing projects, and corrections across sessions without mixing the two.

This plugin is an opinionated take on a common problem: Claude Code's built-in per-project memory is great for code context, but it forgets everything about *you* the moment a new session starts, and it has no sense of which facts belong to your personal life vs. your professional work. This plugin fixes both.

## What it does

| Skill | When it fires | What it does |
| --- | --- | --- |
| `recall-user-memory` | Claude is about to ask you something a returning collaborator would already know, or needs user-specific context to answer | Semantic search against the correct Mem0 store, then silently integrates the result |
| `remember-user-fact` | You state a preference, correct Claude, or share a durable fact | Atomic save to the correct Mem0 store, with duplicate-check before write |
| `commit-learnings` | End of session, handover, or "save what you learned" | Reviews the session, proposes candidate facts, shows them for approval, then batch-saves |

All three skills share a **deduction rule** for picking the right context: default to personal; switch to work only on explicit override, work-repo cwd, or unambiguously business conversation. See `CONTEXT.md` in the plugin for the full rule.

## How context isolation works

The plugin ships **two MCP server instances** pointed at Mem0's hosted MCP:

- `mem0-personal` — authenticated with your personal-project API key
- `mem0-work` — authenticated with your work-project API key

Two separate transports means crossed wires are architecturally impossible: a `mem0-personal` call physically cannot write to the work store, regardless of what Claude decides. The skills add a second layer of safety on top by reasoning about which context applies.

## Installation

### 1. Install the plugin

```bash
claude plugins install danielrosehill/Claude-User-Memory-plugin
```

### 2. Create Mem0 projects

Sign in at [app.mem0.ai](https://app.mem0.ai) and create two projects — e.g. `personal-memory` and `work-memory`. Grab the API key and project ID for each.

### 3. Set environment variables

Export these in your shell (or add to `~/.bashrc` / `~/.zshrc`):

```bash
export MEM0_PERSONAL_API_KEY="m0-..."
export MEM0_PERSONAL_PROJECT_ID="proj_..."
export MEM0_WORK_API_KEY="m0-..."
export MEM0_WORK_PROJECT_ID="proj_..."
```

If you only want a single context to start with, set just one pair and leave the other unset — the unused MCP server will be inert.

### 4. Restart Claude Code

The plugin's MCP servers come online on the next Claude Code start.

### 5. Authenticate each Mem0 MCP (one-time per install)

Mem0's hosted MCP (`https://mcp.mem0.ai/mcp`) uses **OAuth**, not plain Bearer API-key auth. On first use, only two tools will be exposed per server: `authenticate` and `complete_authentication`.

For each server (`mem0-personal` and `mem0-work`):

1. Ask Claude to run the `authenticate` tool on that MCP. It returns a URL.
2. Open the URL in a browser and approve access.
3. Ask Claude to run `complete_authentication` on the same MCP.

After this, the full memory toolset (`add_memory`, `search_memories`, `get_memories`, `update_memory`, `delete_memory`, etc.) becomes available. The API key env vars are still read by the server as a hint during the OAuth exchange — leave them set.

## Usage

Mostly you don't invoke these skills directly — Claude will reach for them when the situation fits. Things that trigger them naturally:

- "Remember that I prefer X." → `remember-user-fact`
- "What's my preferred X?" → `recall-user-memory`
- "Save what you learned this session." → `commit-learnings`
- Any correction you give Claude ("don't do X, do Y") → `remember-user-fact`
- Any question where Claude would otherwise ask you something it should already know → `recall-user-memory` first

To force a context override, say "save this to **work** memory" or "check my **personal** memory".

## Design notes

- **Why two MCPs instead of one with project-scoped args?** Hard separation at the transport layer. Relying on the model to pass the right `project_id` every single call is exactly the kind of thing that goes wrong at the worst moment. Two transports eliminate the class of bug.
- **Why Mem0 rather than a raw vector DB?** Mem0 handles memory semantics — deduplication, updates, temporal reasoning — that you'd otherwise have to build yourself on top of Pinecone or similar. For "facts about a person" specifically, it's the right abstraction.
- **Why three skills rather than one?** Each skill has a different trigger signature. `recall` fires before answering; `remember` fires in the middle of a message when a fact lands; `commit-learnings` fires at session boundaries. Collapsing them into one skill would make the trigger description incoherent.

## Configuration reference

| Env var | Required? | Purpose |
| --- | --- | --- |
| `MEM0_PERSONAL_API_KEY` | yes | Mem0 API key for the personal-context project |
| `MEM0_PERSONAL_PROJECT_ID` | yes | Mem0 project ID for the personal-context project |
| `MEM0_PERSONAL_ORG_ID` | optional | Mem0 org ID — only set if your personal Mem0 account has an explicit org scope |
| `MEM0_WORK_API_KEY` | yes | Mem0 API key for the work-context project |
| `MEM0_WORK_PROJECT_ID` | yes | Mem0 project ID for the work-context project |
| `MEM0_WORK_ORG_ID` | optional | Mem0 org ID — typical for business/paid-tier accounts |

## License

MIT — see `LICENSE`.
