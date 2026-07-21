---
name: os-health
description: >
  Run a health check on a knowledge OS / second brain — a file-based audit that surfaces broken
  links, orphaned notes, naming drift, duplicate or contradictory content, stale claims, and
  missing connections, then applies only the fixes the user approves. Also tracks Knowledge/Frameworks/
  growth and, once it crosses a threshold, proposes and (on approval) builds the vault's first MOCs
  or files newly-unfiled notes into existing ones. Works entirely through file access (no Obsidian
  plugin or local API required), so it runs in Cowork on any connected vault. Use whenever the user
  says 'run os-health', 'health check my vault', 'audit my second brain', 'check my notes for
  problems', 'find broken links / orphans / duplicates', 'tidy up my knowledge base', or wants a
  recurring maintenance pass on their system. Sibling to second-brain-architecture — that skill
  builds the structure, this one keeps it coherent as it grows.
version: 1.0.0
---

# OS Health — knowledge system audit

You are running a **health check** on the user's knowledge OS (a "second brain" — an Obsidian-style
vault of interlinked Markdown notes). This is the maintenance companion to `second-brain-architecture`:
that skill *builds* the structure; **os-health keeps it coherent as it fills up**.

Two things define this skill:

1. **It is file-based.** It works purely through reading and writing the vault's Markdown files — no
   Obsidian plugin, no local REST API, no live app. That is what lets it run in Cowork on any
   connected folder. Every check below is reimplemented over plain files.
2. **It is built to last, not just for week one.** A vault grows from 10 notes to 1,000. The check
   set never shrinks — instead, **scope** is the lever that keeps runs fast and relevant. A young
   vault runs `full`; a mature vault runs `since last run` or a single folder. Position it as a
   recurring ritual (monthly vitals), not a one-time cleanup.

**Golden rule: never destroy or rewrite content without explicit approval.** You surface findings,
the user chooses, you apply only what they pick. Renames and merges break links — treat them with
extra care.

---

## Step 1 — Determine scope

Scope is how this skill scales. Pick the scope from what the user asked, or infer a sensible default:

- **`full`** — audit every substantive note. Right for a small/young vault, or a periodic deep pass.
- **`recent`** — only notes changed in roughly the last 1–2 weeks (use file modification times).
- **`since last run`** — only notes modified since the last recorded health check (see *Recurring
  ritual* below). This is the default for a mature vault and what keeps monthly runs cheap.
- **`<folder>`** — audit one folder the user names.

**Choosing a default when the user didn't specify:**
- First, count the substantive notes (all `.md` minus the exclusions below).
- **≲ 75 notes →** default to `full` and say so.
- **> 75 notes →** default to `since last run` if a prior run exists, otherwise `recent`, and say so.
- Always state the scope you chose and why in one line before you start, and offer to widen it.

**Always exclude from scanning:** daily-note folders, templates, an `Inbox/` or capture staging area,
attachments/media, `.git`, `.obsidian`, `.claude`, any `Archive/` folder, and empty or `.DS_Store`
files. If the vault has a root `CLAUDE.md`, respect any exclusions it names.

## Step 2 — Learn this vault's conventions

**Do not assume any particular house style.** This runs on the *user's own* OS. Before auditing:

- Read the vault's **root `CLAUDE.md`** if one exists. Extract: naming conventions, folder meanings,
  hub-note / MOC pattern, `## Related` footer expectations, and anything marked as a rule.
- If there is no `CLAUDE.md`, infer conventions from the dominant patterns you observe (e.g. most
  session logs are `YYYY-MM-DD - Topic.md`) and note that you're inferring, not enforcing.
- Everything in Steps 3–5 is judged against *this vault's* conventions, not a fixed template.

## Step 3 — Read the notes in scope

Read the in-scope notes. For a large `full` run, read in batches and keep a compact working index
rather than holding every full note in memory:

- `path` → `title` (note basename without `.md`)
- outbound `[[targets]]` found in the body
- whether it has a `## Related` (or equivalent) footer
- frontmatter `type:` / `status:` / dates, if present

## Step 4 — Build the resolution index (this is what keeps link checks accurate at scale)

Broken-link and backlink checks are only trustworthy if you resolve links the way Obsidian does.
Build this once, from the **full file list** (list every `.md` in the vault, even outside scope, so
resolution is correct):

- **Title map:** basename → list of full paths that share it. Most basenames map to exactly one path.
- **Shortest-path resolution:** a link `[[Foo]]` resolves to the note whose basename is `Foo`. If
  several notes share that basename, Obsidian resolves to the shortest / closest vault-relative path;
  flag genuine ambiguity rather than guessing.
- **Link-form normalisation:** strip aliases and headings before matching — `[[Note|alias]]` and
  `[[Note#heading]]` both resolve on `Note`. Ignore `![[embeds]]` for dead-link purposes only if the
  target is an attachment.

This index makes dead-link and orphan detection *accurate*, not approximate — the whole point of the
file-based approach.

## Step 5 — Analyse

Run the checks below. They fall into two tiers; **report both, but rank structural findings first**
because they are deterministic and high-confidence, and coherence findings second because they are
judgement calls. Never invent a problem to fill a section — an empty section is a good result.

**Delegate the two tiers when the subagents are available.** This plugin ships two read-only
subagents that mirror these tiers: `link-integrity-checker` (Tier 1) and `vault-coherence-auditor`
(Tier 2). If your runtime exposes them, dispatch to both — ideally in parallel — instead of running
the checks inline, then fold their findings into Step 6:

- **Tier 1 → `link-integrity-checker`** — dead links, orphans, naming drift, missing backlinks. Pass
  it the scope and the vault root. **Keep MOC coverage yourself** (it's Tier 1 below, but it feeds the
  write procedure in Step 7 and the read-only agent doesn't run it).
- **Tier 2 → `vault-coherence-auditor`** — contradictions, stale claims, duplicates, missing
  cross-references.

Dispatching gives each tier its own context window and lets the two run at once — worth it on any
vault big enough to matter. **If those subagents aren't available in your runtime, run every check
below inline yourself.** The tier definitions here are the canonical spec either way — the agents
follow them, so nothing changes about *what* gets checked, only *who* runs it.

### Tier 1 — Structural integrity (deterministic)

- **Dead links** — a `[[target]]` that resolves to no file in the index. Distinguish "no such note"
  from "ambiguous target" (multiple matches). Quote the link and its source note.
- **Orphans** — a substantive note with **no** outbound links **and** no inbound links (search the
  vault for `[[Title]]`). Exclude intentionally standalone notes (single reference dumps, index/MOC
  pages, hub notes). When unsure, surface it as a *question*, not an assertion.
- **Naming inconsistencies** — filenames that deviate from *this vault's* conventions (from Step 2):
  missing date prefixes, wrong separators, Title Case where kebab-case is expected, etc. Flag only;
  never rename without approval (renames break inbound links).
- **Missing backlinks** — a note mentions another note's exact title as **plain text** where a
  `[[wikilink]]` clearly belongs. Only flag unambiguous proper-noun / title matches, never generic
  words.
- **MOC coverage** — count substantive notes in `Knowledge/Frameworks/*.md`. Two distinct findings
  live under this one check, never both at once:
  - **First graduation** — `Knowledge/MOCs/` is empty or absent, and the Frameworks/ count is at or
    above the graduation threshold (default **20** — check the vault's root `CLAUDE.md` or
    `core-rules.md` first in case this vault set its own number, the same way scope defaults respect
    vault-specific exclusions). Flag: the vault has enough framework notes to organise into MOCs.
  - **Ongoing filing** — `Knowledge/MOCs/*.md` already exist. Count Frameworks/ notes with **no
    inbound `[[wikilink]]` from any MOC file** ("unfiled"). Flag once that count reaches **5** or
    more.
  - If the user defers either finding ("not now"), don't re-flag it on the very next run — note the
    deferral in `os-health-log.md` (see *Recurring ritual*) and only re-surface once the relevant
    count has grown by 10 more, or the user asks directly.

### Tier 2 — Coherence (reasoning — judgement required)

- **Contradictions** — two notes making incompatible claims about the same subject (facts, dates,
  numbers, or a stated principle contradicted by a practice). Quote both lines with paths. **Never
  decide which is correct** — ask.
- **Stale claims** — an assertion that reads as current but is likely outdated: a past date framed as
  upcoming, a `status:` or price contradicted by a newer note, "current/latest/now" language a newer
  note supersedes. Compare against today's date. Quote the claim and the newer evidence; ask.
- **Duplicates / redundancy** — notes covering the same topic that could merge, sections copy-pasted
  across notes (search for distinctive phrases), or empty/near-empty stubs.
- **Missing cross-references** — two notes on clearly related topics whose `## Related` footers omit
  each other. This is *conceptual* overlap, distinct from missing backlinks (an exact-title match).

## Step 6 — Present findings

Lead with a one-line vitals summary, then sections **most-severe first**, each finding numbered so the
user can approve by number. Keep it scannable — plain language, not a data dump. On a large vault,
show the top findings per section and offer to list the rest.

```
## OS Health — [date]
**Scope:** [what was audited] · **Notes checked:** N
**Vitals:** 🔴 N dead links · 🟠 N orphans · 🟠 N naming · 🟣 N MOC coverage · 🟡 N backlinks · 🟡 N contradictions · 🟡 N stale · ⚪ N duplicates · ⚪ N gaps
(Only non-zero categories shown. A clean bill of health is a good outcome — say so plainly.)

### 🔴 Dead links (N)
**[1]** `path/note.md` links `[[Missing Note]]` → no matching file. Fix target or create note?

### 🟠 Orphans (N)
**[1]** `path/note.md` — no inbound or outbound links. Link it to `[[…]]`, or confirm it's standalone?

### 🟣 MOC coverage
**First graduation:** Frameworks/ has grown to N notes with no MOC structure yet. Cluster into MOCs?
**Ongoing filing:** N framework notes aren't filed under any MOC. File them under existing MOCs, or start a new one for [theme]?

…one section per non-empty category, structural tiers first…
```

Then ask plainly: **"Which of these should I fix?"** (e.g. "all dead links", "just #1 and #3",
"skip contradictions"). Accept batch answers.

## Step 7 — Apply only what's approved

Echo each change before making it; confirm after. Never touch anything not explicitly approved.

- **Dead links** — repair the `[[target]]` to the intended existing note, or, only if the user wants
  the missing note created, confirm folder + frontmatter first (don't auto-create).
- **Missing backlinks / cross-references** — replace the exact plain-text string with the bracketed
  form, or add the wikilink to the `## Related` footer. Show the exact edit first.
- **Orphans** — never link blindly. Propose the specific `[[target]]` and where it goes; apply on OK.
- **Naming** — do not rename until approved; when you do, **find and update every inbound `[[link]]`**
  to the old name in the same pass, or the rename breaks them.
- **Duplicates / merges** — draft the merged note, show it in full, apply only on confirmation; then
  handle links pointing at the note being retired.
- **Contradictions / stale claims** — never auto-resolve. Update the outdated line only after the user
  says which version is current, showing the proposed edit first.
- **MOC coverage** — the bigger of the two structural calls in this skill; see *MOC graduation* below
  for the full procedure. Never skip straight to writing files — propose the clustering first.

End with a change log: "Updated `path/note.md`", one line each, and the count fixed vs deferred.

## MOC graduation — building or extending the MOC set

This runs only when the user approves a "MOC coverage" finding from Steps 5–6. It's a bigger
structural call than any other fix in this skill, so it always gets two passes — propose, then
write — never a single-shot creation.

**First graduation (`Knowledge/MOCs/` empty or absent)**

1. Read every note in `Knowledge/Frameworks/` in scope — title, frontmatter, and the "What this is"
   section (or equivalent opening) is usually enough to place it; full-read only if the theme isn't
   clear from that.
2. Group them into 3–6 thematic clusters. A cluster needs at least 2–3 notes to earn its own MOC —
   don't create a one-note MOC, and don't force every note into a cluster; a genuine outlier stays
   unfiled rather than getting mis-filed. Leaving some notes unfiled is a valid outcome of this pass.
3. Name each cluster in this vault's own terms — derive the name from what the notes actually say,
   not a generic label pasted in from elsewhere.
4. **Show the full proposed structure before writing anything**: one line per cluster name, with the
   note filenames listed under it. Same draft-first pattern as duplicates/merges above. Let the user
   rename a cluster, move a note between clusters, or pull one out to stay unfiled — before any file
   gets touched.
5. Once confirmed, write one file per cluster: `Knowledge/MOCs/MOC - <Cluster Name>.md`, matching
   this vault's own MOC convention if one is visible elsewhere in the vault (frontmatter `title` /
   `type: moc` / `created`, an italic one-line scope under the H1, then bulleted `[[wikilink]] —
   description` entries, optionally grouped under thematic sub-headings if a cluster is large enough
   — 6+ notes — to need a second layer). If no existing MOC is visible to pattern-match, use:
   ```markdown
   ---
   title: MOC - <Cluster Name>
   type: moc
   created: YYYY-MM-DD
   ---

   # MOC - <Cluster Name>

   > [One-line scope — what this cluster is about and why these notes sit together]

   - [[note-slug]] — one-line description
   - [[note-slug]] — one-line description

   ## Related maps
   - [[index]]
   - [[MOC - <Other Cluster Name>]]
   ```
   List every sibling MOC created in this same pass under "Related maps," not just this one.
6. For each note now filed, add `[[MOC - <Cluster Name>]]` to that note's own `## Related` footer
   (create the footer if it doesn't have one). MOCs link down to members; members link back up to
   their one MOC — the same bidirectional convention the vault already expects elsewhere.
7. If `index.md` exists and its `## Frameworks` section still lists individual framework notes
   one-by-one (the `second-brain-architecture` setup template), replace that list with one line per
   MOC instead — this is the moment `index.md` graduates from a flat note list to a map-of-maps,
   matching what the folder now actually looks like. Show the proposed replacement before writing it.

**Ongoing filing (MOCs already exist)**

A newly-unfiled note doesn't need a full graduation pass — file it, or a small batch of them, under
the closest existing MOC, the same judgment `capture-framework` makes mid-conversation, just applied
in a batch across everything the scope caught. If a batch of unfiled notes shares a theme with no
existing MOC home, propose one new MOC for that cluster alone (same propose-then-write pattern as
above, just for a single cluster). Update each filed note's `## Related` footer as in step 6 above.

## Recurring ritual (what makes this durable, not a first-week toy)

os-health is meant to be **re-run** — monthly vitals for a growing OS. To support `since last run`:

- After a completed run, append one line to a log note at the vault root — `os-health-log.md`
  (create it if absent): `- YYYY-MM-DD · scope · fixed N / deferred N`.
- On the next run with `since last run`, read the last dated line to get the cutoff, and scope to
  notes modified after it. If no log exists, fall back to `recent`.
- Keep this log to one line per run — it's a lightweight state marker and audit trail, not a report
  archive. Don't create per-run report files unless the user asks.

## Tone

Direct and efficient. Surface findings fast, don't over-explain, don't pad empty sections. If the
user says "ignore that one," move on without argument. A clean result is a valid, good outcome —
report it in one line rather than manufacturing issues.
