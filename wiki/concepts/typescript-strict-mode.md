---
title: "TypeScript Full Strict Mode"
type: concept
tags: [typescript, strictness, quality, react]
created: 2026-04-30
updated: 2026-04-30
sources: 2
---

## Definition

TypeScript's full strict configuration — the set of compiler flags that catch the maximum number of type errors at compile time. "No errors with tsc" is a significantly weaker guarantee than "no errors with full strict."

## How It Works

### The flags

```json
{
  "compilerOptions": {
    "strict": true,                          // Enables all strict checks below as a group
    "noUnusedLocals": true,                  // Error on declared but unused variables
    "noUnusedParameters": true,              // Error on declared but unused function parameters
    "noImplicitAny": true,                   // Error when type is inferred as 'any'
    "noFallthroughCasesInSwitch": true,      // Error on switch case without break/return
    "noImplicitReturns": true,               // Error on function paths that don't return
    "forceConsistentCasingInFileNames": true // Error on import casing mismatches
  }
}
```

Note: `"strict": true` already enables `noImplicitAny` and others. The additional flags above are NOT included in the `strict` umbrella and must be set explicitly.

### What "strict": true includes

- `strictNullChecks` — `null` and `undefined` are not assignable to other types
- `strictFunctionTypes` — stricter checking of function parameter types
- `strictBindCallApply` — stricter types for `.bind()`, `.call()`, `.apply()`
- `strictPropertyInitialization` — class properties must be initialized in constructor
- `noImplicitThis` — error on `this` with implicit `any` type
- `alwaysStrict` — emits `"use strict"` in all files

### The "zero errors without strict" gap

A codebase can pass `tsc --noEmit` without strict and still have:
- Functions returning `undefined` on some paths without declaring it
- `any`-typed variables that bypass all type checking
- Unused imports and variables that inflate bundle size
- Nullable values accessed without null checks

Enabling full strict on [[luxiflow]] required only 3 code changes — which means the codebase was already well-typed. On a poorly typed codebase, strict mode can surface hundreds of errors.

### The 3 fixes required in Luxiflow (first session)

1. `src/hooks/useInView.ts` — narrowed `RefObject<T | null>` to `RefObject<T>` to satisfy React 18's stricter ref typing
2. `src/main.tsx` — replaced `getElementById("root")!` non-null assertion with an explicit null guard:
   ```tsx
   // Before
   ReactDOM.createRoot(document.getElementById("root")!).render(...)
   
   // After
   const root = document.getElementById("root");
   if (!root) throw new Error("Root element not found");
   ReactDOM.createRoot(root).render(...)
   ```
3. `src/components/ui/sonner.tsx` — removed `next-themes` import (package deleted)

### When to enable strict mode

Enable strict from day one. The cost of retrofitting strict mode onto an existing codebase grows exponentially with codebase size. A 50-file codebase with strict enabled from the start requires zero migration work.

## Evidence & Examples

Applied to [[luxiflow]] in [[luxiflow-lovable-fixes-2026-04-30]]. The codebase already passed `tsc --noEmit` from our [[luxiflow-qa-session-2026-04-30]] session. Enabling full strict required only 3 surgical changes and produced zero additional errors after the fixes.

Confirmed again in [[ai-crafted-qa-session-2026-04-30]] on a second Luxiflow ZIP. The `useInView.ts` return type bug (`RefObject<T | null>` → `RefObject<T>`) appeared again — this is a recurring Lovable.dev scaffold pattern, not a one-off. After the fix, zero strict errors.

## Connections

[[react-qa-protocol]] (Phase 1: TypeScript audit) · [[tsconfig-types-pitfall]] · [[luxiflow]]

## Contradictions & Debates

`noUnusedLocals` and `noUnusedParameters` can be overly strict during development when you're iterating on APIs. A common workaround is to prefix unused variables with `_` (TypeScript respects this convention and does not error on `_unusedVar`).

## Open Questions

- Should `exactOptionalPropertyTypes` also be enabled? It makes optional properties (`prop?: string`) not assignable from `undefined` explicitly — stricter than the default but catches a real class of bugs.
- Is there a recommended order for adding strict flags incrementally to a legacy codebase? (e.g. start with `strictNullChecks`, then `noImplicitAny`, then the rest)
