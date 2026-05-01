---
title: "CSS Progressive Enhancement (@supports fallbacks)"
type: concept
tags: [css, browser-compatibility, safari, react]
created: 2026-04-30
updated: 2026-04-30
sources: 2
---

## Definition

The practice of writing CSS that works in all browsers first, then layering in modern features for browsers that support them using `@supports`. Ensures no visual regression in older browsers while still using the best available API where possible.

## How It Works

### The React inline style problem

React's `style` prop is a JavaScript object. Object keys must be unique — you cannot write:

```tsx
// ❌ This does NOT work — duplicate key, second wins, first is silently dropped
style={{
  background: "rgba(247, 247, 245, 0.94)",   // fallback
  background: "color-mix(in srgb, var(--bg) 94%, transparent)",  // modern
}}
```

The only way to provide a CSS fallback from React is to move the property to a CSS class.

### The @supports pattern

```css
/* Base style — works everywhere */
.nav-header-bg {
  background: rgba(247, 247, 245, 0.94);
}

/* Progressive enhancement — only applied where color-mix() is supported */
@supports (background: color-mix(in srgb, white 0%, black)) {
  .nav-header-bg {
    background: color-mix(in srgb, var(--bg) 94%, transparent);
  }
}
```

Then in React, just apply the class:
```tsx
<header className="nav-header-bg">
```

### color-mix() browser support

| Browser | Supported since |
|---------|----------------|
| Chrome | 111 (Mar 2023) |
| Firefox | 113 (May 2023) |
| Safari | 16.2 (Dec 2022) |
| Safari iOS | 16.2 (Dec 2022) |

Older versions (Safari 15.x, Firefox <113) require the plain `rgba()` fallback.

### When to use @supports vs. just dropping the modern feature

Use `@supports` when: the feature provides a meaningful visual improvement (frosted glass, gradient mesh, container queries) AND the fallback is graceful (not broken-looking).

Don't over-engineer: if 95%+ of your audience is on modern browsers and the fallback is acceptable, the `@supports` block adds complexity for marginal gain.

## Evidence & Examples

First applied to [[luxiflow]] nav header in [[luxiflow-qa-session-2026-04-30]].

Confirmed and re-applied in [[ai-crafted-qa-session-2026-04-30]]: the same Nav.tsx component had TWO `color-mix()` instances in inline styles — the nav header background AND the mobile overlay background. Both were extracted to CSS classes (`.nav-bg`, `.nav-overlay-bg`) with `@supports` fallbacks. The CSS `@supports` guard used:

```css
@supports (background: color-mix(in srgb, red 50%, blue)) {
  .nav-bg {
    background: color-mix(in srgb, var(--bg) 94%, transparent);
  }
}
```

Note the test expression uses concrete colors (`red`, `blue`) — not CSS variables — to ensure the `@supports` test itself is universally parseable.

## Connections

[[react-qa-protocol]] (Phase 3 fix) · [[luxiflow]]

## Contradictions & Debates

Some argue that since `color-mix()` is supported in all browsers released after late 2022, the fallback is no longer necessary for most projects. The counter: you don't control when users update their browsers, and the `@supports` wrapper is five lines.

## Open Questions

- Does `color-mix()` in the `in srgb` color space produce the expected result across all browsers, or are there rendering inconsistencies between Chrome and Safari's color management?
