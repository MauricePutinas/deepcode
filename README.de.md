# DeepCode

> [🇬🇧 English](README.md) · 🇩🇪 Deutsch

**Ein agentischer Coding-Assistent als Desktop-App** — der nicht nur Code *vorschlägt*, sondern **handelt**: liest, schreibt und refactored Dateien über ein ganzes Projekt, führt Befehle und Tests aus, automatisiert Arbeit auf einer visuellen Canvas und kann autonome, verifizierte Missionen abarbeiten. Von Grund auf gebaut mit Electron, React und TypeScript; läuft gegen DeepSeek, lokale Modelle (Ollama / LM Studio) und andere OpenAI-kompatible Endpunkte — **komplett lokal und datenschutzfreundlich**.

![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)
![Electron](https://img.shields.io/badge/Electron-33-47848F?logo=electron&logoColor=white)
![React](https://img.shields.io/badge/React-18-61DAFB?logo=react&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-5-3178C6?logo=typescript&logoColor=white)
![Tests](https://img.shields.io/badge/tests-394%20passing-brightgreen)
![CI](https://img.shields.io/badge/CI-typecheck%20·%20lint%20·%20test%20·%20build%20·%20ui--smoke-blue)

![Visueller Workflow-Builder](docs/screenshots/11-workflow-builder.png)

---

## Warum dieses Projekt spannend ist

Ein im Alleingang gebautes End-to-End-Produkt, das weit über einen Chat-Wrapper hinausgeht — die schwierigen Teile sind die Agenten-Laufzeit und das Sicherheitsmodell:

- **Echter Streaming-Tool-Calling-Loop** — mehrstufige Qualitäts-Runden, Token-/Kosten-Budget pro Turn, Context-Compaction und ein Freigabe-Gate mit drei Modi (Interaktiv / Plan / Auto).
- **Verifizierte Autonomie** — „Mission Control" arbeitet ein Ziel als Abhängigkeits-DAG ab und markiert eine Aufgabe **erst dann als erledigt, wenn ihr Verify-Befehl grün ist**, nie auf Zuruf der KI. Begrenzte Nachplanung statt Endlosschleifen.
- **Parallelität, die dein Repo nicht zerstört** — „Swarm" lässt N Agenten gleichzeitig laufen, jeder in einem **eigenen isolierten Git-Worktree + Branch**, mit hartem Kosten-Cap und garantiertem Aufräumen.
- **Mehrschichtiges Sicherheitsmodell** — Arbeitsverzeichnis-Confinement (symlink-sicher), SSRF-gehärteter Web-Fetch (DNS-Pinning + IPv6-Bypass-Blocking), ein „Unattended"-Gate, das risikoreiche Tools blockt, wenn kein Mensch anwesend ist, OS-verschlüsselte Secret-Verwaltung und abbruch-/timeout-gesicherte MCP-Aufrufe.
- **Solide gebaut, nicht zusammengehackt** — 394 Unit-/Integrationstests (inkl. Real-Git-Suiten für die Flagship-Orchestrierung), ESLint und eine CI-Pipeline, die die App **baut und jede Ansicht durch einen Playwright-UI-Smoke-Test** treibt.

> **Keine Secrets in diesem Repo.** API-Keys werden in den App-Einstellungen eingegeben und **OS-verschlüsselt** unter `~/.deepcode` gespeichert — nie im Quellcode, nie in `.env`. Diese öffentliche Kopie enthält keine Keys, Datenbanken oder Logs.

---

## Highlights

| Bereich | Was es kann |
| --- | --- |
| 🧠 **Agenten-Loop** | `read_file` / `write_file` / `edit_file` / `apply_patch`, Shell, Git, Web-Fetch, Subagents, sichtbare To-do-Liste, Streaming + Live-Kosten. |
| 🕸️ **Visueller Workflow-Builder** | n8n-artige Canvas, **20 Node-Typen** (Agent, Tool, Shell, HTTP, Bedingung, Schleife, E-Mail …), Cron- + Datei-Trigger, Self-Healing, „aus Beschreibung generieren", Ausführen-aus-Chat. |
| 🎯 **Mission Control** | Ziel setzen → autonome, verifizierte Abarbeitung eines Plan-Baums, mit Übernacht-/Off-Peak-Operator. |
| 🐝 **Swarm** | Parallele Agenten in isolierten Git-Worktrees + Branches; kosten-gedeckelt; Branches prüfen & mergen. |
| ⏳ **Zeitmaschine** | Vergangene Turns auf einer Zeitleiste scrubben und ab jedem Punkt einen neuen Branch abzweigen (kausaler Replay statt blindes Undo). |
| 🔬 **Run-Traces** | Jeder Turn als korrelierter Kosten-/Zeit-Baum + Wasserfall; Tokens pro Aufruf und ok/Fehler. |
| 🛒 **Marketplace** | Kuratierter **31-Connector**-MCP-Katalog, durchsuchbar, 1-Klick-Aktivierung; plus Plugins/Skills aus einem Git-Repo installieren. |
| 🧩 **Erweiterbar** | Datei-basierte Skills, Subagents, Hooks, Plugins, Slash-Commands, semantisches projekt-bezogenes Memory, geplante Automationen. |

| Mission Control | MCP-Marketplace |
| --- | --- |
| ![Mission Control](docs/screenshots/02-mission-control.png) | ![Marketplace](docs/screenshots/04-marketplace.png) |

---

## Architektur

```
src/
  shared/    Typen + IPC-Kanalnamen (von allen Prozessen genutzt)
  main/      Electron-Main-Prozess
    agent/   Modell-Client, agentische Engine, Prompt-Builder, tools/
    systems/ Skills, Commands, Subagents, Hooks, Memory, MCP, Plugins, Automationen
    missions/ Mission-Overseer + Planer        timemachine/ Timeline + Fork + Reconstruct
    workflows/ Node-Executor, Trigger, Self-Heal
  preload/   contextBridge-API (window.deepcode)
  renderer/  React-UI (Chat, Streaming, Tool-Freigaben, alle Panels)
test/        vitest Unit- + Real-Git/Store-Integrationstests + Renderer-Tests (jsdom)
```

Drei-Prozess-Electron-App: eine typisierte IPC-Brücke verbindet einen React-Renderer mit einer Main-Prozess-Engine, die den Modell-Client, die Tool-Laufzeit und allen Datei-/Shell-Zugriff besitzt. Jede Fähigkeit (Skills, MCP, Hooks, Workflows …) ist datei-basiert und unter `~/.deepcode` editierbar.

## Tech-Stack

**Electron · React 18 · TypeScript · Vite (electron-vite) · React Flow** · markdown-it + highlight.js · Model Context Protocol (MCP) · Vitest · ESLint · Playwright · GitHub Actions.

## Qualität & CI

- **394 Tests** (vitest): reine Logik, Real-Git/Store-Integration (Swarm-Worktrees, Zeitmaschine, Mission-Overseer) und Renderer-Komponenten unter jsdom + Testing Library.
- CI führt **Lint → Typecheck → Test** aus und in einem zweiten Job wird **die App gebaut und der Playwright-UI-Smoke-Test** über jede Ansicht getrieben — ein Build-Bruch, ein Konsolenfehler oder eine kaputte View lässt CI fehlschlagen, nicht nur Unit-Regressionen.

```bash
npm run lint && npm run typecheck && npm test   # worauf CI gated
npm run smoke                                   # baut + treibt jede View (Playwright)
```

## Starten

```bash
npm install
npm run dev                     # Entwicklung mit Hot-Reload
# oder
npm run build && npm run start  # den Produktions-Build starten
npm run package:win             # Windows-Installer (NSIS) nach ./release bauen
```

Beim ersten Start in den **Einstellungen** den DeepSeek-API-Key + das Modell eintragen (oder ein lokales Ollama-/LM-Studio-Modell per `local:`-Präfix nutzen — kostenlos und offline). Keys werden OS-verschlüsselt gespeichert und landen nie in diesem Repo.

> Windows: `START_DEEPCODE.bat` ist ein Ein-Klick-Starter (baut, dann startet er).

## Status & Hinweise

Persönliches Projekt, in aktiver Entwicklung. Modell-ID und Base-URL sind konfigurierbar, sodass jeder OpenAI-kompatible „v4 PRO"-/lokale/DeepInfra-Endpunkt ohne Code-Änderung passt. Die UI-Texte sind aktuell auf Deutsch.

## Lizenz

[MIT](LICENSE) © Maurice
