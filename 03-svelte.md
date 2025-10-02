# Chapter 3: Svelte - The Compiler Will See You Now

## What If Your Framework Just... Disappeared?

Pop quiz: What's better than a fast framework?

*No framework.*

"But wait," your React brain protests, "we NEED a framework! For reactivity! For component lifecycle! For—"

What if I told you the framework could do its job at *build time* and then get out of the way?

That's Svelte. It's not a framework you ship to your users. It's a compiler that turns your components into highly optimized vanilla JavaScript. The framework disappears. Poof. Gone. Like it was never there.

Your bundle? Tiny.
Your runtime? Fast.
Your mind? Blown.

## No Virtual DOM, No Problem

Remember when React explained the virtual DOM?

"We create a virtual representation of the DOM in JavaScript, then we diff it with the previous version, then we calculate the minimal set of changes, then we batch them, then we update the real DOM."

And you nodded along, thinking this sounded reasonable.

It's not. It's a workaround.

Svelte looked at this and said "what if we just... knew exactly what changed?"

```svelte
<script>
  let count = 0
</script>

<button on:click={() => count++}>
  Clicks: {count}
</button>
```

When you click the button, Svelte doesn't:
- Re-run the component function
- Build a virtual DOM
- Diff the virtual DOM
- Calculate what changed
- Update the real DOM

Svelte just updates the text node. That's it. Because the compiler looked at your code and said "oh, when `count` changes, update this specific text node" and generated code to do exactly that.

No runtime overhead. No reconciliation. No "React is working" message in DevTools.

Just surgical DOM updates.

## Reactivity Without the Ceremony

**React's reactivity:**
```jsx
const [count, setCount] = useState(0)
const [doubled, setDoubled] = useState(0)

useEffect(() => {
  setDoubled(count * 2)
}, [count])

// Or with useMemo
const doubled = useMemo(() => count * 2, [count])
```

"Don't forget the dependency array! And yes, you need a whole different hook for computed values. Why? Because that's just how it works. Stop asking questions."

**Svelte's reactivity:**
```svelte
<script>
  let count = 0
  $: doubled = count * 2
</script>
```

That `$:` is a label. A JavaScript label. Svelte's compiler sees it and makes it reactive. When `count` changes, `doubled` updates. No hooks. No arrays. No rules.

"But that's using a weird JavaScript feature!"

Yes. A feature that's been in JavaScript since 1995. Svelte just made it useful.

### Reactive Statements

You can make any statement reactive:

```svelte
<script>
  let firstName = ''
  let lastName = ''

  $: fullName = `${firstName} ${lastName}`
  $: console.log('Full name changed:', fullName)

  $: if (fullName.length > 20) {
    console.log('That is a long name!')
  }
</script>
```

When `firstName` or `lastName` change, `fullName` recalculates, the log runs, and the conditional checks. The compiler figures out the dependencies automatically.

No hooks. No dependency arrays. No useEffect. No crying.

## The Component Model

**React component:**
```jsx
function Counter({ initialCount = 0 }) {
  const [count, setCount] = useState(initialCount)

  const increment = useCallback(() => {
    setCount(c => c + 1)
  }, [])

  useEffect(() => {
    console.log('Count changed:', count)
  }, [count])

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+1</button>
    </div>
  )
}
```

**Svelte component:**
```svelte
<script>
  export let initialCount = 0

  let count = initialCount

  function increment() {
    count++
  }

  $: console.log('Count changed:', count)
</script>

<p>Count: {count}</p>
<button on:click={increment}>+1</button>
```

Same functionality. Way less code. No hooks. No useCallback. No function-that-returns-JSX mental model.

Just: script, markup, done.

### Props

**React:**
```jsx
// Props are function parameters
// Optional props need default values
// TypeScript types are separate
// Destructuring is common but breaks default values sometimes

function Button({
  variant = 'primary',
  disabled = false,
  onClick,
  children
}) {
  // ...
}
```

**Svelte:**
```svelte
<script lang="ts">
  // Props are exported variables
  export let variant: 'primary' | 'secondary' = 'primary'
  export let disabled = false
  export let onClick: () => void
</script>

<button class={variant} {disabled} on:click={onClick}>
  <slot />
</button>
```

Props are just variables you export. Defaults work like normal variable defaults. TypeScript integration is seamless. Slots are like children but better (more on that later).

## Stores: State Management That Doesn't Require a Support Group

**React + Context + useReducer:**
```jsx
const CounterContext = React.createContext()

function counterReducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 }
    case 'decrement':
      return { count: state.count - 1 }
    default:
      return state
  }
}

function CounterProvider({ children }) {
  const [state, dispatch] = useReducer(counterReducer, { count: 0 })
  return (
    <CounterContext.Provider value={{ state, dispatch }}>
      {children}
    </CounterContext.Provider>
  )
}

function Counter() {
  const { state, dispatch } = useContext(CounterContext)
  return (
    <button onClick={() => dispatch({ type: 'increment' })}>
      {state.count}
    </button>
  )
}
```

**Svelte stores:**
```javascript
// stores.js
import { writable } from 'svelte/store'

export const count = writable(0)
```

```svelte
<!-- Counter.svelte -->
<script>
  import { count } from './stores.js'
</script>

<button on:click={() => $count++}>
  {$count}
</button>
```

That `$` prefix automatically subscribes to the store, gets its value, and unsubscribes when the component unmounts. No Provider. No Context. No hooks. Just a dollar sign.

### Derived Stores

```javascript
import { writable, derived } from 'svelte/store'

export const count = writable(0)
export const doubled = derived(count, $count => $count * 2)
```

```svelte
<script>
  import { count, doubled } from './stores.js'
</script>

<p>Count: {$count}</p>
<p>Doubled: {$doubled}</p>
```

When `count` changes, `doubled` updates automatically. It's like useMemo but you don't have to wrap everything in hooks or remember dependency arrays.

### Custom Stores

You can make stores with custom logic:

```javascript
function createCounter() {
  const { subscribe, update } = writable(0)

  return {
    subscribe,
    increment: () => update(n => n + 1),
    decrement: () => update(n => n - 1),
    reset: () => update(() => 0)
  }
}

export const counter = createCounter()
```

```svelte
<script>
  import { counter } from './stores.js'
</script>

<button on:click={counter.decrement}>-</button>
<span>{$counter}</span>
<button on:click={counter.increment}>+</button>
<button on:click={counter.reset}>Reset</button>
```

No classes. No Redux. No ceremony. Just functions that return objects with a `subscribe` method.

## Transitions & Animations: The Good Kind of Magic

**React animations:**
```jsx
// Install react-spring
// Or framer-motion
// Or react-transition-group
// Read 50 pages of docs
// Fight with CSS
// Give up
// Use setTimeout like an animal
```

**Svelte animations:**
```svelte
<script>
  import { fade, fly, slide } from 'svelte/transition'

  let visible = true
</script>

<button on:click={() => visible = !visible}>
  Toggle
</button>

{#if visible}
  <p transition:fade>Fades in and out</p>
  <p transition:fly={{ y: 200 }}>Flies in and out</p>
  <p transition:slide>Slides in and out</p>
{/if}
```

Built in. Works perfectly. No dependencies. No fighting with CSS. Just import, use, done.

### List Animations

```svelte
<script>
  import { flip } from 'svelte/animate'
  import { fade } from 'svelte/transition'

  let items = [1, 2, 3, 4, 5]

  function shuffle() {
    items = items.sort(() => Math.random() - 0.5)
  }
</script>

<button on:click={shuffle}>Shuffle</button>

{#each items as item (item)}
  <div animate:flip transition:fade>
    {item}
  </div>
{/each}
```

The `animate:flip` smoothly animates items when the list order changes. The `transition:fade` handles enter/exit. That's it.

Try doing that in React without a library. Go ahead. I'll wait.

## Migration Strategies: From React to Svelte

### Strategy 1: The Component Swap

Start by converting leaf components—things like buttons, inputs, cards:

**Before (React):**
```jsx
export function Card({ title, children, className }) {
  return (
    <div className={`card ${className || ''}`}>
      <h3>{title}</h3>
      <div className="card-body">
        {children}
      </div>
    </div>
  )
}
```

**After (Svelte):**
```svelte
<script>
  export let title
  export let className = ''
</script>

<div class="card {className}">
  <h3>{title}</h3>
  <div class="card-body">
    <slot />
  </div>
</div>
```

Use them together with a bridge:

```jsx
// ReactSveltebridge.jsx
import { mount } from 'svelte'

export function useSvelteComponent(Component, props) {
  const ref = useRef(null)

  useEffect(() => {
    if (ref.current) {
      const component = mount(Component, {
        target: ref.current,
        props
      })
      return () => component.$destroy()
    }
  }, [props])

  return ref
}
```

### Strategy 2: New Features in Svelte

Like with Vue, you can write new features in Svelte while keeping old ones in React. SvelteKit plays nice with existing backends.

### Strategy 3: The Page-by-Page Rewrite

Move entire pages to SvelteKit:

```javascript
// routes/dashboard/+page.svelte
<script>
  // SvelteKit page
</script>

// routes/old-dashboard/+page.jsx
// Still using React
```

### Converting Hooks to Svelte Patterns

**useState:**
```jsx
const [count, setCount] = useState(0)
```
```svelte
let count = 0
```

**useEffect:**
```jsx
useEffect(() => {
  console.log('count changed')
}, [count])
```
```svelte
$: console.log('count changed', count)
```

**useMemo:**
```jsx
const doubled = useMemo(() => count * 2, [count])
```
```svelte
$: doubled = count * 2
```

**useCallback:**
```jsx
const increment = useCallback(() => {
  setCount(c => c + 1)
}, [])
```
```svelte
function increment() {
  count++
}
```

Notice a pattern? Everything gets simpler.

## The Bundle Size Moment

Remember earlier when I mentioned bundle size? Let's talk numbers.

**A simple counter app:**
- React + ReactDOM: ~140KB (minified, gzipped)
- Svelte: ~2KB (minified, gzipped)

**A real app (e-commerce site):**
- React version: 380KB JavaScript
- Svelte version: 45KB JavaScript

That's not a typo. The Svelte version shipped 1/8th the JavaScript. Same features. Same functionality. Just 335KB less framework overhead.

Your users' phones will thank you.

## When to Use Svelte

Svelte is phenomenal for:

- **Anything user-facing** - Smaller bundles = faster loads = happier users
- **Animations & transitions** - Built-in and beautiful
- **Prototypes** - Less code = faster iteration
- **Teaching** - Beginners understand Svelte in hours, not weeks
- **Literally everything React does but with 1/10th the code**

Svelte might not be ideal for:

- Teams that really, really, really love JSX (try Solid instead)
- Projects that need every possible third-party library (React's ecosystem is bigger)
- People who measure success by bundle size (wait, that's a reason TO use Svelte)

## The Aha Moment

You'll know Svelte has clicked when:

1. You forget the framework exists and just write code
2. You check your bundle size and laugh
3. You add an animation without installing anything
4. You realize you haven't thought about re-renders in days
5. You catch yourself saying "it just works" unironically

## The Trade-offs

**You lose:**
- The virtual DOM (you won't miss it)
- Massive ecosystem (though Svelte's is growing fast)
- Extremely broad hiring pool (fewer Svelte devs than React)

**You gain:**
- Tiny bundles
- Blazing performance
- Code that reads like what it does
- Built-in animations
- The ability to ship features instead of optimizing renders
- Your users' battery life back

## Real Talk: Svelte vs React

React asks: "How do we make JavaScript apps manageable at Facebook scale?"
Svelte asks: "How do we make building websites delightful?"

Different questions. Different answers.

If you're building Facebook, use React.
If you're building literally anything else, maybe try the one that prioritizes your users and your sanity.

---

*"I thought the virtual DOM was necessary. Turns out it was just all I knew."*
— A Developer Who Checked Their Bundle Size

**Up Next:** [Chapter 4: Solid - React's Cooler Younger Sibling](04-solid.md)
