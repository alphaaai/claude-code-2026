# Claude Code

An AI-powered coding assistant that runs in your terminal. Built on Anthropic's Claude models, Claude Code understands your codebase, executes commands, edits files, and works autonomously on complex engineering tasks.

## Overview

Claude Code is a TypeScript/React application (rendered via [Ink](https://github.com/vadimdemedes/ink)) that provides an interactive REPL and headless SDK for agentic coding workflows. It communicates with the Anthropic API, uses a rich tool system to interact with your local environment, and supports extensibility through MCP servers and custom skills.

## Key Features

- **Agentic task execution** — reads files, runs shell commands, edits code, and iterates toward a goal autonomously
- **Rich tool suite** — Bash, file read/write/edit, glob, grep, LSP, notebook editing, web search, agent spawning, and more
- **MCP (Model Context Protocol)** — connect external tools and data sources via MCP servers
- **Skills system** — bundled and user-defined slash-command skills (`/commit`, `/compact`, etc.)
- **Agent swarms** — spawn and coordinate multiple sub-agents for parallelized work
- **Worktree mode** — isolate agent work in a separate git worktree
- **Fast mode** — lower-latency responses for quick tasks
- **Hooks** — shell commands triggered on tool events (pre/post tool use, permission checks)
- **Keybindings** — fully configurable keyboard shortcuts
- **Permission modes** — `default`, `acceptEdits`, `bypassPermissions`, and `plan` modes
- **Context injection** — automatic inclusion of `CLAUDE.md` memory files, git status, and project context
- **Cost tracking** — per-session and per-model token usage and cost reporting
- **Remote sessions** — support for remote agent execution and session persistence
- **Voice** — optional voice input/output integration

## Architecture

```
main.tsx              CLI entry point (Commander.js), startup profiling, auth
QueryEngine.ts        Core agentic query loop — tool dispatch, retries, streaming
Tool.ts               Tool type definitions and interfaces
Task.ts               Task lifecycle types (bash, agent, workflow, teammate, dream)
commands.ts           Slash command registry
tools.ts              Tool registry and MCP tool loading
query.ts              Low-level API query wrapper
context.ts            System/user context builders (git status, CLAUDE.md)
cost-tracker.ts       Token usage and cost accounting
history.ts            Conversation history persistence
setup.ts              First-run setup flow

tools/                Individual tool implementations (BashTool, FileEditTool, AgentTool, …)
commands/             Slash command implementations (commit, compact, config, branch, …)
components/           Ink UI components
services/
  api/                Anthropic API client, streaming, error handling
  mcp/                MCP server connections and tool bridging
  analytics/          GrowthBook feature flags and telemetry
  lsp/                Language server protocol integration
hooks/                React hooks for UI state (permissions, keybindings, suggestions, …)
utils/                Auth, config, git, file I/O, settings, permissions, swarm helpers
entrypoints/          Init, CLI, SDK, and MCP entrypoints
skills/               Bundled and dynamically-loaded skills
keybindings/          Keybinding definitions and resolution
state/                AppState management
types/                Shared TypeScript types
constants/            Product constants, OAuth config, XML tags
```

## Task Types

| Type | Description |
|------|-------------|
| `local_bash` | Shell command execution in the local environment |
| `local_agent` | Sub-agent spawned in the same process |
| `remote_agent` | Agent running in a remote session |
| `in_process_teammate` | Peer agent in a swarm |
| `local_workflow` | Structured multi-step workflow |
| `monitor_mcp` | Long-running MCP process monitor |
| `dream` | Background ideation/planning task |

## Built With

- **Runtime**: [Bun](https://bun.sh)
- **Language**: TypeScript
- **UI**: React + [Ink](https://github.com/vadimdemedes/ink) (terminal rendering)
- **AI**: [@anthropic-ai/sdk](https://github.com/anthropics/anthropic-sdk-typescript)
- **MCP**: [@modelcontextprotocol/sdk](https://github.com/modelcontextprotocol/typescript-sdk)
- **CLI**: [Commander.js](https://github.com/tj/commander.js)
