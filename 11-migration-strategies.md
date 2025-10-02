# Chapter 11: The Great Escape - Practical Migration Strategies

## You Can't Rewrite Everything Tomorrow (And Keep Your Job)

Here's the fantasy: You read this book, get inspired, convince your team, and rewrite your entire React codebase in Svelte over the weekend.

Here's reality: You have a product to ship. Users to support. A boss who will ask "why is nothing working?"

Migrations happen gradually. Smart migrations happen *invisibly*.

This chapter is about escaping React without blowing up your application or your career.

## The Risk Assessment: What Can You Safely Change?

Not all parts of your codebase are equally risky to migrate.

### Low Risk (Start Here)

**New features:**
- Not replacing anything
- Can use new framework
- Easy to rollback (just delete it)
- Perfect for learning

**Leaf components:**
- No children components
- Self-contained
- Easy to test in isolation
- Low blast radius

**Rarely-changed pages:**
- Settings pages
- About pages
- Admin panels
- If migration breaks, few users notice

### Medium Risk

**High-traffic, low-complexity pages:**
- Homepage (lots of users, but simple)
- Landing pages
- Profile pages
- Monitor closely, but manageable

**Shared components:**
- Design system components
- UI primitives
- High reuse, but well-tested

### High Risk (Save for Last)

**Critical path features:**
- Checkout flow
- Authentication
- Payment processing
- Core user journeys

**Complex, interconnected components:**
- Dashboards with lots of state
- Real-time collaborative features
- Anything with "we're not sure how it works"

Start low-risk. Build confidence. Move up.

## Strategy 1: The New Feature Approach

The safest migration: use the new framework for new features only.

### Setup: Multiple Frameworks in One App

**With Vite:**
```javascript
// vite.config.js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import vue from '@vitejs/plugin-vue'
import svelte from '@vitejs/plugin-svelte'

export default defineConfig({
  plugins: [react(), vue(), svelte()]
})
```

All three frameworks work side-by-side.

### Mounting Other Frameworks in React

**Vue in React:**
```jsx
import { useEffect, useRef } from 'react'
import { createApp } from 'vue'
import VueComponent from './Component.vue'

function VueWrapper({ someProp }) {
  const ref = useRef(null)
  const appRef = useRef(null)

  useEffect(() => {
    appRef.current = createApp(VueComponent, { someProp })
    appRef.current.mount(ref.current)

    return () => appRef.current.unmount()
  }, [])

  useEffect(() => {
    if (appRef.current) {
      // Update props
      appRef.current._instance.props.someProp = someProp
    }
  }, [someProp])

  return <div ref={ref} />
}
```

**Svelte in React:**
```jsx
import { useEffect, useRef } from 'react'
import SvelteComponent from './Component.svelte'

function SvelteWrapper({ someProp }) {
  const ref = useRef(null)
  const componentRef = useRef(null)

  useEffect(() => {
    componentRef.current = new SvelteComponent({
      target: ref.current,
      props: { someProp }
    })

    return () => componentRef.current.$destroy()
  }, [])

  useEffect(() => {
    if (componentRef.current) {
      componentRef.current.$set({ someProp })
    }
  }, [someProp])

  return <div ref={ref} />
}
```

Now you can gradually introduce the new framework.

## Strategy 2: The Microfrontend Approach

Run different frameworks as separate applications.

### iframe Approach (Quick & Dirty)

```jsx
function LegacyDashboard() {
  return (
    <iframe
      src="/svelte-dashboard"
      style={{ width: '100%', height: '600px', border: 'none' }}
    />
  )
}
```

Pros:
- Complete isolation
- Zero integration issues
- Deploy independently

Cons:
- Styling isolation (can be a pro or con)
- Communication requires postMessage
- Not great for SEO

### Module Federation (Sophisticated)

```javascript
// React app webpack.config.js
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'reactApp',
      remotes: {
        vueApp: 'vueApp@http://localhost:3001/remoteEntry.js'
      }
    })
  ]
}

// Vue app webpack.config.js
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'vueApp',
      filename: 'remoteEntry.js',
      exposes: {
        './Dashboard': './src/Dashboard.vue'
      }
    })
  ]
}
```

```jsx
// React app
const VueDashboard = React.lazy(() => import('vueApp/Dashboard'))

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <VueDashboard />
    </Suspense>
  )
}
```

Different apps, different frameworks, working together.

## Strategy 3: The Page-by-Page Replacement

Use different frameworks for different routes.

### With a Meta-Framework

**React + Next.js → Astro hybrid:**

```javascript
// astro.config.mjs
export default defineConfig({
  integrations: [react()],
  output: 'server'
})
```

```
pages/
  index.astro          ← New (Astro)
  blog/
    [slug].astro       ← New (Astro)
  dashboard/
    index.jsx          ← Old (React)
    settings.jsx       ← Old (React)
```

Gradually convert pages. Both work during transition.

### With Nginx Routing

```nginx
# nginx.conf
location / {
  # Default React app
  proxy_pass http://react-app:3000;
}

location /new-dashboard {
  # New Svelte app
  proxy_pass http://svelte-app:3001;
}

location /blog {
  # New Astro static site
  root /var/www/astro-blog;
}
```

Different apps, unified under one domain.

## Strategy 4: The Component Library Approach

Rebuild your design system in a framework-agnostic way.

### Using Web Components (Lit)

```javascript
// button.js - Works everywhere
import { LitElement, html, css } from 'lit'

class DSButton extends LitElement {
  static properties = {
    variant: {},
    disabled: { type: Boolean }
  }

  static styles = css`
    button {
      padding: 8px 16px;
      border-radius: 4px;
      cursor: pointer;
    }
  `

  render() {
    return html`
      <button ?disabled=${this.disabled}>
        <slot></slot>
      </button>
    `
  }
}

customElements.define('ds-button', DSButton)
```

**Use in React:**
```jsx
<ds-button variant="primary" onClick={handleClick}>
  Click me
</ds-button>
```

**Use in Vue:**
```vue
<ds-button variant="primary" @click="handleClick">
  Click me
</ds-button>
```

**Use in Svelte:**
```svelte
<ds-button variant="primary" on:click={handleClick}>
  Click me
</ds-button>
```

One component library. Works with any framework. Or no framework.

## Strategy 5: The Strangler Fig Pattern

Gradually wrap and replace React components.

### Phase 1: Identify Boundaries

```
UserDashboard (React)
├── Header (React)
├── Sidebar (React)
├── MainContent (React)
│   ├── UserProfile (React) ← Start here
│   ├── ActivityFeed (React)
│   └── Settings (React)
└── Footer (React)
```

### Phase 2: Replace Leaf Nodes

```
UserDashboard (React)
├── Header (React)
├── Sidebar (React)
├── MainContent (React)
│   ├── UserProfile (Vue) ← Replaced
│   ├── ActivityFeed (React)
│   └── Settings (React)
└── Footer (React)
```

### Phase 3: Work Your Way Up

```
UserDashboard (React)
├── Header (React)
├── Sidebar (React)
├── MainContent (Vue) ← Replaced, contains Vue children
│   ├── UserProfile (Vue)
│   ├── ActivityFeed (Vue) ← Replaced
│   └── Settings (Vue) ← Replaced
└── Footer (React)
```

### Phase 4: Replace Root

```
UserDashboard (Vue) ← Fully migrated
├── Header (Vue)
├── Sidebar (Vue)
├── MainContent (Vue)
│   ├── UserProfile (Vue)
│   ├── ActivityFeed (Vue)
│   └── Settings (Vue)
└── Footer (Vue)
```

## Dealing with Shared State

The hardest part: state management during migration.

### Approach 1: Keep React State, Bridge to New Framework

```javascript
// Shared store (Zustand)
import create from 'zustand'

export const useUserStore = create((set) => ({
  user: null,
  setUser: (user) => set({ user })
}))
```

**React component:**
```jsx
function ReactComponent() {
  const user = useUserStore(state => state.user)
  return <div>{user.name}</div>
}
```

**Bridge to Vue:**
```vue
<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import { useUserStore } from './store'

const user = ref(null)
let unsubscribe

onMounted(() => {
  const store = useUserStore.getState()
  user.value = store.user

  unsubscribe = useUserStore.subscribe((state) => {
    user.value = state.user
  })
})

onUnmounted(() => {
  unsubscribe()
})
</script>

<template>
  <div>{{ user.name }}</div>
</template>
```

### Approach 2: Custom Events for Communication

```javascript
// event-bus.js
export function emit(event, data) {
  window.dispatchEvent(new CustomEvent(event, { detail: data }))
}

export function on(event, callback) {
  window.addEventListener(event, (e) => callback(e.detail))
}
```

**React component (emits):**
```jsx
import { emit } from './event-bus'

function ReactComponent() {
  const handleUpdate = () => {
    emit('user-updated', { name: 'Alice' })
  }

  return <button onClick={handleUpdate}>Update</button>
}
```

**Vue component (listens):**
```vue
<script setup>
import { ref, onMounted } from 'vue'
import { on } from './event-bus'

const user = ref(null)

onMounted(() => {
  on('user-updated', (data) => {
    user.value = data
  })
})
</script>
```

Simple. Decoupled. Works across any framework.

### Approach 3: URL State for Coordination

```javascript
// React component
function ReactFilter() {
  const setFilter = (value) => {
    const url = new URL(window.location)
    url.searchParams.set('filter', value)
    window.history.pushState({}, '', url)
    window.dispatchEvent(new PopStateEvent('popstate'))
  }
}

// Vue component
<script setup>
import { ref, onMounted } from 'vue'

const filter = ref(new URLSearchParams(window.location.search).get('filter'))

onMounted(() => {
  window.addEventListener('popstate', () => {
    filter.value = new URLSearchParams(window.location.search).get('filter')
  })
})
</script>
```

URL is the shared state. Both frameworks react to it.

## Testing During Migration

### Test Both Versions

```javascript
describe('UserProfile', () => {
  describe('React version', () => {
    it('displays user name', () => {
      // Test React component
    })
  })

  describe('Vue version', () => {
    it('displays user name', () => {
      // Test Vue component
    })
  })
})
```

Same tests. Different implementations. Ensures parity.

### Visual Regression Testing

```javascript
// with Playwright
test('UserProfile looks the same', async ({ page }) => {
  // React version
  await page.goto('/profile?version=react')
  await expect(page).toHaveScreenshot('profile-react.png')

  // Vue version
  await page.goto('/profile?version=vue')
  await expect(page).toHaveScreenshot('profile-vue.png')

  // Should be identical
})
```

### Feature Flags for Gradual Rollout

```javascript
// feature-flags.js
export function showVueVersion(userId) {
  // 10% of users
  if (hash(userId) % 100 < 10) return true

  // Or specific users
  if (BETA_USERS.includes(userId)) return true

  return false
}
```

```jsx
function UserProfile({ userId }) {
  if (showVueVersion(userId)) {
    return <VueUserProfile userId={userId} />
  }

  return <ReactUserProfile userId={userId} />
}
```

Rollout gradually. Monitor. Adjust.

## Rollback Plans

Always have a rollback plan.

### Feature Flag Rollback

```javascript
// Instant rollback
const FEATURES = {
  vueUserProfile: false  // Switch back to React
}
```

### Git Rollback

```bash
# Tag the migration commit
git tag -a vue-migration -m "Switched UserProfile to Vue"

# If it goes wrong
git revert vue-migration
```

### Traffic Splitting

```nginx
# Canary deployment
location /profile {
  if ($random_number < 90) {
    proxy_pass http://react-app:3000;
  }

  proxy_pass http://vue-app:3001;
}
```

If the new version has issues, route traffic back to the old version.

## Team Buy-In: The Human Side

### Start Small, Prove Value

Don't propose rewriting everything. Propose:

"I'd like to try Svelte for this one new feature. If it goes well, we can consider more."

Small experiment. Low risk. Easy to approve.

### Show Metrics

After successful migration:

- "Bundle size decreased 40%"
- "Page load time improved 1.2s"
- "Developer velocity increased—feature took 3 days instead of 5"

Numbers convince managers.

### Run a Spike

Time-box an experiment:

"I'll spend 2 days building this feature in both React and Vue. We'll compare and decide."

Concrete comparison beats hypothetical debate.

### Address Concerns

**"We don't know the new framework"**
→ "I'll document patterns and run knowledge-sharing sessions"

**"Hiring will be harder"**
→ "Svelte is easy to learn. We can train React devs in a week"

**"What about maintenance?"**
→ "We'll maintain both during transition. Full migration in 6 months"

## Timeline: What to Expect

**Week 1-2: Setup & Proof of Concept**
- Set up build system for new framework
- Build one small component
- Verify it works in production

**Month 1-2: Low-Risk Migration**
- New features in new framework
- Convert leaf components
- Build confidence

**Month 3-4: Medium-Risk Migration**
- Convert shared components
- Rebuild design system
- Start on high-traffic pages

**Month 5-6: High-Risk Migration**
- Critical path features
- Remove React entirely
- Celebrate

This is conservative. You might go faster. But plan for 6 months minimum for a full migration.

## When to Stop Migrating

Sometimes you don't finish. And that's okay.

**Stop if:**
- The cost exceeds the benefit
- The old code works fine and is low-maintenance
- You're spending more time migrating than building features
- Team morale is suffering

**It's fine to have:**
- 80% new framework, 20% React
- Different frameworks for different parts of the app
- A permanent "hybrid" architecture

Perfection is the enemy of progress.

## Real Talk: Migrations Are Hard

Migrating frameworks is:
- Time-consuming
- Risky
- Sometimes frustrating
- Harder than you think

But it can also be:
- Rewarding
- A massive learning experience
- A huge quality-of-life improvement
- Worth it

Just go in with your eyes open. Plan carefully. Move gradually. Test thoroughly. And have a rollback plan.

---

*"We migrated from React to Svelte over 6 months. The first month was exciting. Month 3 was painful. Month 6 we shipped and never looked back. Worth it."*
— A Developer Who Survived a Migration

**Up Next:** [Chapter 12: Life After React - Finding Your New Framework Family](12-conclusion.md)
