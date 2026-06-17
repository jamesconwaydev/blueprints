# Astro 6.4.7 Starter Theme — Build Blueprint

Here is my blueprint for creating a solid Astro starter theme. It's styled with Tailwind CSS v4 & daisyUI 5 to help you build faster. It also comes fully SEO-ready with canonical URLs, Open Graph, XML Sitemaps, and RSS. It's also made to publish pages, blogs, case studies using native Astro, Markdown, or MDX.

> **Why the specifics matter:** Astro 6 (released March 2026) changed several things that older tutorials get wrong, taking years off my life. This blueprint targets the current architecture: Node 22+, Tailwind via the `@tailwindcss/vite` plugin (not the deprecated `@astrojs/tailwind` integration), daisyUI as a CSS `@plugin`, and the Content Layer API (`glob()` loader + `render()`). Read the **Known Gotchas** section before you build — it will save you hours.

---

## 1. Stack & pinned versions

| Layer | Package | Target version | Role |
|---|---|---|---|
| Framework | `astro` | `6.4.7` | Core framework (static-first) |
| Styling engine | `tailwindcss` | `^4.3` | Utility CSS |
| Tailwind/Astro glue | `@tailwindcss/vite` | `^4.3` | Official Vite plugin (v4 path) |
| Component library | `daisyui` | `^5.5` | Tailwind plugin, semantic components + themes |
| Prose styling | `@tailwindcss/typography` | `^0.5` | `.prose` styles for Markdown/MDX |
| MDX | `@astrojs/mdx` | latest for Astro 6 | `.mdx` support + components in content |
| Sitemap | `@astrojs/sitemap` | latest for Astro 6 | Auto `sitemap-index.xml` |
| RSS | `@astrojs/rss` | `^4.x` | RSS feed generation + `rssSchema` |

> Versions other than Astro are illustrative — install the latest compatible release with `npm install <pkg>@latest`. The blueprint pins **Astro 6.4.7** as requested; confirm it's still current with `npm view astro version` and adjust if a newer 6.x patch exists.

**Hard requirement:** Astro 6 needs **Node.js 22 or later** (Node 18 and 20 support was dropped). Check with `node -v`.

---

## 2. Scaffold the project

```bash
# 1. Create a minimal, strict-TypeScript Astro project
npm create astro@latest astro-starter-theme -- \
  --template minimal --typescript strict --git --yes

cd astro-starter-theme

# 2. Pin Astro to the target version
npm install astro@6.4.7

# 3. Tailwind v4 + the official Vite plugin
npm install tailwindcss @tailwindcss/vite

# 4. daisyUI + typography (dev deps — they only run at build)
npm install -D daisyui@latest @tailwindcss/typography

# 5. Official Astro integrations
npx astro add mdx sitemap --yes
npm install @astrojs/rss
```

> Do **not** run `npx astro add tailwind` on Astro 6 — see Gotcha #1. Wire Tailwind manually as shown below.

---

## 3. Directory structure

```
astro-starter-theme/
├── astro.config.mjs
├── tsconfig.json
├── package.json
├── public/
│   ├── favicon.svg
│   ├── og-default.png            # fallback Open Graph image (1200×630)
│   └── robots.txt
└── src/
    ├── content.config.ts         # collection definitions (root of src/ in v6)
    ├── content/
    │   ├── blog/
    │   │   ├── hello-world.md
    │   │   └── why-astro.mdx
    │   └── case-studies/
    │       └── acme-redesign.mdx
    ├── styles/
    │   └── global.css            # Tailwind + daisyUI entry
    ├── components/
    │   ├── BaseHead.astro        # SEO: title, canonical, Open Graph, Twitter
    │   ├── Header.astro
    │   ├── Footer.astro
    │   ├── ThemeToggle.astro     # daisyUI theme-controller
    │   └── PostCard.astro
    ├── layouts/
    │   ├── BaseLayout.astro      # html shell, header/footer, slots BaseHead
    │   ├── BlogLayout.astro      # article layout (prose)
    │   └── CaseStudyLayout.astro
    ├── pages/
    │   ├── index.astro           # Home
    │   ├── about.astro           # static Page
    │   ├── contact.astro         # static Page
    │   ├── rss.xml.js            # RSS endpoint  -> /rss.xml
    │   ├── blog/
    │   │   ├── index.astro       # blog listing
    │   │   └── [...slug].astro   # blog post (dynamic)
    │   └── case-studies/
    │       ├── index.astro       # case study listing
    │       └── [...slug].astro   # case study (dynamic)
    └── consts.ts                 # SITE_TITLE, SITE_DESCRIPTION, SITE_URL
```

### 3.1 Create the folders and files

The `minimal` template only ships `src/pages/index.astro`. Create everything else in one shot, then fill each file using the sections that follow.

```bash
# --- directories ---
mkdir -p \
  public \
  src/styles \
  src/components \
  src/layouts \
  src/content/blog \
  src/content/case-studies \
  src/pages/blog \
  src/pages/case-studies

# --- top-level src files ---
touch src/consts.ts src/content.config.ts

# --- styles ---
touch src/styles/global.css

# --- components ---
touch src/components/BaseHead.astro \
      src/components/Header.astro \
      src/components/Footer.astro \
      src/components/ThemeToggle.astro \
      src/components/PostCard.astro \
      src/components/Aside.astro

# --- layouts ---
touch src/layouts/BaseLayout.astro \
      src/layouts/BlogLayout.astro \
      src/layouts/CaseStudyLayout.astro

# --- pages (Home already exists; create the rest) ---
touch src/pages/about.astro \
      src/pages/contact.astro \
      src/pages/rss.xml.js \
      src/pages/blog/index.astro \
      src/pages/blog/'[...slug].astro' \
      src/pages/case-studies/index.astro \
      src/pages/case-studies/'[...slug].astro'

# --- example content ---
touch src/content/blog/hello-world.md \
      src/content/blog/why-astro.mdx \
      src/content/case-studies/acme-redesign.mdx

# --- public files ---
touch public/robots.txt
```

> The square-bracket route files (`[...slug].astro`) are quoted so the shell doesn't treat the brackets as a glob pattern.

**Two binary/asset files can't be created with `touch` meaningfully** — add real assets:

- `public/favicon.svg` — drop in any SVG. Minimal placeholder:
  ```svg
  <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 32 32"><rect width="32" height="32" rx="6" fill="#4f46e5"/><text x="16" y="22" font-size="18" text-anchor="middle" fill="#fff" font-family="sans-serif">A</text></svg>
  ```
- `public/og-default.png` — a **1200×630** PNG used as the social-card fallback. Export one from any design tool, or generate a placeholder (requires ImageMagick): `magick -size 1200x630 xc:#4f46e5 public/og-default.png`.

---

## 4. Core configuration

### `astro.config.mjs`

```js
// @ts-check
import { defineConfig } from 'astro/config';
import mdx from '@astrojs/mdx';
import sitemap from '@astrojs/sitemap';
import tailwindcss from '@tailwindcss/vite';

// https://astro.build/config
export default defineConfig({
  // REQUIRED for canonical URLs, Open Graph, sitemap, and RSS.
  site: 'https://www.example.com',

  integrations: [
    mdx(),
    sitemap({
      // Optional: drop drafts/utility routes from the sitemap
      filter: (page) => !page.includes('/draft/'),
      changefreq: 'weekly',
      lastmod: new Date(),
    }),
  ],

  // Tailwind v4 is a Vite plugin — it lives under `vite.plugins`,
  // NOT in the top-level `integrations` array.
  vite: {
    plugins: [tailwindcss()],
  },
});
```

### `src/consts.ts`

```ts
export const SITE_TITLE = 'Astro Starter Theme';
export const SITE_DESCRIPTION =
  'A fast, SEO-friendly Astro starter with Tailwind, daisyUI, blog, and case studies.';
```

### `src/styles/global.css` — Tailwind v4 + daisyUI (CSS-first config)

In v4 there is **no `tailwind.config.js`**. Everything is configured in CSS.

```css
/* 1. Tailwind core */
@import "tailwindcss";

/* 2. Typography plugin (gives .prose for Markdown/MDX) */
@plugin "@tailwindcss/typography";

/* 3. daisyUI + the themes you want exposed */
@plugin "daisyui" {
  themes: light --default, dark --prefersdark, corporate, business;
}

/* 4. Optional: project design tokens */
@theme {
  --font-sans: "Inter", ui-sans-serif, system-ui, sans-serif;
  --color-brand-500: oklch(0.62 0.18 264);
}

/* 5. Optional: customize a daisyUI theme inline */
@plugin "daisyui/theme" {
  name: "corporate";
  --color-primary: oklch(0.62 0.18 264);
}
```

Import this stylesheet **once**, in `BaseLayout.astro`, and every page inherits it.

---

## 5. SEO: canonical URLs + Open Graph

This is the heart of the "SEO-friendly" requirement. One reusable component handles `<title>`, meta description, **canonical link**, **Open Graph**, and Twitter Card tags.

### `src/components/BaseHead.astro`

```astro
---
import { SITE_TITLE } from '../consts';

interface Props {
  title: string;
  description: string;
  /** Page-specific OG image (absolute path or full URL). Falls back to site default. */
  image?: string;
  type?: 'website' | 'article';
  publishedTime?: string; // ISO date, articles only
  modifiedTime?: string;  // ISO date, articles only
}

const {
  title,
  description,
  image = '/og-default.png',
  type = 'website',
  publishedTime,
  modifiedTime,
} = Astro.props;

// Canonical URL = current path resolved against `site` in astro.config.mjs.
const canonicalURL = new URL(Astro.url.pathname, Astro.site);
// Open Graph requires an ABSOLUTE image URL.
const ogImageURL = new URL(image, Astro.site);
const fullTitle = title === SITE_TITLE ? title : `${title} · ${SITE_TITLE}`;
---

<!-- Global metadata -->
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<link rel="icon" type="image/svg+xml" href="/favicon.svg" />
<meta name="generator" content={Astro.generator} />

<!-- Primary SEO -->
<title>{fullTitle}</title>
<meta name="title" content={fullTitle} />
<meta name="description" content={description} />
<link rel="canonical" href={canonicalURL} />

<!-- Open Graph / Facebook -->
<meta property="og:type" content={type} />
<meta property="og:url" content={canonicalURL} />
<meta property="og:title" content={fullTitle} />
<meta property="og:description" content={description} />
<meta property="og:image" content={ogImageURL} />
<meta property="og:site_name" content={SITE_TITLE} />
{type === 'article' && publishedTime && (
  <meta property="article:published_time" content={publishedTime} />
)}
{type === 'article' && modifiedTime && (
  <meta property="article:modified_time" content={modifiedTime} />
)}

<!-- Twitter -->
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:url" content={canonicalURL} />
<meta name="twitter:title" content={fullTitle} />
<meta name="twitter:description" content={description} />
<meta name="twitter:image" content={ogImageURL} />

<!-- RSS autodiscovery -->
<link rel="alternate" type="application/rss+xml" title={SITE_TITLE} href={new URL('rss.xml', Astro.site)} />
```

**Two things make canonical/OG work correctly:**
1. `site` **must** be set in `astro.config.mjs`. Without it, `Astro.site` is `undefined` and canonical/OG URLs break.
2. OG/Twitter images must be **absolute URLs** — that's why we wrap them in `new URL(..., Astro.site)`.

---

## 6. Layouts

`BaseLayout` imports three chrome components — define them first.

### `src/components/Header.astro` (daisyUI navbar)

```astro
---
import ThemeToggle from './ThemeToggle.astro';
import { SITE_TITLE } from '../consts';

const links = [
  { href: '/', label: 'Home' },
  { href: '/about', label: 'About' },
  { href: '/blog', label: 'Blog' },
  { href: '/case-studies', label: 'Case Studies' },
  { href: '/contact', label: 'Contact' },
];
const path = Astro.url.pathname;
---
<header class="navbar bg-base-200 border-b border-base-300 px-4">
  <div class="navbar-start">
    <a href="/" class="btn btn-ghost text-lg font-bold">{SITE_TITLE}</a>
  </div>
  <nav class="navbar-center hidden md:flex">
    <ul class="menu menu-horizontal gap-1">
      {links.map((l) => (
        <li>
          <a href={l.href} class={path === l.href ? 'active' : ''}>{l.label}</a>
        </li>
      ))}
    </ul>
  </nav>
  <div class="navbar-end">
    <ThemeToggle />
  </div>
</header>
```

### `src/components/ThemeToggle.astro` (daisyUI theme-controller, no JS)

```astro
---
// Uses daisyUI's `theme-controller`: the checkbox value swaps `data-theme`
// against the themes declared in global.css. No JavaScript required.
---
<label class="swap swap-rotate btn btn-ghost btn-circle">
  <input type="checkbox" class="theme-controller" value="dark" />
  <svg class="swap-off h-5 w-5 fill-current" viewBox="0 0 24 24"><path d="M5.64 17l-.71.71a1 1 0 001.41 1.41l.71-.71A1 1 0 005.64 17zM5 12a1 1 0 00-1-1H3a1 1 0 000 2h1a1 1 0 001-1zm7-7a1 1 0 001-1V3a1 1 0 00-2 0v1a1 1 0 001 1zM5.64 7.05a1 1 0 001.41 0 1 1 0 000-1.41l-.71-.71a1 1 0 00-1.41 1.41zM17 5.64a1 1 0 00.71-.29l.71-.71a1 1 0 10-1.41-1.41l-.71.71A1 1 0 0017 5.64zM21 11h-1a1 1 0 000 2h1a1 1 0 000-2zm-9 8a1 1 0 00-1 1v1a1 1 0 002 0v-1a1 1 0 00-1-1zM12 6.5A5.5 5.5 0 1017.5 12 5.51 5.51 0 0012 6.5z"/></svg>
  <svg class="swap-on h-5 w-5 fill-current" viewBox="0 0 24 24"><path d="M21.64 13a1 1 0 00-1.05-.14 8.05 8.05 0 01-3.37.73 8.15 8.15 0 01-8.14-8.1 8.59 8.59 0 01.25-2A1 1 0 008 2.36a10.14 10.14 0 1014 11.69 1 1 0 00-.36-1.05z"/></svg>
</label>
```

### `src/components/Footer.astro`

```astro
---
import { SITE_TITLE } from '../consts';
const year = new Date().getFullYear();
---
<footer class="footer footer-center bg-base-200 text-base-content/70 p-6 border-t border-base-300">
  <aside>
    <p>© {year} {SITE_TITLE}. Built with Astro, Tailwind &amp; daisyUI.</p>
    <a href="/rss.xml" class="link link-hover">RSS feed</a>
  </aside>
</footer>
```

### `src/layouts/BaseLayout.astro`

```astro
---
import BaseHead from '../components/BaseHead.astro';
import Header from '../components/Header.astro';
import Footer from '../components/Footer.astro';
import '../styles/global.css';

interface Props {
  title: string;
  description: string;
  image?: string;
  type?: 'website' | 'article';
  publishedTime?: string;
  modifiedTime?: string;
}
const props = Astro.props;
---
<!doctype html>
<html lang="en" data-theme="light">
  <head>
    <BaseHead {...props} />
  </head>
  <body class="min-h-screen bg-base-100 text-base-content flex flex-col">
    <Header />
    <main class="flex-1 container mx-auto px-4 py-10">
      <slot />
    </main>
    <Footer />
  </body>
</html>
```

### `src/layouts/BlogLayout.astro` (articles get `type="article"` + prose)

```astro
---
import BaseLayout from './BaseLayout.astro';

interface Props {
  title: string;
  description: string;
  pubDate: Date;
  updatedDate?: Date;
  heroImage?: { src: string };
  ogImage?: string;
}
const { title, description, pubDate, updatedDate, heroImage, ogImage } = Astro.props;
---
<BaseLayout
  title={title}
  description={description}
  type="article"
  image={ogImage}
  publishedTime={pubDate.toISOString()}
  modifiedTime={updatedDate?.toISOString()}
>
  <article class="prose lg:prose-xl mx-auto">
    {heroImage && <img src={heroImage.src} alt="" class="rounded-box mb-6" />}
    <h1>{title}</h1>
    <p class="text-base-content/60">
      <time datetime={pubDate.toISOString()}>
        {pubDate.toLocaleDateString('en-US', { year: 'numeric', month: 'long', day: 'numeric' })}
      </time>
    </p>
    <slot />
  </article>
</BaseLayout>
```

### `src/layouts/CaseStudyLayout.astro`

Same pattern as `BlogLayout`, with case-study fields (client, industry, results) surfaced in a daisyUI `stats` block.

```astro
---
import BaseLayout from './BaseLayout.astro';

interface Props {
  title: string;
  description: string;
  client: string;
  industry?: string;
  pubDate: Date;
  heroImage?: { src: string };
  ogImage?: string;
  results?: { label: string; value: string }[];
}
const { title, description, client, industry, pubDate, heroImage, ogImage, results = [] } = Astro.props;
---
<BaseLayout
  title={title}
  description={description}
  type="article"
  image={ogImage}
  publishedTime={pubDate.toISOString()}
>
  <article class="prose lg:prose-xl mx-auto">
    {heroImage && <img src={heroImage.src} alt="" class="rounded-box mb-6" />}
    <h1>{title}</h1>
    <p class="text-base-content/60 not-prose">
      <span class="badge badge-primary">{client}</span>
      {industry && <span class="badge badge-ghost ml-2">{industry}</span>}
    </p>

    {results.length > 0 && (
      <div class="stats stats-vertical sm:stats-horizontal shadow not-prose my-8 w-full">
        {results.map((r) => (
          <div class="stat">
            <div class="stat-value text-primary">{r.value}</div>
            <div class="stat-desc">{r.label}</div>
          </div>
        ))}
      </div>
    )}

    <slot />
  </article>
</BaseLayout>
```

---

## 7. Content collections (Markdown + MDX)

Astro 6 uses the **Content Layer API**. The config file lives at **`src/content.config.ts`** (moved to the `src/` root in v5+).

> **Astro 6 change:** importing `z` from `astro:content` is deprecated. Import it from **`astro/zod`** instead.

### `src/content.config.ts`

```ts
import { defineCollection } from 'astro:content';
import { glob } from 'astro/loaders';
import { z } from 'astro/zod';

const blog = defineCollection({
  // glob() loader pulls both .md and .mdx from one directory
  loader: glob({ pattern: '**/*.{md,mdx}', base: './src/content/blog' }),
  schema: ({ image }) =>
    z.object({
      title: z.string(),
      description: z.string(),
      pubDate: z.coerce.date(),
      updatedDate: z.coerce.date().optional(),
      heroImage: image().optional(),   // optimized via Astro's image pipeline
      ogImage: z.string().optional(),  // absolute/relative path for social cards
      tags: z.array(z.string()).default([]),
      draft: z.boolean().default(false),
    }),
});

const caseStudies = defineCollection({
  loader: glob({ pattern: '**/*.{md,mdx}', base: './src/content/case-studies' }),
  schema: ({ image }) =>
    z.object({
      title: z.string(),
      description: z.string(),
      client: z.string(),
      industry: z.string().optional(),
      pubDate: z.coerce.date(),
      heroImage: image().optional(),
      ogImage: z.string().optional(),
      results: z.array(z.object({ label: z.string(), value: z.string() })).default([]),
      draft: z.boolean().default(false),
    }),
});

export const collections = { blog, caseStudies };
```

### Example content entry — `src/content/blog/why-astro.mdx`

```mdx
---
title: "Why I Chose Astro"
description: "Zero-JS by default, great SEO, and a content-first workflow."
pubDate: 2026-06-10
tags: ["astro", "performance"]
draft: false
---

import Aside from '../../components/Aside.astro';

Astro ships **zero JavaScript by default**, which is why it scores so well on Core Web Vitals.

<Aside type="tip">MDX lets you drop components straight into prose.</Aside>
```

A plain `.md` file uses the same frontmatter and renders identically — MDX just additionally allows component imports/JSX.

### Component used inside MDX — `src/components/Aside.astro`

The `why-astro.mdx` example imports this, so it must exist or the build fails.

```astro
---
interface Props { type?: 'note' | 'tip' | 'warning'; }
const { type = 'note' } = Astro.props;
const styles = {
  note: 'alert-info',
  tip: 'alert-success',
  warning: 'alert-warning',
};
---
<aside class={`alert ${styles[type]} my-4 not-prose`}>
  <span><slot /></span>
</aside>
```

### Example post — `src/content/blog/hello-world.md`

```md
---
title: "Hello World"
description: "The first post in your new Astro starter theme."
pubDate: 2026-06-01
tags: ["welcome"]
draft: false
---

This is a plain **Markdown** post. Write headings, lists, code blocks, and links
exactly as you would anywhere else — Astro renders it through the same pipeline
as MDX, just without component support.
```

### Example case study — `src/content/case-studies/acme-redesign.mdx`

```mdx
---
title: "Acme Corp Website Redesign"
description: "A full rebuild that cut load times and lifted conversions."
client: "Acme Corp"
industry: "SaaS"
pubDate: 2026-05-20
results:
  - label: "Faster page loads"
    value: "-62%"
  - label: "Conversion lift"
    value: "+28%"
draft: false
---

import Aside from '../../components/Aside.astro';

We rebuilt Acme's marketing site on Astro, replacing a heavy SPA.

<Aside type="tip">The `results` array above renders as the daisyUI stats block in the layout.</Aside>
```

---

## 8. Routing — Pages, Blog, Case Studies

### Home (`src/pages/index.astro`)

Overwrite the template's default `index.astro` with this.

```astro
---
import BaseLayout from '../layouts/BaseLayout.astro';
import { getCollection } from 'astro:content';
import PostCard from '../components/PostCard.astro';
import { SITE_TITLE, SITE_DESCRIPTION } from '../consts';

const recent = (await getCollection('blog', ({ data }) => !data.draft))
  .sort((a, b) => b.data.pubDate.valueOf() - a.data.pubDate.valueOf())
  .slice(0, 3);
---
<BaseLayout title={SITE_TITLE} description={SITE_DESCRIPTION}>
  <section class="hero bg-base-200 rounded-box mb-12">
    <div class="hero-content text-center py-16">
      <div class="max-w-xl">
        <h1 class="text-5xl font-bold">{SITE_TITLE}</h1>
        <p class="py-6 text-base-content/70">{SITE_DESCRIPTION}</p>
        <a href="/blog" class="btn btn-primary">Read the blog</a>
      </div>
    </div>
  </section>

  <h2 class="text-3xl font-bold mb-6">Latest posts</h2>
  <div class="grid gap-6 md:grid-cols-2 lg:grid-cols-3">
    {recent.map((post) => <PostCard post={post} base="blog" />)}
  </div>
</BaseLayout>
```

### Static Pages (`src/pages/about.astro`)

```astro
---
import BaseLayout from '../layouts/BaseLayout.astro';
---
<BaseLayout title="About" description="About this Astro starter theme.">
  <h1 class="text-4xl font-bold mb-4">About</h1>
  <p>Any standalone page is just an .astro file under src/pages/.</p>
</BaseLayout>
```

### Contact page (`src/pages/contact.astro`)

A second static Page, showing a daisyUI form. (For a working submission, point the form at a service like Formspree or add an SSR adapter + action.)

```astro
---
import BaseLayout from '../layouts/BaseLayout.astro';
---
<BaseLayout title="Contact" description="Get in touch.">
  <h1 class="text-4xl font-bold mb-6">Contact</h1>
  <form class="max-w-md flex flex-col gap-4" method="POST" action="#">
    <label class="form-control">
      <span class="label-text">Name</span>
      <input type="text" name="name" class="input input-bordered" required />
    </label>
    <label class="form-control">
      <span class="label-text">Email</span>
      <input type="email" name="email" class="input input-bordered" required />
    </label>
    <label class="form-control">
      <span class="label-text">Message</span>
      <textarea name="message" class="textarea textarea-bordered" rows="4" required></textarea>
    </label>
    <button type="submit" class="btn btn-primary">Send</button>
  </form>
</BaseLayout>
```

### Blog listing (`src/pages/blog/index.astro`)

```astro
---
import { getCollection } from 'astro:content';
import BaseLayout from '../../layouts/BaseLayout.astro';
import PostCard from '../../components/PostCard.astro';

const posts = (await getCollection('blog', ({ data }) => !data.draft))
  .sort((a, b) => b.data.pubDate.valueOf() - a.data.pubDate.valueOf());
---
<BaseLayout title="Blog" description="Latest articles.">
  <h1 class="text-4xl font-bold mb-8">Blog</h1>
  <div class="grid gap-6 md:grid-cols-2 lg:grid-cols-3">
    {posts.map((post) => <PostCard post={post} base="blog" />)}
  </div>
</BaseLayout>
```

### Blog post (`src/pages/blog/[...slug].astro`)

```astro
---
import { getCollection, render } from 'astro:content';
import BlogLayout from '../../layouts/BlogLayout.astro';

export async function getStaticPaths() {
  const posts = await getCollection('blog', ({ data }) => !data.draft);
  return posts.map((post) => ({
    params: { slug: post.id },   // v6 uses `id`, not `slug`
    props: { post },
  }));
}

const { post } = Astro.props;
const { Content } = await render(post); // v6: render() from astro:content
---
<BlogLayout {...post.data}>
  <Content />
</BlogLayout>
```

### Case Studies listing (`src/pages/case-studies/index.astro`)

```astro
---
import { getCollection } from 'astro:content';
import BaseLayout from '../../layouts/BaseLayout.astro';
import PostCard from '../../components/PostCard.astro';

const studies = (await getCollection('caseStudies', ({ data }) => !data.draft))
  .sort((a, b) => b.data.pubDate.valueOf() - a.data.pubDate.valueOf());
---
<BaseLayout title="Case Studies" description="Selected client work and results.">
  <h1 class="text-4xl font-bold mb-8">Case Studies</h1>
  <div class="grid gap-6 md:grid-cols-2 lg:grid-cols-3">
    {studies.map((study) => <PostCard post={study} base="case-studies" />)}
  </div>
</BaseLayout>
```

### Case Study page (`src/pages/case-studies/[...slug].astro`)

```astro
---
import { getCollection, render } from 'astro:content';
import CaseStudyLayout from '../../layouts/CaseStudyLayout.astro';

export async function getStaticPaths() {
  const studies = await getCollection('caseStudies', ({ data }) => !data.draft);
  return studies.map((study) => ({
    params: { slug: study.id },
    props: { study },
  }));
}

const { study } = Astro.props;
const { Content } = await render(study);
---
<CaseStudyLayout {...study.data}>
  <Content />
</CaseStudyLayout>
```

### `src/components/PostCard.astro` (daisyUI card)

```astro
---
const { post, base } = Astro.props;
---
<a href={`/${base}/${post.id}/`} class="card bg-base-200 shadow-md hover:shadow-xl transition-shadow">
  {post.data.heroImage && (
    <figure><img src={post.data.heroImage.src} alt="" /></figure>
  )}
  <div class="card-body">
    <h2 class="card-title">{post.data.title}</h2>
    <p class="text-base-content/70">{post.data.description}</p>
    <div class="card-actions">
      {post.data.tags?.map((t) => <span class="badge badge-outline">{t}</span>)}
    </div>
  </div>
</a>
```

---

## 9. RSS feed

`@astrojs/rss` generates a feed from your collection at build time.

### `src/pages/rss.xml.js` → served at `/rss.xml`

```js
import rss from '@astrojs/rss';
import { getCollection } from 'astro:content';
import { SITE_TITLE, SITE_DESCRIPTION } from '../consts';

export async function GET(context) {
  const posts = (await getCollection('blog', ({ data }) => !data.draft))
    .sort((a, b) => b.data.pubDate.valueOf() - a.data.pubDate.valueOf());

  return rss({
    title: SITE_TITLE,
    description: SITE_DESCRIPTION,
    site: context.site, // pulled from astro.config.mjs `site`
    items: posts.map((post) => ({
      title: post.data.title,
      description: post.data.description,
      pubDate: post.data.pubDate,
      link: `/blog/${post.id}/`,
    })),
  });
}
```

Autodiscovery is already wired via the `<link rel="alternate" type="application/rss+xml">` tag in `BaseHead.astro`. Optionally add a `rssSchema` validation to your collection schema with `import { rssSchema } from '@astrojs/rss'`.

---

## 10. XML Sitemap

Handled entirely by `@astrojs/sitemap` (added in step 2 / configured in §4). On `npm run build` it emits:

- `/sitemap-index.xml` — the index file to submit to search engines
- `/sitemap-0.xml` — the URL set

Point search consoles and `robots.txt` at the **index** file.

### `public/robots.txt`

```
User-agent: *
Allow: /

Sitemap: https://www.example.com/sitemap-index.xml
```

The sitemap auto-includes every statically rendered route (pages, blog posts, case studies). Use the `filter` option (shown in §4) to exclude drafts or utility routes.

---

## 11. Build, preview, deploy

```bash
npm run dev       # local dev server (HMR)
npm run build     # outputs static site to ./dist
npm run preview   # serve the production build locally
```

Default output is **static** — ideal for SEO and for hosts like Netlify, Vercel, Cloudflare Pages, or GitHub Pages. Set the live domain in `site` before the production build so canonical URLs, OG tags, the sitemap, and RSS all resolve to the real host.

---

## 12. Known gotchas (Astro 6-specific) ⚠️

1. **Don't run `npx astro add tailwind` on Astro 6.** Astro 6 defaults to `rolldown-vite`, and the auto-installed `@tailwindcss/vite` plugin can fail the build with a `Missing field tsconfigPaths` / `BindingViteResolvePluginConfig` error (tracked in `withastro/astro#16542`). **Fix:** wire `@tailwindcss/vite` manually (as in §4). If the build still fails on rolldown-vite, fall back to the **PostCSS path**: `npm install @tailwindcss/postcss postcss`, remove the Vite plugin, and add a `postcss.config.mjs` with `export default { plugins: { '@tailwindcss/postcss': {} } }`. Check whether the issue is resolved in your exact 6.4.x patch first.

2. **Node 22+ is mandatory.** Astro 6 dropped Node 18/20. A wrong Node version produces confusing install/build errors.

3. **No `tailwind.config.js`.** Tailwind v4 is CSS-first — themes, plugins, and tokens go in `global.css` via `@theme` and `@plugin`. Old config-file tutorials are obsolete.

4. **daisyUI is a CSS `@plugin`, not a JS plugin.** `@plugin "daisyui"` in your CSS — there is no `plugins: [require('daisyui')]` array anymore.

5. **Content config moved.** It's `src/content.config.ts` (not `src/content/config.ts`), and `z` should come from `astro/zod`, not `astro:content`.

6. **Content Layer API names changed.** Use `entry.id` (not `entry.slug`) and `render(entry)` imported from `astro:content` (not `entry.render()`).

7. **`site` must be set** in `astro.config.mjs` or canonical URLs, Open Graph images, RSS links, and the sitemap will all be broken.

---

## 13. SEO acceptance checklist

- [ ] `site` set to the production domain in `astro.config.mjs`
- [ ] Every page renders a unique `<title>` and meta description via `BaseHead`
- [ ] `<link rel="canonical">` present and absolute on every page
- [ ] Open Graph (`og:title`, `og:description`, `og:image`, `og:url`, `og:type`) on every page
- [ ] `og:image` is an absolute URL (1200×630 default in `public/og-default.png`)
- [ ] Articles send `type="article"` + `article:published_time`
- [ ] `twitter:card = summary_large_image` with image
- [ ] `/sitemap-index.xml` builds and is referenced in `robots.txt`
- [ ] `/rss.xml` builds and is auto-discoverable via `<link rel="alternate">`
- [ ] Validate with a social-card debugger and Google Rich Results / Search Console after deploy
