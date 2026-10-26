---
name: record
description: >
  Synthesize and record the current work session as a structured note in the active project's
  Sessions/ folder, and keep the project's CLAUDE.md summary current. Use whenever the user
  says 'record this', 'save this session', 'log this', 'capture this session', or invokes
  /record.
---

# Record

This skill captures the substance of the current Claude session and writes it to the connected
vault as a structured note. The goal is to reduce context rot: when starting a new session, the
user (or Claude) can read the note to restore context quickly. It pairs with `resume`, which
reads these notes back.

Operate on paths relative to the connected vault root — never hardcode an absolute path, since
this skill ships to a different vault on every install. Engagement folders live at the vault
root (`Projects/`, and `Clients/`/`Partners/` if the vault opted into them); each holds a
`Sessions/` subfolder.

Tools needed: Read, Write, Edit, Glob. Use Glob for existence checks and Write/Edit for file
creation — not Bash `mkdir`/`ls`. Bash's sandbox mount path and the Read/Write/Edit path diverge
inside a Cowork session, so a Bash listing or write can silently target the wrong place. Get the
current date/time via Bash only (`date`), never for file operations.

## What to capture

The note should be a synthesis, not a transcript. Focus on what would actually help someone
(including a future Claude) pick up where this session left off. Specifically extract:

- **Session topic**: One-line description of what the session was about
- **Key decisions**: Things that were concluded, chosen, or agreed upon
- **Pushback**: Ideas or directions that were raised and rejected — and *why*
- **What landed well**: Elements the user responded positively to or flagged as important
- **What didn't land**: Approaches or framings that didn't resonate
- **Key components / building blocks**: The critical pieces that are central to the project
- **Open threads**: Things that were raised but not resolved — questions, next steps, loose ends
- **Context for next session**: A brief paragraph that could be pasted at the start of a new
  Claude session to restore context quickly

## Workflow

### Step 1: Infer the project

Look at the conversation to determine what project or topic it relates to. Then use `Glob` to
list the vault's top-level structure (`*/`) and identify the matching project folder. Check the
engagement roots that exist in this vault: `Projects/`, `Clients/`, `Partners/` (and their
subfolders).

If the project folder is ambiguous, check whether there's a plausible match — partial name
matches are fine. If no match exists, place the note in a `Sessions/` folder at the vault root.
Tell the user what folder you're using.

### Step 2: Ensure the Sessions subfolder exists

Check whether a `Sessions/` subfolder exists inside the target project folder with
`Glob("[Project Folder]/Sessions/*")`. If it doesn't exist yet, writing the first note there
with the Write tool creates the folder implicitly — no separate creation step needed.

The target path is:
```
[Project Folder]/Sessions/[note filename]
```

If no project folder was found:
```
Sessions/[note filename]
```

### Step 3: Generate the note filename

Use this format:
```
YYYY-MM-DD HH-MM - [Short Topic].md
```

Example: `2026-03-25 14-30 - Pricing Page Rework.md`

Get the current date and time via Bash (`date`) if needed. Keep the topic slug short (3–5 words),
title-cased, derived from the session content.

### Step 4: Write the synthesized note

Use the Write tool to create the note. Structure it as follows:

```markdown
---
date: YYYY-MM-DD
time: "HH:MM"
project: "ProjectName"
type: session
---

# [Session Topic]

*Recorded: [Full date and time] | Project: [Project folder name or "General"]*

## Context for next session

[2–3 sentence paragraph that could be pasted at the start of a new Claude session to
immediately restore context. Should mention what we're building/exploring, where we got to,
and what the most important unresolved question is.]

## What we worked on

[1–2 sentences: the goal and what was actually done in this session.]

## Key decisions

[What was concluded or agreed upon. Be specific — not "we discussed X" but "we decided X
because Y".]

## What didn't work

[Ideas or directions that were explored and rejected, with the reason. This is valuable
context: it prevents relitigating the same ground in future sessions.]

## What landed well

[Framings, approaches, or outputs that the user responded positively to or flagged as
important. Omit if nothing notable.]

## Open threads

[Unresolved questions, next steps, and things that were raised but not concluded. Use a simple
list format here. Omit if nothing open.]

## Related

- [[ProjectName]]
```

### Step 4.5: Update project CLAUDE.md

Maintain a living `CLAUDE.md` file inside the project folder that gives any future Claude
instance fast orientation on the project's current state — without reading all session notes.

**Path:** `[Project Folder]/CLAUDE.md`

#### If CLAUDE.md does not exist

Generate it from scratch. Read the main project file(s) and existing session notes in
`Sessions/`, then synthesize into this structure and write it with the Write tool:

```markdown
last-updated: YYYY-MM-DD

# [ProjectName]

## What this project is
[2-3 sentences: what, for whom, key dates/deadlines]

## Current state
[What's the project status right now — what's done, what's in progress]

## Settled decisions
- [Decisions that are locked and should not be revisited]
- [Include brief reasoning for non-obvious ones]

## Ruled out
- [Approaches explicitly rejected, with why — prevents re-exploring dead ends]

## Active constraints
- [Technical, timeline, licensing, or other constraints shaping the work]

## Key files
- `filename.md` — what it is and when to read it
- [Only list files that matter for orientation, not everything]

## Open threads
- [Unresolved questions or next decisions needed]
```

#### If CLAUDE.md already exists

Read the current CLAUDE.md and compare against this session's decisions and changes. Apply
incremental edits using the Edit tool:

- Add new settled decisions from this session's "Key decisions"
- Move resolved open threads out (to settled decisions or remove)
- Update "Current state" if the project status meaningfully shifted
- Add newly ruled-out approaches from "What didn't work"
- Update "Key files" if new important files were created
- Bump `last-updated` to today
- **Do not rewrite sections that didn't change** — touch only what's new

#### When to skip

If nothing in this session meaningfully changes the project's current state (e.g., a minor
exploratory conversation with no new decisions), skip the CLAUDE.md update entirely.

---

### Step 5: Confirm

After writing, tell the user:
- The full path of the note in the vault
- Project CLAUDE.md status: [updated / created / no changes needed]
- A brief summary of what was captured (1–2 sentences)

Don't reproduce the full note in the chat — just confirm it's saved and where.

## Quality bar

The note should be useful to someone who wasn't in the session. Avoid vague entries like
"discussed the approach" — be specific about *which* approach and *what* was decided about it.
The "Context for next session" section is the most important: it should be immediately usable
as session-opening context for a new Claude conversation.
