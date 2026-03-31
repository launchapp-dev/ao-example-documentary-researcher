# Documentary Research Pipeline

Autonomous documentary research: from a topic brief to a production-ready script with citations, interview guide, and visual notes — fully fact-checked.

## What It Does

Given a topic brief (topic, angle, target audience, duration), this pipeline:
1. Scopes the topic into research dimensions and targeted search queries
2. Gathers 8+ web sources, extracting claims, quotes, and statistics
3. Fact-checks every claim across sources, assigning confidence levels
4. Designs a compelling narrative arc (hook → setup → rising action → climax → resolution)
5. Writes a narration script with inline citations per verified claim
6. Compiles a complete production research package

## Workflow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    DOCUMENTARY RESEARCH                         │
└─────────────────────────────────────────────────────────────────┘

  [topic-brief.json]
        │
        ▼
  ┌──────────────┐  too-broad
  │ scope-topic  │──────────────┐
  │   (Haiku)    │◄─────────────┘ (rework, max 2x)
  └──────┬───────┘
         │ scoped
         ▼
  ┌──────────────────┐  insufficient
  │  gather-sources  │──────────────┐
  │    (Haiku)       │◄─────────────┘ (rework, max 3x)
  └──────┬───────────┘
         │ sufficient
         ▼
  ┌──────────────────┐  disputed ──────────────► gather-sources
  │   check-facts    │
  │    (Sonnet)      │  unreliable ────────────► scope-topic
  └──────┬───────────┘
         │ verified
         ▼
  ┌──────────────────┐  needs-work / restructure
  │  build-narrative │──────────────┐
  │    (Opus)        │◄─────────────┘ (rework, max 3x)
  └──────┬───────────┘
         │ compelling
         ▼
  ┌──────────────────┐
  │  write-script    │
  │    (Sonnet)      │
  └──────┬───────────┘
         │
         ▼
  ┌──────────────────┐
  │ compile-package  │
  │    (Sonnet)      │
  └──────┬───────────┘
         │
         ▼
  output/
  ├── script.md
  ├── interview-guide.md
  ├── visual-notes.md
  ├── research-package.md
  ├── source-bibliography.md
  └── research-stats.json
```

## Quick Start

```bash
# 1. Edit the topic brief
nano config/topic-brief.json

# 2. Start the AO daemon
cd examples/documentary-researcher
ao daemon start

# 3. Enqueue a research job
ao queue enqueue \
  --title "documentary-researcher" \
  --description "Research documentary on [your topic]" \
  --workflow-ref documentary-research

# 4. Watch it run
ao daemon stream --pretty

# 5. Find outputs in output/
```

## Agents

| Agent | Model | Role |
|---|---|---|
| **topic-scoper** | claude-haiku-4-5 | Reads topic brief, decomposes into research dimensions, generates 10-20 targeted search queries |
| **source-gatherer** | claude-haiku-4-5 | Fetches web pages, extracts claims/quotes/statistics, scores source credibility |
| **fact-checker** | claude-sonnet-4-6 | Cross-references claims across sources, assigns confidence levels, validates citation chains |
| **narrative-architect** | claude-opus-4-6 | Designs story arc with scene-by-scene structure, organizes verified facts into cinematic narrative |
| **scriptwriter** | claude-sonnet-4-6 | Writes narration script with inline citations, interview questions, and B-roll suggestions |

## AO Features Demonstrated

- **Multi-model routing**: Haiku for high-volume web fetching, Sonnet for analytical fact-checking, Opus for deep narrative design
- **Decision contracts**: Structured verdicts at each quality gate (scoped/too-broad, sufficient/insufficient, verified/disputed/unreliable, compelling/needs-work/restructure)
- **Rework loops**: Disputed facts route back to `gather-sources`; unreliable research routes back to `scope-topic`; weak narrative iterates on itself
- **Output contracts**: Structured JSON at each phase feeds cleanly into the next
- **Multi-agent pipeline**: 5 specialized agents with distinct expertise and models
- **Phase routing**: Non-linear workflow with up to 3 rework cycles per decision point
- **Production deliverables**: Final outputs are production-ready artifacts, not just raw data

## Directory Structure

```
documentary-researcher/
├── .ao/workflows/
│   ├── agents.yaml           # 5 agent profiles with detailed system prompts
│   ├── phases.yaml           # 6 phases with decision contracts
│   ├── workflows.yaml        # Main pipeline with rework routing
│   └── mcp-servers.yaml      # filesystem, fetch, sequential-thinking
├── config/
│   └── topic-brief.json      # INPUT: edit this with your topic
├── data/                     # RUNTIME: intermediate research artifacts
│   ├── research-scope.json
│   ├── source-database.json
│   ├── gathering-log.md
│   ├── fact-check-report.json
│   ├── fact-check-summary.md
│   ├── narrative-outline.json
│   ├── narrative-outline.md
│   └── narrative-feedback.json  # Written on narrative rework
├── output/                   # FINAL: production-ready deliverables
│   ├── script.md
│   ├── interview-guide.md
│   ├── visual-notes.md
│   ├── research-package.md
│   ├── source-bibliography.md
│   └── research-stats.json
├── sample-data/
│   └── sample-topic-brief.json   # Alternative topic: Opioid Crisis
├── CLAUDE.md
└── README.md
```

## Requirements

No API keys required — uses publicly accessible web sources via `mcp-fetch-server`.

**MCP servers** (installed automatically via npx):
- `@modelcontextprotocol/server-filesystem` — read/write all data and output files
- `mcp-fetch-server` — fetch web pages for source gathering
- `@modelcontextprotocol/server-sequential-thinking` — structured reasoning for scoping and fact-checking

**Models needed** (via AO):
- `claude-haiku-4-5` — topic scoping and source gathering
- `claude-sonnet-4-6` — fact-checking and scriptwriting
- `claude-opus-4-6` — narrative architecture

## Sample Topics

The included `config/topic-brief.json` researches **The Rise and Fall of Theranos**.

A second sample brief in `sample-data/sample-topic-brief.json` covers **The Opioid Crisis and Purdue Pharma**.

Copy either to `config/topic-brief.json` to run, or write your own.
