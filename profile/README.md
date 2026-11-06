<h1 align="center">Tamoz</h1>

<p align="center">
  <strong>We give AI agents sensors, hands, and the rules for using them.</strong><br>
  A physical AI agent for the equipment you already run — it watches your sensors,
  works out what is likely happening, says how sure it is, and proposes a response
  from the actions your team has defined.
</p>

<p align="center">
  <a href="https://ghassan-alhamoud.com/projects/tamoz.html"><img src="https://img.shields.io/badge/Tamoz-Project_overview-0F766E?style=for-the-badge&logo=target&logoColor=white" alt="Tamoz project overview"></a>
  <a href="https://ghassan-alhamoud.com/projects/tamoz.html#pilot"><img src="https://img.shields.io/badge/Industrial-Pilot-B45309?style=for-the-badge&logo=factory&logoColor=white" alt="Industrial pilot"></a>
  <a href="https://ghassan-alhamoud.com"><img src="https://img.shields.io/badge/Website-ghassan--alhamoud.com-0A0A0A?style=for-the-badge&logo=safari&logoColor=white" alt="Website"></a>
</p>

<p align="center">
  <sub><code>pre-release</code> · tested in simulation and on a hardware bench · not yet deployed in the field</sub>
</p>

---

## The problem Tamoz is built for

A cold room warms up overnight. Equipment starts drawing more power. A sensor stops
reporting. An alarm can tell you a reading crossed a line, but not whether a door was
left open, a machine is failing, or the sensor is wrong. Someone still has to work that
out before deciding whether to wait, investigate, or act.

**Tamoz does that first pass.** People decide which actions it can propose and which need
their approval, and every outcome is recorded. While the equipment stays steady, no model
runs. When a sensor stops reporting, Tamoz treats that as missing evidence rather than as
normal, so it becomes *less* certain instead of guessing.

**[→ Read the full project overview](https://ghassan-alhamoud.com/projects/tamoz.html)**

---

## Rules work. Who writes the next one?

Rules have run plants for decades because they are fast, predictable, and easy to check.
Tamoz keeps them. The hard part was never *running* a rule — it is knowing which rule to
write, and keeping up as things change.

A rule only catches what someone understood when they wrote it. A new failure, a machine
that has drifted from its neighbours, or a fault that looks like a different one falls
through to whoever is on shift. Those are the cases Tamoz is for.

It works through the case no rule covers. When the same repair keeps working, it drafts a
new rule with the evidence behind it. Once a person approves it, that rule handles the
problem **without calling a model** — an ordinary rule your team can read, question, and
switch off. If the equipment changes or the repair stops holding, the rule switches itself
off and the model takes over again.

The aim: the model's job shrinks to what is still unknown, while what you have learned
stays in rules you own.

---

## How it is built

Tamoz is one system with three parts and three separate jobs. **The split is the point:**
the part that reasons cannot touch equipment, and the part that touches equipment follows
fixed rules.

| Repository | Job in the system | Stack |
| :--- | :--- | :--- |
| [**tamoz**](https://github.com/ghassan-ai-projects/tamoz) <br> <sub>`public · pre-release`</sub> | **Reasons.** The agent. It gets one immutable snapshot of the situation and a budget for how much reasoning it may spend, reads further evidence through read-only probes an operator declared, and returns a proposal with its uncertainty. It can draft a rule from verified repairs for a person to review. Durable by construction: a Ruby-native checkpointed graph over SQLite that survives `kill -9` and resumes from its last committed barrier. Nothing acts without a reviewed plan bound to its digest; it has no direct path to equipment. | `Ruby` `SQLite` `gRPC` |
| [**agentic-stream**](https://github.com/ghassan-ai-projects/agentic-stream) <br> <sub>`public · development snapshot`</sub> | **Watches and acts.** The runtime that is always on. It records each reading with its event time, quality, and late-data status, publishes immutable versioned **Situations**, and starts a bounded agent episode only when a deterministic cognitive scheduler decides reasoning is useful. Models return typed Decisions and Action Intents; a separate policy plane revalidates every intent immediately before dispatch, and the action plane executes idempotently or records why it could not. Approved learned rules run here, with no model call. | `Go` `SQLite` `CEL` `gRPC` |
| [**streams-simulator**](https://github.com/ghassan-ai-projects/streams-simulator) <br> <sub>`public`</sub> | **The plant and the console.** A deterministic, closed-loop world simulator: compose a plant, fleet, or site from a component library or write a domain as JSON, run it on a seeded clock, inject faults, perturb delivery, and take commands back so the world really changes. Sealed ground truth, a delivery ledger, and independent analytic oracles grade what a consumer detected and did. The plant wire lets a **real board sit beside simulated ones** in one session, with interrupts, approvals, and a twin-versus-board report. A run is a pure function of its inputs and replays byte for byte. | `Go` `MCP` `TypeScript` |

### The loop, from readings to outcomes

```text
1 SENSE        2 SITUATE          3 UNDERSTAND     4 GOVERN        5 ACT           6 LEARN
the world  →   Agentic Stream  →  Tamoz agent   →  policy      →   device edge  →  outcome recorded
sensors,       event time,        what is           freshness,      gateway and     repeated fixes
machines       quality,           likely; a         approval,       firmware:       can become
               silent sensors     proposal          limits          limits,         rules
               → snapshot         → what            → how much      safe stop
```

Evidence is never an executable instruction. The command being received, the device
reporting success, and the world actually changing are kept as three separate facts.

---

## What has actually been tested

| Test | Reasoning | Equipment | Main result |
| :--- | :--- | :--- | :--- |
| Pump station | Real model | Simulated | A learned rule did the model's job on one fault type |
| Hardware bench | Real model | Real board and fan | The whole route ran, in three sessions |
| Earlier rounds | Scripted | Simulated | Machinery exercised across five industries, gaps listed |

**Pump station.** One simulated station, eight pumps plus three healthy look-alikes. Three
fault types injected six times each — 18 faults — and the same test days run twice, with
the learned rule and without it. Where a rule existed it did the model's job with **no
model call**: five of six misalignments, all correct and booked before the deadline. Totals
were 7 of 18 with the rule against 6 of 18 without, and 0 of 3 wrong actions on healthy
look-alikes either way. So the demonstrated gain is **fewer model calls, not more repairs**.
One of three fault types has earned a rule so far; the model handled the rest.

**Hardware bench.** Three recorded sessions ran the whole route end to end — Tamoz's
proposal, through policy and a person's approval, to a command on an ELEGOO Mega 2560
driving a DC fan motor. With the motor disconnected the board still reported success, so
these sessions do **not** measure a physical effect such as cooling. That is why a pilot
starts read-only, and why any later equipment action needs an independent check.

**Where it stands.** Tamoz is pre-release software, tested in simulation and on a small
bench. It has not been deployed in the field. It is **not a certified safety system** —
yours stay responsible for protection. No savings in energy, material, or time have been
measured in a real operation; a pilot is how we would find out.

**[→ Explore the industrial pilot](https://ghassan-alhamoud.com/projects/tamoz.html#pilot)**

---

## How we build

The same operating standard applies to every repository here. It is the point, not the
packaging.

- **Models propose, code decides** — scoring math, validation, policy, and state
  transitions run in code. An LLM is never the arbiter of correctness.
- **Evidence over assertion** — run artifacts, sealed ground truth, audit trails, citable
  sources, and an honest `LIMITATIONS.md` wherever one is warranted.
- **Deterministic where it matters** — a run is reproducible from its inputs; replay
  performs no external effects unless simulation is explicitly selected.
- **`AGENTS.md` in every repo** — one operating contract, read natively by Claude Code,
  Codex, Cursor, and Gemini.
- **One command verifies everything** — `make check` / `npm run check`: format, lint,
  typecheck, tests, build, product gate. If it passes locally, CI agrees.
- **MCP-first** — the agent is a primary interface. CLIs and UIs come second.

---

## Also in this org

Tamoz is the focus. These are the other systems built to the same standard.

| Project | What it does |
| :--- | :--- |
| [**alms**](https://github.com/ghassan-ai-projects/alms) | **Agent Learning Management System** — a self-hosted MCP control plane for the fleet's worst failure mode: one agent learns something useful and the others never hear about it. Registry, heartbeats, gap-safe learning sync, OKF export, protocol distribution by tag. Deliberately out of the hot path. |
| [**automation-miner**](https://github.com/ghassan-ai-projects/automation-miner) | Give it a domain — a one-liner or a whole knowledge base — and get a ranked portfolio of ICE-scored automation opportunity briefs, every claim cited to a numbered source chunk. The critic loop is an LLM; the ICE math and strategic filters are code. |
| [**berlin-insolvency-radar**](https://github.com/ghassan-ai-projects/berlin-insolvency-radar) | Scrapes the official Berlin insolvency portal, extracts structured facts, enriches from public sources, scores across five weighted dimensions, and exports a ranked weekly newsletter. Deterministic guardrails, full audit trail, fail-closed. |
| [**film-pipeline-langgraph**](https://github.com/ghassan-ai-projects/film-pipeline-langgraph) | Film creation from idea to validated handoff. Typed graph state, real human approval gates via LangGraph `interrupt()`, auto-versioned artifacts, and a QC subgraph that fans validators out in parallel. |
| [**email-intelligence-platform**](https://github.com/ghassan-ai-projects/email-intelligence-platform) | Your Maildir becomes a continuously enriched knowledge base — FTS5, vector search, extracted facts and action items — in one SQLite file. Agents get knowledge-level MCP tools, not a raw IMAP wrapper. |
| [**orion-chess-db**](https://github.com/ghassan-ai-projects/orion-chess-db) | A chess database with an MCP server built in. Perft-validated movegen, lossless PGN round-tripping, transposition-aware position search, opening explorer, tactics detection — over a plain PGN file. |
| [**scaleshop-scalability-lab**](https://github.com/ghassan-ai-projects/scaleshop-scalability-lab) <br> [▶ live](https://lab.ghassan-alhamoud.com) | Twelve progressive incidents: read the telemetry, cite the decisive signal, name the bottleneck, choose the *smallest sufficient* intervention. Counterfactual — provisions nothing, sends no traffic. |
| [**human-signals**](https://github.com/ghassan-ai-projects/human-signals) | Interactive visual model of human physiology — hormones, neurotransmitters, organs, brain regions, signal journeys, and the evidence behind them. |

Plus the project blueprints that enforce the standard above:
[**go**](https://github.com/ghassan-ai-projects/go-project-blueprint) ·
[**python**](https://github.com/ghassan-ai-projects/python-project-blueprint) ·
[**rails**](https://github.com/ghassan-ai-projects/rails-project-blueprint)

---

## Writing & reference

**[AI Agent Architecture Handbook](https://ghassan-alhamoud.com/handbook/)** — the patterns behind these systems:
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

## Stack

![Go](https://img.shields.io/badge/Go-00ADD8?style=flat-square&logo=go&logoColor=white)
![Ruby](https://img.shields.io/badge/Ruby-CC342D?style=flat-square&logo=ruby&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white)
![Java](https://img.shields.io/badge/Java-ED8B00?style=flat-square&logo=openjdk&logoColor=white)
![MCP](https://img.shields.io/badge/MCP-Model_Context_Protocol-8B5CF6?style=flat-square)
![gRPC](https://img.shields.io/badge/gRPC-244c5a?style=flat-square&logo=grpc&logoColor=white)
![SQLite](https://img.shields.io/badge/SQLite-003B57?style=flat-square&logo=sqlite&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white)
![LangGraph](https://img.shields.io/badge/LangGraph-1C3C3C?style=flat-square&logo=langchain&logoColor=white)
![Terraform](https://img.shields.io/badge/Terraform-844FBA?style=flat-square&logo=terraform&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat-square&logo=github-actions&logoColor=white)

---

<p align="center">
  Built in Berlin by a founding team of three<br>
  <a href="https://github.com/gh-assan"><strong>Ghassan Alhamoud</strong></a> · engineering &amp; product<br>
  <a href="https://ghassan-alhamoud.com/projects/tamoz.html">Tamoz</a> ·
  <a href="https://ghassan-alhamoud.com">Website</a> ·
  <a href="https://linkedin.com/in/ghassanalhamoud">LinkedIn</a>
</p>

<p align="center">
  <sub>Repositories, teams, and this page are managed as code with Terraform.</sub>
</p>
