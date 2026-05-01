# Raw Session Log — ai-crafted-premium-sites QA

**Date:** 2026-04-30  
**Project:** ai-crafted-premium-sites-main (1) — Luxiflow website, second codebase version  
**Working dir:** `c:\Users\adaml\Downloads\ai-crafted-premium-sites-main (1)\ai-crafted-premium-sites-main`  
**Protocol used:** react-qa-protocol (4-phase)

---

## Project description

Same brand as the Luxiflow sessions (hello@luxiflow.io, luxiflow.io), but a newer / different ZIP download from Lovable.dev. React 18, TypeScript, Vite, Tailwind CSS, Framer Motion, EmailJS. No `node_modules` present at session start.

Price change vs. prior sessions: this version shows €299 Starter / €599 Professional (down from €1,500+). Either repriced or a different product tier.

---

## Phase 1 — Architecture Audit findings

### Critical
- **`tsconfig.app.json` `"types": ["vitest/globals"]`** — same tsconfig-types-pitfall as prior sessions. Blocks `@types/react`, causes TS2875. `strict: false` also.
- **`@/lib/animations` missing** — `Philosophy.tsx` and `Portfolio.tsx` both import `fadeInUp, staggerContainer, viewportConfig` from `@/lib/animations`. Module does not exist. `tsc --noEmit` fails on both files even though neither is imported anywhere in the app. Root cause: TypeScript compiles all files under `"include": ["src"]` regardless of whether they are imported.
- **`@tanstack/react-query`** — `QueryClient` + `QueryClientProvider` wrap the entire app in App.tsx. Zero `useQuery`/`useMutation` calls anywhere. Same dead dependency as prior sessions.
- **`@supabase/supabase-js`** — `src/integrations/supabase/client.ts` exists with placeholder URL `https://placeholder.supabase.co`. Never imported by any component.

### High
- **`color-mix()` in inline styles (Nav.tsx)** — both the nav header and mobile overlay used `color-mix(in srgb, var(--bg) %, transparent)` in React `style={}` props. No fallback possible. Same issue as prior sessions.
- **`role="list"` missing** — Pricing.tsx (feature list), Quote.tsx (feature list + add-ons), Footer.tsx (nav list + legal list), Process.tsx (`<ol>` with `listStyle: none`). All strip VoiceOver list semantics on Safari.
- **Placeholder testimonials live** — `SHOW_TESTIMONIALS = true` in Index.tsx but testimonials are generic ("Thomas K., Founder · SaaS Platform"). Same name appears verbatim in BOTH the Hero.tsx quote block and Testimonials.tsx — duplicate fake social proof.
- **No spam protection** — Contact form and Quote form both lack honeypot and time-trap.
- **Skip link** — CSS class `.skip-link` defined in index.css BUT the `<a href="#main" class="skip-link">` IS already in index.html line 104. False alarm — already implemented.
- **Fake `aggregateRating` in JSON-LD** — schema in index.html contained `"ratingValue": "5", "ratingCount": "3"` based on the fake testimonials.

### Medium
- `originalPrice` field in `Service` interface not used in data — JSX hardcoded `€399` for all cards instead of reading from data.
- `key={i}` index key in Testimonials.tsx.
- `name` attribute missing on all Quote.tsx form inputs (Contact.tsx had them, Quote.tsx didn't).
- Desktop CTA dropdown has `role="menu"` but no ArrowUp/ArrowDown/Escape keyboard navigation.
- No `.env.example` documenting required env vars.

---

## Phase 2 — Devil's Advocate personas

**Senior engineer:** `@/lib/animations` missing module is the most dangerous — `tsc --noEmit` fails. The tsconfig `types` override silently breaks `@types/react`. Two dead dependencies (~80–100 KB). 

**UX director:** Fake testimonials ("Thomas K.") appear in Hero AND Testimonials — duplicate. Savvy founders recognize generic fake social proof immediately. Quote form lacks `name` attrs so autofill won't work.

**Conversion specialist:** The nav `color-mix()` bug breaks ~8–12% Safari users. No spam protection means bot submissions the day it goes live. Fake `aggregateRating` in schema is an SEO/trust liability.

---

## Phase 3 — Fixes applied

| Fix | Files changed |
|-----|--------------|
| tsconfig: removed `types` array, added vitest triple-slash ref to vite-env.d.ts, enabled strict | tsconfig.app.json, vite-env.d.ts |
| Deleted archived broken-import components | Philosophy.tsx, Portfolio.tsx (deleted) |
| Removed dead react-query wrapper | App.tsx |
| `useInView` return type: `RefObject<T | null>` → `RefObject<T>` | hooks/useInView.ts |
| `color-mix()` CSS `@supports` fallback, both nav classes | index.css, Nav.tsx |
| `role="list"` on all bare lists | Pricing.tsx, Footer.tsx, Process.tsx, Quote.tsx |
| `SHOW_TESTIMONIALS = false`, `SHOW_HERO_QUOTE = false` | Index.tsx, Hero.tsx |
| Removed fake `aggregateRating` from JSON-LD schema | index.html |
| Honeypot + time-trap on Contact form (useRef pattern) | Contact.tsx |
| Honeypot + time-trap on Quote form (useRef pattern) | Quote.tsx |
| `name` attrs on all Quote form inputs | Quote.tsx |
| `originalPrice` driven from data | Services.tsx |
| Stable `key={t.name}` in Testimonials | Testimonials.tsx |
| Dead Terminal import removed from Hero | Hero.tsx |
| `.env.example` created | .env.example |
| npm install (node_modules were missing) | — |

**Final TypeScript:** `npx tsc --noEmit` → zero errors under full strict mode.

---

## Phase 4 — Scorecard

- Technical Stability: **9/10** (was ~6 before fixes)
- UI/UX Aesthetics: **9/10**
- Business Conversion: **8/10** (limited by no real social proof yet)

---

## New patterns / insights from this session

1. **Dead source file TypeScript pitfall** — archived components with broken imports still fail `tsc --noEmit` even if never imported, because TypeScript compiles everything under `"include": ["src"]`.

2. **`useRef` is better than `useState` for spam protection** — formLoadTime and honeypot ref don't need to trigger re-renders. `useRef` is correct here; prior wiki docs showed `useState` pattern which was suboptimal.

3. **`RefObject<T>` not `RefObject<T | null>`** — confirmed again: custom hooks returning refs should use `React.RefObject<T>` not `React.RefObject<T | null>` in the signature (even though `.current` is `T | null` at runtime — React's types handle this internally).

4. **JSON-LD `aggregateRating` should be removed when testimonials are fake** — it's not just a UX trust issue, it's a schema.org credibility issue that can backfire with Google.

5. **Skip link: check HTML, not just CSS** — the `.skip-link` CSS class can exist without the actual `<a>` element. The element must be in the HTML shell (`index.html` for SPAs), not in React components (which mount too late for accessibility tools).
