---
name: init
description: One-time scaffolding for the Context Curator knowledge system. Creates docs/, docs/Tactical Docs/, docs/.knowledge-system/routing.yaml, and other files needed by the librarian/learned/curate skills. Idempotent — skips any file that already exists and reports it (NEVER overwrites). Run once per project after installing the plugin.
---

# Init — scaffold the Context Curator knowledge system

You are setting up a fresh project to use the librarian/learned/curate skills.

**Idempotency rule (CRITICAL):** Never overwrite existing files. For each scaffold target, check existence first. If exists, SKIP and report. If absent, create from template. This applies to every single file — no destructive overwrites, ever.

## What to do

### 1. Detect the project root

The current working directory is the project root. Use `Bash("pwd")` to confirm. Display the detected root to the user.

### 2. Compute the project memory key

Claude Code's auto-memory directory for this project is at `~/.claude/projects/<project-key>/memory/`, where `<project-key>` is auto-derived from the project root path: leading `/` → `-`, each subsequent `/` → `-`, spaces → `-`.

Example: `/Users/noahpotter/Desktop/My Project/` → `-Users-noahpotter-Desktop-My-Project`

Compute it now using Bash:
```
pwd | sed 's| |-|g; s|/|-|g'
```

Store this as `<project-key>` for use in template substitution below.

### 3. Scaffold directories

Use a single `mkdir -p` to create all needed directories at once. `mkdir -p` is idempotent — existing dirs are silently skipped.

```
mkdir -p "docs/Tactical Docs/archive" "docs/guides/files" "docs/.knowledge-system"
```

### 4. Write file templates (skip-if-exists)

For each target below: `Glob` it first. If a result exists, SKIP and add it to the "Skipped (already existed)" report. If not, `Write` from the template, substituting `<project-key>` where shown.

#### `docs/README.md`

```markdown
# Project Docs

Working library — system-level guides, file-level operating manuals, and the tactical-doc compost pile.

## Start here
1. **HowWeWork.md** — collaboration contract.
2. **guide-index.md** — module map.
3. Topic guides — load only what's relevant to the current task.

## The knowledge system

| Command | When | Purpose |
|---|---|---|
| `/context-curator:librarian <topic>` | First step of every task | Returns curated guides + memory to load. |
| `/context-curator:learned` | End of session | Writes a tactical doc capturing what was new. |
| `/context-curator:curate` | Periodically | Consolidates compost into durable guides. |

Tactical docs are compost. Durable knowledge lives in topic guides, file guides, and memory.
```

#### `docs/HowWeWork.md`

```markdown
# How We Work

This is your project's collaboration contract — how you and Claude work together. Customize freely. The default is a starter.

## Architecture rules
- Every piece of state has one canonical owner.
- No duplicated derived state.
- If something doesn't fit cleanly, fix the architecture rather than adding a shim.

## Planning
Non-trivial changes: propose a plan, self-rate it out of 10, surface gaps before implementing.

## Collaboration
Be decisive. Be honest about uncertainty. When stuck, say so immediately rather than burning tokens circling.

(Add your project-specific rules below.)
```

#### `docs/guide-index.md`

```markdown
# Guide index

Module map. Maintained by you / `/context-curator:curate`.

| Module | Owns | Guide |
|---|---|---|
| _(example)_ | _what it owns_ | _link to guide_ |

Add entries as the codebase grows.
```

#### `docs/Tactical Docs/README.md`

```markdown
# Tactical Docs — the compost pile

NOT durable knowledge. Raw session material that's mined periodically and culled. Durable knowledge lives in `docs/guide-*.md`, `docs/guides/files/*.md`, and `~/.claude/projects/<project-key>/memory/`.

## What goes here

Session logs written by Claude at end of session via `/context-curator:learned`. Each captures:
- What was NEW in this session
- What Claude got wrong
- Promotion candidates (canonical home for each insight)

## Filename
`YYYY-MM-DD-HHMM-{topic-slug}.md` — minute precision prevents collisions across concurrent sessions.

## Lifecycle
1. Born during `/context-curator:learned`.
2. Lives pending — never auto-loaded.
3. Mined during `/context-curator:curate`.
4. Archived to `archive/YYYY-MM/` after promotion. Never deleted.
```

(Substitute `<project-key>` with the value computed in step 2.)

#### `docs/.knowledge-system/routing.yaml`

```yaml
# /context-curator:librarian routing config
#
# Owns: topic → file load list, topic → alias list, architecture-feedback files.
# /context-curator:curate is the canonical maintainer.
#
# Coverage tags:
#   FULL    — topic guide + file guides for all load-bearing files exist
#   PARTIAL — topic guide exists, but ≥1 load-bearing file lacks a file guide
#
# Path conventions:
#   - Repo-relative for in-repo files (docs/, etc.)
#   - "~/..." absolute paths for memory files (always quoted)

# Auto-attached on NO COVERAGE responses (rules that prevent parallel-path mistakes).
# Add memory file paths here as you build out architecture rules.
architecture_feedback_files: []

# Add topics as you discover them via /context-curator:learned + /context-curator:curate.
topics:
  # Example topic shape (uncomment + customize):
  # tower-sim:
  #   coverage: FULL
  #   aliases:
  #     - tower sim
  #     - simTowers
  #     - firing
  #   files:
  #     - docs/guide-towers.md
  #     - "~/.claude/projects/<project-key>/memory/towers-and-new-types.md"
```

(Substitute `<project-key>`.)

### 5. Memory bootstrap (soft-touch)

The auto-memory directory `~/.claude/projects/<project-key>/memory/` is created lazily by Claude Code when the first memory is saved. **Do NOT create it manually.** If you want to bootstrap an empty `MEMORY.md` index:

- Glob `~/.claude/projects/<project-key>/`. If it exists, also check for `MEMORY.md`.
- If the directory exists AND `MEMORY.md` does not exist → write the template below.
- If the directory does not exist → skip; note in the report that "memory directory will be created automatically when Claude first saves a memory."

Template:
```markdown
# Project Memory

Auto-memory index. Each entry should be one line under ~150 chars:
- [Title](file.md) — one-line hook

(empty)
```

### 6. CLAUDE.md handling (NEVER overwrite)

CLAUDE.md is the canonical rule surface Claude loads every session.

- Glob `CLAUDE.md` in the project root.
- **If it exists:** do NOT touch it. Print the snippet below in the final report and tell the user to paste it manually.
- **If it does not exist:** create it with the snippet wrapped in a minimal header.

Snippet:
```markdown
## Knowledge System — Read This First
Every task starts with `/context-curator:librarian <topic>`. It returns the curated guides + memory to load. If it returns NO COVERAGE, the task is UNDOCUMENTED — surface it before implementing and write `/context-curator:learned` at the end.

Three commands:
- `/context-curator:librarian <topic>` — context pull. FIRST step of every task.
- `/context-curator:learned` — end-of-session tactical doc.
- `/context-curator:curate` — consolidation ritual.

System overview: `docs/README.md`.
```

If creating fresh, prepend:
```markdown
# Project Rules

```

then the snippet.

### 7. Final report

Print a structured report to the user:

```
✅ Context Curator initialized in <project-root>

Created (<count>):
- <list of files actually created>

Skipped (already existed — left untouched):
- <list of files left untouched>

Action required:
- <if CLAUDE.md existed, paste this snippet manually:>
  <reproduce the snippet>
- <if memory directory doesn't exist yet:>
  The memory directory at ~/.claude/projects/<project-key>/memory/
  will be created automatically when Claude first saves a memory.

Next steps:
1. Customize docs/HowWeWork.md with your project's collaboration rules.
2. Try `/context-curator:librarian <something>` — should return NO COVERAGE
   on the empty routing (proves wiring works).
3. As you discover topics worth canonicalizing, add them to
   docs/.knowledge-system/routing.yaml — or let /context-curator:curate add
   them after a few sessions of compost has accumulated.
```

## Failure modes

- **Cannot detect project root**: print error, ask the user to confirm `pwd` and run again from the right directory.
- **Cannot compute project-key**: write template files with `<project-key>` placeholder unsubstituted; tell user to find-replace.
- **`mkdir` fails**: print the error, do not proceed.
- **Cannot write to ~/.claude/projects/...**: skip the memory bootstrap, note it in the report.
- **An existing file has the same NAME but different content from our template**: still skip it (idempotency rule). Note in the report that the user may want to compare manually.

## Args handling

- No args → standard scaffold of the cwd.
- `--dry-run` → list what WOULD be created/skipped, do NOT write anything. Useful for previewing the impact on a project that already has some docs.
