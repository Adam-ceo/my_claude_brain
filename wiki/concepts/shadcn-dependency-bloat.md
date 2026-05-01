---
title: "shadcn/ui Dependency Bloat"
type: concept
tags: [react, shadcn, dependencies, performance, bundle]
created: 2026-04-30
updated: 2026-04-30
sources: 1
---

## Definition

The pattern where shadcn/ui's scaffold ("install everything") approach adds dozens of Radix UI primitives, utility packages, and component files to a project that will only ever use a small subset. Since shadcn components are copied into the codebase rather than imported from a package, unused components and their dependencies stay in `node_modules` and CSS forever unless manually removed.

## How It Works

shadcn/ui works by copying component source code into `src/components/ui/`. The CLI installs all peer dependencies at scaffold time. A typical scaffold installs 20–30 Radix UI packages plus: `class-variance-authority`, `cmdk`, `date-fns`, `embla-carousel-react`, `input-otp`, `react-day-picker`, `react-hook-form`, `@hookform/resolvers`, `react-resizable-panels`, `recharts`, `vaul`, `zod`, `next-themes`.

If the project uses only 2–3 shadcn components, the rest become dead weight.

### What tree-shaking removes (and doesn't)

| Asset | Unused code removed? |
|-------|---------------------|
| JS bundle | ✅ Yes — Vite tree-shakes unused exports |
| CSS bundle | ❌ No — all Tailwind classes from unused components stay |
| node_modules | ❌ No — all packages stay on disk |
| Install time | ❌ No — slower npm install |
| Supply chain | ❌ No — more packages = more attack surface |

**The real cost of shadcn bloat is CSS weight, install time, and supply chain risk — not runtime JS.**

### Build delta from [[luxiflow-lovable-fixes-2026-04-30]]

Removing 41 packages and 45 shadcn component files from [[luxiflow]]:
- CSS: 64.66 KB → 20.17 KB raw, 12.0 KB → 5.17 KB gz (**-57% gz**)
- JS: essentially unchanged (tree-shaking had already excluded unused code)

## Evidence & Examples

Applied to [[luxiflow]] in [[luxiflow-lovable-fixes-2026-04-30]]. Only 2 shadcn components were actually used (sonner.tsx for toasts, tooltip.tsx for nav). 45 other component files were deleted along with their peer dependencies.

## Connections

[[react-qa-protocol]] (Phase 1: check for unused shadcn components) · [[luxiflow]]

## Contradictions & Debates

Some argue that keeping unused shadcn components is acceptable because: (1) they're not shipped to users due to tree-shaking, and (2) having them available makes future development faster. Counter-argument: CSS is not tree-shaken, supply chain risk is real, and you can always re-add components with `npx shadcn@latest add <component>` in seconds.

## Open Questions

- Does shadcn/ui have a "minimal install" option that only adds the specific component requested without all peer dependencies? As of 2026-04-30, the CLI installs all deps even for a single component.
- Should the [[react-qa-protocol]] Phase 1 checklist include a mandatory shadcn audit step?
