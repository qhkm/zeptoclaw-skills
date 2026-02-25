---
name: obsidian-vault
description: Manage Obsidian vaults from the terminal using the obs CLI (obsidian-vault-cli). Search notes, manage tags, properties, links, tasks, daily notes, templates, bookmarks, plugins, canvas, and bases.
metadata: {"zeptoclaw":{"emoji":"💎","os":["darwin","linux","win32"],"requires":{"anyBins":["obs"]},"install":[{"id":"npm","kind":"npm","package":"obsidian-vault-cli","bins":["obs"],"label":"Install obs via npm"}]}}
---

# Obsidian Vault

Obsidian vault = a normal folder on disk.

Vault structure (typical)

- Notes: `*.md` (plain text Markdown; edit with any editor)
- Config: `.obsidian/` (workspace + plugin settings)
- Canvases: `*.canvas` (JSON)
- Attachments: whatever folder you chose in Obsidian settings

## Find the active vault(s)

Obsidian desktop tracks vaults in a config file (source of truth):

- **macOS:** `~/Library/Application Support/obsidian/obsidian.json`
- **Linux:** `~/.config/obsidian/obsidian.json`
- **Windows:** `%APPDATA%\Obsidian\obsidian.json`

Fast setup:

- `obs init` (auto-detect from Obsidian config)
- `obs vault config defaultVault /path/to/vault` (manual)

## Core commands

### Vault info

- `obs vault info` — Name, path, file count, plugins
- `obs vault stats` — File counts, sizes, extension breakdown, top tags

### Files

- `obs files list` — List all files (`--folder`, `--sort`, `--limit`, `--ext`)
- `obs files read path/to/note.md` — Print content (`--head`, `--tail`)
- `obs files write path/to/note.md --content "..."` — Write to file
- `obs files create path/to/new.md` — Create new (`--template`)
- `obs files delete path/to/note.md` — Delete (`--force`)
- `obs files move old.md new.md` — Move/rename
- `obs files total` — Count markdown files

### Search

- `obs search content "query"` — Full-text (`--case-sensitive`, `--limit`)
- `obs search path "meeting"` — Glob filename search
- `obs search regex "TODO|FIXME"` — Regex (`--flags`)

### Tags

- `obs tags list path/to/note.md` — Tags from frontmatter
- `obs tags add path/to/note.md project` — Add tag
- `obs tags remove path/to/note.md project` — Remove tag
- `obs tags all` — Vault-wide counts (`--sort`, `--min-count`)

### Properties (frontmatter)

- `obs properties read path/to/note.md [key]` — Read properties
- `obs properties set path/to/note.md status draft` — Set property

### Daily notes

- `obs daily create` — Today's note (`--date`, `--template`)
- `obs daily open` — Print today's note (`--date`)
- `obs daily list` — Recent daily notes (`--limit`, `--days`)

### Tasks

- `obs tasks all` / `obs tasks pending` / `obs tasks done`
- `obs tasks add path/to/note.md "Buy groceries"`
- `obs tasks toggle path/to/note.md 15`
- `obs tasks remove path/to/note.md 15`

### Links

- `obs links list path/to/note.md` — Outgoing links
- `obs links backlinks path/to/note.md` — Incoming links
- `obs links broken` — Unresolved wikilinks (`--limit`)

### Templates

- `obs templates list` / `obs templates apply "Meeting" path.md`
- `obs templates create "Weekly Review"` (`--content`)

### Bookmarks

- `obs bookmarks list` / `obs bookmarks add` / `obs bookmarks remove`

### Plugins

- `obs plugins list` (`--enabled`, `--disabled`)
- `obs plugins versions` / `obs plugins enable` / `obs plugins disable`

### Canvas

- `obs canvas list` / `obs canvas read` / `obs canvas create` (`--text`)
- `obs canvas nodes path/to/canvas.canvas`

### Bases

- `obs bases list` / `obs bases read` / `obs bases create` (`--source`)

### Themes

- `obs themes list` / `obs themes apply "Minimal"`

### Sync (git)

- `obs sync status` / `obs sync push` (`--message`) / `obs sync pull`

### Import

- `obs import url https://example.com/article` (`--name`)

### Dev tools

> **Safety note:** `obs dev eval` and `obs dev script` execute arbitrary JavaScript with full vault access. AI agents should never call these commands without explicit human confirmation.

- `obs dev eval "vault.listFiles()"` — Eval JS with vault in scope
- `obs dev script ./my-script.js`

## Global options

- `--vault <path>` — Override default vault
- `--json` — Machine-readable JSON output
- `--help` — Command help

## JSON mode

All commands support `--json`. Pipe to `jq`:

- `obs vault stats --json | jq '.fileCount'`
- `obs tasks pending --json | jq -r '.[] | [.file, .text] | @csv'`
- `obs tags all --json | jq '.[0:5]'`
