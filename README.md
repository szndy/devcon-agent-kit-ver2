# DEVCON Jumpstart Agent Kit

A jumpstart kit **built for the DEVCON community** — for members starting their next hackathon entry, code-camp project, workshop build, or personal/community initiative. Turn a one-line product idea into a full set of **development-ready planning documents** — a PRD, a user-flow, an architecture decision record, and a design system — using **free-tier AI models**, inside whichever agent CLI you already use (**OpenCode**, **Claude Code**, or **Cursor**).

It's built for hackathons, workshops, and jumpstarting real projects: describe what you want to build in one sentence, run one command, and get clean Markdown docs a DEVCON team can execute from directly — no paid tools required to get started.

> **Looking for this project's own requirements?** See [`PRD.md`](PRD.md) — the Product Requirements Document for the Jumpstart Kit itself (problem statement, goals, functional/non-functional requirements, success metrics, current scope, and roadmap). This README stays focused on installing and using the kit.

---

## Table of contents

- [What it is](#what-it-is)
- [What it produces](#what-it-produces-expected-output)
- [Features](#features)
- [How it works](#how-it-works)
- [Prerequisites](#prerequisites)
- [Step-by-step: pick your path](#step-by-step-pick-your-path)
- [Install](#install)
- [Usage](#usage)
- [Full walkthrough: build an example project](#full-walkthrough-build-an-example-project)
- [Expected output in detail](#expected-output-in-detail)
- [Models — fully agnostic](#models--fully-agnostic)
- [Directory structure](#directory-structure)
- [Extending the kit](#extending-the-kit)
- [Troubleshooting](#troubleshooting)

---

## What it is

The Jumpstart Kit is a **model-agnostic, tool-agnostic agent kit**. Its guiding principle:

> **The brain is portable; the wiring is an adapter.**

- **`core/` — the portable brain.** All the intelligence lives in vendor-neutral `SKILL.md` skills (a cross-tool open standard), a JSON schema, a deterministic Python renderer, and a shared `AGENTS.md` rulebook. Written once, identical everywhere.
- **`adapters/<tool>/` — thin per-tool wiring.** Only the agent/command definitions and provider config differ per tool. Swapping tools never touches the brain.

Because the whole pipeline is also described in one `workflow-orchestration` skill, a tool **without** subagents (like Cursor) can run the entire thing single-agent. The multi-subagent split used by OpenCode and Claude Code is an optimization (isolation, least-privilege, parallelism), not a requirement.

---

## What it produces (expected output)

Every run creates a self-contained folder `output/<slug>/` (where `<slug>` is a kebab-case of your idea, so runs never overwrite each other):

| File | What it is |
|---|---|
| `intent.json` | Structured intent inferred from your idea, with every assumption recorded explicitly. |
| `prd.md` | **Development-ready PRD** — feature specs with `Given/When/Then` acceptance criteria, a data model, system interactions, non-functional targets, and a concrete WCAG 2.2 AA accessibility section. |
| `flow.json` | The primary user flow as a schema-validated graph (stages, nodes, decision branches). |
| `user-flow.md` | The flow rendered as readable Markdown — numbered steps grouped by stage, `→` transitions and `⤷` decision branches, plus decision-points / exit-points / edge-cases sections. |
| `adr.md` | **Architecture Decision Record** — recommended stack (with alternatives + tradeoffs), system design, long-term engineering choices, and a phased implementation blueprint that maps to the PRD's features. |
| `design-system.md` | **Design system** — a UI approach fitted to the product, foundations with real contrast ratios, a component inventory with states, and design tokens aligned to the ADR's chosen frontend stack. |
| `mvp-scope.md` | *(optional, `/scope` only)* **Time-boxed build plan** — effort-scored features, a protected critical demo path, a Build Now / Parked split, and a per-teammate task split, sized to your actual hours and headcount. |
| `starter/` | *(optional, `/starter-code` only)* **Real runnable boilerplate** — folder structure, `package.json`, a working hello-world page/route, `.env.example`, and a README with exact run commands, generated deterministically from the ADR's chosen stack. |
| `pitch-deck.md` | *(optional, `/pitch-deck` only)* **5-slide demo-day outline** — slide content + speaker notes (~3 min total), pitching what was actually built if `mvp-scope.md` exists. |
| `run.json` | A manifest listing every artifact path produced. |

`/blueprint` produces the first four (`intent` → `prd` → `flow` → `user-flow`). `/jumpstart` produces **everything** — the core four, plus `adr.md`, `starter/`, `design-system.md`, and `pitch-deck.md` — and adds `mvp-scope.md` too if you pass `hours` + `team_size` as extra arguments. Each of `/scope`, `/starter-code`, and `/pitch-deck` also still works standalone (e.g. to rescope after hand-editing the PRD, or regenerate the starter after tweaking the ADR) — they just read whatever's latest in `output/<slug>/` rather than requiring a fresh `/jumpstart` run.

---

## Features

- **Idea → four docs in one command.** PRD, user-flow, ADR, and design system, all cross-referenced by file path.
- **Optional time-boxed MVP scoping.** `/scope <hours> <team_size>` triages the PRD's features against a real deadline and headcount — a protected critical path, a Build Now / Parked split with effort estimates, and a per-teammate task breakdown. Built for hackathons and code camps; downstream ADR and design-system generation automatically respect it if it exists.
- **Optional starter code, not just docs.** `/starter-code` turns the ADR's chosen stack into a real, runnable boilerplate — a script assembles it from known-good templates (React+Vite, Next.js, or plain HTML; Node+Express or none; a zero-dependency JSON-file store, Supabase, or none), so it never hallucinates a broken `package.json`. Anything outside those templates gets an honest TODO stub instead of guessed code.
- **Optional demo-day pitch deck.** `/pitch-deck` drafts a 5-slide outline with speaker notes from the PRD and ADR — pitching what was actually built if you ran `/scope`, not the full original vision.
- **Development-ready depth, not summaries.** Per-feature acceptance criteria, a draftable data model, states & edge cases, and accessibility baked in as a first-class section.
- **Deterministic diagrams.** The user flow is never hand-written by the model — it's rendered by a Python script from a schema-checked JSON graph, which removes structural/syntax hallucination entirely and keeps the flow format consistent.
- **Read-only validator with a 1-retry loop.** A least-privilege reviewer checks each doc against its authoring checklist and returns `PASS` or a concrete `GAPS` list; the author fixes gaps exactly once (respecting free-tier rate limits).
- **Free-tier friendly.** Low temperature everywhere, minimal tool surfaces, capped retries — designed to run reliably on free models.
- **Works in three tools today.** OpenCode, Claude Code, and Cursor — same commands, same output.
- **Runnable from the repo root.** One install command symlinks everything into the root, so you open your tool at the root and the commands are already there — no `cd`-ing into subfolders.
- **Model-agnostic.** No model IDs are hardcoded; every agent inherits whatever model you select in your tool.
- **Scalable by addition.** A new capability = one skill + one agent + one command, with zero rewrites to what exists.

---

## How it works

### The pipeline

```
idea
  └─▶ intent.json      (clarifier — infers intent, records assumptions)
        └─▶ prd.md      (prd-author — loads the prd-authoring skill)
              └─▶ PASS | GAPS   (validator — read-only; author fixes gaps once)
                    └─▶ mvp-scope.md   (scoper — reads prd.md; only if hours+team_size given)
                          └─▶ flow.json        (flow-architect — enriched graph)
                                └─▶ user-flow.md   (render_flow.py — deterministic, no LLM)
                                      └─▶ adr.md         (architect — reads prd.md, + mvp-scope.md if present)
                                            └─▶ starter/       (scaffolder — reads adr.md)
                                                  └─▶ design-system.md   (designer — reads prd.md + adr.md, + mvp-scope.md if present)
                                                        └─▶ pitch-deck.md   (pitch-writer — reads prd.md + adr.md, + mvp-scope.md if present)
                                                              └─▶ run.json      (manifest)
```

Each step reads the previous artifact **by file path**, not by re-deriving from the idea — so downstream docs stay consistent with upstream ones. `/blueprint` runs only as far as `user-flow.md`; `/jumpstart` runs the entire chain (the `mvp-scope.md` branch only if you pass `hours` + `team_size`); each of `/scope`, `/adr`, `/starter-code`, `/design-system`, and `/pitch-deck` runs just its own step against whatever's already on disk.

### The three building blocks

- **Skills** (`core/skills/*/SKILL.md`) — the reusable know-how: how to author a PRD, how to model a flow, how to scope an MVP, how to write an ADR, how to scaffold a starter, how to design a system, how to draft a pitch deck, and how to orchestrate the whole pipeline. These are the portable brain.
- **Agents** (`adapters/<tool>/…`) — narrow specialists (`clarifier`, `prd-author`, `flow-architect`, `scoper`, `architect`, `scaffolder`, `designer`, `pitch-writer`, `validator`) plus, in OpenCode, a primary `jumpstart` orchestrator. Each has a single responsibility and a minimal tool surface (the validator is read-only).
- **Commands** — the slash commands (`/jumpstart`, `/scope`, `/adr`, `/starter-code`, `/design-system`, `/pitch-deck`, `/blueprint`) that kick off a whole run or a single step.

### Why the flow is JSON-then-rendered

The fragile LLM step produces *checkable data* (nodes, edges, conditions, stages). Turning that data into a readable `user-flow.md` is pure Python (`render_flow.py`). This is the single most important reliability decision in the kit: it removes diagram hallucination and makes the flow layer swappable later without touching any agent.

---

## Prerequisites

- **An agent CLI:** [OpenCode](https://opencode.ai/docs), [Claude Code](https://docs.claude.com/en/docs/claude-code), or [Cursor](https://cursor.com).
- **Python 3** on your PATH (`python3`) — runs the deterministic flow renderer.
- **A model API key.** For OpenCode, a free **OpenRouter** or **Groq** key works; authenticate with `opencode auth login`. (Claude Code / Cursor use their own model access.)
- **git** + optionally the **GitHub CLI** (`gh`) if you plan to clone/fork.

---

## Step-by-step: pick your path

Every path runs the exact same commands (`/jumpstart`, `/blueprint`, …) once set up — the only difference is **how you authenticate and pick a model**. Pick the one that matches what you have.

### Option A — Free, no paid plan (OpenCode + a free model key)

1. **Install OpenCode.** Follow [opencode.ai/docs](https://opencode.ai/docs) (one-line install script or `npm install -g opencode-ai`, per their docs).
2. **Clone this repo and install the OpenCode adapter:**
   ```bash
   git clone https://github.com/szndy/DEVCON-community-agent-kit-ver2.git
   cd DEVCON-community-agent-kit-ver2
   ./scripts/install.sh opencode
   ```
3. **Get a free API key** from [OpenRouter](https://openrouter.ai/keys) or [Groq](https://console.groq.com/keys) — both offer free-tier models, no card required.
4. **Authenticate OpenCode with that key:**
   ```bash
   opencode auth login
   ```
   Choose the matching provider (OpenRouter or Groq) and paste your key when prompted.
5. **Launch OpenCode at the repo root:**
   ```bash
   opencode
   ```
6. **Pick a free model** inside OpenCode with `/models` (filter for free-tier OpenRouter/Groq models).
7. **Run your first command:**
   ```
   /blueprint a mobile app for booking barangay health center appointments
   ```
8. Your docs land in `output/<slug>/` — see [What it produces](#what-it-produces-expected-output).

### Option B — Paid plan (Claude Code or Cursor subscription)

No separate API key step — both tools use the model access that comes with your subscription.

**Claude Code:**
1. Install Claude Code and sign in with your Claude subscription: [docs.claude.com/en/docs/claude-code](https://docs.claude.com/en/docs/claude-code).
2. Clone this repo and install the Claude Code adapter:
   ```bash
   git clone https://github.com/szndy/DEVCON-community-agent-kit-ver2.git
   cd DEVCON-community-agent-kit-ver2
   ./scripts/install.sh claude
   ```
3. Open Claude Code at the repo root (`claude`), then run `/jumpstart <your idea>` or `/blueprint <your idea>`.

**Cursor:**
1. Install [Cursor](https://cursor.com) and sign in with your subscription.
2. Clone this repo and install the Cursor adapter:
   ```bash
   git clone https://github.com/szndy/DEVCON-community-agent-kit-ver2.git
   cd DEVCON-community-agent-kit-ver2
   ./scripts/install.sh cursor
   ```
3. Open the repo folder in Cursor, then type `/blueprint <your idea>` in chat. If `.cursor/commands` doesn't surface slash commands in your Cursor version, just type plain text like `blueprint <your idea>` — the `.cursor/rules` guidance picks it up.

Either way, full command reference and examples are in [Usage](#usage); install details (what each installer creates, why it symlinks) are in [Install](#install) below.

---

## Install

Clone the repo, then run the installer for your tool from the repo root. It symlinks the portable `core/` and your tool's adapter **into the repo root**, so the tool is runnable there directly.

```bash
git clone https://github.com/szndy/DEVCON-community-agent-kit-ver2.git
cd DEVCON-community-agent-kit-ver2

./scripts/install.sh opencode      # or: claude | cursor | all
```

What lands at the repo root:

| Command | Creates at root |
|---|---|
| `./scripts/install.sh opencode` | `.opencode/{agent,command,skills}`, `opencode.json`, `AGENTS.md` |
| `./scripts/install.sh claude` | `.claude/{agents,commands,skills}`, `CLAUDE.md`, `AGENTS.md` |
| `./scripts/install.sh cursor` | `.cursor/{rules,commands,skills}`, `AGENTS.md` |
| `./scripts/install.sh all` | all of the above (they coexist) |

The installer is **idempotent** — re-run it any time. Everything is symlinked back to `core/` and the adapters, so editing a skill once updates every tool.

> **Why symlinks?** Cross-directory skill discovery isn't dependable across tools, so the installer links each tool's config next to its skills at the root. The adapters remain the source of truth; the root just points at them. (These root symlinks are regenerated by the installer, so they're git-ignored — always run `install.sh` after cloning.)

Then authenticate and pick a model:

```bash
opencode auth login       # OpenRouter/Groq → free key (once), for OpenCode
opencode                  # open at the repo root; choose a model with /models
```

---

## Usage

Open your tool **at the repo root** and run one step or the whole pipeline:

| Command | Does | Reads |
|---|---|---|
| `/blueprint <idea>` | Builds the PRD + user-flow | your idea |
| `/scope <hours> <team_size> [skill_level]` | Builds a time-boxed MVP plan | the latest `output/<slug>/prd.md` |
| `/adr` | Builds the ADR | the latest `prd.md` (+ `mvp-scope.md` if present) |
| `/starter-code` | Generates a real runnable starter under `output/<slug>/starter/` | the latest `output/<slug>/adr.md` |
| `/design-system` | Builds the design system | the latest `prd.md` + `adr.md` (+ `mvp-scope.md` if present) |
| `/pitch-deck` | Builds a 5-slide demo outline with speaker notes | the latest `prd.md` + `adr.md` (+ `mvp-scope.md` if present) |
| `/jumpstart <idea> [hours] [team_size] [skill_level]` | Builds **every** document end-to-end — PRD, user-flow, ADR, starter code, design system, and pitch deck. Adds MVP scope too if `hours` + `team_size` are given | your idea (+ optional scope numbers) |

**Examples**

```
/blueprint a mobile app for booking barangay health center appointments
/jumpstart a simple event RSVP tool for a campus org
/jumpstart a simple event RSVP tool for a campus org 6 4 beginner
/scope 6 4 beginner
/starter-code
/pitch-deck
```

- Use **`/jumpstart`** for a complete one-shot run — it reads each artifact directly, so there's no "find the latest PRD" guessing. Add `hours` and `team_size` at the end (as in the second example above) to also get a time-boxed `mvp-scope.md`; leave them off and everything except scoping still runs — starter code and the pitch deck are always included.
- Use the **`/blueprint` → `/adr` → `/design-system`** chain when you want to review or edit the PRD before generating the downstream docs. (These standalone steps locate the most recent `output/<slug>/` run, so run `/blueprint` first.)
- **Running a code camp or hackathon?** `/jumpstart <idea> <hours> <team_size>` in one shot covers the whole thing — PRD, scope, flow, ADR, starter code, design system, and pitch deck, all sized to your real deadline. Prefer to review the PRD before committing to a stack? Run the standalone chain instead: `/blueprint` → `/scope <hours> <team_size>` → `/adr` → `/starter-code` → `/design-system` → (build) → `/pitch-deck`. Either way, scoping before the ADR means the ADR's Phase 1, the generated starter, and the design system's component list all match what the team is actually building — and the pitch deck at the end presents that real scope, not the original unscoped idea.
- In **Cursor** (no subagents) the same commands run single-agent via the `workflow-orchestration` skill; if your Cursor version doesn't surface `.cursor/commands`, just type "blueprint &lt;idea&gt;" and the `.cursor/rules` guidance kicks in.

### Render a user flow standalone (no LLM)

The renderer is a plain, deterministic script you can run on any schema-valid `flow.json`:

```bash
python3 core/skills/flow-graph/scripts/render_flow.py output/<slug>/flow.json
```

It validates the graph and writes `user-flow.md` beside it. If validation fails it prints the exact problem — fix the JSON and re-run; never edit the `.md` by hand.

---

## Full walkthrough: build an example project

This section is the tutorial version of [Usage](#usage) — it follows **one concrete idea** from a single sentence to a runnable starter, showing **exactly where you type each command** and **exactly where the output lands**, with real generated content so you know what to expect.

**Example idea:** *"a simple event RSVP tool for a campus org."*

> **Where commands go:** every `/command` below is typed **inside your agent tool's own prompt** (OpenCode's or Claude Code's interactive session, or Cursor's chat panel) — **not** in a plain terminal. Open a terminal only to `cd` into the repo and launch the tool (`opencode`, `claude`, or opening the folder in Cursor); once the tool is running **at the repo root**, every step from here happens inside it.

### Step 1 — Build the PRD + user flow

At the repo root, launch your tool and type:

```
/blueprint a simple event RSVP tool for a campus org
```

The tool runs the `clarifier` → `prd-author` → validator → `flow-architect` → renderer chain on its own (see [How it works](#how-it-works)). When it finishes, look in your **file explorer or a new terminal** — a folder has appeared at:

```
output/campus-event-rsvp/
├── intent.json
├── prd.md
├── flow.json
└── user-flow.md
```

(The exact folder name is a kebab-case slug of your idea, generated automatically — it won't always be `campus-event-rsvp` verbatim.)

Open `prd.md` and `user-flow.md` in any editor (or `cat output/campus-event-rsvp/user-flow.md` in a terminal) to read them. Here's real output the deterministic renderer produces for a flow shaped like this project's RSVP logic:

```markdown
# User Flow — Campus Event RSVP

> A student discovers a campus event and RSVPs, receiving a confirmation.

**Legend:** `1.` step in sequence · `→` leads to · `⤷` decision branch

## Stage: Discovery

1. **Browse events** _(entry)_ — Student opens the app and sees a list of upcoming campus events.
   → **Pick event**
2. **Pick event** — Student taps an event to view details.
   → **Seats available?**

## Stage: RSVP

3. **Seats available?** — System checks remaining capacity for the event.
   - ⤷ *Yes* → **Confirm RSVP** — seats open
   - ⤷ *No* → **Join waitlist** — event full
4. **Confirm RSVP** — Student confirms attendance and receives a confirmation email.
   → **RSVP confirmed**
5. **Join waitlist** — Student is added to the waitlist and notified if a seat opens up.
   → **Waitlisted**

## Stage: Confirmation

6. **RSVP confirmed** _(exit)_ — Student holds a confirmed seat and can view it under 'My Events'.
7. **Waitlisted** _(exit)_ — Student is on the waitlist and will be notified automatically.
```

Read the PRD first — everything downstream (flow, ADR, design system, starter code) is built to match it. Edit `prd.md` by hand now if you want to change scope before continuing (never hand-edit `user-flow.md`; edit `flow.json` and re-render instead — see [Render a user flow standalone](#render-a-user-flow-standalone-no-llm)).

### Step 2 — (optional) Scope it to your deadline

Still inside your tool, at the repo root:

```
/scope 6 4 beginner
```

This reads the `prd.md` you just built and writes `output/campus-event-rsvp/mvp-scope.md` — a Build Now / Parked split sized for 6 hours and a 4-person beginner team. Every step after this one automatically respects it if it exists.

### Step 3 — Build the ADR

```
/adr
```

Reads the latest `prd.md` (and `mvp-scope.md` if you ran Step 2). Writes `output/campus-event-rsvp/adr.md` — open it to see the recommended stack table, alternatives, and a phased blueprint.

### Step 4 — Generate the runnable starter

```
/starter-code
```

Reads `adr.md` and writes real, runnable code to `output/campus-event-rsvp/starter/`. For a React + Node/Express + JSON-file stack, this looks like:

```
output/campus-event-rsvp/starter/
├── frontend/
│   ├── index.html
│   ├── package.json
│   ├── vite.config.js
│   └── src/
│       ├── App.jsx
│       ├── index.css
│       └── main.jsx
├── backend/
│   ├── package.json
│   ├── server.js
│   ├── db.js
│   └── data/db.json
├── .gitignore
├── .env.example
└── README.md
```

**To actually run it**, open a plain terminal (not the agent tool) and follow the exact commands the generated `starter/README.md` gives you, e.g.:

```bash
cd output/campus-event-rsvp/starter/frontend && npm install && npm run dev
cd output/campus-event-rsvp/starter/backend && npm install && npm start   # then visit /api/health
```

You'll see the frontend dev server URL (e.g. `http://localhost:5173`) printed in that terminal — open it in a browser to see the generated hello-world page.

### Step 5 — Design system and pitch deck

```
/design-system
/pitch-deck
```

Writes `output/campus-event-rsvp/design-system.md` (component inventory + real contrast ratios, matching the ADR's frontend stack) and `output/campus-event-rsvp/pitch-deck.md` (a 5-slide demo-day outline). Both are plain Markdown — open them in any editor same as the rest.

### One-shot alternative

All five steps above can run in one command instead of separately:

```
/jumpstart a simple event RSVP tool for a campus org 6 4 beginner
```

This produces `intent.json`, `prd.md`, `mvp-scope.md`, `flow.json`, `user-flow.md`, `adr.md`, `starter/`, `design-system.md`, and `pitch-deck.md` in one go. Drop the trailing `6 4 beginner` and it still produces everything **except** `mvp-scope.md` — `starter/` and `pitch-deck.md` are never opt-in inside `/jumpstart`, only `mvp-scope.md` depends on whether you passed `hours` + `team_size`.

---

## Expected output in detail

A finished `/jumpstart <idea> <hours> <team_size>` run looks like:

```
output/
└── barangay-health-appointment-app/
    ├── intent.json
    ├── prd.md
    ├── mvp-scope.md          # only when hours + team_size were given
    ├── flow.json
    ├── user-flow.md
    ├── adr.md
    ├── starter/
    ├── design-system.md
    ├── pitch-deck.md
    └── run.json
```

A slice of a generated `user-flow.md`:

```markdown
# User Flow — Barangay Health Appointment

> A resident books a health-center appointment and receives confirmation.

**Legend:** `1.` step in sequence · `→` leads to · `⤷` decision branch

## Stage: Booking

3. **Pick service** — Resident chooses the type of consultation needed.
   → **Slot available?**
4. **Slot available?** — System checks the schedule for open slots that day.
   - ⤷ *Yes* → **Confirm booking** — slots open today
   - ⤷ *No* → **Join waitlist** — fully booked
```

The PRD's feature specs use `Given/When/Then` acceptance criteria and a WCAG 2.2 AA table; the ADR includes a per-layer stack table with alternatives and a phased blueprint mapping `FR-xx` ids; the design system states real contrast ratios for each color pair and lists design tokens matching the ADR's frontend stack.

---

## Models — fully agnostic

**No model is hardcoded anywhere.** `opencode.json` has no `model` field and no agent frontmatter has a `model:` line, so every agent inherits **whatever model you select in your tool**. This is deliberate — it prevents "model not found" errors from stale or region-locked free-tier model IDs.

- **OpenCode:** pick a model with `/models`, or set a default in `~/.config/opencode/opencode.json`.
- **Pin specific models (optional):** add a `model:` line back to individual agent frontmatter and/or a project `"model"` in `opencode.json`, using IDs your account can actually reach.

---

## Directory structure

```
DEVCON-community-agent-kit-ver2/
├── README.md
├── PRD.md                                    # this project's own Product Requirements Document
├── scripts/install.sh                       # symlinks core + a tool's adapter INTO the repo root
├── core/                                     # ---- PORTABLE BRAIN (tool-agnostic) ----
│   ├── AGENTS.md                             # shared house rules
│   └── skills/
│       ├── workflow-orchestration/SKILL.md   # runs the full pipeline single-agent if needed
│       ├── prd-authoring/                    # SKILL.md + references/prd-template.md
│       ├── flow-graph/                       # SKILL.md + references/flow-schema.json + scripts/render_flow.py
│       ├── mvp-scoping/                      # SKILL.md + references/mvp-scope-template.md (optional, /scope)
│       ├── starter-scaffold/                 # SKILL.md + references/stack-schema.json + scripts/ (optional, /starter-code)
│       ├── pitch-deck-authoring/              # SKILL.md + references/pitch-deck-template.md (optional, /pitch-deck)
│       ├── adr-authoring/SKILL.md
│       └── design-system-authoring/SKILL.md
├── adapters/                                 # ---- THIN PER-TOOL WIRING ----
│   ├── opencode/    opencode.json + .opencode/{agent,command}/
│   ├── claude-code/ .claude/{agents,commands}/
│   └── cursor/      .cursor/{rules,commands}/
└── output/                                   # generated runs land here as output/<slug>/
```

---

## Extending the kit

**Add a capability** (e.g. a test-plan generator):
1. Add `core/skills/<name>/SKILL.md` (the reusable know-how + any scripts/schemas).
2. Add an agent to each adapter (`adapters/<tool>/…`).
3. Add a command (`<name>.md`) to each adapter.
4. Add one step to the `workflow-orchestration` skill.

Nothing existing changes.

**Add a tool** (e.g. Codex):
1. Create `adapters/<tool>/` with that tool's agent/command schema.
2. Add an `install_<tool>` case to `scripts/install.sh`.
3. Run it. The `core/` brain never changes.

---

## Troubleshooting

- **Commands (`/jumpstart`, `/adr`, …) don't appear.** You opened the tool in a folder without its config. Run `./scripts/install.sh <tool>` from the repo root, then open the tool **at the repo root**. Tools read their config at launch and don't hot-reload — fully quit and relaunch after installing.
- **"Model not found."** The kit ships model-agnostic, so this comes from a stale selection or a hardcoded model you added. Pick a model your account can reach (`/models` in OpenCode). Fully quit and relaunch — running servers cache old config.
- **Broken symlinks after cloning.** The root `.opencode/.claude/.cursor` links are generated. Run `./scripts/install.sh <tool>` (or `all`) once after cloning to (re)create them.
- **Flow renderer prints `FLOW GRAPH INVALID`.** The `flow.json` violates the schema (e.g. a decision with fewer than two labeled branches). Fix the JSON per the printed message and re-run the renderer.

---

Built for the DEVCON community. Idea in, blueprint out.
