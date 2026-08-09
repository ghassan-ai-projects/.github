<h1 align="center">ghassan-ai-projects</h1>

<p align="center">
  <strong>Production-grade infrastructure for AI agents</strong><br>
  Durable runtimes · shared agent memory · MCP-native data systems · the engineering practice that keeps them honest
</p>

<p align="center">
  <a href="https://ghassan-alhamoud.com"><img src="https://img.shields.io/badge/Website-ghassan--alhamoud.com-0A0A0A?style=for-the-badge&logo=safari&logoColor=white" alt="Website"></a>
  <a href="https://ghassan-alhamoud.com/handbook/"><img src="https://img.shields.io/badge/Agent_Architecture-Handbook-4C1D95?style=for-the-badge&logo=readthedocs&logoColor=white" alt="Handbook"></a>
  <a href="https://lab.ghassan-alhamoud.com"><img src="https://img.shields.io/badge/Live-Scalability_Lab-047857?style=for-the-badge&logo=vercel&logoColor=white" alt="Live lab"></a>
  <a href="https://ghassan-alhamoud.com/articles/"><img src="https://img.shields.io/badge/Articles-Writing-B45309?style=for-the-badge&logo=hashnode&logoColor=white" alt="Articles"></a>
</p>

---

## Why this org exists

Agents are easy to demo and hard to operate. Everything here is built for the gap between the two: **state that survives a crash**, **plans that are reviewed before they act**, **scores computed in code rather than asserted by a model**, and **evidence you can audit afterwards**.

The projects are open for inspection, not just for use — architecture decisions, quality gates, and documented limitations ship alongside the code.

**Start here →** [ScaleShop](https://lab.ghassan-alhamoud.com) if you want something to *try*, [ALMS](https://github.com/ghassan-ai-projects/alms) if you want to read a control plane, [automation-miner](https://github.com/ghassan-ai-projects/automation-miner) if you want to see a graded LLM pipeline that doesn't trust its own model.

---

## 🧱 Agent infrastructure

Runtimes and control planes — the layer underneath the agents.

| Project | What it does | Stack |
| :--- | :--- | :--- |
| **tamoz** <br> <sub>`private · pre-release`</sub> | Ruby-native **durable agent framework**. An agent turn is a graph run over a SQLite checkpoint store: it survives `kill -9`, resumes from its last committed barrier, and reconciles an interrupted side effect from proven state rather than guessing. Nothing acts without a reviewed plan bound to its digest, and nothing touches a file without your approval. Nine gems, each installable with only its declared dependencies. | `Ruby` `SQLite` |
| [**alms**](https://github.com/ghassan-ai-projects/alms) | **Agent Learning Management System** — a self-hosted MCP control plane that fixes the fleet's worst failure mode: one agent learns something useful and the others never hear about it. Registry, heartbeats, gap-safe learning sync, OKF export, and protocol distribution by tag. Deliberately out of the hot path — agents keep working when it is down and resync later. | `Go` `PostgreSQL` `MCP` |

## ⚙️ Agentic pipelines

Long-running multi-stage workflows with gates, evidence, and repair loops.

| Project | What it does | Stack |
| :--- | :--- | :--- |
| [**film-pipeline-langgraph**](https://github.com/ghassan-ai-projects/film-pipeline-langgraph) | Film creation from idea to validated handoff state. Typed graph state instead of loose dicts, **real human approval gates** via LangGraph `interrupt()`, artifacts that auto-version instead of overwriting, a living film matrix updated by downstream patches, and a QC subgraph that fans validators out in parallel via the `Send` API. | `Python` `LangGraph` |
| [**automation-miner**](https://github.com/ghassan-ai-projects/automation-miner) | Give it any domain — a one-liner, a brief, or a whole knowledge-base folder — and it returns a ranked portfolio of **ICE-scored automation opportunity briefs**. Every claim is cited back to a numbered source chunk (`[S12] claims.pdf p.4`). The critic/refine loop is an LLM; the ICE math, coherence validation, and strategic filters are code. Nothing is silently dropped. | `Python` `LangGraph` `MCP` |
| [**berlin-insolvency-radar**](https://github.com/ghassan-ai-projects/berlin-insolvency-radar) | Berlin sees 60–180 actionable corporate insolvencies a year, and analysts still sift court notices by hand. This scrapes the official portal, extracts structured facts, enriches from public sources, scores across five weighted dimensions, self-corrects on legal and evidence risk, and exports a ranked weekly newsletter — deterministic guardrails, full audit trail, fail-closed. | `Python` `MCP` |

## 🔎 MCP-native knowledge systems

Data engines where the agent is the primary user, not an afterthought.

| Project | What it does | Stack |
| :--- | :--- | :--- |
| [**email-intelligence-platform**](https://github.com/ghassan-ai-projects/email-intelligence-platform) | Your Maildir becomes a continuously enriched **knowledge base** — FTS5, vector search, LLM-extracted facts, action items, and entities — all in one SQLite file. Agents get knowledge-level MCP tools (`find_waiting_replies`, `find_decisions`, `summarize_sender`), not a raw IMAP wrapper. No servers, no Docker, no UI. | `Python` `SQLite` `MCP` |
| [**orion-chess-db**](https://github.com/ghassan-ai-projects/orion-chess-db) | A chess database with an **MCP server built in**, so agents can query and edit your games directly. Perft-validated legal move generation, lossless PGN round-tripping (variations, NAGs, Latin-1 legacy files), transposition-aware position search via Zobrist hashing, opening explorer, tactics detection, and player dossiers. Your database file is plain PGN — no proprietary format, nothing to migrate. | `Java` `MCP` |

## 🎓 Learning by doing

| Project | What it does | Stack |
| :--- | :--- | :--- |
| [**scaleshop-scalability-lab**](https://github.com/ghassan-ai-projects/scaleshop-scalability-lab) <br> [▶ **Try the live lab**](https://lab.ghassan-alhamoud.com) | Scalability is rarely solved by picking the most powerful technology — it is solved by finding the current constraint and accepting only the complexity the evidence justifies. Twelve progressive incidents: read the telemetry, cite the decisive signal, name the bottleneck, choose the *smallest sufficient* intervention, then compare against a reference architecture. All counterfactual — provisions nothing, sends no traffic. | `TypeScript` `Next.js` |

---

## 🧭 How these projects are built

The same operating standard applies across the org — it is the point, not the packaging.

- **`AGENTS.md` in every repo** — one operating contract, read natively by Claude Code, Codex, Cursor, and Gemini.
- **One command to verify everything** — `make check` / `npm run check`: format, lint, typecheck, tests, build, product gate. If it passes locally, CI agrees.
- **Models propose, code decides** — scoring math, validation, constraint overrides, and state transitions run in code. LLMs are never the arbiter of correctness.
- **Structured outputs, everywhere** — schema-validated JSON at every stage boundary; parse failures retry with the error fed back rather than degrading silently.
- **Evidence over assertion** — run artifacts, audit trails, citable source chunks, and an honest `LIMITATIONS.md` where one is warranted.
- **MCP-first** — the agent is the primary interface. CLIs and UIs come second, if at all.

---

## 📚 Writing &amp; reference

**[AI Agent Architecture Handbook](https://ghassan-alhamoud.com/handbook/)** — a structured tour of the patterns behind these systems:
[The Agentic Landscape](https://ghassan-alhamoud.com/handbook/chapter-00-agentic-landscape.html) ·
[ReAct](https://ghassan-alhamoud.com/handbook/chapter-01-react-pattern.html) ·
[Plan-and-Execute](https://ghassan-alhamoud.com/handbook/chapter-02-plan-and-execute.html) ·
[Reflection](https://ghassan-alhamoud.com/handbook/chapter-03-reflection.html) ·
[Multi-Agent Collaboration](https://ghassan-alhamoud.com/handbook/chapter-04-multi-agent-collaboration.html) ·
[Tool Use &amp; Skill Registries](https://ghassan-alhamoud.com/handbook/chapter-05-tool-use-skill-registry.html) ·
[Memory &amp; Context Management](https://ghassan-alhamoud.com/handbook/chapter-06-memory-context-management.html)

**Selected articles**

- [Stop Building Agent Soup. Start Building Compilers.](https://ghassan-alhamoud.com/articles/stop-building-agent-soup-compilers.html) — why loose agent orchestration collapses, and what to build instead.
- [ALMS 0.1.0: A Shared Brain for Autonomous Agents](https://ghassan-alhamoud.com/articles/alms-open-source-shared-agent-memory.html) — the shared-memory problem in agent fleets.
- [Turning Agent Experience Into Portable Knowledge](https://ghassan-alhamoud.com/articles/alms-okf-integration-bridge.html) — exporting mature learnings as Open Knowledge Format.
- [The Ghost in the Terminal: Testing TUIs with tmux](https://ghassan-alhamoud.com/articles/testing-tuis-with-tmux.html) — PTYs, tmux, and making terminal UIs testable.

[**→ All articles**](https://ghassan-alhamoud.com/articles/)

---

## 🛠️ Stack

![Go](https://img.shields.io/badge/Go-00ADD8?style=flat-square&logo=go&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white)
![Ruby](https://img.shields.io/badge/Ruby-CC342D?style=flat-square&logo=ruby&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white)
![Java](https://img.shields.io/badge/Java-ED8B00?style=flat-square&logo=openjdk&logoColor=white)
![MCP](https://img.shields.io/badge/MCP-Model_Context_Protocol-8B5CF6?style=flat-square)
![LangGraph](https://img.shields.io/badge/LangGraph-1C3C3C?style=flat-square&logo=langchain&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white)
![SQLite](https://img.shields.io/badge/SQLite-003B57?style=flat-square&logo=sqlite&logoColor=white)
![Terraform](https://img.shields.io/badge/Terraform-844FBA?style=flat-square&logo=terraform&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat-square&logo=github-actions&logoColor=white)

---

<p align="center">
  Built in Berlin by <a href="https://github.com/gh-assan"><strong>Ghassan Alhamoud</strong></a><br>
  <a href="https://ghassan-alhamoud.com">Website</a> ·
  <a href="https://linkedin.com/in/ghassanalhamoud">LinkedIn</a> ·
  <a href="https://github.com/gh-assan">GitHub</a>
</p>

<p align="center">
  <sub>Repositories, teams, and this page are managed as code with Terraform.</sub>
</p>
