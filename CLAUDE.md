# Documentary Research Pipeline — Agent Context

## What This Repo Does

This is a multi-agent documentary research pipeline. Given a topic brief, it autonomously gathers web sources, fact-checks every claim, designs a narrative arc, writes a narration script with citations, and compiles a complete production research package.

## File Layout

### Inputs (you read these)
- `config/topic-brief.json` — the documentary topic, angle, target audience, duration, key questions, and tone

### Intermediate Artifacts (you read and write these)
- `data/research-scope.json` — topic dimensions, search queries, source categories, key questions (written by topic-scoper)
- `data/source-database.json` — all gathered sources with extracted claims, quotes, statistics, credibility scores (written by source-gatherer)
- `data/gathering-log.md` — running log of what was fetched (written by source-gatherer)
- `data/fact-check-report.json` — every claim cross-referenced across sources with confidence levels (written by fact-checker)
- `data/fact-check-summary.md` — human-readable fact-check results (written by fact-checker)
- `data/narrative-outline.json` — scene-by-scene structure with claim and source citations (written by narrative-architect)
- `data/narrative-outline.md` — readable version of the narrative outline (written by narrative-architect)
- `data/narrative-feedback.json` — feedback for narrative rework (written by narrative-architect on needs-work verdict)

### Final Outputs (you write these)
- `output/script.md` — full narration script with inline citations (written by scriptwriter)
- `output/interview-guide.md` — consolidated interview suggestions with questions (written by scriptwriter)
- `output/visual-notes.md` — B-roll and visual suggestions by scene (written by scriptwriter)
- `output/research-package.md` — executive summary, methodology, key findings (written by fact-checker in compile-package)
- `output/source-bibliography.md` — formatted bibliography with credibility ratings (written by fact-checker in compile-package)
- `output/research-stats.json` — statistics: source count, claim confidence distribution, citation integrity (written by fact-checker in compile-package)

## Agent Roles

| Agent | When It Runs | What It Produces |
|---|---|---|
| topic-scoper | Phase 1 | research-scope.json |
| source-gatherer | Phase 2 | source-database.json, gathering-log.md |
| fact-checker | Phase 3 and 6 | fact-check-report.json, fact-check-summary.md (phase 3); research-package.md, source-bibliography.md, research-stats.json (phase 6) |
| narrative-architect | Phase 4 | narrative-outline.json, narrative-outline.md |
| scriptwriter | Phase 5 | script.md, interview-guide.md, visual-notes.md |

## Decision Points

- **scope-topic**: `scoped` → proceed | `too-broad` → rework (max 2x)
- **gather-sources**: `sufficient` → proceed | `insufficient` → rework (max 3x)
- **check-facts**: `verified` → proceed | `disputed` → back to gather-sources | `unreliable` → back to scope-topic
- **build-narrative**: `compelling` → proceed | `needs-work` → rework self (max 3x) | `restructure` → rework self

## Citation Integrity Rule

Every factual claim in `output/script.md` must include an inline citation in the format `[Source: Title, URL]`.
The compile-package phase validates that every citation resolves to a real entry in `data/source-database.json`.
Only cite claims marked `high` or `medium` confidence in `data/fact-check-report.json`.

## Quality Standards

- Minimum 8 sources in source-database.json
- Sources must span at least 3 different credibility levels (1-5 scale)
- >80% of claims must be high or medium confidence to pass fact-check gate
- Narrative must cover all key_questions from the topic brief
- Every scene must link to at least 2 verified claims with source citations
