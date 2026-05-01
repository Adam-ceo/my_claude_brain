---
title: "Luxiflow Website QA Session"
type: source
tags: [react, typescript, qa, accessibility, css, performance]
created: 2026-04-30
updated: 2026-04-30
sources: 1
raw: "raw/luxiflow-qa-session-2026-04-30.md"
---

## Summary

A full QA and optimization session on the [[luxiflow]] production website — a one-person AI web development agency site targeting founders at €1,500–€3,500. The site was built in React 18, TypeScript, Vite, Tailwind CSS, and Framer Motion.

The session ran through a four-phase protocol: architecture audit, devil's advocate analysis, automated optimization, and a final scorecard. 19 concrete changes were made across 9 files. The codebase had strong visual design and a coherent CSS custom property system, but contained several recurring anti-patterns: placeholder content in production, hardcoded color values in JSX, inline `<style>` tags, a dead dependency, and a TypeScript configuration bug that silently broke the JSX runtime.

The final scores were: Technical Stability 9/10, UI/UX Aesthetics 9/10, Business Conversion 8/10. The remaining gap is content (no real testimonials or case studies), not code quality.

## Key Claims

- The `"types": ["vitest/globals"]` entry in tsconfig.app.json broke `@types/react` loading, causing TS2875 — a non-obvious gotcha that affects any project mixing vitest globals with React.
- `color-mix(in srgb, ...)` is unsupported in Safari <16.2 and Firefox <113; React inline styles cannot provide a fallback, requiring a CSS class + `@supports` pattern.
- Safari VoiceOver silently strips list semantics from `<ol>/<ul>` elements that have `list-style: none` — `role="list"` is required to restore them.
- ARIA `role="menu"` elements must implement ArrowUp/ArrowDown/Escape keyboard navigation to pass accessibility audits.
- `@tanstack/react-query` was completely unused but wrapped the entire app, adding dead bundle weight.
- The `[[feature-flag-pattern]]` (`const SHOW_X = false`) is the correct way to ship placeholder-free production code while keeping the UI ready for real content.

## Entities

[[luxiflow]]

## Concepts

[[react-qa-protocol]] · [[feature-flag-pattern]] · [[css-progressive-enhancement]] · [[aria-accessibility-patterns]] · [[tsconfig-types-pitfall]]

## Connections

This source establishes the first practical case study in the knowledge base. It demonstrates the [[react-qa-protocol]] applied end-to-end on a real commercial project. The lessons — especially the TypeScript pitfall and the CSS @supports pattern — are directly reusable on any React/Vite project.

## Contradictions

None yet — this is the first source in the wiki.

## Open Questions

- Does the `color-mix()` support gap matter in practice given that Safari 16.2 launched in late 2022? Worth checking analytics for actual browser distribution of Luxiflow's target audience.
- The Supabase dependency is imported but unused. Should it be removed entirely or is it planned for a future feature (e.g. a blog CMS or lead capture)?
- What is the conversion rate benchmark for a comparable one-person agency site? The 8/10 business conversion score needs a real baseline to be meaningful.
