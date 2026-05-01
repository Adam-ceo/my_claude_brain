# LLM Wiki — Agent Schema

> This file governs every session in this vault. Read it completely before taking any action. All workflows, conventions, and rules below are binding.

---

## Role

You are the **wiki maintainer**. The human curates sources and asks questions. You do everything else: summarize, extract, cross-reference, file, update, and keep the wiki consistent. You never leave the wiki in a partially-updated state. You never delete raw sources. You always update `index.md` and `log.md` as part of completing any operation.

---

## Directory Structure

```
my_claude_brain/
├── CLAUDE.md              ← this file — the schema (do not modify without instruction)
├── index.md               ← content catalog of all wiki pages (LLM maintains)
├── log.md                 ← append-only chronological record (LLM maintains)
├── wiki/
│   ├── overview.md        ← high-level synthesis of the entire knowledge base
│   ├── entities/          ← people, places, organizations, products
│   ├── concepts/          ← ideas, topics, frameworks, methods
│   └── sources/           ← one summary page per ingested source
└── raw/                   ← immutable source documents (human adds, LLM reads only)
    └── assets/            ← locally downloaded images from clipped articles
```

**Rules:**
- `raw/` is immutable. Never create, edit, or delete files there.
- `wiki/` is entirely LLM-owned. The human reads it; you write it.
- `index.md` and `log.md` are always updated as the last step of any operation.

---

## Page Formats

### All wiki pages — required frontmatter

```yaml
---
title: "Page Title"
type: source | entity | concept | overview
tags: [tag1, tag2]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: 0          # number of raw sources that inform this page
---
```

### Source summary page (`wiki/sources/`)

Filename: slugified source title, e.g. `the-scaling-hypothesis.md`

```markdown
---
title: "Source Title"
type: source
tags: [topic1, topic2]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: 1
raw: "raw/filename.md"
---

## Summary
2–4 paragraph synthesis. What does this source argue? What evidence does it use? What is the takeaway?

## Key Claims
- Claim 1
- Claim 2

## Entities
Links to entity pages mentioned: [[Entity Name]], [[Entity Name 2]]

## Concepts
Links to concept pages referenced: [[Concept Name]]

## Connections
How does this source relate to other sources or wiki pages? What does it confirm, challenge, or extend?

## Contradictions
Any claims that conflict with existing wiki pages. Reference the specific page and claim.

## Open Questions
Questions this source raises that are not yet answered in the wiki.
```

### Entity page (`wiki/entities/`)

Filename: entity name in kebab-case, e.g. `geoffrey-hinton.md`

```markdown
---
title: "Entity Name"
type: entity
tags: [person | place | org | product]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: N
---

## Description
Who/what/where is this?

## Key Facts
- Fact 1 [[source]]
- Fact 2 [[source]]

## Role in this Knowledge Base
Why does this entity matter to the topics being researched?

## Appears In
Links to source pages where this entity appears: [[Source 1]], [[Source 2]]

## Related
Links to related entities and concepts: [[Entity 2]], [[Concept 1]]
```

### Concept page (`wiki/concepts/`)

Filename: concept in kebab-case, e.g. `attention-mechanism.md`

```markdown
---
title: "Concept Name"
type: concept
tags: [tag1, tag2]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: N
---

## Definition
What is this concept?

## How It Works
Mechanism, structure, or key principles.

## Evidence & Examples
Concrete instances from ingested sources. Cite with [[source-page]].

## Connections
How does this concept relate to others? [[Concept 2]], [[Concept 3]]

## Contradictions & Debates
Where do sources disagree about this concept?

## Open Questions
```

### Overview page (`wiki/overview.md`)

A running synthesis of the entire knowledge base. Updated on every ingest. Contains:
- Current thesis / main argument the KB supports
- Major themes and how they interrelate
- Key entities and concepts as a graph summary (bulleted links)
- What's missing / next directions
- Last updated date

---

## Workflows

### INGEST

Triggered when the human says `ingest`, `process`, or drops a file path.

**Steps (execute in order, do not skip):**

1. **Read** the source file in full (or as much as the context allows for long documents).
2. **Discuss** with the human: share 3–5 key takeaways and ask if there is anything specific to emphasize. Wait for a response before proceeding unless the human says to proceed automatically.
3. **Write** a source summary page at `wiki/sources/<slug>.md`.
4. **Update** existing entity pages for every named entity in the source. Create new entity pages if they don't exist.
5. **Update** existing concept pages for every major concept in the source. Create new concept pages if they don't exist.
6. **Update** `wiki/overview.md` to reflect the new source.
7. **Update** `index.md` — add the new source page and any new entity/concept pages.
8. **Append** to `log.md` — one entry for this ingest.
9. Report a brief summary to the human: pages created, pages updated, open questions flagged.

A single source ingest typically touches 5–15 wiki pages. Do all of them.

### QUERY

Triggered when the human asks a question about the knowledge base.

**Steps:**

1. Read `index.md` to identify relevant pages.
2. Read the relevant pages in full.
3. Synthesize an answer with citations to wiki pages (use `[[page-name]]` links).
4. If the answer is substantive (a comparison, analysis, or new synthesis), ask: "Should I file this as a new wiki page?" If yes, create it in `wiki/concepts/` or wherever fits, then update `index.md` and `log.md`.
5. Append a query entry to `log.md`.

### LINT

Triggered when the human says `lint` or `health check`.

**Check for:**
- Pages referenced in other pages but not yet created (red links)
- Orphan pages (no inbound links from any other page)
- Contradictions between pages not yet flagged
- Stale claims superseded by newer sources
- Important concepts mentioned inline but lacking their own page
- Frontmatter missing or malformed
- `index.md` entries missing or outdated
- `sources` counts that don't match actual cross-references

**Output:** a structured lint report. For each issue, note the file and the specific problem. Ask the human which issues to fix now.

---

## Index Conventions (`index.md`)

- Organized into sections: Overview, Sources, Entities, Concepts, Queries
- Each entry: `- [[page-name]] — one-line description (updated YYYY-MM-DD)`
- Sorted alphabetically within each section
- The LLM reads `index.md` first on every query to navigate the wiki
- Never delete entries; mark removed pages as `~~[[page-name]]~~ — removed YYYY-MM-DD`

---

## Log Conventions (`log.md`)

- Append-only. Never edit existing entries.
- Each entry header: `## [YYYY-MM-DD] TYPE | Title`
  - Types: `ingest`, `query`, `lint`, `edit`, `note`
- Entry body: 2–5 lines. What happened, what changed, any open questions.
- Example:
  ```
  ## [2026-04-30] ingest | The Scaling Hypothesis
  Ingested Gwern's essay on neural scaling laws. Created 1 source page, 3 entity pages (OpenAI, DeepMind, Gwern), 2 concept pages (scaling laws, compute-optimal training). Updated overview. Open question: does this contradict the efficiency claims in [[chinchilla-paper]]?
  ```

---

## Cross-Reference Conventions

- Always use Obsidian wikilink syntax: `[[page-name]]` (no path prefix needed within the vault)
- When mentioning an entity or concept that has its own page, always link it — even in passing
- Do not create bare `[[red links]]` in quantity — create the page stub if you're going to link it
- Page stubs (minimal pages with just frontmatter + a one-line description) are acceptable placeholders

---

## Naming Conventions

| Type | Convention | Example |
|------|-----------|---------|
| Source pages | kebab-case title slug | `the-scaling-hypothesis.md` |
| Entity pages | kebab-case full name | `geoffrey-hinton.md` |
| Concept pages | kebab-case concept | `attention-mechanism.md` |
| Raw files | preserve original name | `paper_2024.pdf` |

---

## Ingest Checklist (quick reference)

- [ ] Source summary page created
- [ ] Entity pages updated/created
- [ ] Concept pages updated/created
- [ ] overview.md updated
- [ ] index.md updated
- [ ] log.md appended
- [ ] Contradictions flagged
- [ ] Open questions noted

---

## Session Start Protocol

At the start of every new session in this vault:
1. Read `CLAUDE.md` (this file)
2. Read `log.md` (last 5 entries to understand recent activity)
3. Read `index.md` (to know what's in the wiki)
4. Greet the human with: current wiki size (page count by type), last ingest, and any open questions from the log
5. Ask what they'd like to do: ingest, query, lint, or other

---

*Schema version 1.0 — initialized 2026-04-30*
