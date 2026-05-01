---
title: "TypeScript tsconfig types Restriction Pitfall"
type: concept
tags: [typescript, tsconfig, vitest, react, gotcha]
created: 2026-04-30
updated: 2026-04-30
sources: 1
---

## Definition

A non-obvious TypeScript configuration gotcha where adding an explicit `types` array in `tsconfig.app.json` restricts TypeScript to loading *only* the listed type packages, silently breaking all other `@types/*` packages including `@types/react`.

## How It Works

### The trap

When you add vitest globals to a React project (so `describe`, `it`, `expect` are available in test files without imports), a common instruction is:

```json
// tsconfig.app.json
{
  "compilerOptions": {
    "types": ["vitest/globals"]
  }
}
```

This looks harmless. It is not.

TypeScript's `types` field in `compilerOptions` means: **load ONLY these type declaration packages, and ignore everything else in `node_modules/@types/`**.

The result: `@types/react` is no longer loaded → `react/jsx-runtime` cannot be found → every `.tsx` file reports:

```
TS2875: This module is declared with 'export =', and can only be used with a
default import when using the 'esModuleInterop' flag.
```

or more commonly:

```
TS2307: Cannot find module 'react/jsx-runtime' or its corresponding type declarations.
```

### The fix

**Step 1:** Remove `"types"` from `tsconfig.app.json` entirely.

```json
// ✅ tsconfig.app.json — no types restriction
{
  "compilerOptions": {
    "target": "ES2020",
    "jsx": "react-jsx"
    // ... no "types" field
  }
}
```

**Step 2:** Restore vitest globals in the test setup file using a triple-slash directive:

```ts
// src/test/setup.ts
/// <reference types="vitest/globals" />
import "@testing-library/jest-dom";
```

The triple-slash reference applies only to the file where it appears (and files that include it via the test config). It does not affect the main application TypeScript compilation.

### Why does vitest/globals work this way?

Vitest's `globals: true` config option makes `describe`, `it`, `expect` etc. available globally in test files. The type definitions for these globals live in `vitest/globals`. They only need to be loaded in the test compilation context, not the application context. Putting them in `tsconfig.app.json` was always the wrong location.

### The correct location for test types

```ts
// Option A: triple-slash in setup.ts (simple, no extra config)
/// <reference types="vitest/globals" />

// Option B: separate tsconfig.test.json with its own types array
// (correct for larger projects where test and app compilation are separate)
```

## Evidence & Examples

Encountered in [[luxiflow]] during [[luxiflow-qa-session-2026-04-30]]. The project had `"types": ["vitest/globals"]` in `tsconfig.app.json`. Removing it + adding the triple-slash reference in `src/test/setup.ts` resolved all TS2875 errors. Confirmed with `npx tsc --noEmit` → zero errors.

## Connections

[[react-qa-protocol]] (Phase 1 architecture audit) · [[luxiflow]]

## Contradictions & Debates

Some setups intentionally use a separate `tsconfig.test.json` that extends the main config and adds `"types": ["vitest/globals"]`. This is correct and does not have the problem described above, because the restriction applies only to the test compilation, not the application compilation. The pitfall only occurs when the restriction is placed in the shared/application tsconfig.

## Open Questions

- Does the same pitfall apply to Jest with `@types/jest`? Yes — the same mechanism applies if `"types": ["jest"]` is put in the application tsconfig.
- Vite projects often have two tsconfig files (`tsconfig.app.json` for the app, `tsconfig.node.json` for Vite config). Which one should the vitest types go in? Neither — use the triple-slash reference in the test setup file.
