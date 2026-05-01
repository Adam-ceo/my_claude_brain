---
title: "ai-crafted-premium-sites QA Session"
type: source
tags: [react, typescript, qa, accessibility, css, performance, spam-protection]
created: 2026-04-30
updated: 2026-04-30
sources: 1
raw: "raw/ai-crafted-qa-session-2026-04-30.md"
---

## Summary

A full 4-phase QA and optimization session on a second version of the [[luxiflow]] production website — same brand, different ZIP from Lovable.dev, pricing repriced to €299–€599. The session ran the complete [[react-qa-protocol]]: architecture audit, devil's advocate analysis, optimization pass, and scorecard. `node_modules` were absent at session start; `npm install` was run before the TypeScript check.

The codebase repeated most of the anti-patterns from prior Luxiflow sessions — `@tanstack/react-query` dead wrapper, `@supabase/supabase-js` unused client, `color-mix()` inline style with no fallback, `role="list"` missing on bare lists — confirming that these are not one-off mistakes but systematic Lovable.dev scaffold patterns. Two new patterns surfaced: archived components with broken imports (`@/lib/animations` missing) that still fail TypeScript compilation, and `useRef` as the correct implementation for spam protection (not `useState`).

After 15 fixes across 14 files, TypeScript compiles with zero errors under full strict mode. Scorecard: Technical 9/10, UI/UX 9/10, Business Conversion 8/10.

## Key Claims

- **TypeScript compiles all files under `"include": ["src"]` regardless of import graph** — `Philosophy.tsx` and `Portfolio.tsx` were "archived" (not imported by any route) but still caused `tsc --noEmit` to fail because they imported `@/lib/animations` which didn't exist. The fix: delete or move the files.
- **`useRef` is the correct primitive for spam protection timers and honeypot refs** — `useState` triggers a re-render on mount; `useRef` does not. The formLoadTime ref is a side-effect value with no display consequence.
- **`useInView` hook return type must be `RefObject<T>` not `RefObject<T | null>`** — React 18's DOM `ref` prop accepts `RefObject<T>` (where `.current` is `T | null` internally). Returning `RefObject<T | null>` from a hook produces TS2322 under strict mode.
- **JSON-LD `aggregateRating` with fake review count is an SEO liability** — Google can verify schema.org ratings against actual review signals; mismatches can suppress or penalize rich snippets.
- **Skip link must be in `index.html`, not in a React component** — the `.skip-link` CSS was defined but a dev check confirmed the `<a href="#main">` element was already in `index.html` line 104. SPAs mount React after the HTML shell, so accessibility-critical elements (skip links, noscript) belong in the static shell.

## Entities

[[luxiflow]]

## Concepts

[[react-qa-protocol]] · [[dead-source-file-typescript]] · [[spam-protection-honeypot-timetrap]] · [[css-progressive-enhancement]] · [[aria-accessibility-patterns]] · [[tsconfig-types-pitfall]] · [[typescript-strict-mode]] · [[feature-flag-pattern]]

## Connections

Extends [[luxiflow-qa-session-2026-04-30]] (same project, same anti-patterns confirmed) and [[luxiflow-lovable-fixes-2026-04-30]] (same strict mode approach). Together these three sessions form a three-pass QA record on the Luxiflow codebase.

Introduces new pattern [[dead-source-file-typescript]] not previously documented in the wiki.

Updates implementation detail in [[spam-protection-honeypot-timetrap]]: `useRef` > `useState` for formLoadTime.

## Contradictions

The spam protection implementation in [[luxiflow-lovable-fixes-2026-04-30]] used `useState` for the formLoadTime; this session used `useRef`. Both work, but `useRef` is correct because formLoadTime has no display consequence. The `useState` example in `spam-protection-honeypot-timetrap.md` should be updated.

## Open Questions

- Should archived/disabled components in a Lovable.dev project be moved to a `src/_archived/` folder outside the TypeScript include path, rather than deleted?
- The Luxiflow pricing changed from €1,500+ to €299 between sessions — was this a deliberate reprice or a different site variant? Needs clarification from Adam.
- Nav dropdown `role="menu"` still lacks ArrowUp/ArrowDown keyboard navigation — left as a known issue (not fixed in this session). Is this acceptable for an MVP?
