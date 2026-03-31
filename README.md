# Claude Code

I once asked Claude Code to "clean up the auth module." It read 14 files, refactored three of them, ran the tests, noticed a race condition I hadn't mentioned, fixed that too, and committed everything with a sensible message. I got up to make coffee.

That was a Tuesday. By Friday I'd stopped reviewing most of my own PRs.

---

## What Is This?

Claude Code is Anthropic's AI coding assistant that lives in your terminal. It's not a chatbot with a `vim` plugin bolted on. It's a full development environment — custom React reconciler, Yoga flexbox layout engine, 60+ tools, a multi-agent swarm architecture, a persistent memory system, and (inexplicably, wonderfully) a virtual pet companion — all shipping as a CLI.

The people who built this are not normal.

## How It Works

You type something. Claude thinks. Then it reads your files, runs your tests, edits your code, spawns sub-agents to parallelize the boring parts, tracks every token spent, and reports back. The whole loop is a resilient state machine that handles streaming failures, context overflow, model overload, and auth errors — mostly without you ever knowing something went wrong.

```
You → QueryEngine → Claude API → Tools → Your codebase
              ↑                              ↓
         Permission layer ←── Results ←── Execution
```

## The Tools (60+)

Every tool has a Zod schema, permission checks, and concurrency safety markers. The interesting ones:

- **BashTool** — runs shell commands. Spills to disk when output exceeds 30K chars. Handles the fact that humans write very long `find` commands.
- **FileEditTool** — surgical string replacement with git-aware diffing. Not "rewrite the whole file."
- **AgentTool** — spawns a child Claude in its own git worktree. For when the task is too big for one agent and too important to do yourself.
- **LSPTool** — nine language intelligence operations. Claude reads your code *and* understands its types.
- **ToolSearchTool** — discovers ~18 hidden tools on demand, keeping the base prompt under 200K tokens. Elastic tool discovery. Very clever.

## Permission Modes

Because "give the AI full disk access" is a spectrum, not a binary:

| Mode | Vibe |
|------|------|
| `default` | Asks before anything destructive |
| `plan` | Read-only. Good for paranoid Monday mornings. |
| `acceptEdits` | Trust file edits, eyeball shell commands |
| `bypassPermissions` | You're a responsible adult. Allegedly. |
| `dontAsk` | Auto-denies unsafe ops. For the cautious automators. |

## Memory That Persists

Claude Code remembers things across sessions. Not in a "it kind of remembers" way — in a file-based memory system with four distinct types (user profile, feedback, project state, external references), a 200-line index loaded into every conversation, and a Sonnet-powered relevance selector that picks the 5 most useful files from up to 200 candidates.

You correct it once. It doesn't make the same mistake again.

## When the Context Window Fills Up

It doesn't just crash. It runs a 5-stage compaction playbook:
1. Strip images from old messages
2. Summarize old rounds with a compaction model
3. Microcompact oversized tool results
4. Truncate history at a snip boundary
5. Circuit-break after 3 consecutive failures so it doesn't thrash

The goal: you never see a "context too long" error. You just... keep going.

## Multi-Agent (For the Really Ambitious)

Coordinator mode turns Claude Code into an orchestrator directing parallel workers through a four-phase workflow: research → synthesize → implement → verify. Workers communicate via file-based IPC (disk, not sockets — more durable across sessions). Each agent can get its own isolated git worktree for safe destructive experimentation.

I've run 6 agents on the same codebase simultaneously. It felt like managing a very fast, very literal intern army.

## The Rendering Engine

This is where it gets unhinged (in a good way).

The terminal UI uses a custom React reconciler feeding into the Yoga flexbox engine, writing to a double-buffered screen cell array with three interning pools (chars, styles, hyperlinks) for O(1) comparisons, and blitting unchanged regions between frames. **These are techniques from game engines.** Applied to a terminal. Because someone on the team apparently thought: why not?

The result is a terminal app that renders like a web browser.

## The Hidden Features Nobody Talks About

**Vim mode.** Full modal editing. Real motions, operators, text objects. `ciw`, `dap`, `gg`, `G`. Not decorative.

**ULTRAPLAN.** Send a hard problem to a remote Claude Code instance for up to 30 minutes of exploration. Tolerates 5 consecutive network failures mid-session. Someone stress-tested this.

**BUDDY.** A Tamagotchi-style companion. 18 species, 5 rarity tiers, ASCII animations at 500ms, seeded from your account ID via a Mulberry32 PRNG. It reacts to your conversation. I don't know why this exists. I'm glad it does.

## Error Recovery (The Unsexy Superpower)

The mark of production software is what happens when things go wrong. Claude Code:

- **Overload errors** → retry with backoff, fall back to alternate model, strip thinking blocks
- **Context too long** → run compaction pipeline before surfacing the error
- **Token limit hit** → escalate budget up to 3 times per turn
- **Streaming fails mid-response** → retry as a non-streaming request
- **Auth fails** → surface immediately. Retrying a 401 is not optimism, it's denial.

Users almost never see raw API errors. The system fails gracefully enough that you often don't realize it recovered.

## The Engineering Lessons (If You're Building Something Similar)

1. **Defer everything.** Load tools, schemas, commands on first use. Your startup time will thank you.
2. **Intern for performance.** Deduplicating chars and styles by integer ID makes terminal diffing fast.
3. **Fail closed.** Permissions default to "ask." Safe is the default, not an option.
4. **File-based IPC beats sockets** for multi-agent state that needs to survive process restarts.
5. **Sort your tools alphabetically.** It keeps the prompt cache stable. Boring optimization. Enormous savings.
6. **Feature flags as dead-code elimination.** Bun's `feature()` removes unused paths at build time. Ship less, run faster.

---

Built with TypeScript, Bun, React, and an apparent disregard for the conventional limits of what a terminal app should be.

[Deep technical analysis by Sathwick](https://sathwick.xyz/blog/claude-code.html) — where most of the above is sourced from.
