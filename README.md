# Claude Code

A complete development environment that happens to run in your terminal. Not an API wrapper with delusions of grandeur — a genuinely sophisticated system with a custom React reconciler, a Yoga flexbox layout engine, game-engine blitting optimizations, and a virtual pet. Yes, really.

## What It Does

Talks to Claude, executes tools, edits your code, runs shell commands, spawns sub-agents, and generally handles the parts of programming you'd rather not do. Interactive REPL or headless SDK — your call.

## The Stack

- **Runtime:** Bun (fast)
- **UI:** React 18 + custom terminal reconciler → Yoga layout → ANSI sequences → TTY
- **AI:** Anthropic API with streaming, retries, and token-budget management
- **CLI:** Commander.js
- **Validation:** Zod everywhere

## Architecture at a Glance

```
main.tsx          — Entry point. Parallelizes MDM policy + OAuth reads before imports load.
QueryEngine.ts    — The brain. Resilient state machine: context → API → tools → repeat.
Tool.ts           — Unified tool interface (Zod schemas, permissions, 4-tier UI rendering)
Task.ts           — Task lifecycle: bash / agent / workflow / teammate / dream
commands.ts       — 100+ slash commands, lazily loaded and memoized
query.ts          — Low-level API wrapper (streaming, fallback, error recovery)
context.ts        — Builds system prompt from git status, CLAUDE.md, memory files
cost-tracker.ts   — Per-model token accounting down to fractions of a cent

tools/            — 60+ tool implementations
commands/         — Slash command implementations
components/       — 146 Ink UI components
services/         — API, MCP, analytics, LSP, OAuth, compaction
hooks/            — React hooks for permissions, keybindings, suggestions
utils/            — 330+ utility files (auth, config, git, swarm, settings…)
skills/           — Bundled + user-defined prompt skills
```

## Tools (60+)

Every tool has a Zod schema, permission-aware descriptions, and concurrency safety declarations. Highlights:

| Tool | What it does |
|------|-------------|
| `BashTool` | Shell execution, 30K char limit, disk-spills for larger output |
| `FileEditTool` | Surgical string replacement with git-aware diffing |
| `AgentTool` | Spawns subagents with worktree or remote isolation |
| `LSPTool` | Nine language intelligence ops via Language Server Protocol |
| `WebSearchTool` | Server-side search, max 8 per invocation, domain-filterable |
| `ToolSearchTool` | Discovers ~18 deferred tools on demand to keep prompts under 200K tokens |

## Permission Modes

| Mode | Behavior |
|------|----------|
| `default` | Asks before destructive operations |
| `plan` | Read-only + question-asking only |
| `acceptEdits` | Auto-approves file edits, asks for shell commands |
| `bypassPermissions` | Full access. You've been warned. |
| `dontAsk` | Auto-denies unsafe commands |

Rules support glob patterns, ML classifiers for edge cases, and a priority cascade across MDM → remote settings → user → project → global → defaults.

## Context & Memory

- **CLAUDE.md files** — project/user-level instruction files injected into every conversation
- **File-based memory** — persistent across sessions at `~/.claude/projects/<project>/memory/`
- **Four memory types:** user profile, feedback/corrections, project state, external references
- **Relevance selection** — Sonnet picks the 5 most relevant files from up to 200 candidates

## When the Context Window Fills Up

Multi-strategy compaction kicks in automatically:
1. Strip images from old messages
2. Summarize older rounds with a compaction model
3. Microcompact tool results by age/size
4. Snip history at a boundary
5. Circuit breaker after 3 consecutive failures (no thrashing)

## Multi-Agent

- Spawn child agents with shared filesystem, worktree, or remote isolation
- File-based IPC for cross-session durability (no sockets)
- Coordinator mode: research (parallel) → synthesize → implement (workers) → verify
- Worktree isolation: each agent gets a fresh git branch for safe destructive work

## Execution Modes

One codebase, seven contexts:

`interactive CLI` · `headless/CI` · `MCP server` · `bridge to claude.ai` · `remote/teleport` · `local agent` · `coordinator`

## Extensibility

- **Skills** — Markdown prompt templates with frontmatter, discoverable from project, user, bundled, plugin, or MCP sources
- **Plugins** — Bundle skills + hooks + MCP servers with configurable variables
- **MCP** — stdio, HTTP/WebSocket, embedded SDK, or Claude.ai proxy; 8 configuration scopes
- **Hooks** — Shell commands, LLM evaluations, HTTP POSTs, or full agent loops triggered on tool events

## Hidden Depths

**Rendering:** Double-buffered screen cells with pointer swapping between frames, three interning pools (chars, styles, hyperlinks) for O(1) comparisons, and blitting to skip unchanged regions — techniques borrowed directly from game engines.

**Startup:** 16-stage init sequence with overlapping async ops. MDM policy and keychain reads fire before module evaluation completes. Profiling sampled at 0.5% for external users.

**Vim mode:** Full modal editing — motions, operators, text objects, visual mode. Not a toy implementation.

**BUDDY:** A Tamagotchi-style companion. 18 species, 5 rarity tiers, ASCII sprite animations at 500ms, seeded deterministically from your account ID. Technically optional. Spiritually essential.

**ULTRAPLAN:** Farms complex planning to a remote Claude Code instance for up to 30 minutes. Tolerates 5 consecutive network failures across 600+ API calls. Keyword detection avoids false positives inside code identifiers and delimiters.

## Error Recovery

Users almost never see raw API errors. The system exhausts every option first:

- **Overload (529/429):** Retry with backoff, fall back to alternate model
- **Prompt too long:** Collapse → compact → surface only if both fail
- **Max output tokens:** Escalate budget up to 3 times per turn
- **Streaming failure:** Retry as non-streaming request
- **Auth errors (401/403):** Surface immediately — no point retrying

## Key Engineering Choices Worth Stealing

1. **Defer everything** — tools, schemas, commands, modules; load on first use
2. **Intern for performance** — chars, styles, hyperlinks by integer ID
3. **Fail closed** — permissions default to "ask"; safe defaults everywhere
4. **File-based IPC** — disk beats sockets for cross-session multi-agent state
5. **Sort tools alphabetically** — keeps prompt cache stable across requests
6. **Feature flags as dead-code elimination** — Bun's `feature()` removes unused code paths at build time
