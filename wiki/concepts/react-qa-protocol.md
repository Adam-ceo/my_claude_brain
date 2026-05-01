---
title: "React QA Protocol"
type: concept
tags: [react, qa, methodology, typescript, accessibility]
created: 2026-04-30
updated: 2026-04-30
sources: 3
---

## Definition

A four-phase systematic quality assurance methodology for React/TypeScript websites before they go to production. It combines automated checks (TypeScript, build), manual code review, and accessibility auditing into a repeatable process.

## How It Works

### Phase 1 — Architecture Audit
- Read every component and page file
- Extract the design system (CSS vars, typography, color tokens)
- Check for dead imports and unused dependencies
- Verify TypeScript configuration is sound (`tsconfig.app.json`, `tsconfig.node.json`)
- Check environment variable handling (`.env.example` exists, no hardcoded secrets)
- **Check for archived/disabled components** under `src/` — they compile even if not imported. See [[dead-source-file-typescript]]
- **Verify skip link is in `index.html`** — not just CSS. The `<a href="#main">` must be in the static HTML shell, not a React component

### Phase 2 — Devil's Advocate Analysis
Three expert-persona critiques per section: senior engineer, UX director, conversion specialist. Goal: surface issues the builder is blind to.

Key checklist items:
- `name` attributes on all form inputs [[luxiflow-qa-session-2026-04-30]]
- No `<style>` tags inside `.map()` or component render
- `@media (hover: none)` guards on all hover effects
- CSS vars used instead of hardcoded `rgba()` in JSX strings
- `[[feature-flag-pattern]]` applied to all placeholder content
- Stable keys (not index) in all `.map()` renders
- **JSON-LD `aggregateRating` only present if reviews are real** — fake rating data is an SEO liability [[ai-crafted-qa-session-2026-04-30]]
- **Spam protection on all forms** — honeypot + time-trap (`useRef` pattern) [[spam-protection-honeypot-timetrap]]

### Phase 3 — Optimization Pass
Execute fixes in this order:
1. Remove dead dependencies (check bundle impact)
2. Fix TypeScript errors (`npx tsc --noEmit`)
3. Apply feature flags to placeholder content
4. Fix ARIA / accessibility issues
5. Move inline styles to CSS classes where they block fallbacks or re-render
6. Verify production build (`npm run build`)

### Phase 4 — Scorecard
Score three dimensions 1–10: Technical Stability, UI/UX Aesthetics, Business Conversion.
Declare whether the site reaches the target quality tier.

## Evidence & Examples

Applied in full to [[luxiflow]] three times:
- Pass 1: 19 changes across 9 files. [[luxiflow-qa-session-2026-04-30]]
- Pass 2: full strict mode, 41 dead deps removed, spam protection, per-route SEO. [[luxiflow-lovable-fixes-2026-04-30]]
- Pass 3 (different ZIP): 15 changes across 14 files, zero strict errors, new patterns: [[dead-source-file-typescript]], `useRef` spam protection. [[ai-crafted-qa-session-2026-04-30]]

## Connections

Produces outputs that inform: [[feature-flag-pattern]], [[css-progressive-enhancement]], [[aria-accessibility-patterns]], [[tsconfig-types-pitfall]], [[dead-source-file-typescript]], [[spam-protection-honeypot-timetrap]]

## Contradictions & Debates

The protocol is opinionated about moving inline styles to CSS — some teams prefer keeping styles co-located with components (CSS-in-JS philosophy). The tradeoff: CSS classes enable `@supports` and `@media` rules; inline styles do not.

## Open Questions

- How does this protocol adapt for Next.js (SSR) vs Vite (SPA)? Some checks (SEO, meta tags, OG images) become more critical in SSR contexts.
- Should Phase 2 include automated Lighthouse CI rather than manual estimation?
