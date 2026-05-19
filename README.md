# LLM Wiki

This repository is a clean starting point for an agent-maintained knowledge base following the LLM Wiki pattern.

Raw sources stay immutable in `raw/`. The AI agent reads those sources, synthesizes reusable knowledge into `wiki/`, maintains links between pages, records operations in `wiki/log.md`, and answers future questions from the wiki before falling back to raw files.

## Agent Contract

The canonical operating instructions are in `AGENTS.md`.

Any model used with this repository should follow that file first. In particular, it must:

- treat `raw/` as read-only and append-only;
- read `wiki/index.md` before answering wiki questions or creating new wiki pages;
- create summaries in `wiki/summaries/` for ingested sources;
- create or update concepts, entities, and syntheses only when supported by sources;
- use Obsidian-style wikilinks such as `[[concepts/example]]`;
- keep `wiki/index.md`, `wiki/overview.md`, and `wiki/log.md` current after wiki changes;
- mark contradictions, uncertainty, and missing evidence explicitly.

## Directory Structure

```text
raw/                         # raw source files; never rewritten by the agent
wiki/
  index.md                   # operational catalog; read first for queries
  log.md                     # append-only operation log
  overview.md                # living summary of the whole knowledge base
  summaries/                 # one summary page per raw source
  concepts/                  # concepts, patterns, methods, strategies
  entities/                  # people, products, tools, companies, systems
  syntheses/                 # comparisons, decisions, cross-source analyses
  journal/                   # optional research/session notes
  presentations/             # optional Marp presentations generated from the wiki
  _maintenance/              # health checks, backlog, link maps, audits
```

## Core Workflows

| Workflow | Typical trigger | Expected behavior |
|---|---|---|
| Ingest | `ingest raw/source.md` | Read the source, create/update wiki pages, link them, update index, overview when needed, and log. |
| Query | natural-language question | Read `wiki/index.md` first, then relevant wiki pages; use `raw/` only when the wiki is incomplete. |
| Lint | `lint`, `health check`, `controlla la wiki` | Audit frontmatter, links, sources, duplicates, sections, orphan pages, and contradictions. |
| Synthesis | `confronta`, `fammi una sintesi`, `decision matrix` | Create or update `wiki/syntheses/` only for reusable cross-page analysis. |
| Graph | `build graph`, `mappa collegamenti` | Generate maintenance reports or link maps in `wiki/_maintenance/`. |
| Presentation | `presentazione`, `slides`, `marp` | Generate Marp slides in `wiki/presentations/` from existing wiki knowledge. |

## Starting State

This repository intentionally starts with no ingested knowledge:

- `raw/` contains only `.gitkeep`;
- summary, concept, entity, synthesis, journal, and presentation directories are empty;
- `wiki/index.md` and `wiki/overview.md` describe the empty baseline;
- `wiki/log.md` records the baseline setup.

To begin, add one or more source files under `raw/` and ask the model to ingest them.

```text
ingest raw/my-first-source.md
```

## License

MIT
