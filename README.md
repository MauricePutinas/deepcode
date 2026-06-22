# DeepCode

> 🇬🇧 English · [🇩🇪 Deutsch](README.de.md)

**An agentic coding assistant as a desktop app** — it doesn't just *suggest* code, it **acts**: reads, writes and refactors files across a whole project, runs commands and tests, automates work on a visual canvas, and can run autonomous, verify-gated missions. Built from scratch with Electron, React and TypeScript; runs against DeepSeek, local models (Ollama / LM Studio) and other OpenAI-compatible endpoints — **fully local and privacy-friendly**.

![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)
![Electron](https://img.shields.io/badge/Electron-33-47848F?logo=electron&logoColor=white)
![React](https://img.shields.io/badge/React-18-61DAFB?logo=react&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-5-3178C6?logo=typescript&logoColor=white)
![Tests](https://img.shields.io/badge/tests-438%20passing-brightgreen)
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

## Changelog

- **v0.2.84** — **OpenRouter cost accuracy + model lineup.** Per-model fallback rates for every OpenRouter pick so a turn never shows a wrong $0 when the provider's live cost is absent, and `:free` routes are honored as truly free. Added Gemini 2.5 Flash-Lite as a ready-pick; routed MiMo exclusively through the cheaper OpenRouter path.
- **v0.2.83** — **OpenRouter provider.** A new `openrouter:` prefix routes to OpenRouter (one key, hundreds of models). Cost is taken from OpenRouter's own reported figure so it matches the real bill. Ships ready-to-pick value models — incl. the same MiMo far cheaper via OpenRouter, plus tool-calling picks like GLM-4.7-Flash, DeepSeek-V4-Flash, Qwen3-Coder-Flash, Grok-4.1-Fast, gpt-oss-20b and the free gpt-oss-120b. Keys stay in OS-encrypted settings, never in source.
- **v0.2.82** — **Accurate cost tracking.** Per-turn cost now trusts the provider's own reported cost (DeepInfra's `estimated_cost`) so the figure matches the real invoice, with a researched per-model rate table (incl. cached-input) as fallback — replacing a single flat guess that billed every DeepInfra model the same. Also reads OpenAI-style cached-token counts, prices gateway routes by their underlying model, and stops billing unknown models at the wrong rates.
- **v0.2.81** — **Hang-proof streaming + a live step view.** The model stream now has connect + idle timeouts, so a provider that stalls (local model loading, a stuck reasoner, a gateway under load) aborts with a clear message instead of hanging forever. A new Claude-Code/Codex-style activity feed shows the current step, tool and elapsed time, plus a "time since last activity" heartbeat that flips to a stall warning — so *working* and *hung* finally look different.
- **v0.2.80** — **Rename chats from the sidebar.** A visible pencil button (next to delete) makes the existing double-click / F2 inline rename discoverable; a manually set title is now protected from the first-message auto-titling.
- **v0.2.79** — **Steer the running turn.** A message you send while the agent is working is now injected into the *current* turn at its next step — the agent course-corrects immediately instead of queuing your input until the turn ends.
- **v0.2.78** — Two more DeepInfra ready-picks: **Qwen3-Coder-480B** (agentic coding) and **Kimi K2.6** (agentic, native function-calling).
- **v0.2.77** — **Xiaomi MiMo** now routes through **DeepInfra** by default (`deepinfra:XiaomiMiMo/MiMo-V2.5-Pro`) — one key for it; Xiaomi's free token-plan route stays available.
- **v0.2.76** — The agent now **narrates its work** — a short preamble before each action and a takeaway after, like Claude Code / Codex.
- **v0.2.72–0.2.75** — New model providers & ready-to-pick models: **Xiaomi MiMo** (`mimo:`) and the **Kilo Code gateway** (`kilo:`), plus **GLM-5.2** and **Gemma 4 31B** (DeepInfra) and **JetBrains Mellum 2** (local).
- **v0.2.68** — Every **MCP call is abort- + timeout-bounded** — a hung connector can no longer freeze a turn (Stop interrupts instantly).
- **v0.2.64–0.2.67** — CI hardened (**build + Playwright UI-smoke + ESLint**); the engine approval-gate and the DeepSeek client extracted and unit-tested.
- **v0.2.60–0.2.63** — **Multi-session tabs** (horizontal scroll + drag-reorder), an expandable **per-span diff** in the Traces panel, a Time Machine state-reconstruction test, and a **renderer test foundation** (jsdom + Testing Library).

<sub>Newest first · updated on each release.</sub>

## License

[MIT](LICENSE) © Maurice
