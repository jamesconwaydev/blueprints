# Astro 7 Starter Theme — Build Blueprint

Here is my second copy-paste blueprint for a production-ready Astro starter theme. This time, it has been verified against **Astro v7.0.2** (Vite 8, Rust compiler, Sätteri Markdown pipeline) and **Tailwind CSS v4.3** (Vite plugin, CSS-first config) as of June 24, 2026.

> My first blueprint for creating an [Astro 6 starter theme](https://github.com/jamesconwaydev/blueprints/blob/main/astro-6-starter-theme.md) covered what most tutorials get wrong. In this second blueprint, I've taken things to the next level by adding `JSON-LD` to complete the full SEO package, a `custom 404 page` for users who land on the wrong URL, and a modern, `minimal corporate design` instead of standard, out-of-the-box Tailwind CSS elements.
---

## 1. The Stack

| Layer | Choice | Notes |
|---|---|---|
| Framework | Astro 7.x | Vite 8, Rust compiler (only compiler) |
| Styling | Tailwind CSS 4.x | Via `@tailwindcss/vite`, CSS-first `@theme` config |
| Content | MDX | `@astrojs/mdx` + Content Layer `glob()` loader |
| SEO | Custom `<Seo />` | Canonical URL + Open Graph + Twitter cards + JSON-LD |
| Sitemap | `@astrojs/sitemap` | Auto-generates `sitemap-index.xml` |
| RSS | `@astrojs/rss` | Endpoint at `/rss.xml` from the blog collection |
| Typography | `@tailwindcss/typography` | Tailwind CSS typography plugin |
| Runtime | Node 22.12.0+ | Hard requirement; Astro 6+ dropped Node 18/20 |

---

## 2. Project Structure

```
corporate-starter/
├── astro.config.mjs
├── package.json
├── tsconfig.json
├── src/
│   ├── content.config.ts          # Content collection schema (NOT src/content/config.ts)
│   ├── styles/
│   │   └── global.css             # @import "tailwindcss" + @theme tokens
│   ├── components/
│   │   ├── seo.astro              # Canonical + OG + Twitter meta + JSON-LD
│   │   ├── navigation.astro       # Responsive nav + logo
│   │   ├── footer.astro           # Corporate footer
│   │   ├── button.astro
│   │   └── container.astro
│   ├── layouts/
│   │   ├── BaseLayout.astro       # <html> shell, pulls in SEO, Nav, Footer
│   │   └── PostLayout.astro       # Blog post wrapper
│   ├── content/
│   │   └── blog/
│   │       ├── building-trust-at-scale.mdx
│   │       ├── why-performance-is-strategy.mdx
│   │       └── the-quiet-power-of-clarity.mdx
│   └── pages/
│       ├── index.astro
│       ├── about.astro
│       ├── contact.astro
│       ├── 404.astro              # Custom not-found page
│       ├── rss.xml.js             # RSS feed
│       └── blog/
│           ├── index.astro        # Blog listing
│           └── [...slug].astro    # Dynamic blog post route
└── public/
    ├── favicon.svg
    └── og-default.png             # Fallback Open Graph image (1200×630)
```

---

## 3. Setup Commands

```bash
# 1. Scaffold an empty Astro 7 project (Node 22.12+ must be active)
npm create astro@latest corporate-starter -- \
  --template minimal --typescript strict --git --yes

cd corporate-starter

# 2. Add integrations via the CLI
npx astro add mdx sitemap

# 3. Add Tailwind 4 — the CLI installs the VITE PLUGIN, not the old integration
npx astro add tailwind

# 4. Add the remaining packages
npm install @astrojs/rss @tailwindcss/typography

# 5. Do a quick run test
npm run dev          # http://localhost:4321
```

> `npx astro add tailwind` on Astro 7 installs `tailwindcss` + `@tailwindcss/vite` and registers the plugin under `vite.plugins`. If you see any tutorial telling you to `npm install @astrojs/tailwind`, stop — that package only targets Tailwind 3.

---

## 4. `astro.config.mjs`

The `site` field is **mandatory** — canonical URLs, the sitemap, and RSS links all derive from it.

```js
// @ts-check
import { defineConfig } from 'astro/config';
import mdx from '@astrojs/mdx';
import sitemap from '@astrojs/sitemap';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  // REQUIRED for canonical URLs, sitemap, and RSS. Use your real domain.
  site: 'https://www.example.com',

  // Pick ONE trailing-slash policy and keep it consistent everywhere
  trailingSlash: 'never',

  integrations: [
    mdx(),
    sitemap(),
  ],

  vite: {
    // Tailwind 4 lives HERE, as a Vite plugin — not in `integrations`
    plugins: [tailwindcss()],
  },
});
```

---

## 5. `src/styles/global.css` — Fresh Corporate Design Tokens

This is what stops the theme from looking like default Tailwind elements. A custom palette, type scale, and radii is defined in `@theme`. This example palette creates a more modern corporate feel rather than the stock blue/indigo everyone knows.

```css
/* src/styles/global.css */
@import "tailwindcss";
@plugin "@tailwindcss/typography";

@theme {
  /* --- Brand palette (oklch for perceptual consistency) --- */
  --color-ink-950: oklch(0.18 0.02 250);   /* near-black headings */
  --color-ink-900: oklch(0.25 0.02 250);
  --color-ink-700: oklch(0.42 0.02 250);   /* body text */
  --color-ink-400: oklch(0.62 0.02 250);   /* muted text */

  --color-brand-50:  oklch(0.97 0.02 195);
  --color-brand-100: oklch(0.93 0.04 195);
  --color-brand-400: oklch(0.74 0.10 195);
  --color-brand-500: oklch(0.66 0.12 195); /* primary accent */
  --color-brand-600: oklch(0.58 0.12 195); /* hover */
  --color-brand-700: oklch(0.48 0.11 195);

  --color-signal-500: oklch(0.72 0.16 55); /* warm CTA / highlights */

  --color-surface:     oklch(0.99 0.004 250);
  --color-surface-alt: oklch(0.96 0.006 250); /* section banding */
  --color-line:        oklch(0.90 0.008 250); /* hairline borders */

  /* --- Typography --- */
  --font-sans: "Inter Variable", "Inter", ui-sans-serif, system-ui, sans-serif;
  --font-display: "Inter Variable", "Inter", ui-sans-serif, system-ui, sans-serif;

  /* Tighter, more editorial scale than the defaults */
  --text-display: 3.5rem;
  --text-display--line-height: 1.05;
  --text-display--letter-spacing: -0.02em;

  /* --- Shape & depth --- */
  --radius-xl: 1rem;
  --radius-2xl: 1.5rem;
  --shadow-card: 0 1px 2px rgb(15 23 42 / 0.04), 0 8px 24px -8px rgb(15 23 42 / 0.10);
}

/* Base layer: set defaults so the whole site inherits the corporate feel */
@layer base {
  html {
    scroll-behavior: smooth;
    -webkit-font-smoothing: antialiased;
  }
  body {
    background-color: var(--color-surface);
    color: var(--color-ink-700);
    font-family: var(--font-sans);
    font-feature-settings: "cv02", "cv03", "cv04", "cv11";
  }
  h1, h2, h3, h4 {
    color: var(--color-ink-950);
    font-family: var(--font-display);
    letter-spacing: -0.02em;
    text-wrap: balance;
  }
  ::selection {
    background-color: var(--color-brand-100);
    color: var(--color-ink-950);
  }
}
```

> The custom token names (`ink-*`, `brand-*`, `signal-*`, `surface`, `line`) generate utilities like `text-ink-700`, `bg-brand-500`, `border-line` automatically. Use *those* everywhere instead of `gray-*`/`blue-*` — that's what makes it not look like stock Tailwind.

---

## 6. `src/content.config.ts` — Blog Collection

Content Layer API: file lives at `src/content.config.ts` (project root of `src/`), uses the `glob()` loader, and a Zod schema.

```ts
// src/content.config.ts
import { defineCollection, z } from 'astro:content';
import { glob } from 'astro/loaders';

const blog = defineCollection({
  loader: glob({ pattern: '**/*.{md,mdx}', base: './src/content/blog' }),
  schema: ({ image }) =>
    z.object({
      title: z.string(),
      description: z.string().max(160),   // keeps meta descriptions SEO-sane
      pubDate: z.coerce.date(),
      updatedDate: z.coerce.date().optional(),
      author: z.string().default('The Team'),
      heroImage: image().optional(),       // image() gives optimized assets
      ogImage: z.string().optional(),       // path under /public for social
      tags: z.array(z.string()).default([]),
      draft: z.boolean().default(false),
    }),
});

export const collections = { blog };
```

Run `npx astro sync` after any schema change to regenerate types.

---

## 7. `src/components/Seo.astro` — Canonical + Open Graph + JSON-LD

Emits canonical/OG/Twitter tags **and** a schema.org `@graph` with an `Organization`, a `WebSite`, and a per-page node (`WebPage` for normal pages, `BlogPosting` for articles). Callers can append extra nodes (e.g. a `BreadcrumbList`) via the optional `jsonLd` prop. The graph uses `@id` references so the `Organization` is defined once and reused as the article publisher.

```astro
---
// src/components/Seo.astro
interface Props {
  title: string;
  description: string;
  image?: string;          // path or absolute URL
  type?: 'website' | 'article';
  publishedTime?: Date;
  modifiedTime?: Date;
  author?: string;
  /** Extra schema.org node(s) appended to the @graph (e.g. BreadcrumbList) */
  jsonLd?: Record<string, unknown> | Record<string, unknown>[];
}
const {
  title,
  description,
  image = '/og-default.png',
  type = 'website',
  publishedTime,
  modifiedTime,
  author,
  jsonLd,
} = Astro.props;

const siteName = 'Northbeam'; // your brand name

// Canonical: resolve the current path against the configured `site`
const canonical = new URL(Astro.url.pathname, Astro.site).href;
// Absolute OG image URL (social platforms reject relative paths)
const ogImage = new URL(image, Astro.site).href;
const siteUrl = new URL('/', Astro.site).href;

// --- JSON-LD graph nodes ---
const organization = {
  '@type': 'Organization',
  '@id': `${siteUrl}#organization`,
  name: siteName,
  url: siteUrl,
  logo: { '@type': 'ImageObject', url: new URL('/favicon.svg', Astro.site).href },
};

const website = {
  '@type': 'WebSite',
  '@id': `${siteUrl}#website`,
  name: siteName,
  url: siteUrl,
  publisher: { '@id': `${siteUrl}#organization` },
};

const pageNode =
  type === 'article'
    ? {
        '@type': 'BlogPosting',
        headline: title,
        description,
        image: ogImage,
        url: canonical,
        mainEntityOfPage: canonical,
        datePublished: publishedTime?.toISOString(),
        dateModified: (modifiedTime ?? publishedTime)?.toISOString(),
        author: { '@type': 'Person', name: author ?? siteName },
        publisher: { '@id': `${siteUrl}#organization` },
      }
    : {
        '@type': 'WebPage',
        url: canonical,
        name: title,
        description,
        isPartOf: { '@id': `${siteUrl}#website` },
      };

const extra = jsonLd ? (Array.isArray(jsonLd) ? jsonLd : [jsonLd]) : [];
const structuredData = {
  '@context': 'https://schema.org',
  '@graph': [organization, website, pageNode, ...extra],
};
---
<title>{title}</title>
<meta name="description" content={description} />
<link rel="canonical" href={canonical} />

<!-- Open Graph -->
<meta property="og:type" content={type} />
<meta property="og:title" content={title} />
<meta property="og:description" content={description} />
<meta property="og:url" content={canonical} />
<meta property="og:image" content={ogImage} />
<meta property="og:site_name" content={siteName} />
{publishedTime && (
  <meta property="article:published_time" content={publishedTime.toISOString()} />
)}
{modifiedTime && (
  <meta property="article:modified_time" content={modifiedTime.toISOString()} />
)}

<!-- Twitter -->
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:title" content={title} />
<meta name="twitter:description" content={description} />
<meta name="twitter:image" content={ogImage} />

<!-- JSON-LD structured data.
     MUST use set:html + JSON.stringify — see gotcha #15. -->
<script type="application/ld+json" set:html={JSON.stringify(structuredData)} />
```

**Using the `jsonLd` prop** — e.g. add breadcrumbs from a page or layout:

```astro
<BaseLayout
  title="Why Performance Is Strategy — Northbeam"
  description="..."
  jsonLd={{
    '@type': 'BreadcrumbList',
    itemListElement: [
      { '@type': 'ListItem', position: 1, name: 'Blog', item: new URL('/blog', Astro.site).href },
      { '@type': 'ListItem', position: 2, name: 'Why Performance Is Strategy' },
    ],
  }}>
  ...
</BaseLayout>
```

---

## 8. `src/layouts/BaseLayout.astro`

```astro
---
// src/layouts/BaseLayout.astro
import '../styles/global.css';
import Seo from '../components/Seo.astro';
import Navigation from '../components/Navigation.astro';
import Footer from '../components/Footer.astro';

interface Props {
  title: string;
  description: string;
  image?: string;
  type?: 'website' | 'article';
  publishedTime?: Date;
  modifiedTime?: Date;
  author?: string;
  jsonLd?: Record<string, unknown> | Record<string, unknown>[];
}
const { title, description, image, type, publishedTime, modifiedTime, author, jsonLd } = Astro.props;
---
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <link rel="icon" type="image/svg+xml" href="/favicon.svg" />
    <link rel="sitemap" href="/sitemap-index.xml" />
    <link
      rel="alternate"
      type="application/rss+xml"
      title="Northbeam Blog"
      href={new URL('rss.xml', Astro.site)}
    />
    <Seo {title} {description} {image} {type} {publishedTime} {modifiedTime} {author} {jsonLd} />
  </head>
  <body class="flex min-h-screen flex-col">
    <Navigation />
    <main class="flex-1">
      <slot />
    </main>
    <Footer />
  </body>
</html>
```

---

## 9. `src/components/Navigation.astro` — Full-Screen Responsive Nav + Logo

Sticky header on desktop; a full-screen overlay menu on mobile. The overlay is rendered as a **sibling of the header, not a child** — this is deliberate (see gotcha #14: a `backdrop-filter`/`filter`/`transform` ancestor breaks `position: fixed`, which is what makes a nested overlay look transparent). The overlay carries a solid `bg-surface` background and sits at `z-[60]`, above the `z-50` header. The tiny inline script is vanilla JS (no framework island needed) and is the only client-side code in the theme.

```astro
---
// src/components/Navigation.astro
const links = [
  { href: '/', label: 'Home' },
  { href: '/about', label: 'About' },
  { href: '/blog', label: 'Blog' },
  { href: '/contact', label: 'Contact' },
];
const path = Astro.url.pathname;
const isActive = (href: string) =>
  href === '/' ? path === '/' : path.startsWith(href);
---
<header class="sticky top-0 z-50 border-b border-line bg-surface/80 backdrop-blur-md">
  <nav class="mx-auto flex h-16 max-w-6xl items-center justify-between px-6">
    <!-- Logo -->
    <a href="/" class="flex items-center gap-2 font-display text-lg font-semibold text-ink-950">
      <span class="grid h-8 w-8 place-items-center rounded-lg bg-brand-500 text-surface">N</span>
      Northbeam
    </a>

    <!-- Desktop links -->
    <ul class="hidden items-center gap-8 md:flex">
      {links.map((l) => (
        <li>
          <a
            href={l.href}
            class:list={[
              'text-sm font-medium transition-colors hover:text-brand-600',
              isActive(l.href) ? 'text-brand-600' : 'text-ink-700',
            ]}
          >
            {l.label}
          </a>
        </li>
      ))}
      <li>
        <a href="/contact"
           class="rounded-lg bg-ink-950 px-4 py-2 text-sm font-semibold text-surface transition-colors hover:bg-ink-900">
          Get in touch
        </a>
      </li>
    </ul>

    <!-- Mobile hamburger -->
    <button id="navToggle" aria-label="Open menu" aria-expanded="false"
            class="md:hidden text-ink-950">
      <svg width="24" height="24" fill="none" stroke="currentColor" stroke-width="2">
        <path d="M3 6h18M3 12h18M3 18h18" stroke-linecap="round" />
      </svg>
    </button>
  </nav>
</header>

<!--
  Full-screen mobile overlay.
  IMPORTANT: this lives OUTSIDE <header> on purpose. The header uses
  `backdrop-blur-md` (backdrop-filter), and any ancestor with a filter,
  backdrop-filter, or transform becomes the containing block for
  `position: fixed` descendants. If the overlay were nested inside the
  header, `fixed inset-0` would clip to the 4rem header instead of the
  viewport, leaving the page visible behind it (the "transparent" bug).
  As a top-level sibling, it fills the whole screen with a solid background.
-->
<div id="navOverlay"
     class="fixed inset-0 z-[60] hidden flex-col bg-surface md:hidden">
  <div class="flex h-16 items-center justify-between border-b border-line px-6">
    <span class="font-display text-lg font-semibold text-ink-950">Menu</span>
    <button id="navClose" aria-label="Close menu" class="text-ink-950">
      <svg width="24" height="24" fill="none" stroke="currentColor" stroke-width="2">
        <path d="M6 6l12 12M18 6L6 18" stroke-linecap="round" />
      </svg>
    </button>
  </div>
  <ul class="flex flex-1 flex-col justify-center gap-2 px-6">
    {links.map((l) => (
      <li>
        <a href={l.href}
           class="block py-4 font-display text-3xl font-semibold text-ink-950 hover:text-brand-600">
          {l.label}
        </a>
      </li>
    ))}
  </ul>
  <div class="px-6 pb-10">
    <a href="/contact"
       class="block rounded-xl bg-ink-950 py-4 text-center text-base font-semibold text-surface">
      Get in touch
    </a>
  </div>
</div>

<script>
  const toggle = document.getElementById('navToggle');
  const overlay = document.getElementById('navOverlay');
  const close = document.getElementById('navClose');
  const open = () => {
    overlay?.classList.remove('hidden');
    overlay?.classList.add('flex');
    document.body.style.overflow = 'hidden';
    toggle?.setAttribute('aria-expanded', 'true');
  };
  const shut = () => {
    overlay?.classList.add('hidden');
    overlay?.classList.remove('flex');
    document.body.style.overflow = '';
    toggle?.setAttribute('aria-expanded', 'false');
  };
  toggle?.addEventListener('click', open);
  close?.addEventListener('click', shut);
  overlay?.querySelectorAll('a').forEach((a) => a.addEventListener('click', shut));
</script>
```

---

## 10. `src/components/Footer.astro` — Corporate Footer

```astro
---
// src/components/Footer.astro
const year = new Date().getFullYear();
const cols = [
  { title: 'Company', links: [['About', '/about'], ['Blog', '/blog'], ['Contact', '/contact']] },
  { title: 'Resources', links: [['RSS Feed', '/rss.xml'], ['Sitemap', '/sitemap-index.xml']] },
  { title: 'Legal', links: [['Privacy', '/privacy'], ['Terms', '/terms']] },
];
---
<footer class="border-t border-line bg-surface-alt">
  <div class="mx-auto max-w-6xl px-6 py-16">
    <div class="grid gap-12 md:grid-cols-[1.5fr_repeat(3,1fr)]">
      <div>
        <a href="/" class="flex items-center gap-2 font-display text-lg font-semibold text-ink-950">
          <span class="grid h-8 w-8 place-items-center rounded-lg bg-brand-500 text-surface">N</span>
          Northbeam
        </a>
        <p class="mt-4 max-w-xs text-sm text-ink-400">
          Building dependable digital foundations for ambitious teams.
        </p>
      </div>
      {cols.map((col) => (
        <div>
          <h3 class="text-sm font-semibold text-ink-950">{col.title}</h3>
          <ul class="mt-4 space-y-3">
            {col.links.map(([label, href]) => (
              <li>
                <a href={href} class="text-sm text-ink-700 transition-colors hover:text-brand-600">
                  {label}
                </a>
              </li>
            ))}
          </ul>
        </div>
      ))}
    </div>
    <div class="mt-12 flex flex-col items-center justify-between gap-4 border-t border-line pt-8 text-sm text-ink-400 sm:flex-row">
      <p>© {year} Northbeam, Inc. All rights reserved.</p>
      <p>Built with Astro &amp; Tailwind CSS.</p>
    </div>
  </div>
</footer>
```

---

## 11. Sample Pages

### `src/pages/index.astro`

```astro
---
import BaseLayout from '../layouts/BaseLayout.astro';
---
<BaseLayout
  title="Northbeam — Dependable digital foundations"
  description="We help ambitious teams ship fast, reliable, and beautiful products.">

  <!-- Hero -->
  <section class="mx-auto max-w-6xl px-6 py-24 md:py-32">
    <p class="text-sm font-semibold uppercase tracking-widest text-brand-600">Strategy · Design · Engineering</p>
    <h1 class="mt-4 max-w-3xl text-5xl font-bold leading-[1.05] tracking-tight md:text-6xl">
      Dependable digital foundations for ambitious teams.
    </h1>
    <p class="mt-6 max-w-xl text-lg text-ink-700">
      We partner with companies to build products that are fast, accessible, and
      built to last — no rewrites, no surprises.
    </p>
    <div class="mt-10 flex flex-wrap gap-4">
      <a href="/contact" class="rounded-lg bg-ink-950 px-6 py-3 font-semibold text-surface hover:bg-ink-900">Start a project</a>
      <a href="/about" class="rounded-lg border border-line px-6 py-3 font-semibold text-ink-950 hover:border-ink-400">Learn more</a>
    </div>
  </section>

  <!-- Feature band -->
  <section class="border-y border-line bg-surface-alt">
    <div class="mx-auto grid max-w-6xl gap-8 px-6 py-20 md:grid-cols-3">
      {[
        ['Performance first', 'Sub-second loads as a baseline, not an afterthought.'],
        ['Accessible by default', 'WCAG-aligned interfaces every user can rely on.'],
        ['Built to scale', 'Architecture that grows with you, not against you.'],
      ].map(([t, d]) => (
        <div class="rounded-2xl border border-line bg-surface p-8 shadow-[var(--shadow-card)]">
          <h3 class="text-lg font-semibold">{t}</h3>
          <p class="mt-2 text-ink-700">{d}</p>
        </div>
      ))}
    </div>
  </section>
</BaseLayout>
```

### `src/pages/about.astro`

```astro
---
import BaseLayout from '../layouts/BaseLayout.astro';
---
<BaseLayout
  title="About — Northbeam"
  description="Who we are, how we work, and why clients trust us with their most important products.">
  <section class="mx-auto max-w-3xl px-6 py-24">
    <h1 class="text-4xl font-bold md:text-5xl">About Northbeam</h1>
    <p class="mt-6 text-lg text-ink-700">
      Northbeam is a small, senior team that has spent the last decade shipping
      software for startups and enterprises alike. We believe the best digital
      products feel inevitable — clear, quick, and quietly excellent.
    </p>
    <h2 class="mt-12 text-2xl font-semibold">How we work</h2>
    <p class="mt-4 text-ink-700">
      Every engagement starts with understanding the problem, not the solution.
      We prototype early, measure constantly, and treat performance and
      accessibility as features rather than chores.
    </p>
  </section>
</BaseLayout>
```

### `src/pages/contact.astro`

```astro
---
import BaseLayout from '../layouts/BaseLayout.astro';
---
<BaseLayout
  title="Contact — Northbeam"
  description="Tell us about your project. We typically reply within one business day.">
  <section class="mx-auto max-w-xl px-6 py-24">
    <h1 class="text-4xl font-bold md:text-5xl">Get in touch</h1>
    <p class="mt-4 text-ink-700">
      Tell us a little about what you're building. We usually respond within one business day.
    </p>

    <!-- Static form: wire `action` to Formspree / Netlify Forms / your endpoint -->
    <form action="https://formspree.io/f/your-id" method="POST" class="mt-10 space-y-6">
      <div>
        <label for="name" class="block text-sm font-medium text-ink-950">Name</label>
        <input id="name" name="name" type="text" required
               class="mt-2 w-full rounded-lg border border-line bg-surface px-4 py-3 outline-none focus:border-brand-500 focus:ring-2 focus:ring-brand-100" />
      </div>
      <div>
        <label for="email" class="block text-sm font-medium text-ink-950">Email</label>
        <input id="email" name="email" type="email" required
               class="mt-2 w-full rounded-lg border border-line bg-surface px-4 py-3 outline-none focus:border-brand-500 focus:ring-2 focus:ring-brand-100" />
      </div>
      <div>
        <label for="message" class="block text-sm font-medium text-ink-950">Message</label>
        <textarea id="message" name="message" rows="5" required
                  class="mt-2 w-full rounded-lg border border-line bg-surface px-4 py-3 outline-none focus:border-brand-500 focus:ring-2 focus:ring-brand-100"></textarea>
      </div>
      <button type="submit"
              class="rounded-lg bg-ink-950 px-6 py-3 font-semibold text-surface hover:bg-ink-900">
        Send message
      </button>
    </form>
  </section>
</BaseLayout>
```

### `src/pages/404.astro` — Custom Not-Found Page

Astro builds this to `404.html`. Static hosts (Netlify, Vercel, Cloudflare Pages, GitHub Pages) serve it automatically for unmatched routes, and `astro dev` / `astro preview` serve it locally too. It reuses `BaseLayout`, so it gets the same nav, footer, and SEO scaffolding as every other page.

```astro
---
import BaseLayout from '../layouts/BaseLayout.astro';
---
<BaseLayout
  title="Page not found — Northbeam"
  description="Sorry, we couldn't find the page you were looking for.">
  <section class="mx-auto flex max-w-xl flex-col items-center px-6 py-32 text-center">
    <p class="font-display text-7xl font-bold tracking-tight text-brand-500">404</p>
    <h1 class="mt-6 text-3xl font-bold md:text-4xl">This page wandered off.</h1>
    <p class="mt-4 text-ink-700">
      The page you're looking for doesn't exist or may have moved.
      Let's get you back on track.
    </p>
    <div class="mt-10 flex flex-wrap justify-center gap-4">
      <a href="/" class="rounded-lg bg-ink-950 px-6 py-3 font-semibold text-surface hover:bg-ink-900">
        Back to home
      </a>
      <a href="/blog" class="rounded-lg border border-line px-6 py-3 font-semibold text-ink-950 hover:border-ink-400">
        Read the blog
      </a>
    </div>
  </section>
</BaseLayout>
```

> The page returns a real `404` status on the hosts above, so it doesn't need a `noindex` meta. On a plain static file server (or an SSR adapter), confirm the response status is `404` and not `200` — a `200` "soft 404" can get the page indexed. See gotcha #16.

---

## 12. Blog

### `src/layouts/PostLayout.astro`

```astro
---
// src/layouts/PostLayout.astro
import BaseLayout from './BaseLayout.astro';
import type { CollectionEntry } from 'astro:content';

interface Props { post: CollectionEntry<'blog'> }
const { post } = Astro.props;
const { title, description, pubDate, updatedDate, author, ogImage } = post.data;
const dateFmt = (d: Date) =>
  d.toLocaleDateString('en-US', { year: 'numeric', month: 'long', day: 'numeric' });
---
<BaseLayout
  {title}
  {description}
  image={ogImage}
  type="article"
  publishedTime={pubDate}
  modifiedTime={updatedDate}
  {author}
>
  <article class="mx-auto max-w-2xl px-6 py-20">
    <header class="mb-10">
      <h1 class="text-4xl font-bold leading-tight md:text-5xl">{title}</h1>
      <p class="mt-4 text-sm text-ink-400">
        By {author} · {dateFmt(pubDate)}
        {updatedDate && <span> · Updated {dateFmt(updatedDate)}</span>}
      </p>
    </header>
    <!-- prose comes from @tailwindcss/typography -->
    <div class="prose prose-lg max-w-none prose-headings:font-display prose-a:text-brand-600">
      <slot />
    </div>
  </article>
</BaseLayout>
```

### `src/pages/blog/index.astro` — Listing

```astro
---
import BaseLayout from '../../layouts/BaseLayout.astro';
import { getCollection } from 'astro:content';

const posts = (await getCollection('blog', ({ data }) => !data.draft))
  .sort((a, b) => b.data.pubDate.getTime() - a.data.pubDate.getTime());
const dateFmt = (d) => d.toLocaleDateString('en-US', { year: 'numeric', month: 'short', day: 'numeric' });
---
<BaseLayout
  title="Blog — Northbeam"
  description="Notes on engineering, design, and building products that last.">
  <section class="mx-auto max-w-3xl px-6 py-24">
    <h1 class="text-4xl font-bold md:text-5xl">From the blog</h1>
    <p class="mt-4 text-ink-700">Notes on engineering, design, and building products that last.</p>

    <ul class="mt-14 divide-y divide-line">
      {posts.map((post) => (
        <li class="py-8">
          <a href={`/blog/${post.id}`} class="group block">
            <p class="text-sm text-ink-400">{dateFmt(post.data.pubDate)}</p>
            <h2 class="mt-2 text-2xl font-semibold transition-colors group-hover:text-brand-600">
              {post.data.title}
            </h2>
            <p class="mt-2 text-ink-700">{post.data.description}</p>
            <span class="mt-3 inline-block text-sm font-semibold text-brand-600">Read more →</span>
          </a>
        </li>
      ))}
    </ul>
  </section>
</BaseLayout>
```

### `src/pages/blog/[...slug].astro` — Dynamic Post Route

Note `render()` is imported from `astro:content` (the Content Layer API form) — `post.render()` no longer exists.

```astro
---
import { getCollection, render } from 'astro:content';
import PostLayout from '../../layouts/PostLayout.astro';

export async function getStaticPaths() {
  const posts = await getCollection('blog', ({ data }) => !data.draft);
  return posts.map((post) => ({
    params: { slug: post.id },
    props: { post },
  }));
}

const { post } = Astro.props;
const { Content } = await render(post);
---
<PostLayout {post}>
  <Content />
</PostLayout>
```

### Sample Posts

`src/content/blog/building-trust-at-scale.mdx`

```mdx
---
title: "Building Trust at Scale"
description: "Why reliability is the feature your customers notice most, and how to engineer for it from day one."
pubDate: 2026-05-12
author: "Mara Holloway"
tags: ["engineering", "reliability"]
---

Trust is not a marketing exercise. It is the cumulative result of a thousand
small moments where your product did exactly what the user expected.

## Reliability compounds

Every outage costs more than the minutes it lasts. It costs the next decision a
customer makes about whether to depend on you again.

> The most expensive bug is the one that quietly erodes confidence.

When you treat uptime, correctness, and graceful failure as first-class
features, you are investing in the one asset that is hardest to rebuild.
```

`src/content/blog/why-performance-is-strategy.mdx`

```mdx
---
title: "Why Performance Is Strategy"
description: "Speed is not a technical detail. It is a business advantage that shapes how people perceive your entire brand."
pubDate: 2026-05-28
author: "Devin Park"
tags: ["performance", "strategy"]
---

A faster site is not just nicer to use. It is measurably more profitable,
more accessible, and more resilient.

## The first 100 milliseconds

Users form an impression of your product before they read a single word.
That impression is set by how quickly the page responds.

Astro's zero-JS-by-default model means most of this work is already done for
you — you ship HTML and CSS, and add interactivity only where it earns its place.
```

`src/content/blog/the-quiet-power-of-clarity.mdx`

```mdx
---
title: "The Quiet Power of Clarity"
description: "Great interfaces disappear. A short case for designing with restraint rather than decoration."
pubDate: 2026-06-09
author: "Mara Holloway"
tags: ["design", "ux"]
---

The best compliment an interface can receive is that no one noticed it.

## Less to decide

Every element you add is a decision you ask the user to make. Clarity is the
discipline of removing decisions that do not serve them.

Restraint is not the absence of design. It is design confident enough to
get out of the way.
```

---

## 13. `src/pages/rss.xml.js` — RSS Feed

Pulls from the blog collection. `link` uses `post.id` and matches the `trailingSlash: 'never'` policy.

```js
// src/pages/rss.xml.js
import rss from '@astrojs/rss';
import { getCollection } from 'astro:content';

export async function GET(context) {
  const posts = (await getCollection('blog', ({ data }) => !data.draft))
    .sort((a, b) => b.data.pubDate.getTime() - a.data.pubDate.getTime());

  return rss({
    title: 'Northbeam Blog',
    description: 'Notes on engineering, design, and building products that last.',
    site: context.site,
    trailingSlash: false, // match astro.config trailingSlash: 'never'
    items: posts.map((post) => ({
      title: post.data.title,
      description: post.data.description,
      pubDate: post.data.pubDate,
      link: `/blog/${post.id}`,
      categories: post.data.tags,
    })),
    customData: `<language>en-us</language>`,
  });
}
```

Feed will be live at `https://www.example.com/rss.xml`.

---

## 14. XML Sitemap

Already handled by `@astrojs/sitemap` (added in step 2 and registered in config). On `npm run build` it emits `sitemap-index.xml` + `sitemap-0.xml` covering every static route, including blog posts. It is referenced from `<head>` in `BaseLayout.astro`. Add it to `robots.txt` too:

```
# public/robots.txt
User-agent: *
Allow: /
Sitemap: https://www.example.com/sitemap-index.xml
```

---

## 15. Known Gotchas

These are the failure modes that waste the most time on this exact stack.

**1. Do NOT use `@astrojs/tailwind`.** That integration targets Tailwind 3 and will either error or silently misbehave with v4. Tailwind 4 is a Vite plugin: `@tailwindcss/vite` registered under `vite.plugins`, never under `integrations`.

**2. No `tailwind.config.js`.** Tailwind 4 is CSS-first. All theming goes in `@theme { ... }` inside `global.css`. If you scaffold one out of habit, it's ignored (unless you explicitly bridge it with `@config`).

**3. `@apply` inside scoped `<style>` blocks needs a reference.** If you use Tailwind utilities via `@apply` in an Astro component's `<style>` block, add `@reference "tailwindcss";` at the top of that block — otherwise the build can't resolve your tokens. Prefer utility classes in markup to avoid this entirely.

**4. Astro 7's Rust compiler enforces valid HTML nesting.** The old Go compiler silently fixed bad nesting; the Rust compiler does not. A `<div>` (or any block element) inside a `<p>` will break your layout because the browser closes the `<p>` early. Audit components for invalid nesting if output looks wrong after building. This commonly bites `<p>` wrappers around MDX/component content.

**5. Astro 7 changed the default Markdown pipeline (Sätteri).** remark/rehype are no longer bundled by default. Plain Markdown/MDX works out of the box. **If** you need remark or rehype plugins, install `@astrojs/markdown-remark` separately, then your `markdown.remarkPlugins` / `markdown.rehypePlugins` config will work again.

**6. `site` must be set in `astro.config.mjs`.** Canonical URLs, the sitemap, and RSS links all derive from it. Without it, canonical tags resolve to `undefined`, the sitemap integration warns and skips, and RSS links are relative/broken.

**7. Content config path & API.** The file is `src/content.config.ts` (root of `src/`), not the old `src/content/config.ts`. Use the `glob()` loader from `astro/loaders`. `Astro.glob()` was removed — use `getCollection()` instead.

**8. Render API changed.** Use `import { render } from 'astro:content'` and `const { Content } = await render(post)`. The old `post.render()` method is gone. Post identifier is `post.id`, not `post.slug`.

**9. OG images must be absolute URLs.** Social scrapers reject relative paths. Always resolve with `new URL(image, Astro.site).href` (done in `Seo.astro`). Provide a real 1200×630 `public/og-default.png` fallback.

**10. Trailing-slash consistency.** Pick one policy (`trailingSlash: 'never'` here) and mirror it everywhere: canonical URLs, internal links, and `trailingSlash: false` in the `rss()` call. RSS defaults to adding a trailing slash regardless of your config, so set it explicitly or your feed links won't match your real post URLs.

**11. Node 22.12.0+ is required.** Astro 6 dropped Node 18 and 20. Pin it with an `.nvmrc` containing `22` and set the same in your host's build settings (Netlify/Vercel/Cloudflare), or the build fails on deploy even though it worked locally.

**12. `<ClientRouter />`, not `<ViewTransitions />`.** If you add view transitions, the component was renamed and its event timing changed. Don't copy old `<ViewTransitions />` snippets.

**13. Run `npx astro sync` after schema edits.** Type generation for `astro:content` doesn't always refresh on its own; sync regenerates types so your editor and build agree.

**14. A `backdrop-filter` / `filter` / `transform` ancestor breaks `position: fixed` children.** This is what causes a "transparent" mobile menu. The sticky header uses `backdrop-blur-md` (which is `backdrop-filter`), and any element with that — or with `filter` or `transform` — becomes the containing block for its `fixed` descendants. A full-screen overlay nested inside such a header gets clipped to the header's height instead of covering the viewport, so the page shows through and the menu looks transparent. **Fix:** render the overlay as a sibling of the header (top level in the component), not a child, and give it a solid background (`bg-surface`) plus a higher z-index (`z-[60]`) than the header (`z-50`). The Navigation component in §9 is structured this way. The same rule applies to any modal, dropdown, or toast you position with `fixed`.

**15. JSON-LD must use `set:html` + `JSON.stringify`.** Never drop a raw object expression between `<script type="application/ld+json">` tags as text — Astro HTML-escapes the output, turning `"` into `&quot;`, which produces invalid JSON that crawlers silently reject. Always `<script type="application/ld+json" set:html={JSON.stringify(data)} />`. Because the `type` is `application/ld+json` (not a JS module), Astro leaves the script inline and does **not** bundle it, which is exactly what you want. Validate with Google's Rich Results Test or schema.org's validator after building.

**16. A custom 404 must return a `404` status, not `200`.** Astro builds `src/pages/404.astro` to `404.html`. Managed hosts (Netlify, Vercel, Cloudflare Pages, GitHub Pages) serve it with a real `404` status automatically. But a naïve static file server — or a misconfigured SSR adapter — can serve it with `200`, creating a "soft 404" that search engines may index. Verify the response code (`curl -I https://yoursite.com/does-not-exist`) on your actual host. Only add a `noindex` meta if you can't control the status code.
