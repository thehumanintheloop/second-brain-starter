---
name: resume
description: >
  Resume a previous work session by loading context from the latest Sessions/ note in a
  project folder. Use this skill whenever the user says 'resume', 'pick up where we left off',
  'continue from last time', 'what were we working on', 'load context for [project]', or
  invokes /resume. Also trigger when the user points at a folder or names a project and
  implies they want to continue prior work rather than start fresh. Always use this skill
  rather than asking the user to manually summarise the previous session — that's exactly what
  this skill is for.
---

# Resume

This skill reads the most recent session note from a project's `Sessions/` folder and
immediately restores context — surfacing what was decided, what's in progress, and what to
work on next. The goal is zero-friction continuation: the user shouldn't have to re-explain
anything that was already captured.

Operate on paths relative to the connected vault root — never hardcode an absolute path, since
this skill ships to a different vault on every install. The vault's engagement folders live at
the root (`Projects/`, and `Clients/`/`Partners/` if the vault opted into them); each holds a
`Sessions/` subfolder and a hub note.

Tools needed: Read, Glob, Grep. Use Glob to list and locate folders, not Bash — Bash's sandbox
mount path and the Read/Write/Edit path diverge inside a Cowork session, so a Bash listing can
silently miss files the file tools can see.

## When to use this skill

Any time the user wants to continue work on a project that has prior session notes. The trigger
is intent to continue — not necessarily the word "resume". If they say "let's get back to the
pricing work" or "where were we on the launch plan?", this skill should fire.

## What the user provides

One of the following:
- A **folder path** pointing at a project directory (e.g., `Projects/Website Rebuild/`)
- A **project or topic name** (e.g., "the pricing engagement", "the launch plan")
- Nothing explicit — if the current Cowork session already has a mounted folder, use that

If they provide a path, use it directly. If they provide a name, search the vault for a matching
project folder. If nothing is provided, use the currently mounted workspace folder.

## Step 1: Find the project folder

**If a path was given:** navigate directly to it.

**If a name was given:** search the vault for a folder whose name matches the topic. Check the
engagement roots that exist in this vault:
- `Projects/`
- `Clients/` (if present)
- `Partners/` and its subfolders (if present)

Use `Glob` (e.g. `**/[Name]*/`) for the match. Fuzzy matching is fine — "launch" matches
`Projects/Product Launch/`. If there are multiple plausible matches, briefly list them and ask
the user to confirm before proceeding.

**If nothing was given:** use the mounted workspace folder (check the Cowork directory context).

## Step 2: Find the most recent session note

List the contents of `[ProjectFolder]/Sessions/` with `Glob`. Sessions are named
`YYYY-MM-DD HH-MM - Topic.md`. Sort by filename (lexicographic = chronological) and take the
most recent.

If there is no `Sessions/` folder or it's empty, say so clearly and offer to either:
- Check the project's `CLAUDE.md` instead (it may contain a living summary)
- Start a fresh session

## Step 3: Also check the project CLAUDE.md

If `[ProjectFolder]/CLAUDE.md` exists, read it too. It's a living summary that the `record`
skill maintains — it often has a crisper picture of current state than the session note alone.
Use both: the session note for specifics of the last interaction, the CLAUDE.md for the broader
project state.

## Step 4: Synthesise and surface

Don't just paste the session note. Read it and produce a **resumption brief** — a concise,
actionable summary that answers:

1. **What we're working on**: One or two sentences on the project and the active task
2. **Where we got to**: The concrete end-state of the last session — what was finished, what was
   in-flight
3. **Key decisions in force**: The 3–5 most consequential settled decisions that shape current work
   (don't list every historical decision — only those that are load-bearing right now)
4. **Open threads**: The unresolved questions and next steps from the last session
5. **Suggested next actions**: Based on the open threads and session context, offer 2–3 concrete
   things to do now — in priority order

Format this as a conversational summary in chat, not a document. Prose over bullets where
possible. Keep it tight — the goal is to restore context in 30 seconds, not to reproduce the
session note verbatim.

End with a direct question or prompt that invites the user to pick back up: e.g.
"The most pressing open thread was [X] — want to start there, or is there something else on
your mind?"

## Quality bar

The brief should pass this test: if someone who wasn't in the last session read it, they could
immediately contribute usefully to the work. Vague entries like "discussed the approach" fail
this bar. Be specific — name the decision, name the option that was rejected and why.

The suggested next actions should feel like a natural continuation, not generic advice. They
should come from the open threads and context in the note, not from general knowledge about the
project type.

## Edge cases

**Multiple recent sessions on the same project:** If there are several sessions from the same
week, skim the most recent 2–3 to build a fuller picture before synthesising. Older sessions
are lower priority unless a key decision there hasn't been superseded.

**Session note is sparse or low-quality:** If the last session note is thin (e.g., just a
brief paragraph with no clear decisions), say so, read the CLAUDE.md as primary source, and
note that the brief may be less precise than usual.

**No prior context at all:** If there are no session notes and no CLAUDE.md, say so clearly.
Offer to read other project files (a main project note, a brief, etc.) and synthesise from
those instead.
