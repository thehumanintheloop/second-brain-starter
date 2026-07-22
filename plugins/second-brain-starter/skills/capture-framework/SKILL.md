---
name: capture-framework
description: >
  Turns a durable idea, distinction, or recurring lens into a Knowledge/Frameworks/ note, in
  one pass — no separate review step. Works two ways: capture something just discussed in the
  current conversation, or open it fresh in a new chat and hand it the idea directly (pasted
  text, a link, or a described concept). Checks existing framework notes and MOCs first:
  deepens an existing note instead of forking a near-duplicate when the idea extends something
  already there, and files the note under a fitting MOC if one exists. Lightweight, single-turn,
  no external sync (no Slack, no scheduled batch). Use when the user says "capture this as a
  framework", "save this as a framework note", "write this up as a framework", "add this to my
  frameworks folder", "turn this into a framework note", or opens a new chat asking to capture a
  specific idea as a framework.
---

# Capture Framework — File a Durable Idea into Frameworks/, in One Pass

Turns a durable idea into a proper `Knowledge/Frameworks/` note and files it under a fitting
MOC in the same pass, if one exists. The idea can come from the conversation you're already in,
or from material the user hands over when they invoke the skill cold in a fresh chat — either
way the steps below are identical once you have the source material. This is the lightweight,
single-pass version of a fuller capture workflow — same core judgment (new note vs. deepen an
existing one, MOC placement), with the batch/sync mode and the Research/-cross-linking step
removed. One trigger, one pass, done.

Tools needed: Read, Write, Edit, Grep, Glob — plus web fetch when a cold-start capture points
at a link (fetch and read it before writing the note). No Bash for existence checks or file
creation — Bash's sandbox mount path and the Read/Write/Edit path diverge inside a Cowork
session, so Glob is the reliable way to check what already exists.

Operate on paths relative to the connected vault root (`Knowledge/Frameworks/`,
`Knowledge/MOCs/`, `index.md`) — never hardcode an absolute path, since this skill ships to a
different vault on every install.

---

## When this runs

Two entry points, same steps once you have the source material. Their explicit ask ("capture
this", "write this up as a framework") already establishes the idea is worth keeping — there's
no worthiness test to apply here (that's Mode B territory in the fuller version, and this skill
doesn't have a Mode B).

- **Mid-conversation** — the user has just said something worth keeping: a distinction they
  explained, a rule of thumb they repeated, a correction they gave more than once, a one-liner
  that landed. The source material is the conversation itself.
- **Fresh / cold start** — the user opens a new chat and invokes the skill with the idea
  attached in the same message: pasted text, a link, or a concept described in a line or two.
  That attached material is the source. This works from an empty conversation — nothing needs
  to have been discussed first.

**Getting the source material:**

- If the invoking message contains the idea (inline text, a pasted passage, a described
  concept), use that as the source — go straight to the steps below.
- If the invoking message points at a **link**, fetch and read it first, then treat its
  substance as the source. Record the URL as the note's `origin` (see Step 2b).
- If the skill is invoked cold with **no idea attached** ("capture a framework for me" and
  nothing else), ask one short question — "What's the idea you want to capture?" — then proceed.
- If it's genuinely unclear what specifically to capture because the conversation covered
  several distinct ideas, ask one short clarifying question before proceeding.

Otherwise, don't ask — just do it and report back.

---

## Step 1 — Check what already exists

Before writing anything:

1. `Glob("Knowledge/Frameworks/*.md")` to see what's there. If the folder is empty or doesn't
   exist yet, skip straight to Step 2 (new note).
2. For a non-empty folder, `Grep` the idea's key terms across `Knowledge/Frameworks/*.md` and
   read any hits that look promising.
3. Judge the match:
   - **Strong match** — the idea deepens, extends, or adds a case to a concept that note
     already covers. Default to updating that note. Forking a near-duplicate file is the
     drift this check exists to prevent, so lean toward updating whenever the connection is
     more than superficial, not only when it's an obvious restatement.
   - **No strong match** — create a new note.

---

## Step 2a — Updating an existing note

Append a new section at the end of the note:

```markdown

## Update — YYYY-MM-DD: <short title for this addition>

[The new idea, synthesised — not pasted verbatim from the conversation. Write it the way the
rest of the note is written.]
```

Bump `last-updated` in the note's frontmatter to today's date. Leave `created`, `origin`, `type`,
and `status` untouched, and leave the rest of the note's body untouched. Don't touch the MOC —
it already lists this note.

---

## Step 2b — Creating a new note

1. Build a kebab-case slug from the idea's title (describing the concept, not a category).
2. Check it's unique vault-wide before writing — wikilinks resolve by filename alone, so a
   name collision anywhere in the vault breaks link resolution: `Glob("**/<slug>.md")`. If it
   collides, adjust the slug.
3. Write `Knowledge/Frameworks/<slug>.md`:

```markdown
---
title: <Title>
type: framework
status: active
created: YYYY-MM-DD
last-updated: YYYY-MM-DD
origin: <conversation | pasted text | described in conversation | the source URL>
---

# <Title>

*How [you/the team] approach[es] <theme> — a durable lens, not a snapshot of current work in
this area.*

## What this is
[Synthesise the idea properly — don't paste the raw conversation excerpt. A clear statement
of the concept, and why it matters.]

## How it's applied
[Optional — leave as a short placeholder if there's not yet a concrete example; expand later
as it gets used.]

## Related
[Optional — leave the header even if empty at first. Add a `[[wikilink]]` here only if this
capture connects to another existing framework note in a genuinely non-obvious way (the
"surprise test" — don't link just because two notes are topically adjacent). Fine to ship
with nothing under it; a future capture on a related theme can fill it in then.]
```

Frontmatter — `title` / `type` / `status` / `created` / `last-updated` / `origin` — is the
standard every note in `Knowledge/Frameworks/` carries, regardless of which skill wrote it.
Set `origin` to match where the idea actually came from: `conversation` when captured from the
chat you're already in, `pasted text` or the source URL when handed the material cold in a fresh
chat, `described in conversation` when the user described the concept to you directly. This
matches the setup skill's framework notes (from `second-brain-architecture`'s 1d,
imported/decomposed material), which carry a real URL, `pasted text`, or `described in
conversation` — see that skill's SKILL.md, file 7. `created` and `last-updated` match at note
creation; `last-updated` is the only property Step 2a below touches when deepening an existing
note.

---

## Step 3 — File into a MOC, if one fits

1. `Glob("Knowledge/MOCs/*.md")`. If nothing comes back, the vault hasn't grown a MOC yet —
   skip this step entirely, silently. That's expected for a young vault; MOCs are meant to
   emerge once framework notes cluster, not exist from day one.
2. If MOC files exist, skim them (and `index.md` if it references themes) for the closest fit
   to this note's theme.
3. **Strong fit** — add the note under the relevant section, in this same pass, as a plain
   `[[wikilink]]` bullet with a short description. A note whose theme belongs under an
   existing MOC section but never gets listed there is the exact failure mode this step
   prevents.
4. **No fit, or ambiguous** — don't force it and don't invent a new MOC unattended, that's a
   bigger structural call than this skill should make on its own. Leave the note unfiled.
5. **Soft cluster nudge** — while you're in `Knowledge/Frameworks/`, if this capture brings
   the count of un-filed notes sharing a clear common theme to three or more, mention it in
   the confirmation (Step 4) as a one-line suggestion — "you've now got three notes on X with
   no MOC — worth running `os-health` to see if it's time to graduate one?" — rather than
   creating the MOC yourself. This skill only ever notices an early cluster forming; `os-health`
   is what actually builds MOCs, once the Frameworks/ count crosses its own graduation threshold
   (see that skill's "MOC coverage" check) — this nudge just points there instead of dead-ending.

---

## Step 4 — Confirm

One or two sentences, not a step-by-step recap: what got captured, where it landed (new note
or an update to an existing one), and whether it was filed under a MOC. Example: *"Captured as
a new note — `plausibility-vs-usability.md` — and filed under MOC - AI Fluency."* Or: *"Added
as an update to `augmentation-vs-automation.md` — it read as a deepening of that one rather
than a new concept."*

---

## Rules

- Note writing and MOC filing happen in the same pass — never leave a note written without
  also deciding on its MOC placement, even if the decision is "no fit, left unfiled."
- Never create a new MOC unattended — surface the suggestion, let the user decide.
- Never skip the new-vs-update check to save time — that check is the entire point of this
  skill over just writing a fresh file.
- Never paste the raw conversation excerpt into the note as if it were the finished concept —
  synthesise it into a clear, standalone statement, the way every other note in the folder
  reads.
- This skill has no batch mode and no external sync (no Slack, no scheduled runs) — it only
  ever runs on explicit request, against a single idea: either one from the current
  conversation or one the user hands it at invocation. That single-idea, one-pass scope is what
  keeps it distinct from the fuller capture workflow.
