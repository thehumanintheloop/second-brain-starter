---
name: create-folder
description: >
  Create a new engagement folder in the vault — client, partner-mediated client, or internal
  project — following the vault's standard architecture. Creates the folder, a Sessions/
  subfolder, a hub note named after the folder, and a CLAUDE.md orientation note. Use whenever
  the user wants to start a new client, partner-mediated client, or internal project — 'create
  a folder for X', 'set up a new client', 'start an internal project', 'new engagement for Y',
  'scaffold a project folder'. Works directly in Cowork with the vault connected.
---

# Create Engagement Folder

Creates a new engagement folder in the connected vault — client, partner-mediated client, or
internal project — and writes an orientation CLAUDE.md plus a hub note. No Cowork
custom-instructions panel to generate or paste; CLAUDE.md is the sole orientation layer.

Operate on paths relative to the connected vault root — never hardcode an absolute path, since
this skill ships to a different vault on every install. Standing rules live once, at the vault
root's `core-rules.md`. This skill doesn't read or duplicate them — CLAUDE.md just points back
to that file.

Tools needed: Read, Write, Edit, Glob. Use Glob for existence checks and Write/Edit for file
creation — never Bash `mkdir`/`ls`. Bash's sandbox mount path and the Read/Write/Edit path
diverge inside a Cowork session, so a hardcoded Bash path silently breaks there.

If the vault isn't already connected in this session, creating the folder will naturally prompt
for access to it — that's expected, not an error.

---

## Step 1 — Engagement type

If not already clear from how the user phrased the request, ask:

> **What type of engagement is this?**
> - **Client** — direct relationship with an external client
> - **Partner + Client** — client work delivered through a business partner
> - **Internal Project** — internal work, no external client

The taxonomy is opt-in. `Projects/` always exists. `Clients/` and `Partners/` exist only if the
vault opted into them at setup. If the user picks a type whose root folder doesn't exist yet,
that's fine — this skill creates the root as part of scaffolding the first engagement of that
type. Confirm briefly if you're about to introduce a new top-level root: "You don't have a
`Clients/` folder yet — want me to create it and add this client inside?"

## Step 2 — Name(s) and path

- **Client**: ask "What's the client/company name?" → `Clients/[ClientName]/`
- **Partner + Client**: ask partner name and client name → `Partners/[PartnerName]/[ClientName]/`
- **Internal Project**: ask "What's the project name?" → `Projects/[ProjectName]/`

Check whether the folder already exists using **Glob** (e.g. `Clients/[ClientName]/**`). If it
exists, confirm with the user before writing anything: "This folder already exists — confirm this
is a new CLAUDE.md setup, not an overwrite?"

## Step 3 — Orientation questions

Ask in one message, numbered:

1. **What is this engagement?** (1–3 sentences — what it is, why it exists, what it's trying to achieve)
2. **Current status?** (1–2 lines)
3. **Open items?** (up to 5 — skip if none yet)
4. **Hard constraints?** (branding rules, platform limits, budget ceiling — skip if none)

Wait for all answers before proceeding.

## Step 4 — Create the folder, hub note, and CLAUDE.md

Create the engagement folder with a `Sessions/` subfolder and two notes. Use Write to create
files (which creates parent folders implicitly) — never Bash `mkdir`.

**a. Hub note** — `[path]/[EngagementName].md`, named after the folder, per core-rules rule 7.
This is the master note new notes link back to. Start it minimal:

```markdown
---
title: [Engagement Name]
type: [client | project]
status: active
created: [today YYYY-MM-DD]
---

# [Engagement Name]

[Answer from Q1 — what this engagement is.]

## Notes

[Links to notes inside this folder accumulate here as work happens.]
```

**b. Sessions/** — create the subfolder (writing the hub note and CLAUDE.md doesn't create it;
place a `Sessions/.gitkeep` or the first session file via Write if you need it to exist now).

**c. CLAUDE.md** — `[path]/CLAUDE.md`:

```markdown
last-updated: [today YYYY-MM-DD]

# [Engagement Name]

## What this is
[Answer from Q1]

## Status
[Answer from Q2]
[Open items from Q3 as checkboxes — omit section if none]
- [ ] item 1

## Settled decisions
[Leave empty — append only as decisions are made]

## Ruled out
[Leave empty — append only as options are ruled out]

## Active constraints
[Answer from Q4 — omit entire section if none]

## Key files
- `[EngagementName].md` — hub note; read first for full context
- `Sessions/` — timestamped work logs; most recent = most current state
- `core-rules.md` (vault root) — standing rules for the vault
```

## Step 5 — Confirm

Report:

> **Created: [Engagement Name]**
> - Folder: `[path]/`
> - Hub note: `[EngagementName].md`
> - Sessions/: created
> - CLAUDE.md: written

Nothing to paste anywhere else. Once the engagement has prior sessions, `resume` picks it back
up and `record` keeps its CLAUDE.md current.

---

## Rules for this skill

- Never overwrite an existing CLAUDE.md or hub note without explicit confirmation.
- Use Glob/Write for existence checks and file creation, not Bash — keeps behavior identical
  across a direct filesystem and Cowork's sandboxed one.
- Every engagement folder gets a hub note named after the folder (core-rules rule 7) — don't
  skip it, later notes are expected to link back to it.
- Create a new top-level root (`Clients/`, `Partners/`) only when the user opts into that
  engagement type and confirms — don't pre-create roots the vault hasn't asked for.
