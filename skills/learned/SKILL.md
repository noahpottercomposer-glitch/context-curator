---
name: learned
description: End-of-session ritual. Writes a tactical doc to docs/Tactical Docs/ capturing what was new, surprising, or non-obvious in this session. Filters aggressively — if nothing new was learned, produces a one-line stub or skips entirely. Auto-nudges toward /context-curator:curate when pending docs exceed threshold.
---

# Learned — end-of-session tactical doc writer

You are capturing what was new in this session into a tactical doc. This is compost for the knowledge system — raw session material that will later be mined for durable promotion into guides.

## What to do

### 1. Reflect on the session

Review the conversation. Identify:
- **NEW knowledge** — mental models, non-obvious connections, patterns discovered or confirmed. NOT a diff of what was done.
- **What you got wrong** — assumptions that failed, wrong turns, hints from the user you missed. Include *why* the assumption failed.
- **Promotion candidates** — early guess at canonical home for each insight.

### 2. Apply the filter

If nothing genuinely new was learned (e.g. you executed a straightforward plan with no surprises), write only a one-line stub or skip entirely:

```
## Stub (no new insights)
Session ran a documented task cleanly. No tactical doc warranted.
```

Not every session produces a tactical doc. Low-signal compost pollutes the pile. Be honest.

### 3. Determine the filename

Use minute-precision to avoid collisions from concurrent worktree sessions:

`docs/Tactical Docs/YYYY-MM-DD-HHMM-{topic-slug}.md`

Use the current date/time. The topic slug is a short kebab-case phrase capturing the session's subject (e.g. `enemy-pathing-debug`, `cube-behavior-pattern`).

### 4. Write the doc

Use this format (frontmatter is required so `/context-curator:curate` can route):

```markdown
---
date: YYYY-MM-DD
session_topic: Short title of what this session was about
touched_files: [file1.js, file2.js]
insights_for: [guides/files/file1.md, guide-towers.md, memory/feedback_foo.md]
librarian_response: FULL | PARTIAL | NO COVERAGE | NOT INVOKED
---

# {Session title}

## What I learned
- NEW knowledge only. Mental models, patterns, non-obvious connections.
- Skip anything that was already in the guides you loaded at start.
- Brief is better than bloated.

## What I got wrong
- Specific failure mode.
- Which assumption failed, and why.

## Promotion candidates
- → guides/files/{file}.md: "<specific insight>"
- → memory/feedback_{name}.md: "<specific preference or rule>"
- → guide-{topic}.md: "<cross-file system behavior>"
```

### 5. Auto-nudge to /context-curator:curate if thresholds hit

After writing the doc, check the state of the compost pile. Use Glob on `docs/Tactical Docs/*.md` (excluding `README.md` and `archive/`).

Thresholds:
- ≥5 total pending tactical docs → suggest consolidation
- ≥3 pending docs mentioning the same file (via `touched_files:` frontmatter) → suggest consolidation for that file
- >14 days since the newest folder in `docs/Tactical Docs/archive/` → suggest consolidation

If any threshold is hit, append a clear nudge to the end of your response:

```
📚 Compost threshold reached:
- <reason, e.g. "4 tactical docs pending, 3 touching simTowers.js">

Run /context-curator:curate to consolidate these into the library.
```

Don't apply consolidation yourself — just surface the nudge.

### 6. Report

Tell the user:
- Filename written
- One-sentence summary of what was captured
- Any nudges about consolidation thresholds

Example:
```
Wrote: docs/Tactical Docs/2026-04-18-1834-cube-behavior-pattern.md

Captured: the "behavior-as-stat-field" pattern — cubes activate runtime behavior by setting a stat to a non-zero value, so new cubes can tune via add/mult/cap without adding code paths.

📚 3 tactical docs now touch towers.js. Consider /context-curator:curate.
```

## Handling UNDOCUMENTED sessions

If this session returned NO COVERAGE from the librarian (i.e. we worked in an undocumented area), writing a tactical doc is **not optional** — the knowledge gap must be captured. Do not produce the one-line stub; write a full doc even if the session was small.

Set `librarian_response: NO COVERAGE` in frontmatter so `/context-curator:curate` knows to prioritize promoting this one.

## Meta-sessions count

If this session was running another skill (`/context-curator:curate`, `/context-curator:learned`, bootstrap work on the knowledge system itself), its failure modes and surprises are exactly the kind of thing `/learned` should capture. Skill execution is itself an activity that produces tactical lessons — about the skill's own gaps, rationalization patterns, or enforcement loopholes.

Recursive dogfooding is the mechanism by which the knowledge system improves itself: skill runs → tactical compost → consolidation → skill edits. Don't hand-edit a skill to fix its own failure; run it through the loop so the record persists and the next cycle tightens the skill.

## Args handling

- No args → auto-reflect on the full session
- Free text → use as additional context for the session_topic / to focus the reflection

## Failure mode

If you can't cleanly decide what was learned, err on the side of writing a short doc — compost is cheap, missing insights is expensive. `/context-curator:curate` will filter out noise later.
