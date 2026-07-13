---
name: import-transcript
description: >
  Import a meeting transcript into the vault as a structured context note, routing to whichever
  transcript or document source is connected (a meeting-notes tool, a cloud-drive doc, or pasted
  text). Use when the user says 'import this meeting', 'add transcript', 'save meeting context',
  'log this call', or invokes /import-transcript. Optionally accepts a meeting link, a document
  URL, a transcript ID, or a meeting keyword as an argument.
---

# Import Meeting Transcript

Imports meeting context into the connected vault as a structured, scannable note.

This skill is **source-agnostic and ships dormant**. It doesn't assume any particular
transcription tool. At runtime it routes to whatever source the user actually has connected — a
meeting-notes MCP (Fireflies, Otter, Fathom, Granola, etc.), a cloud-drive doc (Google Drive,
OneDrive, Dropbox), or transcript text pasted directly into the conversation. If no transcript
source is connected and nothing is pasted, say so plainly and offer to file pasted text instead.

Operate on paths relative to the connected vault root — never hardcode an absolute path, since
this skill ships to a different vault on every install. Tools needed: Read, Write, Edit, Glob,
plus whatever connector MCP supplies the transcript. **Connector tools are usually deferred —
load them via ToolSearch before calling them** (search a keyword like the tool's name or
"transcript" / "meeting" / "drive").

---

## Step 1: Identify the meeting and the source

Determine two things:

1. **Where the content lives** — a connected meeting-notes tool, a connected cloud drive, or
   pasted text in the conversation.
2. **Which meeting** — a link, an ID, a keyword, or a date.

**Source routing:**

| Input | Action |
|---|---|
| A meeting-tool link or bare transcript ID | Route to the connected meeting-notes MCP |
| A cloud-drive document URL | Extract the file ID and route to the connected drive MCP |
| Transcript text pasted into the chat | Use it directly — no connector needed |
| A keyword or date, source unclear | Search whatever transcript sources are connected; present results; ask the user to confirm |
| No input at all | Ask: "Which meeting? Give me a name, date, a link, or paste the transcript." |

To discover what's available, check which connectors are present this session (a registry search
or the tool list). Don't assume a specific product — match by category (meeting notes, cloud
drive) and use whatever is connected.

---

## Step 2: Retrieve the meeting data

Once the source is known, load its tools via ToolSearch, then pull the content.

- **Meeting-notes MCP:** fetch the summary/overview first; if it lacks detail on discussion
  points, fetch the sentence-level transcript too. If you only have a keyword or date, list
  matching meetings (title + date), have the user confirm, then fetch the chosen one.
- **Cloud-drive doc:** if you have the file ID, read it directly; if only a keyword, search
  (up to 5 results, name + last-modified), confirm, then read.
- **Pasted text:** work straight from what's in the conversation.

Extract these fields regardless of source:
- **Title** — the meeting title
- **Date** — ISO date if available
- **Attendees** — participant names/emails if available
- **Overview** — 2–3 sentence summary
- **Key discussion points** — 4–8 substantive topics
- **Decisions made** — things explicitly resolved or agreed
- **Action items** — named actions with an owner where attributable
- **Open questions** — topics left unresolved
- **Source reference** — the link or a short note of where it came from ("pasted transcript")

---

## Step 3: Determine the destination folder

Infer from the meeting content and attendees, or ask. Route to the engagement roots that exist
in this vault:

| Meeting type | Destination |
|---|---|
| Direct client meeting | `Clients/[ClientName]/Sessions/` |
| Partner-sourced client meeting | `Partners/[PartnerName]/[ClientName]/Sessions/` |
| Internal project | `Projects/[ProjectName]/Sessions/` |
| General / no clear project | `Meetings/` (at vault root) |

Only use `Clients/` or `Partners/` if those roots exist in this vault. If uncertain, ask:
> "Which project or client is this meeting for? (Options: [inferred list from attendees/title], or 'general' to file it in Meetings/)"

---

## Step 4: Write the note

**Filename convention:**
- Inside `Sessions/`: `YYYY-MM-DD HH-MM - [Topic].md` (topic = 3–5 word slug from the title)
- Inside `Meetings/`: `YYYY-MM-DD Meeting with [Name].md`

**Template:**

```markdown
---
date: YYYY-MM-DD
attendees: [Name1, Name2]
project: "[ProjectName or General]"
type: meeting
source: "[source category — e.g. meeting-notes, cloud-drive, pasted]"
source-id: "[transcript ID or file ID, if any]"
source-url: "[link, if any]"
---

# Meeting — [Title]

*[YYYY-MM-DD] | Attendees: [comma-separated list]*

## Overview

[2–3 sentences: what the meeting covered, the main outcome, and why it mattered.]

## Key discussion points

- [Substantive topic 1]
- [Substantive topic 2]
- [Continue for all main topics — aim for 4–8 bullets]

## Decisions made

- [Decision — reasoning if non-obvious]

## Action items

- [ ] [Action — Owner if attributable]

## Open questions

- [Unresolved question to carry forward]

## Source

- **[Source]:** [link or short note of where it came from]

## Related

- [[ProjectName]]
```

Write the note with the Write tool.

**Synthesis rules:**
- Key discussion points: substantive topics only, not meta-conversation or small talk
- Decisions: only things explicitly agreed, not maybes
- Action items: only named actions with an implicit or explicit owner
- Open questions: things that came up but were explicitly left unresolved
- Do not reproduce large chunks of the transcript verbatim — synthesize
- If a meeting has no real decisions or action items, omit those sections rather than leaving them empty

---

## Step 5: Update project context (if session-filed)

If the note was filed in a `Sessions/` folder (not general `Meetings/`):

1. Check with Glob whether a `CLAUDE.md` exists in the parent engagement folder.
2. If it does: read it, and if the meeting surfaced new decisions or changed status, append them
   to `## Settled decisions` or update `## Status`. Use Edit, not Write, and bump `last-updated`.
3. If no CLAUDE.md: skip — a meeting note alone doesn't warrant creating one from scratch.

---

## Step 6: Wrap up

Print:

> **Imported.**
> - Note: `[path to note]`
> - Source: [link or short note]
> - Project: [[ProjectName]] *(or "General — filed in Meetings/")*
> - CLAUDE.md: [updated / no changes needed / skipped — general meeting]

---

## Tone

Efficient and precise. The goal is a note the user can scan in 30 seconds to understand what
happened in the meeting and what needs to happen next. Do not narrate the process — just execute
and confirm.
