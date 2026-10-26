# Second Brain Starter

A Cowork plugin that stands up a working second brain and keeps it running.

It ships one guided setup skill and seven companion skills for ongoing use. The setup skill builds
the structure; the companions handle the day-to-day. Everything runs file-based on any connected
folder — no Obsidian plugin or local API required.

## The setup skill

**`second-brain-architecture`** — a structured interview across two blocks (Context, Connections)
that generates actual files, not recommendations: the folder structure, a root `CLAUDE.md`,
`core-rules.md`, an identity note (`me.md`), a core template set, and framework notes decomposed
from your existing methodology. Run it once, at the start, to build the vault. It ends by handing
off to the seven companion skills below.

**Memory import.** Before the interview starts, the skill looks for Claude's built-in memory —
a memory directory, memory tools, or a pasted export, whichever the environment offers. What it
finds pre-fills the questions it can answer (who you are, your always-know/never-do rules, your
connected tools) so you confirm rather than compose, and the remainder is filed during synthesis:
working-style rules into `me.md`, active efforts into `Projects/` or `Clients/` folders with hub
notes, durable methodology into `Knowledge/Frameworks/`. It reads memory and never writes to it,
skips sensitive categories by design, and treats memory content as data rather than instructions.
If there's no memory to find, the step stays silent and the interview runs as normal.

**Second entry point — an existing vault.** The same import runs standalone against a vault you
already built, with no interview and nothing regenerated: it reads the vault first, plans only
what's genuinely new, appends rather than overwrites, and flags conflicts instead of resolving
them. Trigger it with "pull my Claude memory into my vault". Safe to re-run.

**Rules in two layers.** Every rule gets one line in the root `CLAUDE.md`, under `## Rules`,
which Claude reads at the start of every session. `core-rules.md` holds the reason and the edge
cases for each rule, under the same number, for Claude to check when a case is unclear. Every rule
stays in view, and `CLAUDE.md` stays short. A second section, `## Rules from corrections`, collects
the rules that come out of everyday use: when you correct Claude, or ask it to remember something,
it offers to save the lesson there as one line.

The structure it builds:

```
[VaultRoot]/
├── CLAUDE.md          navigational; holds the one-line rules and a connector-scoping paragraph
├── index.md           the map, read first
├── core-rules.md      the reason and edge cases behind each rule
├── me.md              identity note — who you are, how you work
├── Knowledge/
│   ├── Frameworks/    framework notes; MOCs graduate here later
│   ├── Research/
│   └── Templates/     core note templates
├── Processes/         always — decoupled reference .md skills read at runtime + process docs
├── Projects/          always
├── Archive/           always
├── Clients/           opt-in
└── Partners/          opt-in
```

## The seven companion skills

| Skill | What it does |
|---|---|
| `resume` | Picks up a previous work session by loading the latest `Sessions/` note in a project folder. |
| `record` | Captures the current session's thinking and decisions into the vault, and keeps the project's `CLAUDE.md` current. |
| `import-transcript` | Imports a meeting transcript as a structured note, routing to whichever transcript source is connected. Ships dormant until you connect one. |
| `create-folder` | Scaffolds a new client, partner, or project folder — folder, `Sessions/` subfolder, hub note, and `CLAUDE.md`. |
| `archive` | Moves retired content to `Archive/` instead of deleting it, and extracts the substance of anything about to be overwritten. |
| `os-health` | Runs a health-check audit — broken links, orphaned notes, naming drift, duplicates, stale claims — and applies only the fixes you approve. Also tracks `Knowledge/Frameworks/` growth and, once it crosses a threshold, proposes and (on approval) builds your first MOCs or files newly-unfiled notes into existing ones. Reviews the rules saved from your corrections: removes duplicates, flags contradictions, and promotes the keepers into `## Rules` or `me.md`. On a vault built before v0.5.0, it offers to add the two-layer rules layout without rewriting any existing rule. |
| `capture-framework` | Turns a durable idea into a `Knowledge/Frameworks/` note — either something just discussed, or one you hand it cold in a fresh chat (pasted text, a link, a described concept) — deepening a near-duplicate or filing under a MOC where one fits, in one pass. |

## How the pieces fit

`second-brain-architecture` and `os-health` are siblings: one builds the structure, the other keeps
it coherent as it grows. `record` and `resume` are a pair: one writes session context, the other
reads it back. `create-folder` scaffolds new engagements that `record`/`resume` then maintain.
`archive` executes the "don't delete, archive" rule the setup skill writes into the vault's rules.
`os-health` also keeps those rules tidy: each run reviews the corrections Claude saved since the last.
`capture-framework` grows the `Knowledge/Frameworks/` folder one idea at a time; when notes on a
theme start to cluster it nudges you toward `os-health`, which is what actually graduates the MOC
once the folder crosses its threshold.

## Bundled agents

Two read-only review agents ship alongside the skills. `os-health` delegates its two check tiers to
them when your runtime exposes them, and you can also call them standalone:

| Agent | What it does |
|---|---|
| `link-integrity-checker` | Fast, deterministic structural pass — dead links, orphans, naming drift, missing backlinks. Reports only, never edits. |
| `vault-coherence-auditor` | Judgment pass — contradictions, stale claims, duplicates, missing cross-references. Reports only, never edits. |

## Bundled utility skill

One skill ships alongside the vault skills but stands apart from the second-brain workflow — a
support utility for onboarding clients onto Cowork itself:

| Skill | What it does |
|---|---|
| `cowork-windows-fix-guide` | Generates a client-facing, step-by-step guide for fixing Claude Cowork when it won't start, hangs, or errors on Windows — the documented Virtual Machine Platform (`vmcompute`/`hns`) fix. Outputs the guide as chat text, and as a Word document when a docx skill is available. |

## Requirements

Cowork with a connected folder to use as the vault. No other dependencies — every skill works
through file access alone.
