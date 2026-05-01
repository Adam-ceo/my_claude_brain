---
title: "Feature Flag Pattern (React placeholder content)"
type: concept
tags: [react, pattern, ux, content]
created: 2026-04-30
updated: 2026-04-30
sources: 1
---

## Definition

A simple React pattern for hiding placeholder or unfinished content from production visitors without deleting the UI. A boolean constant at the top of the component controls whether the section renders. Flipping it to `true` re-enables the section when real content is ready.

## How It Works

```tsx
// At the top of the component file — easy to find, easy to flip
const SHOW_TESTIMONIALS = false;  // set true once you have real client testimonials

// In JSX — the section simply does not render when false
{SHOW_TESTIMONIALS && (
  <section>...</section>
)}
```

The constant naming convention is `SHOW_` + uppercase section name. This makes it instantly greppable across the codebase.

### Why not just delete the placeholder?

Deleting the UI means rebuilding it when the real content arrives. The flag keeps the component, the design, and the animations intact and battle-tested — only the render is gated.

### Where to apply it

Any UI element that exists but has no real data yet:
- Testimonials / social proof sections
- Hero quotes or case study callouts
- Portfolio grids
- "As seen in" logo bars
- Blog sections (before first post)
- Pricing tables (before pricing is confirmed)

## Evidence & Examples

Applied to [[luxiflow]] in [[luxiflow-qa-session-2026-04-30]]:
- `SHOW_TESTIMONIALS = false` in `src/pages/Index.tsx`
- `SHOW_HERO_QUOTE = false` in `src/components/Hero.tsx`

Both prevented placeholder `[Client Name]` / `[Company]` strings from appearing to real visitors.

## Connections

Part of the [[react-qa-protocol]] Phase 2 checklist. Related to the broader concept of progressive disclosure in product design.

## Contradictions & Debates

Some developers prefer deleting placeholder content entirely and building it fresh with real data. The argument: dead code is worse than no code. Counter-argument: the layout/animation work is non-trivial to recreate and the flag is a 2-second toggle.

## Open Questions

- Should the flag live in the component file or in a central `features.ts` config? Central config is better for multi-flag coordination but adds indirection.
