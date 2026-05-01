# Luxiflow — Lovable.dev Fix Session — 2026-04-30

## Context

Second QA pass on the Luxiflow website, applied by Lovable.dev on top of the 2026-04-30 
codebase delivered after our first QA session. Two fix groups applied: TypeScript strict 
mode + dependency trim (#2), and spam protection + per-route SEO (#3).

## Fix #2 — TypeScript Full Strict + Dependency Trim

### tsconfig.app.json — strict flags added
- "strict": true
- "noUnusedLocals": true
- "noUnusedParameters": true
- "noImplicitAny": true
- "noFallthroughCasesInSwitch": true
- "noImplicitReturns": true
- "forceConsistentCasingInFileNames": true

### Code fixes required by strict mode (3 surgical changes)
1. src/hooks/useInView.ts — return type narrowed from RefObject<T | null> to RefObject<T>
2. src/main.tsx — replaced non-null assertion getElementById("root")! with explicit guard
3. src/components/ui/sonner.tsx — removed next-themes dependency

### Dependencies removed (41 packages)
All unused Radix UI primitives (accordion, alert-dialog, aspect-ratio, avatar, checkbox, 
collapsible, context-menu, dialog, dropdown-menu, hover-card, label, menubar, navigation-menu, 
popover, progress, radio-group, scroll-area, select, separator, slider, switch, tabs, toast, 
toggle, toggle-group), plus: @supabase/supabase-js, @tanstack/react-query, 
class-variance-authority, cmdk, date-fns, embla-carousel-react, input-otp, lovable-tagger, 
next-themes, react-day-picker, react-hook-form, @hookform/resolvers, react-resizable-panels, 
recharts, vaul, zod.

### Files deleted
- 45 unused src/components/ui/*.tsx shadcn primitives (kept only sonner.tsx, tooltip.tsx)
- src/hooks/use-mobile.tsx, use-toast.ts
- src/components/NavLink.tsx, Philosophy.tsx, Portfolio.tsx, Testimonials.tsx
- src/integrations/ (unused Supabase client)

### Build delta
| Asset      | Before            | After             | Change     |
|------------|-------------------|-------------------|------------|
| index.css  | 64.66 KB / 12.0 KB gz | 20.17 KB / 5.17 KB gz | -57% gz |
| index.js   | 444 KB / 141 KB gz | 468 KB / 149 KB gz | +5% (react-helmet-async) |
| Total gz   | ~153 KB           | ~155 KB           | ~unchanged |

Key insight: tree-shaking already removed most Radix JS from the bundle. Real wins are 
install time, supply chain risk, typesafety, and CSS weight.

## Fix #3 — Spam Protection + Per-Route SEO + Sitemap

### src/components/Contact.tsx — spam protection
- Honeypot field: hidden "website" input (position absolute, left -10000px, aria-hidden, 
  tabIndex=-1). If filled: silent fake-success without sending email.
- Time-trap: formMountedAtRef records mount time. Submission rejected with friendly toast 
  if elapsed < 3000ms. Bots submit in <500ms; humans take >3s.
- role="status" + aria-live="polite" on success container for screen reader announcement.

### src/components/SEO.tsx — new reusable SEO component
Built with react-helmet-async. Props: title, description, canonical, publishedTime 
(ISO date for articles), type ("website"|"article"), noIndex (boolean).
Sets: <title>, meta description, canonical, robots (if noIndex), og:type, og:url, 
og:title, og:description, og:image, article:published_time, twitter:card, twitter:title, 
twitter:description, twitter:image.
SITE constant = "https://luxiflow.io", DEFAULT_OG = "/og-image.jpg".

### Per-page SEO wired in
- / — home meta
- /blog — blog index meta
- /blog/:slug — per-post title, excerpt, ISO publishedTime, og:type=article
- /privacy-policy, /cookie-policy, /terms-of-service — own meta
- /404 — own title + noindex, follow

### SEO infrastructure
- public/sitemap.xml — 8 URLs (home, blog index, 3 posts, 3 legal pages)
- public/robots.txt — Sitemap: https://luxiflow.io/sitemap.xml line added

## Final State of Dependencies

Kept (production): @emailjs/browser, @radix-ui/react-slot, @radix-ui/react-tooltip, 
framer-motion, lucide-react, react, react-dom, react-helmet-async, react-router-dom, 
sonner, clsx, tailwind-merge.

Kept (dev): @eslint/js, @testing-library/jest-dom, @testing-library/react, @types/node, 
@types/react, @types/react-dom, @vitejs/plugin-react-swc, autoprefixer, eslint, 
eslint-plugin-react-hooks, eslint-plugin-react-refresh, globals, jsdom, postcss, 
tailwindcss, tailwindcss-animate, typescript, typescript-eslint, vite, vitest.

## Verified
- tsc --noEmit -p tsconfig.app.json → zero errors
- vite build → success, 5.14s
- vite preview → home loads, sitemap.xml + robots.txt served, all meta tags in HTML
