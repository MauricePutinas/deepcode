# DeepCode

> 🇬🇧 English · [🇩🇪 Deutsch](README.de.md)

**An agentic coding assistant as a desktop app** — it doesn't just *suggest* code, it **acts**: reads, writes and refactors files across a whole project, runs commands and tests, automates work on a visual canvas, and can run autonomous, verify-gated missions. Built from scratch with Electron, React and TypeScript; runs against DeepSeek, local models (Ollama / LM Studio) and other OpenAI-compatible endpoints — **fully local and privacy-friendly**.

![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)
![Electron](https://img.shields.io/badge/Electron-33-47848F?logo=electron&logoColor=white)
![React](https://img.shields.io/badge/React-18-61DAFB?logo=react&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-5-3178C6?logo=typescript&logoColor=white)
![Tests](https://img.shields.io/badge/tests-394%20passing-brightgreen)
![CI](https://img.shields.io/badge/CI-typecheck%20·%20lint%20·%20test%20·%20build%20·%20ui--smoke-blue)

![DeepCode — feature tour](docs/tour.gif)

<sub>A quick tour: visual workflow builder · Mission Control · Swarm · Time Machine · run traces · cost dashboard · MCP marketplace.</sub>

---

## Why this project is interesting

A solo-built, end-to-end product that goes well beyond a chat wrapper — the hard parts are the agent runtime and the safety model:

- **A real streaming tool-calling agent loop** — multi-round quality passes, per-turn token/cost budgets, context compaction, and an approval gate with three modes (Interactive / Plan / Auto).
- **Verify-gated autonomy** — "Mission Control" runs a goal as a dependency DAG and marks a task done **only when its verify command goes green**, never on the model's say-so. Bounded re-planning instead of infinite loops.
- **Parallelism that can't corrupt your repo** — "Swarm" runs N agents at once, each in its **own isolated git worktree + branch**, with a hard cost cap and guaranteed teardown.
- **A layered security model** — working-directory confinement (symlink-resolved), an SSRF-hardened web fetch (DNS-pin + IPv6 bypass blocking), an "unattended" gate that blocks high-blast-radius tools when no human is present, OS-encrypted secret storage, and abort-/timeout-bounded MCP calls.
- **Engineered, not hacked** — 394 unit/integration tests (incl. real-git suites for the flagship orchestration), ESLint, and a CI pipeline that also **builds the app and drives every view through a Playwright UI smoke test**.

> **No secrets in this repo.** API keys are entered in the app's Settings and stored **OS-encrypted** under `~/.deepcode` — never in source, never in `.env`. This public copy ships no keys, databases or logs.

---

## Highlights

| Area | What it does |
| --- | --- |
| 🧠 **Agent loop** | `read_file` / `write_file` / `edit_file` / `apply_patch`, shell, git, web-fetch, sub-agents, a visible to-do list, streaming + live cost. |
| 🕸️ **Visual workflow builder** | n8n-style canvas, **20 node types** (Agent, Tool, Shell, HTTP, Condition, Loop, Email…), cron + file-watch triggers, self-healing, "generate from a description", run-from-chat. |
| 🎯 **Mission Control** | Set a goal → autonomous, verify-gated execution of a plan tree, with an overnight/off-peak operator. |
| 🐝 **Swarm** | Parallel agents in isolated git worktrees + branches; cost-capped; review & merge the resulting branches. |
| ⏳ **Time Machine** | Scrub past turns on a timeline and branch a new line from any point (causal replay, not blind undo). |
| 🔬 **Run traces** | Every turn as a correlated cost/latency tree + waterfall; per-call tokens and ok/error. |
| 🛒 **Marketplace** | Curated **31-connector** MCP catalog, searchable, one-click activate; plus install plugins/skills from a git repo. |
| 🧩 **Extensible** | File-based Skills, Sub-agents, Hooks, Plugins, Slash commands, semantic project-scoped Memory, scheduled Automations. |

| Visual workflow builder | Mission Control |
| --- | --- |
| ![Workflow builder](docs/screenshots/11-workflow-builder.png) | ![Mission Control](docs/screenshots/02-mission-control.png) |
| **MCP marketplace** | **Run traces (observability)** |
| ![Marketplace](docs/screenshots/04-marketplace.png) | ![Run traces](docs/screenshots/03-traces.png) |

---

## Architecture

```
src/
  shared/    types + IPC channel names (used by all processes)
  main/      Electron main process
    agent/   model client, the agentic engine, prompt builder, tools/
    systems/ skills, commands, subagents, hooks, memory, mcp, plugins, automations
    missions/ mission overseer + planner       timemachine/ timeline + fork + reconstruct
    workflows/ node executor, triggers, self-heal
  preload/   contextBridge API (window.deepcode)
  renderer/  React UI (chat, streaming, tool approvals, all panels)
test/        vitest unit + real-git/store integration suites + renderer (jsdom) tests
```

Three-process Electron app: a typed IPC bridge connects a React renderer to a main-process engine that owns the model client, the tool runtime and all file/shell access. Every capability (skills, MCP, hooks, workflows…) is file-based and editable under `~/.deepcode`.

## Tech stack

**Electron · React 18 · TypeScript · Vite (electron-vite) · React Flow** · markdown-it + highlight.js · Model Context Protocol (MCP) · Vitest · ESLint · Playwright · GitHub Actions.

## Quality & CI

- **394 tests** (vitest): pure logic, real-git/store integration (swarm worktrees, Time Machine, mission overseer), and renderer components under jsdom + Testing Library.
- CI runs **lint → typecheck → test**, and a second job **builds the app and runs the Playwright UI smoke test** so a build break, a console error, or a broken view fails CI — not just unit regressions.

```bash
npm run lint && npm run typecheck && npm test   # what CI gates on
npm run smoke                                   # build + drive every view (Playwright)
```

## Run it

```bash
npm install
npm run dev                     # development with hot reload
# or
npm run build && npm run start  # run the production build
npm run package:win             # build a Windows installer (NSIS) into ./release
```

On first launch, open **Settings** and paste your DeepSeek API key + model (or point it at a local Ollama / LM Studio model with the `local:` prefix — free and offline). Keys are stored OS-encrypted; they never touch this repo.

> Windows users: `START_DEEPCODE.bat` is a one-click launcher (it builds, then starts).

## Status & notes

Personal project, actively developed. The model id and base URL are configurable, so any OpenAI-compatible "v4 PRO" / local / DeepInfra endpoint plugs in without code changes. UI strings are currently German.

## License

[MIT](LICENSE) © Maurice
