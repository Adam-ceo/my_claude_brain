---
title: "ARIA Accessibility Patterns"
type: concept
tags: [accessibility, aria, react, safari, keyboard]
created: 2026-04-30
updated: 2026-04-30
sources: 2
---

## Definition

Specific ARIA attribute and keyboard navigation patterns required for web components to be accessible to screen reader users and keyboard-only users. Two patterns in particular are easy to miss and fail audits silently.

## How It Works

### Pattern 1 — Safari VoiceOver: list-style:none kills list semantics

When you apply `list-style: none` via CSS to a `<ul>` or `<ol>`, Safari VoiceOver stops announcing the element as a list. Sighted users see no difference; blind users lose navigation context.

**Fix:** Add `role="list"` explicitly.

```tsx
// ❌ VoiceOver treats this as a generic container on Safari
<ul style={{ listStyle: "none" }}>

// ✅ Semantics preserved regardless of CSS
<ul role="list" style={{ listStyle: "none" }}>
```

This is a Safari-specific behavior. Chrome/Firefox VoiceOver equivalents do not strip the semantics. It was reported as a bug by Scott O'Hara and the ARIA working group but Safari has not changed it.

---

### Pattern 2 — role="menu" requires keyboard navigation

Any element with `role="menu"` must support the ARIA menu keyboard interaction model:

| Key | Action |
|-----|--------|
| `ArrowDown` | Move focus to next menu item |
| `ArrowUp` | Move focus to previous menu item |
| `Escape` | Close menu, return focus to trigger |
| `Enter` / `Space` | Activate focused item |

Without this, keyboard users can open the menu but cannot navigate it.

**React implementation:**

```tsx
const dropdownItemRefs = useRef<(HTMLButtonElement | null)[]>([]);

const handleDropdownKeyDown = useCallback((e: React.KeyboardEvent, index: number) => {
  const items = dropdownItemRefs.current.filter(Boolean) as HTMLButtonElement[];
  if (e.key === "ArrowDown") {
    e.preventDefault();
    items[(index + 1) % items.length]?.focus();
  } else if (e.key === "ArrowUp") {
    e.preventDefault();
    items[(index - 1 + items.length) % items.length]?.focus();
  } else if (e.key === "Escape") {
    setDropdownOpen(false);
  }
}, []);

// In JSX:
<button
  role="menuitem"
  ref={(el) => { dropdownItemRefs.current[idx] = el; }}
  onKeyDown={(e) => handleDropdownKeyDown(e, idx)}
>
```

Note: use `React.KeyboardEvent` (not the DOM `KeyboardEvent`) for the parameter type to avoid conflicts with existing `window.addEventListener("keydown", ...)` handlers in the same file.

---

### Pattern 3 — @media (hover: none) for touch devices

CSS `:hover` on touch devices is "sticky" — it fires on tap and remains active until the user taps elsewhere. This means hover styles (rings, glows, color changes) stay permanently visible after a tap.

**Fix:**

```css
@media (hover: none) {
  .process-step:hover .step-circle {
    border-color: rgba(184, 150, 46, 0.35) !important;
    box-shadow: none !important;
  }
}
```

Apply this guard to any hover effect that would look wrong if frozen in the hover state on mobile.

## Evidence & Examples

All three patterns applied to [[luxiflow]] in [[luxiflow-qa-session-2026-04-30]]:
- `role="list"` added to Process section `<ol>`
- ArrowUp/Down/Escape keyboard nav added to Nav.tsx CTA dropdown
- `@media (hover: none)` guard added for `.process-step:hover .step-circle`

`role="list"` confirmed and re-applied across 4 more elements in [[ai-crafted-qa-session-2026-04-30]]: Pricing.tsx feature list, Quote.tsx feature list, Footer.tsx navigation `<ul>`, Footer.tsx legal `<ul>`, and the Process.tsx `<ol>`. Confirms this is a systematic gap in Lovable.dev scaffolds. Also confirmed: the pattern applies to `<ol>` (ordered lists) just as much as `<ul>` — any list with `listStyle: none` and `padding: 0` needs `role="list"` on Safari.

**Skip link note:** the `.skip-link` CSS class may exist in a stylesheet without the `<a>` element being present in the HTML. For SPAs, the skip link `<a href="#main">` must be in `index.html` (the static shell), not in a React component — React components mount too late for screen readers. Always verify the element exists in the HTML, not just the CSS.

## Connections

[[react-qa-protocol]] · [[luxiflow]]

## Contradictions & Debates

The `role="list"` fix is sometimes criticized as over-engineering since it only affects Safari VoiceOver, which has a small share of screen reader usage. Counter: accessibility is a legal requirement in many jurisdictions (EN 301 549, WCAG 2.1 AA), and a 2-character attribute addition has zero downside.

## Open Questions

- Does `role="menu"` also require `aria-orientation="vertical"` for vertical menus? The ARIA spec says it defaults to vertical, but explicit is safer.
- Should focus be trapped inside a `role="menu"` (Tab should not exit the menu)? ARIA spec says Tab closes the menu rather than cycling through items — different behavior than a modal dialog.
