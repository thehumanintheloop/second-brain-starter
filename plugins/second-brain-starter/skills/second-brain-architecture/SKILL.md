---
name: second-brain-architecture
description: >
  Run this skill to build a client's second brain architecture from scratch. Conducts a
  structured interview across two question blocks (Context, Connections), then generates
  a working folder structure, root CLAUDE.md, core-rules.md, an identity note, a core
  template set, and any durable framework notes — actual files, not recommendations. Use
  at the start of any new client engagement (typically Session 2 of a structured AI
  onboarding sequence) or whenever a user needs to set up their knowledge architecture in
  Cowork. Trigger on:
  "build my second brain", "set up my vault", "second brain architecture", "set up my
  folder structure", "create my CLAUDE.md", "knowledge architecture", or any request to
  start a new Cowork setup from scratch.
---

# Second Brain Architecture Skill

A guided interview that builds a client's second brain architecture — actual files — in a
single session. The client answers two question blocks; Claude synthesises the answers
and creates the root CLAUDE.md, core-rules.md, identity note, folder structure, core
templates, and any durable framework notes. Setup ends by handing over seven pre-installed
skills — a working system, not a reading list. The session facilitator then explores
what's been created with the client and adjusts live.

Designed for Session 2 of a structured client onboarding sequence. Reusable for any facilitator, agency, or consultancy running this kind of session.

---

## Before you begin

Read the following context before running the interview:

**The 4Cs framework (Nate Herk):**
- **Context** — evergreen, locked-in data worth keeping for a year. Decisions made, business status, frameworks, outcomes. Lives *inside* the second brain as files.
- **Connections** — live, frequently-changing data (Slack, email, CRM, calendar). Should NOT be stored in the second brain — it becomes noise. Instead, the second brain knows *where to look* and retrieves this on demand via connectors. Connections = access routes, not storage.
- **Capabilities** — recurring outputs and workflows; what can be templated or automated.
- **Cadence** — timing and triggers; when information flows.

**Session 2 scope:** Context and Connections only. Capabilities and Cadence are not covered
here — there's no real usage yet to draw on. They're the subject of Session 3, once the
client has worked in the vault for a week.

**Design principle — guided spine, adapt at the edges.** The folder skeleton, core rules, and
note conventions come from a fixed, proven baseline. The interview personalises the edges
(names, which optional folders, tone, durable frameworks) — it does not invent the skeleton
from an open question.

**The CLAUDE.md structure to generate** (lean, navigational — not a knowledge dump):
```
# [Vault Name]

## What this is
[Who the person is, what this vault holds, what it's for — 2–3 sentences]

## How to navigate
[Where to start, which files to read first and when — concise pointers]

## Vault structure
[Folder tree as a code block, one-line description per folder/file]

## Operational notes
[Naming conventions, standing rules, what not to do — lean pointers only, plus the
connector-scoping paragraph from Block 2]
```

---

## Interaction model

**Use `AskUserQuestion` for every question that has a defined option set.** It renders as clickable options in the chat and reliably returns the answer to Claude — unlike a display widget, whose buttons don't send events back (Cowork's widget sandbox doesn't pass button clicks to Claude, only `AskUserQuestion` does, since it's a first-class tool call rather than rendered HTML). Use plain chat text only for genuinely open-ended questions, where no fixed option set makes sense.

**Rules:**
- Defined option set → `AskUserQuestion` with those options. The tool always adds an "Other" free-text option automatically — don't add your own "Other" or "Something else" option to the list.
- Multiple answers allowed → set `multiSelect: true`.
- Purely open-ended, or the answer needs elaboration a popup can't capture → ask in plain text in chat instead. `AskUserQuestion` requires at least 2 options, so it isn't the right tool for these.
- After each answer, acknowledge it in one sentence, then move straight to the next question — no filler paragraphs, no re-explaining what's coming.
- Collect all answers silently in memory. File creation happens only after Block 2 is complete (see Synthesis).
- Run questions strictly in sequence — one at a time, wait for the answer before firing the next `AskUserQuestion` call. Don't batch multiple calls in one turn.

---

## Step 0 — Folder setup

**This runs first, before any interview questions.** Claude needs a destination folder before generating anything.

Ask in the chat:
> "First, let's set up where your vault will live. Select a folder — this can be a new empty folder you've created anywhere (Desktop works fine), or an existing location you want to build inside."

Then call `mcp__cowork__request_cowork_directory` (load via ToolSearch if it's deferred) to open the native folder picker. Wait for the user to select a folder.

Once connected, confirm in one sentence: *"Got it — I'll build your vault inside [folder name]."* Then move straight to Block 1.

All files and folders created during synthesis go inside this connected directory. Note that this folder-picker step names the *location*; the vault-name question at the end of the interview names the *vault* itself — the folder the client picked may already have its own name.

---

## Block 1 — Context

*Feeds: root CLAUDE.md, core-rules.md, me.md, top-level folder structure, framework notes.*

Open with one sentence in chat:
> "We're building the foundation of your second brain — the layer that tells Claude who you are, how you work, and where everything lives. Every answer here loads automatically before you type a single word in Cowork."

Then run these in order:

**1a. Role and identity**
```
AskUserQuestion:
  question: "What is your primary role?"
  header: "Your role"
  multiSelect: false
  options:
    - label: "Founder / CEO"
      description: "You run the organisation or a significant part of it"
    - label: "Consultant / Advisor"
      description: "You work across multiple clients or engagements"
    - label: "Practitioner / Specialist"
      description: "You own a functional domain (e.g. marketing, product, ops)"
    - label: "Researcher / Analyst"
      description: "Your primary output is insight, synthesis, or analysis"
```

Then ask in plain text — this needs elaboration a popup can't capture, so it's chat, not `AskUserQuestion`:
> "In a sentence or two, who are you and what do you actually do — the version you'd give someone who's never heard of your work, not a job title?"

Wait for the response. The role select is a quick anchor; this answer is what actually populates
"Who I am" (file 3) and the opening of root CLAUDE.md's "What this is" (file 1) — a role label
alone isn't enough to describe a person.

**1b. The spine**

Present the guided spine as reference text in chat (this is information, not a question):
> "Here's the standard spine every second brain we build starts from:
> ```
> Knowledge/  — always. Frameworks, Research, Templates.
> Projects/   — always. Internal work, no external client.
> Archive/    — always. Retired work moves here, never deleted.
> ```
> Everything else is a fit to how you work."

Then run one `AskUserQuestion` call:

```
AskUserQuestion:
  question: "Do you do external work with clients or through partners?"
  header: "External"
  multiSelect: true
  options:
    - label: "Direct clients"
      description: "You work directly with external clients — opts in Clients/"
    - label: "Partner-mediated clients"
      description: "You deliver client work through a business partner — opts in Partners/"
    - label: "No external clients"
      description: "Everything here is internal work"
```
*(If "No external clients" is selected alongside another option, that's contradictory — ask a quick one-line clarifying follow-up before moving on.)*

Move straight to 1c — no open-ended "what would you change about the spine" question. A first-time client doesn't yet have the context to answer that; the spine is a fixed baseline they adjust later once they've actually worked in it.

**1c. Work-style flags — not asked, pre-packaged**

"Always know" and "never do" are not interview questions. Asking a client to articulate
their working style in the abstract, before they've seen Claude do anything, produces
vague answers ("be direct", "no filler") that don't encode a real preference. Instead,
`me.md` (file 3) ships with one illustrative example in each section, and the client edits
it over time as they notice actual friction — a flag added the moment something bugs them,
not a forecast at setup time.

Seed content:
- **Always know** example: "Avoid excessive em dashes in writing."
- **Never do** example: "Don't guess when a request is ambiguous — ask."

These are placeholders to demonstrate the mechanism — swap them for whatever's actually
true once the client has a real preference.

**1d. Core methodology / USP**

Ask in plain text — this needs elaboration a popup can't capture, so it's chat, not `AskUserQuestion`:
> "What's the core approach or methodology that makes your work distinct — the thing you'd point someone to if they asked 'how do you actually do this?' Give me whatever you've got: a website link, a services or about page, a one-pager, deck text, more than one source if you have them. Paste in as much as you want — I'll split it into separate framework notes for each distinct piece, not one giant note. Nothing written down anywhere? Just describe it in your own words: a name for it if it has one, and its main parts."

Wait for the response, then process whatever comes back:

- **A URL** — fetch it (find a web-fetch tool via ToolSearch if one isn't already loaded; never assume a fixed tool name, it varies by environment). Use the page content as source material.
- **Pasted text** (services page copy, deck text, a one-pager) — use it directly.
- **Described in chat** — use the description as-is; there's no separate written source to fetch.
- **Multiple sources** — process all of them into the same decomposition pass below, not one at a time.

This feeds "Framework notes" (file 7) — see that section for the decomposition and template mechanism. There's no note count to hit here and no six-month follow-up question to ask the client directly; the durability check happens silently during decomposition instead (see file 7).

Confirm before advancing: *"Foundation mapped — moving to Connections."*

---

## Block 2 — Connections

*Feeds: the connector-scoping paragraph in root CLAUDE.md; primes `import-transcript` if a transcript source is named.*

Open with one sentence:
> "Your second brain stores evergreen knowledge — decisions, frameworks, reference material. It doesn't try to hold live, changing data (emails, Slack threads, CRM records) because that becomes noise. Instead, it knows where to look. This block maps those access routes — no files get created from this, just a paragraph in your CLAUDE.md telling Claude where to reach."

**2a. Live data tools**
```
AskUserQuestion:
  question: "Which tools hold your live, changing data — the things you'd want Claude to check for recent conversations, current status, or real-time context?"
  header: "Live data"
  multiSelect: true
  options:
    - label: "Gmail / Google Workspace"
      description: "Email, Drive, Docs, Calendar"
    - label: "Microsoft 365"
      description: "Outlook, Teams, SharePoint"
    - label: "Slack"
      description: "Team conversations and decisions"
    - label: "Notion"
      description: "Team wikis, project tracking, databases"
```

Then ask in plain text: *"Any other tools that hold live data? (e.g. a CRM, Linear, Jira, a calendar tool, something sector-specific) — list them or skip."* Wait for the response.

**2b. What lives where**

Ask in plain text:
> "For each tool you mentioned, what type of data actually lives there? One line per tool is fine — e.g. 'Slack = team conversations and decisions / Notion = project tracker'."

Wait for the response, then move to synthesis.

---

## Synthesis and file generation

After Block 2 is submitted, synthesise all responses and create the files. Do this silently — no step-by-step narration. Present the completed architecture for review once all files are created.

**Ask the user one thing before creating files:**
> "What would you like to call this vault? (e.g. your name, company name, or project name)"

Use this as the root folder name and vault title throughout.

---

### Files to create

**1. Root `CLAUDE.md`**
Follow the lean structure: What this is / How to navigate / Working in subfolders / Vault structure / Operational notes.
- "What this is": synthesise from 1a (role + the who-you-are/what-you-do answer) + the vault purpose
- "How to navigate": point to index.md first; note when to open project (or client/partner) folders vs the root; note that `core-rules.md` and `me.md` sit at root alongside it
- **"Working in subfolders"**: a standing, non-optional directive that forces Claude to pick up local context before acting inside any folder. Emit this section **verbatim** into the root CLAUDE.md — it's the mechanism, not a suggestion:
  ```markdown
  ## Working in subfolders — read the local CLAUDE.md first

  Folders in this vault carry their own `CLAUDE.md` files. Each one holds the local context —
  status, settled decisions, constraints, and what's in the folder — that this root file
  cannot.

  **Before you browse, read, create, edit, or reason about anything inside a subfolder, first
  read that subfolder's `CLAUDE.md` if one exists.** If you are working several levels deep,
  read the `CLAUDE.md` of each folder on the way down, outermost first. This is mandatory, not
  optional: acting inside a folder without having read its `CLAUDE.md` means working without the
  context that folder was set up to give you. When you open a folder and find a `CLAUDE.md`,
  reading it is the first thing you do.
  ```
- "Vault structure": embed the full folder tree as a code block, one line per folder — only the folders actually opted into (see file 4)
- "Operational notes": naming conventions implied by the folder structure, plus **the connector-scoping paragraph** synthesised from Block 2 — one or two sentences naming the connected tools and what each is for, written as prose, not a bullet-per-tool file structure. Point to `me.md` for the always-know/never-do work-style flags rather than duplicating them here.

**2. `core-rules.md`** — trimmed operating rules baseline, verbatim:
```markdown
# Core Rules

1. Do not delete notes — move them to `Archive/` instead.
2. Archive meaningful content before removing or replacing it.
3. `Sessions/` files are append-only — never edit or delete past entries.
4. Context-first before deliverables: save a `.md` context note before generating any output document.
5. Do not create notes outside the folder structure without asking first.
6. Use unique, descriptive filenames vault-wide — wikilinks resolve by filename alone, so two notes sharing a name anywhere in the vault break each other's links.
7. Every engagement folder (client, partner, or project) has a hub note named after the folder. New notes in that folder link back to it.
8. Before working inside any folder, read its `CLAUDE.md` first if one exists — and the `CLAUDE.md` of each parent folder on the way down. Never act on a folder's contents without its local context.
```

**3. `me.md`** — identity note, home for 1a plus the always-know/never-do work-style flags so CLAUDE.md stays lean. The two flag sections ship with one illustrative example each (never blank) — the client edits them over time as real preferences surface:
```markdown
# Me

## Who I am
[1a answer — role + the who-you-are/what-you-do description, written as prose, not a label]

## What Claude should always know
- Avoid excessive em dashes in writing.
- [Add more here as you notice preferences]

## What Claude should never do
- Don't guess when a request is ambiguous — ask.
- [Add more here as you notice friction points]
```

**4. Top-level folder structure**
Create folders matching the tree in CLAUDE.md, built from the guided spine plus the External-work answer in 1b:
```
[VaultName]/
├── CLAUDE.md
├── index.md
├── core-rules.md
├── me.md
├── Knowledge/
│   ├── Frameworks/
│   ├── Research/
│   └── Templates/
├── Projects/
├── Archive/
├── Clients/             (only if opted in)
└── Partners/            (only if opted in)
```
`Knowledge/`, `Projects/`, and `Archive/` are always present. `Clients/` and `Partners/` are included only if the client opted into them in 1b. `Inbox/` and `Daily/` are no longer offered at setup — they're easy to add later (ask Claude, or use `create-folder`) once the client actually knows whether they want a capture/journal layer, rather than guessing at Session 2. `Knowledge/Templates/` is always present and pre-populated (see file 6). `Knowledge/Frameworks/` always exists but ships empty of MOCs — it only holds the framework notes from 1d (file 7); a MOC graduates later, once a cluster of notes on one theme accumulates.

Create a `.gitkeep` placeholder in each empty folder so the structure is visible.

**5. Sub-folder `CLAUDE.md` stubs** — one per opted-in top-level category folder (e.g. `Projects/CLAUDE.md`, `Clients/CLAUDE.md` if opted in)

Every top-level category folder gets its own lean CLAUDE.md. These are instruction files for Claude, not vault content: index the folder's contents by **filename only, in a plain list — never with `[[wikilinks]]`**. Wikilinks pull CLAUDE.md into the note graph as if it were a content note, which it isn't, and it breaks the moment a linked file is renamed.

File path: `[Category]/CLAUDE.md`
```markdown
# [Category Name]

## What this holds
[One sentence — the scope of what belongs in this folder]

## Files in this folder
- `filename.md` — one-line description
- `filename.md` — one-line description
*(Update this list as files are added or removed — filenames only, no wikilinks)*

## Read before working here
[Any file(s) to read first, if applicable — reference by filename, not by link]
```

At setup time these folders are empty, so this stub is mostly scaffolding — it becomes load-bearing once the client starts creating engagement folders inside `Clients/`/`Partners/`/`Projects/` via the `create-folder` skill, which generates the matching hub note per core-rules rule 7.

**6. `Knowledge/Templates/`** — six core note templates, minimal frontmatter so a first-time client isn't overwhelmed:
- `Meeting.md`
- `Project.md`
- `Session.md`
- `Contact.md`
- `Company.md`
- `Document.md`

Each template file is just the frontmatter block plus a couple of empty headers matching that note type — no organisation-specific fields. Keep it minimal: `title`, `date`/`type`-appropriate fields only.

**7. Framework notes** — decomposed from whatever 1d returned (not a MOC)
File path: `Knowledge/Frameworks/[Framework Name].md`

**How many notes:** not a fixed count — driven entirely by what 1d's material actually contains. A one-line description usually yields one note. Richer material (a named methodology plus a multi-part model) yields several, one per genuinely distinct, separable concept — the same atomicity judgment used elsewhere (a note per idea, not a note per paragraph, and not one giant note covering everything). Don't force a target number in either direction.

**Six-month test, applied per piece, not as a question to the client:** while decomposing, silently leave out any part of the submitted material that's a live deal, a pitch to one named prospect, or something tied to current circumstances rather than the underlying method — that belongs in a project/client folder, not here. This replaces the old "is that a live project or a durable way of working?" follow-up question — the check still happens, just inside decomposition instead of as an extra question in the interview.

**Template** — richer than a captured-insight note, since this is imported material with real substance behind it, not a single captured line. Frontmatter carries the note's properties; the body stays plain prose, with no provenance line buried in the text:

```markdown
---
title: [Framework Name]
type: framework
status: active
created: YYYY-MM-DD
last-updated: YYYY-MM-DD
origin: [source]
---

# [Framework Name]

*How [Name/Business] approaches [theme] — the underlying method, not a snapshot of current work.*

## What this is
[1–3 sentences: what this approach or methodology is, in plain terms.]

## Why it's different
[Optional — what distinguishes this from how others approach the same work. Omit if the source material doesn't cover it.]

## Core components
[Optional — the named parts, stages, or pillars, if the approach has them. Omit for a single unified idea with no sub-parts.]

## Related
- [[other-note-from-the-same-submission]] — if this material produced more than one note, cross-link them to each other here
```

`origin:` — the URL if one was fetched, `pasted text` if copied in directly, or `described in conversation` if there was nothing written down. `created` and `last-updated` are the same date at setup time; `last-updated` only moves later, if the note is deepened (e.g. by `capture-framework` — see that skill's Step 2a). `status` starts `active`; later states are an `archive`-skill concern, not something this interview sets.

These six properties — `title`, `type`, `status`, `created`, `last-updated`, `origin` — are the standard frontmatter for every note in `Knowledge/Frameworks/`, regardless of which skill wrote it. This template is specific to setup-generated foundation notes; `capture-framework`'s mid-conversation captures carry the same frontmatter block but a lighter body — see that skill's own SKILL.md.

No MOC file is generated at setup. Once enough framework notes on a related theme exist, a MOC graduates from them — that's a later, organic step, not part of this interview.

**8. `index.md`** — vault map
```markdown
# [Vault Name] — Index

*Claude: read this first when navigating. It maps where everything lives.*

## Active areas
[One line per top-level folder — what it holds, when to open it]

## Frameworks
[One line per framework note in Knowledge/Frameworks/ — what it covers]
```

---

## Setup ends — the seven-skill handoff

Once all files are created, present the seven pre-installed skills, plain — no flags, no tiers, no recommendation cards:

| Skill | What it does |
|---|---|
| `resume` | Picks up a previous work session by loading the latest Sessions/ note in a project folder |
| `record` | Captures session thinking and decisions into the vault as a structured note |
| `import-transcript` | Imports a meeting transcript into the vault as a structured context note (routes to whichever transcript source is connected) |
| `create-folder` | Scaffolds a new client, partner, or project folder — creates the folder, a Sessions/ subfolder, and a hub note |
| `archive` | Moves retired or superseded content to `Archive/`, keeping or cutting links as appropriate |
| `os-health` | Runs a health-check audit on the vault — dead links, orphaned notes, naming drift, duplicates, stale claims |
| `capture-framework` | Turns an idea from the current conversation into a `Knowledge/Frameworks/` note, checking for a near-duplicate to deepen and a MOC to file it under, in one pass |

One line each. No install decisions to make — they're already installed.

---

## After generation — the facilitation prompt

Once all files are created and the six-skill handoff is shown, output this closing prompt for the facilitator:

---

**Architecture built. Here's how to explore it with the client:**

Open the vault together and work through these questions:

1. **CLAUDE.md** — Does this feel right? Is anything missing from "What Claude should always know"? Is there anything in "Never do" that's missing?

2. **Folder structure** — Does this match how you actually work? Are any folders missing? Are any unnecessary?

3. **Framework notes** — Does this capture the actual methodology accurately? Anything missing, or overstated? (These map what you already do — finer-grained notes accumulate later, one at a time, via `capture-framework`.)

4. **Connections** — Does the CLAUDE.md paragraph capture the right data sources? Which should get connected first?

5. **The six skills** — Walk through what each one does. Which will get used first?

Adjust live. The architecture is a 70% draft — the conversation in the room takes it to 90%. The remaining 10% comes from the week of actual use before Session 3.

**Assignment to give the client:** Use the new architecture for at least three real work tasks this week. Note: what the architecture gets right, where it feels missing or clunky, what you had to explain that should have been in the harness already. Those observations open Session 3.

---
