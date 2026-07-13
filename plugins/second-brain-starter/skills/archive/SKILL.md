---
name: archive
description: >
  Move retired or superseded content into the vault's Archive/ instead of deleting it, and
  extract the meaningful idea from anything about to be overwritten before the edit happens.
  This skill executes the vault's core rule that meaningful content is archived, never deleted.
  Use when the user says 'archive this', 'retire this note', 'move this to the archive', 'this
  is superseded', or when you are about to overwrite, replace, or cut substantial content from
  an existing note and want to preserve what's being lost.
---

# Archive

Retires content into `Archive/` rather than deleting it, and preserves the substance of anything
about to be overwritten. This is the tool behind two core rules of the vault:

1. Do not delete notes — move them to `Archive/` instead.
2. Archive meaningful content before removing or replacing it.

Operate on paths relative to the connected vault root — never hardcode an absolute path, since
this skill ships to a different vault on every install. Tools needed: Read, Write, Edit, Glob.
Existence checks and file creation go through Glob and Write, not Bash `ls`/`mkdir` — the sandbox
mount path and the Read/Write/Edit path diverge inside a Cowork session. The one sanctioned Bash
use is removing an original file after its archived copy has been written and verified (see Mode
A, step 4).

The skill runs in two modes.

---

## Mode A — Explicit archival

The user asks to retire, archive, or supersede a specific note or file.

### Step 1 — Identify the target

Confirm which note. If the reference is ambiguous, use Glob to find candidates and ask which one.
Read it so you know what's being archived and can index it accurately.

### Step 2 — Ensure the Archive scaffold exists

Check with Glob for `Archive/` and `Archive/CLAUDE.md`. `Archive/` ships with the vault; if
`Archive/CLAUDE.md` (the index) doesn't exist yet, create it on first use:

```markdown
last-updated: YYYY-MM-DD

# Archive

Retired and superseded content lives here. Nothing is deleted — it's moved here so it stays
recoverable and its history stays legible. `Concepts/` holds ideas extracted from notes that
were overwritten or heavily cut (see the `archive` skill, Mode B).

## Archived notes

| Note | Archived | Why |
|---|---|---|
```

### Step 3 — Move the note

Write the note's content to `Archive/[original-filename].md`, preserving the filename (wikilinks
resolve by filename alone, so inbound links keep working after the move). If the note is a whole
retired engagement rather than a single file, mirror its folder under `Archive/` instead.

Verify the archived copy exists with Glob before touching the original.

### Step 4 — Remove the original

Only after the copy is confirmed present in `Archive/`, remove the original from its old
location. Use the file-delete tool if one is available; otherwise remove it with Bash `rm` on the
vault-mounted path as the final step. This ordering means a failed removal leaves a harmless
duplicate, never data loss.

### Step 5 — Update the index and inbound links

- Add a row to `Archive/CLAUDE.md`'s table: note name, today's date, one-line reason. Bump
  `last-updated`.
- **Links:** by default, leave inbound `[[wikilinks]]` intact — they still resolve to the moved
  file by filename. If the user wants the note fully retired from active view, offer to annotate
  or remove the inbound references (Grep for the filename across the vault to find them). Keep or
  cut them as the user prefers; don't silently rewrite links either way.

### Step 6 — Confirm

One or two sentences: what was archived, where it landed, whether inbound links were kept or cut.

---

## Mode B — Proactive extraction before a destructive edit

This mode fires on its own, without an explicit "archive" request. Before you overwrite a note,
replace a meaningful section, or cut substantial content as part of some other task, pause and
preserve what's about to be lost — that's core rule 2.

### When it triggers

Any edit that would destroy meaningful content: rewriting a note wholesale, replacing an
argument or framework with a different one, deleting a section that captured a real idea. It does
**not** trigger for routine edits — fixing a typo, tightening a sentence, appending new content,
or updating a status line loses nothing worth keeping.

### What to do

1. Identify the specific idea, framing, or decision the edit would remove — not the whole note,
   just the part with durable value.
2. Ensure `Archive/Concepts/` exists (create it on first use).
3. Write it to `Archive/Concepts/[concept-slug].md`:

```markdown
---
title: [Concept title]
type: archived-concept
status: superseded
created: YYYY-MM-DD
origin: [source note filename, and what replaced it]
---

# [Concept title]

[The extracted idea, written so it stands on its own — enough that someone could revive it later
without the original context.]

## Why it was set aside

[One or two sentences: what replaced it and why, so the archival is legible, not just a dump.]
```

4. Add a row to `Archive/CLAUDE.md` under the archived-notes table (or a `## Extracted concepts`
   section) so the extraction is indexed.
5. Then proceed with the original edit.

Keep Mode B light. It's a reflex that protects against quietly losing good thinking during an
unrelated edit — not a reason to stop and interrogate every change. If in doubt about whether
something is worth extracting, a one-line check with the user is fine ("before I replace this —
want me to archive the old version of the framework first?").

---

## Rules

- Never delete meaningful content outright — Mode A moves it, Mode B extracts it. Deletion
  without archival is the exact failure these two rules exist to prevent.
- Always write and verify the archived copy before removing any original.
- Keep the `Archive/CLAUDE.md` index current — an archive nobody can search is just a graveyard.
- Preserve filenames on move so inbound wikilinks keep resolving; only rewrite links when the
  user asks.
