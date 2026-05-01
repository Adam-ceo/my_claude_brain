# Wiki Log

Append-only chronological record of all wiki operations. Never edit existing entries.

Entry format: `## [YYYY-MM-DD] TYPE | Title`
Types: `ingest`, `query`, `lint`, `edit`, `note`

---

## [2026-04-30] note | Wiki initialized

Initialized wiki structure from the LLM Wiki idea file. Created CLAUDE.md schema (v1.0), index.md, log.md, overview.md stub, and folder structure (wiki/entities/, wiki/concepts/, wiki/sources/, raw/assets/). No sources ingested yet. Ready for first ingest.

## [2026-04-30] ingest | Luxiflow Website QA Session

First source ingested. Raw file: `raw/luxiflow-qa-session-2026-04-30.md`. Created 1 source page, 1 entity page (Luxiflow), 5 concept pages (react-qa-protocol, feature-flag-pattern, css-progressive-enhancement, aria-accessibility-patterns, tsconfig-types-pitfall). Updated overview.md and index.md. Total wiki pages: 8.

Key patterns extracted: SHOW_X = false feature flag, @supports color-mix() fallback, role="list" for Safari VoiceOver, role="menu" keyboard navigation, tsconfig types restriction bug with @types/react.

Open questions flagged: Is color-mix() fallback still necessary given 2022+ browser support? Is Supabase planned for a future feature or dead code? What is the real conversion rate baseline for one-person agency sites?

## [2026-04-30] ingest | ai-crafted-premium-sites QA Session

Third QA pass on Luxiflow — different ZIP download (ai-crafted-premium-sites-main). Raw file: `raw/ai-crafted-qa-session-2026-04-30.md`. Created 1 source page, 1 new concept page (dead-source-file-typescript). Updated 6 existing concept pages (react-qa-protocol, spam-protection-honeypot-timetrap, css-progressive-enhancement, aria-accessibility-patterns, typescript-strict-mode, react-qa-protocol), 1 entity page (luxiflow), overview.md, index.md. Total wiki pages: 15.

New patterns: archived components with broken imports fail tsc even when never imported; useRef > useState for spam protection timestamps; useInView return type bug confirmed recurring in Lovable.dev scaffolds; JSON-LD aggregateRating removed when testimonials are fake; skip link must be in index.html not React component.

Open questions flagged: Luxiflow pricing dropped from €1,500+ to €299 — reprice or different variant? Nav dropdown role="menu" keyboard nav still unimplemented. Should archived components go to _archived/ outside src/?

## [2026-04-30] ingest | Luxiflow — Lovable.dev Fix Session

Second source ingested from Lovable.dev ZIP. Raw file: `raw/luxiflow-lovable-fixes-2026-04-30.md`. Created 1 source page, 4 concept pages (shadcn-dependency-bloat, typescript-strict-mode, spam-protection-honeypot-timetrap, react-seo-helmet). Updated luxiflow entity, overview.md, index.md. Total wiki pages: 13.

Key patterns extracted: shadcn install-everything bloat (41 packages removed, CSS -57% gz), TypeScript full strict mode (7 extra flags beyond basic tsc), honeypot + time-trap spam protection, react-helmet-async SEO component pattern.

Contradiction noted with Session 1: Session 1 preserved Testimonials behind a feature flag; Session 2 deleted the component. Both defensible — trade-off documented in feature-flag-pattern.

Open questions: Dynamic sitemap generation as blog grows. Time-trap 3s threshold accessibility concern. react-helmet-async vs SSR-native solutions in 2026.
