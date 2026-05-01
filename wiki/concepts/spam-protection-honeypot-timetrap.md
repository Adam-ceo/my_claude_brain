---
title: "Spam Protection — Honeypot + Time-Trap"
type: concept
tags: [security, forms, spam, accessibility, react]
created: 2026-04-30
updated: 2026-04-30
sources: 2
---

## Definition

A CAPTCHA-free spam protection technique that combines two passive checks: a hidden form field (honeypot) that bots fill but humans never see, and a minimum time-on-page check (time-trap) that rejects submissions made faster than a human could realistically read and fill the form.

## How It Works

### Honeypot field

A text input visually hidden far off-screen using absolute positioning (`left: -10000px`). Real users never interact with it. Bots that auto-fill all form fields will populate it.

```tsx
// State
const [website, setWebsite] = useState("");

// Hidden field in JSX — do NOT use display:none or visibility:hidden
// (some bots detect those). Position absolute off-screen fools them.
<div aria-hidden="true" style={{ position: "absolute", left: "-10000px", width: 1, height: 1, overflow: "hidden" }}>
  <label htmlFor="contact-website">Your website (leave blank)</label>
  <input
    id="contact-website"
    type="text"
    name="website"
    tabIndex={-1}
    autoComplete="off"
    value={website}
    onChange={(e) => setWebsite(e.target.value)}
  />
</div>

// On submit — silent fake-success, no email sent
if (website.trim().length > 0) {
  setSuccess(true);  // bot thinks it succeeded
  return;
}
```

**Why not `display: none`?** Sophisticated bots check `display` and `visibility` CSS and skip those fields. `position: absolute; left: -10000px` is harder to detect programmatically.

**Why silent success?** Telling a bot it failed teaches it to adapt. Fake success wastes its time and gives no signal.

### Time-trap

Records the timestamp when the form mounts. Rejects submissions made before a minimum elapsed time (3 seconds). Real humans read instructions, fill fields, and review before submitting — this takes at minimum 3–5 seconds. Automated submissions happen in milliseconds.

```tsx
// ✅ Correct: useRef — no re-render needed, it's a side-effect value
const formLoadTime = useRef(Date.now());

// On submit
if (Date.now() - formLoadTime.current < 3000) return; // silent drop, same as honeypot
```

**Why `useRef` not `useState`?** The load timestamp has no display consequence — it never changes the UI. `useState` would trigger an unnecessary re-render on mount. `useRef` is the correct primitive for imperative/side-effect values that don't affect rendering.

Unlike the honeypot (which silently succeeds), you can either silently drop or show a friendly error. Silent drop is simpler and consistent with the honeypot behavior.

### Accessibility consideration

The 3-second threshold may affect users with motor disabilities using switch controls or screen readers, who may interact more slowly. Consider raising the threshold to 1500ms (less aggressive) if accessibility is a priority. The honeypot alone catches most bots.

## Evidence & Examples

First implemented in [[luxiflow]] Contact.tsx in [[luxiflow-lovable-fixes-2026-04-30]]. Confirmed and applied again (with corrected `useRef` pattern) to BOTH Contact.tsx and Quote.tsx in [[ai-crafted-qa-session-2026-04-30]]. Applying to multiple forms in the same app: each form independently declares its own `honeypot` ref and `formLoadTime` ref — no shared state needed.

## Connections

[[react-qa-protocol]] (Phase 3: form hardening) · [[luxiflow]]

## Contradictions & Debates

Some argue CAPTCHA (reCAPTCHA v3, hCaptcha, Turnstile) is more reliable because it uses ML signals across millions of requests. Counter-arguments: CAPTCHA adds a third-party dependency, has privacy implications (GDPR), can be bypassed by CAPTCHA farms, and creates friction for legitimate users. Honeypot + time-trap handles 90%+ of spam with zero UX cost.

## Open Questions

- Should the time-trap threshold be configurable via an environment variable rather than hardcoded?
- Does the `autoComplete="off"` on the honeypot field prevent browser autofill from accidentally populating it for real users who have autofill enabled? Testing suggests yes, but worth verifying across browsers.
