---
layout: default
---

# Chapter 10: Astro - Content Sites Don't Need Virtual DOMs

## Ship Zero JavaScript (By Default)

Picture this: You're building a blog. It has:
- Articles
- An about page
- A contact form
- Some syntax highlighting
- Maybe a dark mode toggle

You reach for React because, well, that's what you know.

Final bundle size: 300KB of JavaScript for a site that's 95% static content.

Astro looked at this and asked a radical question: "What if we shipped zero JavaScript by default and only added it where actually needed?"

## The Islands Architecture

Traditional SPAs (React, Vue, etc.):
```
┌─────────────────────────────────┐
│     Entire Page is JavaScript   │
│  ┌──────────┐  ┌──────────┐    │
│  │ Header   │  │  Nav     │    │
│  └──────────┘  └──────────┘    │
│  ┌──────────────────────────┐  │
│  │   Content (static!)      │  │
│  │   Why is this JS???      │  │
│  └──────────────────────────┘  │
│  ┌──────────┐  ┌──────────┐    │
│  │ Sidebar  │  │  Footer  │    │
│  └──────────┘  └──────────┘    │
│                                 │
│  Bundle: 300KB                  │
└─────────────────────────────────┘
```

Astro (Islands):
```
┌─────────────────────────────────┐
│       Static HTML               │
│  ┌──────────┐  ┌──────────┐    │
│  │ Header   │  │  Nav     │    │
│  └──────────┘  └──────────┘    │
│  ┌──────────────────────────┐  │
│  │   Content (HTML!)        │  │
│  └──────────────────────────┘  │
│  ┌──────────┐  ┌──────────┐    │
│  │ Static   │  │ 🏝️Island│←JS │
│  └──────────┘  └──────────┘    │
│                                 │
│  Bundle: 5KB (just the island)  │
└─────────────────────────────────┘
```

Most of the page is static HTML. Interactive parts are "islands" of JavaScript.

## Astro Components: HTML-First

```astro
---
// This runs at BUILD TIME, not in the browser
const posts = await fetch('https://api.example.com/posts').then(r => r.json())
const title = 'My Blog'
---

<html>
  <head>
    <title>{title}</title>
  </head>
  <body>
    <h1>{title}</h1>

    {posts.map(post => (
      <article>
        <h2>{post.title}</h2>
        <p>{post.excerpt}</p>
      </article>
    ))}
  </body>
</html>
```

The API call happens at build time. The data gets baked into HTML. Zero JavaScript ships to the browser.

In React, this would:
1. Ship React runtime
2. Ship component code
3. Fetch data in useEffect
4. Show loading state
5. Re-render with data

In Astro, users just get HTML. Fast. Instant. No loading states.

## Bringing Your Own Framework

Here's the brilliant part: you can use React, Vue, Svelte, Solid, or anything else for the interactive islands:

```astro
---
import ReactCounter from './ReactCounter.jsx'
import VueSearch from './VueSearch.vue'
import SvelteNav from './SvelteNav.svelte'
---

<html>
  <body>
    <h1>Static HTML Header</h1>

    <!-- React island, only loads React for this component -->
    <ReactCounter client:load />

    <!-- Vue island, only loads Vue for this component -->
    <VueSearch client:idle />

    <!-- Svelte island -->
    <SvelteNav client:visible />

    <!-- Static content, no JS -->
    <article>
      <p>This is just HTML, no framework needed!</p>
    </article>
  </body>
</html>
```

Different frameworks in the same page. Each island is independent. Or use no framework at all.

## Client Directives: When to Load JavaScript

Astro gives you fine control over when JavaScript loads:

### client:load
```astro
<ReactCounter client:load />
```
Loads immediately on page load. Use for critical interactive elements.

### client:idle
```astro
<VueSearch client:idle />
```
Loads when the browser is idle (using requestIdleCallback). Use for important but not critical elements.

### client:visible
```astro
<SvelteCarousel client:visible />
```
Loads when the component scrolls into view. Perfect for below-the-fold content.

### client:media
```astro
<MobileMenu client:media="(max-width: 768px)" />
```
Loads only if media query matches. Ship mobile menu JS only to mobile users.

### client:only
```astro
<ReactWidget client:only="react" />
```
Never server-renders, only runs in browser. Use for things that need browser APIs.

### Default (no directive)
```astro
<ReactComponent />
```
Renders to static HTML at build time. Zero JavaScript shipped.

## Content Collections: Type-Safe Content

Astro treats content as data:

```typescript
// src/content/config.ts
import { defineCollection, z } from 'astro:content'

const blog = defineCollection({
  schema: z.object({
    title: z.string(),
    description: z.string(),
    publishDate: z.date(),
    author: z.string(),
    tags: z.array(z.string())
  })
})

export const collections = { blog }
```

Now your blog posts are type-checked:

```markdown
---
title: "Why React Made Me Sad"
description: "A journey through useEffect"
publishDate: 2024-01-15
author: "Recovering Dev"
tags: ["react", "recovery", "therapy"]
---

Your markdown content here...
```

Query it with type safety:

```astro
---
import { getCollection } from 'astro:content'

const posts = await getCollection('blog')
// TypeScript knows the shape!
---

{posts.map(post => (
  <article>
    <h2>{post.data.title}</h2>
    <p>By {post.data.author}</p>
  </article>
))}
```

## Markdown and MDX Support

Write content in Markdown:

```markdown
---
title: My Post
---

# Heading

This is **markdown**!
```

Or use MDX for components in your content:

```mdx
---
title: Interactive Post
---

import { Chart } from '../components/Chart.jsx'

# My Post

Here's some content.

<Chart data={salesData} client:visible />

And more content!
```

## Migration from React: The Content-First Rewrite

### Step 1: Identify What's Actually Static

Most React sites have way more static content than interactive features:

**React blog (all interactive):**
- Header: Could be static
- Nav: Could be static
- Article content: Could be static
- Sidebar: Could be static
- Footer: Could be static
- Comments section: Actually needs JavaScript

You shipped 300KB of React for the comments section.

**Astro blog (mostly static):**
```astro
---
const article = await getArticle(Astro.params.id)
---

<!-- All of this is static HTML -->
<Layout>
  <Header />
  <Nav />

  <article>
    <h1>{article.title}</h1>
    <div set:html={article.content} />
  </article>

  <Sidebar />

  <!-- Only this needs JavaScript -->
  <Comments articleId={article.id} client:visible />

  <Footer />
</Layout>
```

Comments component can still be React! But now you only ship React code for that one component.

### Step 2: Convert Pages to Astro

**Before (React + Next.js):**
```jsx
// pages/blog/[slug].jsx
import { useEffect, useState } from 'react'

export default function BlogPost({ slug }) {
  const [post, setPost] = useState(null)

  useEffect(() => {
    fetch(`/api/posts/${slug}`)
      .then(r => r.json())
      .then(setPost)
  }, [slug])

  if (!post) return <Loading />

  return (
    <article>
      <h1>{post.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  )
}
```

**After (Astro):**
```astro
---
// src/pages/blog/[slug].astro
import { getEntry } from 'astro:content'

const { slug } = Astro.params
const post = await getEntry('blog', slug)
const { Content } = await post.render()
---

<article>
  <h1>{post.data.title}</h1>
  <Content />
</article>
```

No loading state. No useEffect. No client-side data fetching. It's pre-rendered at build time.

### Step 3: Keep React for Interactive Parts

```astro
---
// src/pages/dashboard.astro
import ReactDashboard from '../components/ReactDashboard'
import { getStats } from '../lib/stats'

const stats = await getStats() // Build-time or server
---

<Layout>
  <!-- Static header -->
  <header>
    <h1>Dashboard</h1>
  </header>

  <!-- Interactive React island -->
  <ReactDashboard
    client:load
    initialStats={stats}
  />
</Layout>
```

Your existing React components work. But now they're islands in a sea of static HTML.

## Adapters: SSR When You Need It

Astro defaults to static site generation. But it can do server-side rendering:

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config'
import node from '@astrojs/node'

export default defineConfig({
  output: 'server',  // or 'hybrid'
  adapter: node()
})
```

Now you can have dynamic routes:

```astro
---
// src/pages/api/users/[id].json.js
export async function get({ params }) {
  const user = await db.getUser(params.id)
  return {
    body: JSON.stringify(user)
  }
}
---
```

Or server-render specific pages:

```astro
---
// src/pages/dashboard.astro
export const prerender = false // This page is server-rendered

const user = await getCurrentUser(Astro.request)
---

<h1>Welcome, {user.name}!</h1>
```

## View Transitions: SPAs Without the Overhead

Astro has built-in view transitions:

```astro
---
// src/layouts/Layout.astro
import { ViewTransitions } from 'astro:transitions'
---

<html>
  <head>
    <ViewTransitions />
  </head>
  <body>
    <slot />
  </body>
</html>
```

Now page navigations feel like an SPA, with smooth transitions, without shipping a router or framework.

## When Astro Shines

Astro is perfect for:

- **Blogs** - Mostly content, minimal interaction
- **Documentation sites** - Static content with search
- **Marketing sites** - Fast load times matter
- **Portfolios** - Show off that Lighthouse score
- **E-commerce content pages** - Product pages, category pages
- **Any content-heavy site** - News, magazines, etc.

Astro might not be ideal for:

- **Highly interactive apps** - Dashboards, tools, games
- **Real-time collaboration** - Google Docs-style apps
- **Apps with mostly dynamic content** - Social feeds, messaging
- **Things that are actually applications, not websites**

## The Performance Numbers

Let's talk Lighthouse scores.

**Typical React blog:**
- First Contentful Paint: 2.1s
- Time to Interactive: 3.8s
- Total Bundle: 312KB
- Lighthouse Performance: 67

**Same blog in Astro:**
- First Contentful Paint: 0.4s
- Time to Interactive: 0.4s
- Total Bundle: 0KB (static pages), 8KB (with one island)
- Lighthouse Performance: 100

Users notice. Google notices. Your boss notices.

## Real Talk: Astro vs React

React wants to be your entire application framework.

Astro wants to be your content framework that can use React when needed.

Different goals.

If you're building Facebook, use React.

If you're building a blog that happens to have a React component for comments, maybe don't ship the entire React runtime for the header, footer, and article content.

Astro gives you the escape hatch: use frameworks where they add value, skip them where they don't.

## The Philosophy

React's philosophy: "Everything is a component. Components are JavaScript. JavaScript runs in the browser."

Astro's philosophy: "Most content is static. Ship HTML. Add JavaScript only where it improves the experience."

One ships JavaScript by default.
One ships HTML by default.

For content sites, HTML-first wins.

---

*"I migrated my blog from Next.js to Astro. My Lighthouse score went from 68 to 100, and my hosting costs went to zero because I could host it on Netlify's free tier."*
— A Developer Who Stopped Over-Engineering Blogs

**Up Next:** [Chapter 11: The Great Escape - Practical Migration Strategies](11-migration-strategies.md)
