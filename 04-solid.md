---
layout: default
---

# Chapter 4: Solid.js - React's Cooler Younger Sibling

## JSX Without the Baggage

"I actually like JSX," you say sheepishly, like you're confessing to a crime.

That's fine! JSX isn't the problem. JSX is great. The problem is what React does with it.

Solid looked at React and said: "What if we kept the JSX, but fixed literally everything else?"

```jsx
import { createSignal } from 'solid-js'

function Counter() {
  const [count, setCount] = createSignal(0)

  return (
    <button onClick={() => setCount(count() + 1)}>
      Count: {count()}
    </button>
  )
}
```

Look familiar? It should. It's JSX. It's a component function. It even has something that looks like useState.

But here's the twist: this function runs **once**. Not on every render. Just once. Ever.

Your React brain just short-circuited.

## Fine-Grained Reactivity Explained

Remember React's model?

1. State changes
2. Component function re-runs
3. New JSX tree is created
4. Virtual DOM diffing happens
5. Real DOM updates

Now here's Solid's model:

1. State changes
2. DOM updates

That's it. No re-running. No diffing. No virtual DOM.

"But HOW?!"

Solid's compiler turns your JSX into reactive primitives. The `count()` call isn't just reading a value—it's creating a dependency. When `setCount` runs, only the specific text node that depends on `count()` updates.

```jsx
function Counter() {
  const [count, setCount] = createSignal(0)

  console.log('Component function runs!')

  return (
    <button onClick={() => setCount(count() + 1)}>
      Count: {count()}
    </button>
  )
}
```

That console.log runs once. Click the button a thousand times. Still just one log.

The component function is initialization, not render.

Mind. Blown.

## No More useCallback, No More useMemo

In React, you need to memoize everything:

```jsx
function TodoList({ todos }) {
  const [filter, setFilter] = useState('all')

  // Memoize the filtered list
  const filteredTodos = useMemo(() => {
    return todos.filter(todo => {
      if (filter === 'active') return !todo.done
      if (filter === 'completed') return todo.done
      return true
    })
  }, [todos, filter])

  // Memoize the handler
  const handleFilterChange = useCallback((newFilter) => {
    setFilter(newFilter)
  }, [])

  return (
    <div>
      <FilterButtons onChange={handleFilterChange} />
      <TodoItems items={filteredTodos} />
    </div>
  )
}
```

In Solid, you just... write the code:

```jsx
function TodoList(props) {
  const [filter, setFilter] = createSignal('all')

  const filteredTodos = () => {
    return props.todos.filter(todo => {
      if (filter() === 'active') return !todo.done
      if (filter() === 'completed') return todo.done
      return true
    })
  }

  const handleFilterChange = (newFilter) => {
    setFilter(newFilter)
  }

  return (
    <div>
      <FilterButtons onChange={handleFilterChange} />
      <TodoItems items={filteredTodos()} />
    </div>
  )
}
```

No useMemo. No useCallback. No dependency arrays. The filtered function only re-runs when `props.todos` or `filter()` changes because Solid tracks the dependencies automatically.

Functions are just functions. They don't need optimization hints.

## Props Don't Change (In a Good Way)

In React, props are new objects on every render:

```jsx
function Parent() {
  const [count, setCount] = useState(0)

  return (
    <Child
      user={{ name: 'Alice' }} // New object every render!
      onUpdate={() => setCount(c => c + 1)} // New function every render!
    />
  )
}
```

Cue the React.memo, useCallback, and useMemo dance.

In Solid, props are proxy objects with getters:

```jsx
function Parent() {
  const [count, setCount] = createSignal(0)

  return (
    <Child
      user={{ name: 'Alice' }}
      onUpdate={() => setCount(count() + 1)}
    />
  )
}

function Child(props) {
  // props.user is the same object
  // props.onUpdate is the same function
  // But accessing props.user.name creates a dependency
}
```

Props are stable. They don't change. What changes is the reactive values inside them. No memoization needed.

## createEffect: useEffect Done Right

React's useEffect is a footgun:

```jsx
useEffect(() => {
  console.log('Count changed:', count)
}, [count]) // Forget this array? Infinite loop.
```

Solid's createEffect just works:

```jsx
createEffect(() => {
  console.log('Count changed:', count())
})
```

No dependency array. It tracks dependencies automatically by seeing what you call. When `count()` changes, the effect re-runs.

### Cleanup

**React:**
```jsx
useEffect(() => {
  const timer = setInterval(() => {
    console.log('tick')
  }, 1000)

  return () => clearInterval(timer)
}, []) // Empty array for "mount only"
```

**Solid:**
```jsx
createEffect(() => {
  const timer = setInterval(() => {
    console.log('tick')
  }, 1000)

  onCleanup(() => clearInterval(timer))
})
```

The `onCleanup` function registers cleanup code. When the effect re-runs or the component unmounts, cleanup runs first. No weird return function. No dependency array confusion.

## Control Flow: No More && and ?:

React's conditional rendering:

```jsx
function UserGreeting({ user, loading, error }) {
  return (
    <div>
      {loading && <Spinner />}
      {error && <Error message={error} />}
      {!loading && !error && user && (
        <h1>Hello, {user.name}</h1>
      )}
    </div>
  )
}
```

Every branch gets evaluated on every render. Even the ones that aren't shown.

Solid has built-in control flow components:

```jsx
import { Show, Switch, Match } from 'solid-js'

function UserGreeting(props) {
  return (
    <div>
      <Show when={props.loading}>
        <Spinner />
      </Show>

      <Show when={props.error}>
        <Error message={props.error} />
      </Show>

      <Show when={!props.loading && !props.error && props.user}>
        <h1>Hello, {props.user.name}</h1>
      </Show>
    </div>
  )
}
```

Or better yet, with Switch/Match:

```jsx
function UserGreeting(props) {
  return (
    <Switch>
      <Match when={props.loading}>
        <Spinner />
      </Match>

      <Match when={props.error}>
        <Error message={props.error} />
      </Match>

      <Match when={props.user}>
        <h1>Hello, {props.user.name}</h1>
      </Match>
    </Switch>
  )
}
```

Only the active branch exists in the DOM. Only the active branch is reactive. No wasted computations.

## For Loops That Don't Destroy Everything

React's list rendering:

```jsx
function TodoList({ todos }) {
  return (
    <ul>
      {todos.map(todo => (
        <TodoItem key={todo.id} todo={todo} />
      ))}
    </ul>
  )
}
```

When the `todos` array changes, React diffs everything, uses keys to match components, and tries to minimize DOM updates.

Solid's For component:

```jsx
import { For } from 'solid-js'

function TodoList(props) {
  return (
    <ul>
      <For each={props.todos}>
        {(todo) => <TodoItem todo={todo} />}
      </For>
    </ul>
  )
}
```

The `For` component tracks items by reference. When an item changes, only that item updates. When items are reordered, nothing re-renders—the DOM nodes just move. No keys needed (though you can use them for by-value comparison).

### Index Tracking

```jsx
<For each={props.todos}>
  {(todo, index) => (
    <li>
      {index() + 1}. {todo.name}
    </li>
  )}
</For>
```

The `index` is a signal. It only updates when the item's position changes. Beautiful.

## Stores: Nested Reactivity

React's useState is flat:

```jsx
const [user, setUser] = useState({
  name: 'Alice',
  profile: {
    age: 30,
    email: 'alice@example.com'
  }
})

// To update nested state:
setUser(u => ({
  ...u,
  profile: {
    ...u.profile,
    age: 31
  }
}))
```

Immer helps, but it's still ceremony.

Solid's createStore provides nested reactivity:

```jsx
import { createStore } from 'solid-js/store'

const [user, setUser] = createStore({
  name: 'Alice',
  profile: {
    age: 30,
    email: 'alice@example.com'
  }
})

// To update nested state:
setUser('profile', 'age', 31)

// Or with a function:
setUser('profile', 'age', age => age + 1)

// Or multiple levels:
setUser('profile', { age: 31, email: 'new@example.com' })
```

Any component that reads `user.profile.age` will update when it changes. Components that read `user.name` won't. Fine-grained reactivity all the way down.

## Migration from React: Keep the Syntax, Lose the Pain

The migration is surprisingly smooth because the syntax is so similar.

### Converting useState

**React:**
```jsx
const [count, setCount] = useState(0)
setCount(count + 1)
```

**Solid:**
```jsx
const [count, setCount] = createSignal(0)
setCount(count() + 1)
```

Just add parentheses to read the value. That's it.

### Converting useEffect

**React:**
```jsx
useEffect(() => {
  document.title = `Count: ${count}`
}, [count])
```

**Solid:**
```jsx
createEffect(() => {
  document.title = `Count: ${count()}`
})
```

Remove the dependency array, add parentheses to the signal.

### Converting useMemo

**React:**
```jsx
const doubled = useMemo(() => count * 2, [count])
```

**Solid:**
```jsx
const doubled = () => count() * 2
```

Just a function. Or use `createMemo` if the computation is expensive:

```jsx
const doubled = createMemo(() => count() * 2)
```

### Converting useCallback

**React:**
```jsx
const increment = useCallback(() => {
  setCount(c => c + 1)
}, [])
```

**Solid:**
```jsx
const increment = () => {
  setCount(count() + 1)
}
```

Just a function. No hook needed.

### Converting Context

**React:**
```jsx
const ThemeContext = createContext()

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

**Solid:**
```jsx
import { createContext, useContext } from 'solid-js'

const ThemeContext = createContext()

function App() {
  const [theme, setTheme] = createSignal('dark')

  return (
    <ThemeContext.Provider value={[theme, setTheme]}>
      <Dashboard />
    </ThemeContext.Provider>
  )
}

function Dashboard() {
  const [theme] = useContext(ThemeContext)
  return <div classList={{ [theme()]: true }}>...</div>
}
```

Almost identical. The API is intentionally similar.

## The Performance Story

Remember when you had to optimize everything in React?

```jsx
// Preventing unnecessary re-renders
export default React.memo(Component)

// Memoizing computed values
const value = useMemo(() => expensive(data), [data])

// Memoizing callbacks
const handler = useCallback(() => doThing(), [])

// Splitting code to reduce bundle
const Component = React.lazy(() => import('./Component'))
```

In Solid, components don't re-render. Ever. There's nothing to optimize. You write code and it's fast by default.

Benchmarks (js-framework-benchmark):
- React: Pretty good
- React + optimization: Better
- Solid: Faster than optimized React
- Solid + optimization: Unnecessary, but still faster

## When to Use Solid

Solid is perfect for:

- **React refugees who love JSX** - Same syntax, better everything else
- **Performance-critical apps** - Fastest reactive framework
- **Teams that hate optimization work** - Fast by default
- **Complex UIs** - Fine-grained updates shine here
- **Developers who want to understand what's happening** - No magic, just reactivity

Solid might not be ideal for:

- **Teams invested in React ecosystem** - Some libraries don't have Solid equivalents yet
- **Projects that need template-based frameworks** - Use Vue or Svelte instead
- **Developers who refuse to understand signals** - You need to grok the model

## The Learning Curve

If you know React:
- Hour 1: "This is just React!"
- Hour 2: "Wait, components don't re-render?"
- Hour 3: "Where's useCallback? Oh, I don't need it."
- Hour 4: "This is SO FAST"
- Day 2: "Can I convert my entire React codebase?"

The syntax similarity is a feature. You can be productive immediately. The conceptual differences make you more productive over time.

## Real Talk: Solid vs React

**React says:** "Components are functions that return JSX and re-run when state changes."

**Solid says:** "Components are functions that return JSX and run once to set up reactive bindings."

Same syntax. Opposite execution models.

React optimizes by minimizing re-renders through diffing.
Solid optimizes by never re-rendering at all.

One approach needs constant vigilance and optimization.
The other is fast by default.

Choose accordingly.

---

*"I switched from React to Solid and my app got faster, my code got clearer, and I stopped having stress dreams about dependency arrays."*
— A Developer Who Stopped Re-Rendering

**Up Next:** [Chapter 5: Angular - The Framework With Opinions (And a Therapist)](05-angular.md)
