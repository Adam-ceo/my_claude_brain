---
title: "Luxiflow — Lovable.dev Fix Session"
type: source
tags: [react, typescript, seo, spam-protection, dependency-audit]
created: 2026-04-30
updated: 2026-04-30
sources: 1
raw: "raw/luxiflow-lovable-fixes-2026-04-30.md"
---

## Summary

A second QA pass on [[luxiflow]] applied by Lovable.dev, building on top of our own first QA session (see [[luxiflow-qa-session-2026-04-30]]). Two major fix groups: TypeScript full strict mode with aggressive dependency trimming, and spam protection plus per-route SEO infrastructure.

The dependency trim removed 41 npm packages and 45 shadcn UI component files that had been installed but never used. CSS gzip weight dropped 57% (12.0 KB → 5.17 KB). The JS bundle was essentially unchanged because Vite's tree-shaking had already eliminated most of the unused JS — confirming that the real cost of [[shadcn-dependency-bloat]] is in install time, supply chain risk, and CSS, not in runtime JS.

The spam protection added a honeypot field and a 3-second time-trap to the contact form — a CAPTCHA-free approach that silently rejects bots without any user-facing friction. The SEO component (`<SEO>`) uses react-helmet-async and provides per-route meta tags, Open Graph, Twitter cards, and article-level publishing metadata for blog posts.

## Key Claims

- Enabling TypeScript full strict mode required only 3 surgical code changes on a codebase that already passed `tsc --noEmit` — meaning "no errors" without strict is a meaningfully lower bar than full strict.
- The 41 removed packages were all installed as shadcn/ui's "install everything" scaffold and never called by any component. Tree-shaking removes JS but not CSS — which is why CSS dropped 57% while JS was unchanged.
- A honeypot + time-trap combination provides effective bot protection with zero CAPTCHA friction and no third-party dependency.
- A reusable `<SEO>` component built on react-helmet-async is the correct approach for per-route meta in a React SPA; individual pages should not manage their own `<head>` directly.
- Blog posts need `og:type=article` + ISO `article:published_time` for proper social sharing cards on LinkedIn and Twitter/X.

## Entities

[[luxiflow]]

## Concepts

[[shadcn-dependency-bloat]] · [[typescript-strict-mode]] · [[spam-protection-honeypot-timetrap]] · [[react-seo-helmet]]

## Connections

Directly extends [[luxiflow-qa-session-2026-04-30]] — this session applied further fixes to the same codebase. Together the two sessions represent a complete two-pass QA cycle on a production React site.

The [[shadcn-dependency-bloat]] concept confirms the cost model suspected in Session 1 when we noted Supabase and React Query were imported but unused. The full cleanup was more extensive than initially estimated.

## Contradictions

[[luxiflow-qa-session-2026-04-30]] preserved the Testimonials component behind a feature flag (`SHOW_TESTIMONIALS = false`). This session deleted the component entirely. Both approaches are defensible — the trade-off is discussed in [[feature-flag-pattern]].

## Open Questions

- The sitemap.xml is hardcoded with 8 URLs. As the blog grows, this needs to be auto-generated at build time. What is the right approach for Vite + React Router — a Vite plugin, a build script, or a server-side solution?
- react-helmet-async adds ~8 KB gz to the bundle. For a marketing site that doesn't need client-side meta updates post-render, would a static HTML template approach be lighter?
- The time-trap threshold is 3 seconds. Is this aggressive enough? Some accessibility tools (screen readers, switch controls) may cause slower form interactions. Should this be tunable?
