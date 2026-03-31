# Documentary Researcher — Build Plan

## Overview

A documentary research pipeline that takes a topic, gathers primary and secondary sources from the web, fact-checks claims against multiple sources, builds narrative structure, writes a cited script, and produces a complete research package for a production team.

## Agents

| Agent | Model | Role |
|---|---|---|
| **topic-scoper** | claude-haiku-4-5 | Reads topic brief, defines research scope, generates search queries and source categories |
| **source-gatherer** | claude-haiku-4-5 | Fetches web sources, extracts key claims/quotes/data, builds structured source database |
| **fact-checker** | claude-sonnet-4-6 | Cross-references claims across multiple sources, flags contradictions, rates confidence levels |
| **narrative-architect** | claude-opus-4-6 | Designs story arc, organizes themes, builds scene-by-scene outline with source citations |
| **scriptwriter** | claude-sonnet-4-6 | Writes narration script with inline citations, interview question suggestions, B-roll notes |

## Phase Pipeline

### Phase 1: `scope-topic` (topic-scoper, Haiku)
- Reads `config/topic-brief.json` (user-provided topic, angle, target audience, duration)
- Uses sequential-thinking to break topic into research dimensions
- Generates 10-20 specific search queries across primary/secondary source categories
- Outputs `data/research-scope.json`: dimensions, queries, source categories, key questions
- **Decision:** `scoped` (proceed) | `too-broad` (rework with narrower focus)

### Phase 2: `gather-sources` (source-gatherer, Haiku)
- Reads `data/research-scope.json` for queries and categories
- Uses fetch MCP to retrieve web pages for each query
- Extracts: key claims, direct quotes, statistics, dates, people mentioned, source credibility indicators
- Builds `data/source-database.json`: array of source entries with fields: url, title, author, date, credibility_score (1-5), extracted_claims[], quotes[], statistics[]
- Minimum 8 sources required; re-fetches if under threshold
- Outputs `data/source-database.json` and `data/gathering-log.md`
- **Decision:** `sufficient` (enough diverse sources) | `insufficient` (need more — rework)

### Phase 3: `check-facts` (fact-checker, Sonnet)
- Reads `data/source-database.json`
- Cross-references every major claim across sources
- For each claim: how many sources support it, any contradictions, confidence level
- Uses sequential-thinking for methodical claim-by-claim analysis
- Outputs `data/fact-check-report.json`: claims[] with {claim, sources_supporting[], sources_contradicting[], confidence: high|medium|low|disputed, notes}
- Outputs `data/fact-check-summary.md`: human-readable report
- **Decision:** `verified` (>80% claims high/medium confidence) | `disputed` (major claims disputed — rework gather-sources with targeted queries) | `unreliable` (too many low-confidence claims — rework scope-topic)

### Phase 4: `build-narrative` (narrative-architect, Opus)
- Reads `data/fact-check-report.json` and `data/source-database.json`
- Designs documentary narrative arc: hook, rising action, climax, resolution
- Organizes verified facts into thematic segments/scenes
- Each scene: theme, key claims (with source citations), suggested interviews, visual notes
- Uses sequential-thinking for structural reasoning
- If `data/narrative-feedback.json` exists (rework loop), addresses all feedback
- Outputs `data/narrative-outline.json`: scenes[], arc_description, estimated_duration, themes[]
- Outputs `data/narrative-outline.md`: readable version
- **Decision:** `compelling` (strong narrative arc) | `needs-work` (specific issues — rework with feedback) | `restructure` (fundamental arc problems — rework)

### Phase 5: `write-script` (scriptwriter, Sonnet)
- Reads `data/narrative-outline.json`, `data/source-database.json`, `data/fact-check-report.json`
- Writes narration script with inline citations [Source: title, url]
- Includes: narrator text, interview question suggestions per segment, B-roll/visual notes, transition cues
- Every factual claim must cite at least one verified source
- Outputs `output/script.md`: the full narration script
- Outputs `output/interview-guide.md`: suggested interviewees and question sets
- Outputs `output/visual-notes.md`: B-roll and visual suggestions per scene

### Phase 6: `compile-package` (fact-checker, Sonnet)
- Reads all data/ and output/ files
- Compiles final research package:
  - `output/research-package.md`: executive summary, methodology, key findings, source list, script overview
  - `output/source-bibliography.md`: formatted bibliography of all sources with credibility ratings
  - `output/research-stats.json`: total sources, claims verified/disputed, confidence distribution, coverage by theme
- Validates all citations in script resolve to real sources in database
- **No decision** — final assembly phase

## Workflow Routing

```
scope-topic → gather-sources → check-facts → build-narrative → write-script → compile-package
                    ↑                |               |
                    |           (disputed)      (needs-work/
                    |           rework to        restructure)
                    |          gather-sources    rework to
                    |                           build-narrative
                    |
               (insufficient)
               rework to self
```

- `check-facts` on `disputed`: rework back to `gather-sources` (get better sources for disputed claims)
- `check-facts` on `unreliable`: rework back to `scope-topic` (narrow the scope)
- `build-narrative` on `needs-work` or `restructure`: rework to self (writes feedback to `data/narrative-feedback.json`)
- `gather-sources` on `insufficient`: rework to self
- Max rework attempts: 3 per decision point

## MCP Servers

| Server | Package | Purpose |
|---|---|---|
| filesystem | `@modelcontextprotocol/server-filesystem` | Read/write all data and output files |
| fetch | `@modelcontextprotocol/server-fetch` | Fetch web pages for source gathering |
| sequential-thinking | `@modelcontextprotocol/server-sequential-thinking` | Structured reasoning for scoping, fact-checking, narrative design |

## Directory Structure

```
documentary-researcher/
├── .ao/workflows/
│   ├── agents.yaml
│   ├── phases.yaml
│   ├── workflows.yaml
│   └── mcp-servers.yaml
├── config/
│   └── topic-brief.json        # INPUT: topic, angle, audience, duration target
├── data/                        # RUNTIME: intermediate artifacts
│   ├── research-scope.json
│   ├── source-database.json
│   ├── gathering-log.md
│   ├── fact-check-report.json
│   ├── fact-check-summary.md
│   ├── narrative-outline.json
│   ├── narrative-outline.md
│   └── narrative-feedback.json  # Written on narrative rework
├── output/                      # FINAL: production-ready deliverables
│   ├── script.md
│   ├── interview-guide.md
│   ├── visual-notes.md
│   ├── research-package.md
│   ├── source-bibliography.md
│   └── research-stats.json
├── sample-data/
│   └── sample-topic-brief.json  # Demo: "The Rise and Fall of Theranos"
├── CLAUDE.md
└── README.md
```

## Sample Topic Brief (for testing)

```json
{
  "topic": "The Rise and Fall of Theranos",
  "angle": "How regulatory gaps and celebrity culture enabled a decade of deception in health tech",
  "target_audience": "General audience, tech-curious, interested in corporate fraud",
  "target_duration_minutes": 45,
  "key_questions": [
    "How did Theranos avoid regulatory scrutiny for so long?",
    "What role did board composition play in the fraud?",
    "How did media coverage shift from hype to investigation?",
    "What regulatory changes resulted from the scandal?"
  ],
  "tone": "investigative, measured, empathetic to victims"
}
```

## Key Design Decisions

1. **Haiku for fetching, Opus for narrative**: Source gathering is high-volume/low-complexity (Haiku is fast and cheap). Narrative structure requires deep creative reasoning (Opus excels here).

2. **Fact-checker as gatekeeper**: The fact-check phase sits between raw research and narrative construction, ensuring only verified claims flow into the script. This is the critical quality gate.

3. **Rework loops target the right phase**: Disputed facts rework to gather-sources (get better sources), not to fact-checker (which would just re-check the same bad sources). Narrative issues rework to self (iterate on structure).

4. **Citation chain is enforced**: Every claim in the script must trace back through fact-check-report to source-database to an actual URL. The compile-package phase validates this chain.

5. **Research package as deliverable**: The final output isn't just a script — it's a complete research package that a production team can use: bibliography, interview guide, visual notes, and methodology documentation.
