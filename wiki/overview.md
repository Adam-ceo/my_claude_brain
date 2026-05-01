---
title: "Overview"
type: overview
tags: [synthesis, meta]
created: 2026-04-30
updated: 2026-04-30
sources: 3
---

# Overview

*This page is the living synthesis of the entire knowledge base. Updated on every ingest.*

---

## Current Thesis

**Production-ready React sites consistently fail on the same 8–9 categories of issues** — placeholder content, accessibility gaps, TypeScript misconfiguration, CSS browser compatibility, inline style anti-patterns, dead dependencies (especially shadcn bloat), missing spam protection, missing per-route SEO, and archived source files with broken imports. All of these are fixable in a two-to-three-pass QA cycle. The [[luxiflow]] case study is now the most thoroughly documented example in this knowledge base: three full QA passes across two ZIP versions.

A secondary thesis is emerging: **Lovable.dev scaffold projects share a predictable set of anti-patterns**. Across three sessions on the same brand's codebase, the same issues recur (`@tanstack/react-query` dead wrapper, `color-mix()` inline, `role="list"` missing, `tsconfig types` override, `useInView` return type). This suggests a checklist-first approach — run the known Lovable.dev footguns before a full audit.

---

## Major Themes

### 1. Systematic QA for React Projects
The [[react-qa-protocol]] provides a four-phase process. Three full passes on [[luxiflow]] across two ZIP versions: Pass 1 ([[luxiflow-qa-session-2026-04-30]]) = 19 fixes. Pass 2 ([[luxiflow-lovable-fixes-2026-04-30]]) = strict mode + 41 dead deps + SEO. Pass 3 ([[ai-crafted-qa-session-2026-04-30]]) = 15 fixes, new patterns confirmed.

### 2. Recurring React Anti-Patterns (confirmed across 3 sessions)
Eight patterns appear reliably and should be checked on every project:
- **Placeholder content in production** → [[feature-flag-pattern]]
- **TypeScript "zero errors" ≠ strict mode** → [[typescript-strict-mode]]
- **TypeScript misconfiguration** → [[tsconfig-types-pitfall]]
- **CSS fallbacks impossible via inline styles** → [[css-progressive-enhancement]]
- **Accessibility oversights** → [[aria-accessibility-patterns]]
- **shadcn install-everything bloat** → [[shadcn-dependency-bloat]]
- **Archived files with broken imports still compile** → [[dead-source-file-typescript]]
- **Inline `<style>` tags, index keys, dead imports** → [[react-qa-protocol]]

### 3. Browser Compatibility Nuances
Modern CSS features require `@supports` fallbacks in React. `color-mix()` confirmed in two sessions, both nav header and overlay. Safari has unique accessibility behaviors. See [[css-progressive-enhancement]] and [[aria-accessibility-patterns]].

### 4. Security & SEO Baseline
Every commercial React site needs: CAPTCHA-free spam protection on all forms ([[spam-protection-honeypot-timetrap]]) and per-route meta tags for search and social sharing ([[react-seo-helmet]]). Both are low-effort, high-impact additions.

### 5. TypeScript as Safety Net
Enabling strict mode catches real bugs invisible to basic `tsc --noEmit`. Across Luxiflow sessions, the recurring fixes are: `useInView` return type (`RefObject<T | null>` → `RefObject<T>`), `tsconfig types` array blocking `@types/react`, and unused strict mode flags. See [[typescript-strict-mode]].

---

## Key Entities

- [[luxiflow]] — AI web agency site; three-pass QA case study across two ZIP versions

---

## Key Concepts

- [[react-qa-protocol]] — 4-phase QA methodology for React/TypeScript sites (3 sources)
- [[dead-source-file-typescript]] — archived components still compiled by TypeScript; must delete or exclude
- [[feature-flag-pattern]] — `SHOW_X = false` for placeholder content in production
- [[css-progressive-enhancement]] — `@supports` + `color-mix()` fallback pattern (2 sources)
- [[aria-accessibility-patterns]] — `role="list"`, `role="menu"` keyboard nav, `hover:none` guards (2 sources)
- [[tsconfig-types-pitfall]] — explicit `types` array in tsconfig breaking `@types/react`
- [[typescript-strict-mode]] — full strict flags and what they catch beyond basic `tsc` (2 sources)
- [[shadcn-dependency-bloat]] — CSS weight and supply chain cost of unused shadcn components
- [[spam-protection-honeypot-timetrap]] — CAPTCHA-free bot blocking via honeypot + time-trap (2 sources)
- [[react-seo-helmet]] — per-route SEO component with react-helmet-async

---

## What's Missing / Next Directions

- **More project diversity** — all three sessions are the same brand's codebase. Patterns need confirmation across a different stack (Next.js, non-Lovable.dev scaffold).
- **Next.js / SSR variant** — documented for Vite SPAs; SSR introduces hydration, meta, and rendering failure modes.
- **Automated checklist** — [[react-qa-protocol]] is manual. Could be partially automated with ESLint rules and Lighthouse CI.
- **Dynamic sitemap** — the current Luxiflow sitemap.xml is hardcoded; a build-time generator is needed as the blog grows.
- **Lovable.dev footgun list** — a dedicated page cataloging the known Lovable.dev scaffold anti-patterns would accelerate future QA passes on similar projects.
- **Price discrepancy** — Luxiflow repriced from €1,500+ to €299 between sessions. Unknown if intentional or different variant.

---

## Stats

| Type | Count |
|------|-------|
| Sources | 3 |
| Entities | 1 |
| Concepts | 10 |
| Query pages | 0 |
| **Total** | **15** |

*Last updated: 2026-04-30 — Third source ingested: [[ai-crafted-qa-session-2026-04-30]]*
