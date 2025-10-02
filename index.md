---
layout: home
title: Home
---

# Over-Reacting
## A Recovery Guide for React Developers

<p class="subtitle">For developers moving from the world of React to build with better tools</p>

---

### The Five Stages of React Grief

1. **Denial:** "useState is fine. useEffect is fine. Everything is fine."
2. **Anger:** "WHY IS IT RE-RENDERING?!"
3. **Bargaining:** "Maybe if I just wrap everything in useMemo..."
4. **Depression:** *stares at dependency array at 3 AM*
5. **Acceptance:** "There are other frameworks. Good ones. Better ones, even."

**Welcome to acceptance.**

---

### What You'll Learn

This book covers real alternatives to React, with practical migration strategies:

- **Vue** - Reactivity that actually makes sense
- **Svelte** - The compiler that disappears at build time
- **Solid** - JSX without the re-rendering pain
- **Angular** - Opinionated structure that prevents chaos
- **Alpine** - Lightweight interactivity without the build step
- **Lit** - Web Components for the modern era
- **HTMX** - What if the backend was right all along?
- **Vanilla Web Standards** - The platform can do more than you think
- **Astro** - Zero JavaScript by default for content sites

Plus **practical migration strategies** that won't get you fired.

---

### Start Reading

<div class="chapter-grid">
  {% for chapter in site.chapters %}
  <div class="chapter-card">
    <h3><a href="{{ site.baseurl }}/{{ chapter.file }}">Chapter {{ forloop.index }}</a></h3>
    <p>{{ chapter.title }}</p>
  </div>
  {% endfor %}
</div>

Or start with the [Table of Contents]({{ site.baseurl }}/00-table-of-contents).

---

### About This Book

This book is for developers who:
- Are frustrated with React's complexity
- Want to explore better alternatives
- Need practical migration strategies
- Deserve frameworks that feel like they're on your side

You are not a bad developer for using React. But you deserve better than prop drilling, useEffect dependency arrays, and re-render optimization hell.

---

*"I spent three years thinking I just wasn't smart enough to understand useEffect. Turns out it just doesn't make sense."*
— Every Developer Ever

---

<div class="cta">
  <a href="{{ site.baseurl }}/01-introduction" class="button">Start Reading →</a>
</div>
