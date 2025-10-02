# Chapter 2: Vue - The Gentle Intervention

## Reactivity That Actually Makes Sense

Remember the first time you tried to understand React's reactivity model?

"So the component is a function that returns JSX, and when state changes, the whole function runs again, which is why you can't put hooks in conditionals, and that's why you need useCallback to prevent functions from being recreated, but you also need useMemo for values, and useEffect for side effects, but only after render, except when—"

Now let me explain Vue's reactivity:

"The data is reactive."

That's it. That's the whole model.

```vue
<script setup>
import { ref } from 'vue'

const count = ref(0)
const increment = () => count.value++
</script>

<template>
  <button @click="increment">
    Count: {{ count }}
  </button>
</template>
```

Notice what's missing? No useState. No useCallback. No dependency arrays. No memoization. No rules about where you can and can't put things. The button knows it depends on `count`. When `count` changes, the button updates. Done.

Your React brain is screaming "but HOW does it know?"

It uses Proxies. Magic compiler transforms. Actual computer science. The details don't matter—it just works. And it works *correctly*, without you having to prove to the framework that you understand its dependency system.

## Templates vs JSX: A Peace Treaty

I know what you're thinking. "Templates? Isn't that, like, from 2012? Aren't we past that?"

Here's the thing: templates are actually... good?

**React's pitch:** "It's just JavaScript! You have the full power of JavaScript in your markup!"

**Reality:** You end up with:
```jsx
{items && items.length > 0 && items.map(item => (
  item.visible && item.active ? (
    <Item key={item.id} {...item} />
  ) : null
))}
```

**Vue's pitch:** "Here are directives for common patterns."

**Reality:**
```vue
<Item
  v-for="item in items"
  v-if="item.visible && item.active"
  :key="item.id"
  v-bind="item"
/>
```

One is "just JavaScript." The other is just *readable*.

### The Conditional Rendering Showdown

**React:**
```jsx
{isLoading ? (
  <Spinner />
) : error ? (
  <Error message={error} />
) : data ? (
  <Content data={data} />
) : null}
```

**Vue:**
```vue
<Spinner v-if="isLoading" />
<Error v-else-if="error" :message="error" />
<Content v-else-if="data" :data="data" />
```

Which one would you rather debug at 2 AM?

### Your Designer's Perspective

**Before Vue:**
"Can you just make this section purple?"
*You change CSS*
*Three components break because of CSS-in-JS cascade issues*
*Spend two hours debugging*

**After Vue:**
"Can you just make this section purple?"
*Designer changes CSS*
*It's purple*
*You continue with your life*

Vue uses actual CSS. In actual files. With actual scoping that works correctly. Your designer can work in the `<style>` section. You can work in the `<script>` section. Nobody needs to understand CSS-in-JS, styled-components, emotion, or whatever we're pretending is better than CSS this week.

## Pinia vs Redux: A Tragedy in One Act

**Redux introduction (2015):**
"We need predictable state management!"
*Creates three files per feature*
*Writes ten lines of boilerplate per action*
*Adds middleware*
*Adds thunks*
*Adds sagas*
*Achieves predictable suffering*

**Redux Toolkit (2019):**
"Okay, we made Redux less painful!"
*Still requires ceremony*
*Still has immutability gotchas*
*Still feels like work*

**Pinia (2020):**
"What if state management was just... nice?"

```javascript
// stores/counter.js
import { defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', {
  state: () => ({ count: 0 }),
  actions: {
    increment() {
      this.count++  // Yes, just mutate it. It's fine.
    }
  }
})
```

```vue
<!-- Component -->
<script setup>
import { useCounterStore } from './stores/counter'

const counter = useCounterStore()
</script>

<template>
  <button @click="counter.increment">
    {{ counter.count }}
  </button>
</template>
```

That's the whole thing. No reducers. No actions files. No dispatch. No middleware. Just stores with state and methods. Like JavaScript objects, but reactive.

"But type safety!"

It's fully typed. TypeScript just works.

"But DevTools!"

Better DevTools than Redux. Time travel and everything.

"But what about—"

It's good. It's all good. Just use it.

## Migrating from React: The Methadone Approach

You can't rewrite everything at once (your manager would have opinions). Here's how to migrate gradually while keeping your job:

### Phase 1: New Features Only

Start writing new features in Vue. Your build system can handle both:

```javascript
// vite.config.js
import vue from '@vitejs/plugin-vue'
import react from '@vitejs/plugin-react'

export default {
  plugins: [vue(), react()]
}
```

Mount Vue components in your React app:
```jsx
// React component
import { createApp } from 'vue'
import MyVueComponent from './MyVueComponent.vue'

function VueWrapper() {
  const ref = useRef(null)

  useEffect(() => {
    const app = createApp(MyVueComponent)
    app.mount(ref.current)
    return () => app.unmount()
  }, [])

  return <div ref={ref} />
}
```

It's not pretty, but it works. Think of it as a nicotine patch.

### Phase 2: Convert Leaf Components

Start with components that don't have children. No props drilling, no context, just simple components:

**Before (React):**
```jsx
function Button({ onClick, disabled, loading, children }) {
  return (
    <button
      onClick={onClick}
      disabled={disabled || loading}
      className={loading ? 'loading' : ''}
    >
      {loading ? <Spinner /> : children}
    </button>
  )
}
```

**After (Vue):**
```vue
<script setup>
defineProps(['disabled', 'loading'])
defineEmits(['click'])
</script>

<template>
  <button
    @click="$emit('click')"
    :disabled="disabled || loading"
    :class="{ loading }"
  >
    <Spinner v-if="loading" />
    <slot v-else />
  </button>
</template>
```

Same functionality. Half the code. No hooks. No memoization. Just a component.

### Phase 3: Move Pages/Routes

Once you've converted enough components, move entire routes to Vue:

```javascript
// router.js
import { createRouter } from 'vue-router'

const router = createRouter({
  routes: [
    { path: '/old-dashboard', component: ReactBridge }, // Still React
    { path: '/dashboard', component: VueDashboard }      // New hotness
  ]
})
```

### Phase 4: Context to Provide/Inject

React Context → Vue Provide/Inject is nearly 1:1:

**Before:**
```jsx
const ThemeContext = React.createContext()

function App() {
  const [theme, setTheme] = useState('dark')
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <Dashboard />
    </ThemeContext.Provider>
  )
}

function Dashboard() {
  const { theme } = useContext(ThemeContext)
  return <div className={theme}>...</div>
}
```

**After:**
```vue
<!-- App.vue -->
<script setup>
import { provide, ref } from 'vue'

const theme = ref('dark')
provide('theme', theme)
</script>

<!-- Dashboard.vue -->
<script setup>
import { inject } from 'vue'

const theme = inject('theme')
</script>

<template>
  <div :class="theme">...</div>
</template>
```

Same concept. Less ceremony. No Context.Provider wrapper components.

## The Composition API: Hooks Done Right

If you loved the idea of hooks but hated the rules, Composition API is for you:

**React hooks:**
```jsx
function useWindowSize() {
  const [size, setSize] = useState({ width: 0, height: 0 })

  useEffect(() => {
    const handleResize = () => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight
      })
    }

    window.addEventListener('resize', handleResize)
    handleResize()

    return () => window.removeEventListener('resize', handleResize)
  }, []) // Don't forget the dependency array!

  return size
}
```

**Vue composables:**
```javascript
import { ref, onMounted, onUnmounted } from 'vue'

function useWindowSize() {
  const width = ref(0)
  const height = ref(0)

  function handleResize() {
    width.value = window.innerWidth
    height.value = window.innerHeight
  }

  onMounted(() => {
    window.addEventListener('resize', handleResize)
    handleResize()
  })

  onUnmounted(() => {
    window.removeEventListener('resize', handleResize)
  })

  return { width, height }
}
```

Look familiar? It's the same pattern. But notice:
- No dependency array
- Lifecycle hooks instead of useEffect mystery box
- No "stale closure" gotchas
- No "Rules of Hooks"
- You can call it in a loop if you want (don't, but you *could*)

## The Stuff That Just Works

### Computed Values

**React:**
```jsx
const expensiveValue = useMemo(() => {
  return items
    .filter(item => item.active)
    .reduce((sum, item) => sum + item.value, 0)
}, [items]) // Don't forget dependencies!
```

**Vue:**
```javascript
import { computed } from 'vue'

const expensiveValue = computed(() =>
  items.value
    .filter(item => item.active)
    .reduce((sum, item) => sum + item.value, 0)
)
```

No dependency array. It tracks dependencies automatically. It's cached. It only recomputes when needed. It just *works*.

### Watchers

**React:**
```jsx
useEffect(() => {
  if (userId) {
    fetchUserData(userId)
  }
}, [userId]) // What about fetchUserData? Should that be in the array?
```

**Vue:**
```javascript
import { watch } from 'vue'

watch(userId, (newId) => {
  if (newId) {
    fetchUserData(newId)
  }
})
```

It watches `userId`. When it changes, the callback runs. No array. No rules. No existential questions about function identity.

## Real Talk: The Trade-offs

Vue isn't perfect. Nothing is. Here's what you're trading:

**You lose:**
- "It's just JavaScript" bragging rights
- The world's largest ecosystem (React is bigger)
- Some cutting-edge experimental features that show up in React first
- The ability to complain about useEffect at parties

**You gain:**
- Reactivity that makes sense
- Templates that are readable
- Actual CSS scoping
- State management that doesn't require a PhD
- Your sanity
- Time with your family
- The ability to read your own code later

## When Vue Clicks

You'll know Vue has clicked when:

1. You stop reaching for useMemo
2. You write a watcher and it just works
3. Your designer changes CSS and nothing breaks
4. You realize you haven't thought about dependency arrays in weeks
5. You catch yourself writing more features and less infrastructure

## What About React?

Here's the thing: React isn't bad at what it was designed for—large applications where Facebook's specific constraints matter. Facebook needed something that could handle their scale, their build system, their everything.

You are not Facebook.

You don't have 50,000 components. You don't have specialized build tooling teams. You don't have "move fast and let the framework gaslight you" as a company value.

You have features to ship, bugs to fix, and a life to live.

Vue remembers that.

---

*"I spent six months learning React. I spent two days learning Vue. Vue took two days because I kept waiting for the other shoe to drop. It never did."*
— A Recovering React Developer

**Up Next:** [Chapter 3: Svelte - The Compiler Will See You Now](03-svelte.md)
