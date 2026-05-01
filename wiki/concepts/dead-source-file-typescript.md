---
title: "Dead Source File TypeScript Pitfall"
type: concept
tags: [typescript, build, react, vite, architecture]
created: 2026-04-30
updated: 2026-04-30
sources: 1
---

## Definition

TypeScript compiles **all files matched by the `include` glob in `tsconfig.json`**, regardless of whether those files are imported by any other module. An "archived" or "disabled" component that is never imported anywhere but still lives under `src/` will be type-checked — and if it has broken imports, it will fail `tsc --noEmit`.

## How It Works

### The trap

A typical Vite + React tsconfig contains:

```json
{
  "include": ["src"]
}
```

This means every `.ts` and `.tsx` file under `src/` is compiled, whether or not any other file imports it. Vite itself uses a module graph (only bundles imported files), so the dev server and production build succeed. But `tsc --noEmit` — which is what CI and most editors use for type checking — sees all files.

### Example from [[ai-crafted-qa-session-2026-04-30]]

```
// Philosophy.tsx — archived, never imported by any route
import { staggerContainer, fadeInUp, viewportConfig } from "@/lib/animations";
//                                                           ^^^^^^^^^^^^^^^^
//                                                           This module does not exist.
```

`Philosophy.tsx` and `Portfolio.tsx` both imported `@/lib/animations` which had been deleted (or never created in this branch). Neither file was imported anywhere. Vite's dev server never errored. But `tsc --noEmit` failed on both.

### Why Vite doesn't catch it

Vite resolves modules on-demand from the entry point. If no entry point imports `Philosophy.tsx`, Vite never touches it. This creates a silent divergence: `npm run dev` and `npm run build` pass, but `npx tsc --noEmit` fails.

### Fix options

1. **Delete the file** — cleanest, appropriate when the component is truly abandoned.
2. **Move it outside `src/`** — e.g., `_archived/Philosophy.tsx` at the project root is not matched by `"include": ["src"]`.
3. **Add to `.gitignore` equivalent** — not possible in tsconfig (there's no exclude-by-pattern for "if not imported").
4. **Fix the import** — create a stub `src/lib/animations.ts` if the module is intentionally planned.

Option 1 is almost always correct. Option 2 is useful if the file might be re-enabled. Option 4 is useful if the module is in active development.

### Related: `tsconfig.exclude`

You can explicitly exclude files from TypeScript:

```json
{
  "include": ["src"],
  "exclude": ["src/components/Philosophy.tsx"]
}
```

But listing individual files is fragile. Better to establish a convention: archived files go outside `src/`.

## Evidence & Examples

Confirmed in [[ai-crafted-qa-session-2026-04-30]]: two Lovable.dev scaffold components (`Philosophy.tsx`, `Portfolio.tsx`) were commented "archived" and not rendered in any route, but still caused `tsc --noEmit` to error on the missing `@/lib/animations` module. Deleting the files resolved all errors.

## Connections

[[tsconfig-types-pitfall]] (another tsconfig footgun) · [[react-qa-protocol]] (Phase 1: check for dead files) · [[typescript-strict-mode]]

## Contradictions & Debates

Some teams prefer keeping archived components in `src/` as documentation of what was tried. If this is the practice, the archived file must still compile — meaning broken imports must be fixed or the file must be added to `tsconfig.exclude`.

## Open Questions

- Should Lovable.dev projects have a `_archived/` convention at the root level for disabled components? Would prevent this class of error without deleting history.
- Does `tsc --watch` (IDE language server mode) also error on unimported files? Yes — VS Code's TypeScript language service uses the same tsconfig and will show errors in archived files.
