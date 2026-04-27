# Context Curator

Context Curator is a Claude Code plugin that keeps project knowledge useful by routing only the most relevant guides and memory into context for each task. It creates a closed loop: /librarian finds what Claude should load, /learned captures new session insights, and /curate promotes useful tactical notes into durable documentation. Its key differentiators are freshness checks, metrics for whether the library is actually changing behavior, and a strong “NO COVERAGE” state that forces undocumented work to be captured instead of lost. The system is project-portable because the protocol lives in the plugin while each project’s routing lives in its own routing.yaml.

Three-stage knowledge management for Claude Code, built around the insight that **finite context is the binding constraint** of working with an LLM. Most projects either flood the context window with everything or starve Claude of the rules it needs. This plugin gives you a closed loop:

- **`/context-curator:librarian <topic>`** — per-task context routing. Returns the minimal set of guides + memory to load before starting work. FIRST step of every task.
- **`/context-curator:learned`** — per-session insight capture. Writes a tactical doc to `docs/Tactical Docs/`.
- **`/context-curator:curate`** — library-level consolidation. Promotes tactical compost into durable guides.
- **`/context-curator:init`** — one-time scaffolding for a fresh project.

The library grows; the loaded context per task stays small. Future sessions start better-informed than today's.

## What makes this different

- **Closed-loop content pipeline.** Tactical compost → consolidation → durable guides → librarian routing → next session better-informed than today's. Most knowledge bases are write-only.
- **Self-instrumenting.** `/curate` reports `X` (librarian-not-invoked rate), `Y` (re-derivations the library already had answers to), `Z` (citations). Y is THE metric — measures whether the library is actually changing behavior, not just accumulating files.
- **Semantic freshness stamps.** `last_verified: YYYY-MM-DD` on memory files means "curate re-checked the claim against current code," not "file mtime." The librarian flags stale claims (>30 days) for re-verification.
- **NO COVERAGE has teeth.** Forced scope interrogation, mandatory canonical-path audit, mandatory tactical doc on undocumented territory. The system treats "we don't know about this" as a first-class state, not an error.
- **Fail-open.** Broken routing is a papercut, not a crash.

## Installation

### Local (test before publishing)

If you cloned this repo to `/path/to/context-curator/`:

```
cd ~/your/project
/plugin marketplace add /path/to/context-curator
/plugin install context-curator@context-curator
```

### From GitHub

```
cd ~/your/project
/plugin marketplace add github:noahdpotter/context-curator
/plugin install context-curator@<auto-derived-marketplace-name>
```

(The marketplace name auto-derives from the GitHub repo. If unclear, list with `/plugin list`.)

## Quick start

After installing in a fresh project:

```
/context-curator:init
```

This scaffolds (skip-if-exists, NEVER overwrites):
- `docs/`, `docs/Tactical Docs/`, `docs/guides/files/`
- `docs/HowWeWork.md` — starter collaboration contract
- `docs/README.md` — system overview
- `docs/guide-index.md` — empty module map
- `docs/Tactical Docs/README.md` — compost spec
- `docs/.knowledge-system/routing.yaml` — empty routing + commented example
- `MEMORY.md` template (if memory dir exists)
- A snippet for `CLAUDE.md` (printed for you to paste; never overwritten)

Then verify:

```
/context-curator:librarian foo
```

Should return NO COVERAGE on empty routing — proves the system is wired up. Add topics to `docs/.knowledge-system/routing.yaml` as you discover them, or let `/context-curator:curate` add them after a few sessions of compost has accumulated.

## Architecture

- **Skill bodies** are project-agnostic protocol (response shapes, freshness model, scope interrogation, canonical-path audit, footer logic).
- **Project-specific routing** lives in `docs/.knowledge-system/routing.yaml` — the only file you customize per project. Topics map to coverage tags, alias lists, and ordered file lists.
- **Tactical compost** lives in `docs/Tactical Docs/` (mined periodically, never the primary reference).
- **Durable knowledge** lives in `docs/guide-*.md`, `docs/guides/files/*.md`, and `~/.claude/projects/<project-key>/memory/`.

The split between protocol and routing is what makes this plugin portable. Each project owns its own `routing.yaml`; the skills themselves never need editing.

## The three rituals in detail

### `/context-curator:librarian <topic>`

Reads `routing.yaml`, fuzzy-matches the query against the topic alias table, returns the load list with one of three response shapes:

- **FULL COVERAGE (DOCUMENTED)** — topic guide + file guides exist; load and proceed.
- **PARTIAL COVERAGE** — topic guide exists but ≥1 load-bearing file lacks a file guide; proceed with the gap in mind; tactical doc captures what you learn.
- **NO COVERAGE (UNDOCUMENTED)** — task is uncharted; forced scope interrogation; mandatory tactical doc at end.

Includes a library-health footer: pending tactical docs, days since last consolidation, undocumented-topic counter, threshold-triggered "Consider /curate" nudges.

### `/context-curator:learned`

Writes a tactical doc at session end. Filters aggressively — if nothing genuinely new was learned, produces a one-line stub or skips entirely. Mandatory if the session was UNDOCUMENTED (no auto-skip). Auto-nudges toward `/curate` when compost thresholds hit.

### `/context-curator:curate`

The consolidation ritual. Reads pending tactical docs, **verifies each insight against current code** (refactored-away insights are skipped, not promoted), proposes a structured diff report, awaits approval, applies promotions, archives consumed tactical docs.

Decision tree for canonical homes:
- Cross-session preference / feedback / workflow rule → memory `feedback_*.md`
- File-specific invariant / pattern / gotcha → `docs/guides/files/{file}.md`
- Cross-file system behavior → `docs/guide-{topic}.md`
- Project-wide architectural or collaborative rule → `docs/HowWeWork.md`

Bumps `last_verified:` stamps on memory files re-checked this cycle. Updates `routing.yaml` when new file guides land.

## Status

v0.1 — actively used on a brutalist tower-defense game (~60 sessions of dogfooding before extraction). Format conventions are still settling; expect rough edges and breaking changes through v1.0.

## License

MIT.

## Author

Noah Potter.
