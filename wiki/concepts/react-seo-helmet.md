---
title: "React SPA SEO — react-helmet-async Pattern"
type: concept
tags: [seo, react, helmet, meta, opengraph]
created: 2026-04-30
updated: 2026-04-30
sources: 1
---

## Definition

The standard approach for per-route SEO in a React single-page application: a reusable `<SEO>` component built on `react-helmet-async` that injects `<head>` tags (title, description, canonical, Open Graph, Twitter cards) for each page independently.

## How It Works

### Setup

```bash
npm install react-helmet-async
```

Wrap the app in `<HelmetProvider>` in `main.tsx`:

```tsx
import { HelmetProvider } from "react-helmet-async";

root.render(
  <HelmetProvider>
    <App />
  </HelmetProvider>
);
```

### The reusable SEO component

```tsx
// src/components/SEO.tsx
import { Helmet } from "react-helmet-async";

interface SEOProps {
  title: string;
  description: string;
  canonical: string;
  publishedTime?: string;   // ISO date — for blog posts
  type?: "website" | "article";
  noIndex?: boolean;        // for legal pages, 404
}

const SITE = "https://yourdomain.io";
const DEFAULT_OG = `${SITE}/og-image.jpg`;

export default function SEO({ title, description, canonical, publishedTime, type = "website", noIndex = false }: SEOProps) {
  const fullCanonical = canonical.startsWith("http") ? canonical : `${SITE}${canonical}`;
  return (
    <Helmet>
      <title>{title}</title>
      <meta name="description" content={description} />
      <link rel="canonical" href={fullCanonical} />
      {noIndex && <meta name="robots" content="noindex, follow" />}
      <meta property="og:type" content={type} />
      <meta property="og:url" content={fullCanonical} />
      <meta property="og:title" content={title} />
      <meta property="og:description" content={description} />
      <meta property="og:image" content={DEFAULT_OG} />
      {publishedTime && <meta property="article:published_time" content={publishedTime} />}
      <meta name="twitter:card" content="summary_large_image" />
      <meta name="twitter:title" content={title} />
      <meta name="twitter:description" content={description} />
      <meta name="twitter:image" content={DEFAULT_OG} />
    </Helmet>
  );
}
```

### Usage per page

```tsx
// Home page
<SEO title="Luxiflow — Premium Websites in 14 Days" description="..." canonical="/" />

// Blog post
<SEO
  title={post.title}
  description={post.excerpt}
  canonical={`/blog/${post.slug}`}
  type="article"
  publishedTime={post.isoDate}
/>

// 404
<SEO title="Page Not Found" description="..." canonical="/404" noIndex />
```

### Blog posts specifically need

- `og:type` = `"article"` (not `"website"`) — LinkedIn and Twitter use this to display richer cards
- `article:published_time` in ISO 8601 format (`2026-04-30T00:00:00Z`) — shown in search results and social previews
- Per-post canonical URL to prevent duplicate content issues

### sitemap.xml + robots.txt

For a small static site, a handwritten `public/sitemap.xml` is acceptable:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url><loc>https://yourdomain.io/</loc><priority>1.0</priority></url>
  <url><loc>https://yourdomain.io/blog</loc></url>
  <!-- ... -->
</urlset>
```

Add to `public/robots.txt`:
```
Sitemap: https://yourdomain.io/sitemap.xml
```

**Limitation:** Hardcoded sitemap requires manual updates as content grows. For blogs with frequent posts, a Vite plugin or build-time script that generates sitemap.xml from `blog-posts.ts` is a better long-term solution.

## Evidence & Examples

Implemented in [[luxiflow]] in [[luxiflow-lovable-fixes-2026-04-30]]. 8 routes now have unique meta tags. `og-image.jpg` placed in `/public`.

## Connections

[[react-qa-protocol]] (Phase 3: SEO hardening) · [[luxiflow]]

## Contradictions & Debates

react-helmet-async adds ~8 KB gz to the bundle. For a marketing site where pages are not dynamically rendered (no SSR), the meta tags are set after JS loads — meaning crawlers that don't execute JS (rare, but some) won't see them. For fully correct SEO, SSR (Next.js) or pre-rendering (vite-plugin-ssg) is more reliable than react-helmet-async.

## Open Questions

- Should the sitemap be auto-generated at build time from `blog-posts.ts`? This would avoid manual updates as the blog grows.
- Is react-helmet-async still the right choice in 2026, or have alternatives (e.g. `@tanstack/router`'s built-in head management) become more ergonomic for React SPAs?
