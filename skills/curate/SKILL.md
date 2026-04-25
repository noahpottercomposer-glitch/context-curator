---
name: curate
description: Consolidation ritual for the Context Curator knowledge system. Reads pending tactical docs in docs/Tactical Docs/, verifies insights against current code, proposes structured diffs to promote them into durable guides (memory / file guides / topic guides / HowWeWork.md), awaits approval, applies the diffs, and archives the consumed tactical docs. Also updates the librarian routing.yaml when new guides are created.
---

# Curate — consolidation ritual

You are running the consolidation ritual. Your job is to turn the tactical compost pile into durable library growth, carefully and with the user's approval.

## The procedure (run in order)

### 1. Scan the compost pile

Use Glob to list `docs/Tactical Docs/*.md`, excluding `README.md` and the `archive/` subdirectory. Read each file. Parse frontmatter (`date`, `touched_files`, `insights_for`, `session_topic`, `librarian_response`).

If scope is restricted (user passed a topic in args), filter tactical docs whose `touched_files` or topic overlap with the scope.

### 1.5. Librarian-effectiveness instrumentation

Before clustering, compute four numbers from the scanned tactical docs. They go at the TOP of the report in step 6 — they are the primary signal for whether the librarian / memory system is changing behavior, not just accumulating files.

**N — Pending docs.** Total count of tactical docs scanned in this pass (excluding README + archive).

**X — Librarian NOT INVOKED.** Count of tactical docs whose frontmatter `librarian_response: NOT INVOKED` (or is missing the field entirely). Diagnoses harness firing, not memory quality. If X/N stays high across cycles, the fix is a hook or `CLAUDE.md` auto-load, NOT more memory files.

**Y — Re-derivations.** For each "What I learned" bullet across all pending docs, check whether the same claim is already encoded in an existing memory file or topic/file guide. Count the duplicates.
- "Already encoded" means: a substantive grep against the project's memory directory + docs/ surfaces a file whose content expresses the same rule, even if phrased differently. Don't require verbatim match.
- Re-derivation means the library HAD the answer but the session didn't use it. That's the failure mode the librarian exists to prevent.
- Quick shortcut: for each bullet, think "is there a memory file I could have cited instead?" — if yes, Y += 1.
- This is THE metric. Trend line for Y/N over cycles tells you whether the system is actually working. Should decline as library matures.

**Z — Citations.** Grep pending tactical docs for `feedback_` / `per memory/` / `applied X.md` / explicit cross-refs to library files. Count them. Wins leave fingerprints. Zero citations across a full cycle = library is write-only even when loaded.

Surface the four numbers at the top of step 6's report. Include a one-line delta vs the prior cycle if the user's been tracking (pull from previous /curate's consolidation-report entry in `MEMORY.md` if present).

### 2. Cluster insights

Group the "What I learned" and "Promotion candidates" items across the pending docs. Look for:
- Repeated themes (same insight surfaces in multiple sessions → high-confidence promotion)
- Contradictions (newer doc disagrees with older → newer wins; flag to the user)
- Stale references (a file mentioned no longer exists → skip the insight)

### 3. Verify against current code

**This is the critical step.** For each candidate insight:
- If it makes a claim about a function, file, pattern, or invariant: Grep/Read the current code to confirm it still holds.
- If a refactor has obsoleted the insight, **skip it** and note the obsolescence in your report.
- Do not promote anything you haven't verified is still accurate.
- **When an insight maps to content in an EXISTING memory file and the claim still holds in current code, plan to bump that memory's `last_verified: YYYY-MM-DD` to today — even if no text edit is needed.** The stamp is the librarian's freshness signal; it only moves when you've actually re-checked.

Consolidation that blindly promotes encodes outdated knowledge into guides. Slow is smooth, smooth is fast.

### 4. Route each insight to its canonical home

Decision tree:
- **Cross-session preference, feedback, workflow rule, or lesson about collaborating with the user** → `~/.claude/projects/<project-key>/memory/feedback_*.md` (new file or existing). `<project-key>` is auto-derived by Claude Code from the project root.
- **Fact about a specific file's internals — invariant, pattern, gotcha** → `docs/guides/files/{file}.md` (create if not exists and file has earned a guide)
- **Cross-file system behavior, contract, pipeline** → `docs/guide-{topic}.md` (existing topic guide)
- **Project-wide architectural or collaborative rule** → `docs/HowWeWork.md`

No insight lands in more than one place. Cross-references point to the canonical home.

### 5. Check if any file has earned a file guide

A file earns a guide when:
- It's been touched in ≥3 tactical docs, OR
- The user has flagged it directly, OR
- Consolidation has produced enough material to justify one.

**Step 5 is not optional judgment.** If a file meets ANY of the three criteria above, the file guide MUST appear as a concrete proposal in the main Promotions section of the report (step 6) — NOT in a "candidates surveyed" side-section, NOT deferred with "holding" / "not proposing yet" language.

Anti-patterns to reject at this step:
- "File is enormous, topic guides already cover most of it" — both conditions are arguments FOR a file guide (more invariants buried, more navigation value), not against.
- "NOT proposing yet — holding for next cycle." If the criteria are met THIS cycle, propose THIS cycle.
- Side-sections labeled "surveyed but not advancing" — these silently exit the user's approval gate. Anything categorized away from the decision-set is outside the gate.

If you genuinely believe a qualifying file should be deferred, surface it as an **explicit question** in the report ("Criteria met for X; propose this cycle or defer?") — a surfaced decision, never a buried categorization.

If so, propose creating `docs/guides/files/{filename}.md` using this template (update `last_reviewed` to today):

```markdown
---
file: {filename}.js
last_reviewed: YYYY-MM-DD
covers: [short, topic, tags]
---

# guide: {filename}.js

## Purpose (why this file exists)
One paragraph for future-you.

## Responsibility
One-sentence: what this file owns.

## Canonical patterns used here
- ...

## Invariants (must not break)
- ...

## Known gotchas / dead-ends
- ...

## Related
- Topic guides: ...
- Peer files: ...
- Memory: ...
```

### 6. Propose a structured diff report

Before editing anything, present the user with a clear report. Format:

```
## Consolidation Report — <date>

Scope: <all / topic-X>
Tactical docs reviewed: <N>
Insights proposed: <M>  |  Obsolete/skipped: <K>

### Librarian-effectiveness signal

  Pending docs (N):              <N>
  Librarian NOT INVOKED (X):     <X>/<N>   (harness — if high, fix CLAUDE.md or hooks, not memory)
  Re-derivations (Y):            <Y>       (library had it, session didn't use it — THE metric)
  Citations (Z):                 <Z>       (wins leave fingerprints — should climb over time)

  Prior cycle (<prior-date>):   X=<x>/<n>  Y=<y>  Z=<z>   <delta interpretation>

### Promotions

#### → memory/feedback_foo.md (NEW)
- Insight: "..."
- Source: tactical doc YYYY-MM-DD-bar.md
- Verified against: <file:line>

#### → docs/guides/files/simTowers.md (NEW FILE GUIDE)
- Purpose: ...
- Invariants captured: ...
- Sourced from 3 tactical docs

#### → docs/guide-renderer.md (edit: "Draw-call scaling" section)
- Add: "..."
- Verified: render loop at renderer3d.js:1234

### Obsolete (NOT promoting)
- From YYYY-MM-DD-baz.md: "..." — obsoleted by refactor in commit abc123

### Librarian routing updates needed
- Add file guide link for `tower-sim` topic → now FULL COVERAGE
- New topic: `enemy-visuals` → maps to ...

### Tactical docs to archive after approval
- YYYY-MM-DD-foo.md
- YYYY-MM-DD-bar.md
- YYYY-MM-DD-baz.md
```

### 7. Await approval

Do NOT apply anything yet. Stop here and let the user read the report. The user may:
- Approve everything (proceed to step 8)
- Approve specific items only (apply those, leave the rest)
- Reject or edit specific items (adjust per feedback, re-present)
- Ask for more detail on any item (provide and wait)

### 8. Apply approved diffs

For each approved item:
- Use Edit to modify existing files (preserve surrounding content exactly).
- Use Write to create new files (file guides, new memory files).
- Update `last_reviewed: YYYY-MM-DD` frontmatter on every edited guide to today.
- **Update `last_verified: YYYY-MM-DD` frontmatter on every memory file you've re-verified against current code THIS CYCLE** — whether you edited its body or not. Absence or ≥30-day-old stamps are how the librarian flags staleness. Stamp only what you actually re-checked; don't mass-bump.
- **New memory files born in this cycle** get `last_verified: YYYY-MM-DD` set to today alongside `name:`, `description:`, `type:` in frontmatter.
- Update `docs/.knowledge-system/routing.yaml` — add the new file to the appropriate topic's `files:` list, update `coverage:` (PARTIAL → FULL if a file guide now exists). The librarian skill body never holds routing — it reads the YAML at invocation time.
- If `MEMORY.md` index in `~/.claude/projects/<project-key>/memory/` needs a new entry pointer for a new memory file, add it.

### 9. Archive consumed tactical docs

Create `docs/Tactical Docs/archive/YYYY-MM/` if it doesn't exist. Move each consumed tactical doc into that folder using Bash `mv`:

```
mv "docs/Tactical Docs/YYYY-MM-DD-foo.md" "docs/Tactical Docs/archive/YYYY-MM/"
```

(Do one at a time so each move is visible/reviewable.)

### 10. Summarize

Report what was applied, what was archived, and the new library-coverage state.

Example:
```
Consolidation complete.
- 4 insights promoted (1 new file guide, 2 memory entries, 1 HowWeWork rule)
- 3 tactical docs archived
- Librarian now has FULL COVERAGE for tower-sim
- Next: run /context-curator:librarian tower-sim to see the improved routing
```

## Scope handling (args)

- No args → consolidate everything pending
- `/context-curator:curate tower-sim` → only tactical docs touching tower-sim-related files
- `/context-curator:curate all` → same as no args, explicit

## Non-goals

- Do NOT rewrite existing guide content that isn't being amended. Preserve it.
- Do NOT promote an insight without code-verification.
- Do NOT skip the approval step. The user is the editor-in-chief.
- Do NOT delete tactical docs — always move to archive. History matters.

## Failure mode

If the code-verification step fails for most insights (e.g., a large refactor landed and obsoleted most of the compost), say so plainly and propose archiving the obsolete tactical docs without promotion. The user decides.
