# Second Brain Starter

A Claude plugin marketplace by [Humanatia](https://humanatia.com).

**`second-brain-starter`** stands up a working second brain in Cowork and keeps it running. A guided setup skill builds your folder structure, rules, identity note, templates, and framework notes — actual files, not recommendations — then hands off to seven companion skills for day-to-day upkeep.

## What's inside

| Skill | What it does |
|---|---|
| `second-brain-architecture` | Guided interview that builds your vault from scratch |
| `resume` | Pick up a previous session from its latest work log |
| `record` | Save the current session as a structured note |
| `import-transcript` | Pull a meeting transcript in as a context note |
| `create-folder` | Scaffold a new client / project folder with a hub note |
| `archive` | Retire content to `Archive/` instead of deleting it |
| `os-health` | Audit the vault for broken links, orphans, duplicates, stale claims |
| `capture-framework` | Turn a durable idea into a filed framework note |

## Install

In **Cowork** or **Claude Code**, add the marketplace, then install the plugin:

```
/plugin marketplace add thehumanintheloop/second-brain-starter
/plugin install second-brain-starter@second-brain-starter
```

Then start the setup with a prompt like **"build my second brain"**.

## Updating

Maintainers push changes to this repo. To pull the latest version:

```
/plugin marketplace update second-brain-starter
```

---

Built and maintained by Humanatia.
