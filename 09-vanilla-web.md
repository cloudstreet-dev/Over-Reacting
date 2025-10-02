---
layout: default
---

# Chapter 9: Vanilla Web Standards - The Framework Was Inside You All Along

## What If You Don't Need Any Framework?

Here's a thought experiment: What if React, Vue, Svelte, and all the frameworks are solving problems you don't have?

What if the browser already has everything you need?

"Impossible!" your React brain screams. "I need components! I need state! I need reactivity!"

The browser: "I have components. I have state. I have events."

Let me introduce you to... the web platform. It's been here the whole time.

## Web Components: No Library Required

```javascript
class CounterElement extends HTMLElement {
  constructor() {
    super()
    this.attachShadow({ mode: 'open' })
    this.count = 0
  }

  connectedCallback() {
    this.render()
    this.shadowRoot.querySelector('button').addEventListener('click', () => {
      this.count++
      this.render()
    })
  }

  render() {
    this.shadowRoot.innerHTML = `
      <style>
        button {
          padding: 8px 16px;
          font-size: 16px;
          cursor: pointer;
        }
      </style>
      <button>Count: ${this.count}</button>
    `
  }
}

customElements.define('counter-element', CounterElement)
```

Usage:

```html
<counter-element></counter-element>
```

No React. No Vue. No Svelte. No build step. No npm install. Just JavaScript and web standards.

It works in every modern browser. Forever. Because it's a web standard.

## querySelector: The Original Component Selection

React has `useRef`:

```jsx
function MyComponent() {
  const inputRef = useRef(null)

  useEffect(() => {
    inputRef.current.focus()
  }, [])

  return <input ref={inputRef} />
}
```

The browser has `querySelector`:

```javascript
document.querySelector('input').focus()
```

That's it. No ref. No useEffect. No hook rules.

### Event Delegation

React re-invents event handling:

```jsx
<button onClick={handleClick}>Click me</button>
```

The browser has event listeners:

```javascript
document.querySelector('button').addEventListener('click', (e) => {
  console.log('Clicked!', e)
})
```

Or use event delegation for dynamic content:

```javascript
document.addEventListener('click', (e) => {
  if (e.target.matches('.delete-button')) {
    e.target.closest('.item').remove()
  }
})
```

One listener handles all delete buttons, even ones added dynamically.

## Template Elements: Reusable HTML

React:
```jsx
function UserCard({ user }) {
  return (
    <div className="card">
      <h3>{user.name}</h3>
      <p>{user.email}</p>
    </div>
  )
}
```

Vanilla:
```html
<template id="user-card-template">
  <div class="card">
    <h3 class="name"></h3>
    <p class="email"></p>
  </div>
</template>

<script>
  function createUserCard(user) {
    const template = document.getElementById('user-card-template')
    const clone = template.content.cloneNode(true)

    clone.querySelector('.name').textContent = user.name
    clone.querySelector('.email').textContent = user.email

    return clone
  }

  // Usage
  const card = createUserCard({ name: 'Alice', email: 'alice@example.com' })
  document.body.appendChild(card)
</script>
```

No JSX. No build step. Just HTML and JavaScript.

## Fetch API: No Axios Needed

React (with axios):
```jsx
import axios from 'axios'

function UserProfile() {
  const [user, setUser] = useState(null)

  useEffect(() => {
    axios.get('/api/user').then(res => setUser(res.data))
  }, [])

  // ...
}
```

Vanilla:
```javascript
fetch('/api/user')
  .then(res => res.json())
  .then(user => {
    document.querySelector('.user-name').textContent = user.name
  })
```

The Fetch API is built in. It works great.

### Async/Await

```javascript
async function loadUser() {
  const response = await fetch('/api/user')
  const user = await response.json()

  document.querySelector('.user-name').textContent = user.name
  document.querySelector('.user-email').textContent = user.email
}

loadUser()
```

Clean. Simple. No library.

## FormData: Forms Without State Management

React forms:
```jsx
function ContactForm() {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    message: ''
  })

  const handleChange = (e) => {
    setFormData({
      ...formData,
      [e.target.name]: e.target.value
    })
  }

  const handleSubmit = (e) => {
    e.preventDefault()
    fetch('/api/contact', {
      method: 'POST',
      body: JSON.stringify(formData)
    })
  }

  return (
    <form onSubmit={handleSubmit}>
      <input name="name" value={formData.name} onChange={handleChange} />
      <input name="email" value={formData.email} onChange={handleChange} />
      <textarea name="message" value={formData.message} onChange={handleChange} />
      <button type="submit">Send</button>
    </form>
  )
}
```

Vanilla:
```html
<form id="contact-form">
  <input name="name" required>
  <input name="email" type="email" required>
  <textarea name="message" required></textarea>
  <button type="submit">Send</button>
</form>

<script>
  document.getElementById('contact-form').addEventListener('submit', async (e) => {
    e.preventDefault()

    const formData = new FormData(e.target)

    await fetch('/api/contact', {
      method: 'POST',
      body: formData
    })

    alert('Sent!')
    e.target.reset()
  })
</script>
```

No state. No onChange handlers. Just FormData.

Need JSON instead?

```javascript
const data = Object.fromEntries(new FormData(e.target))

await fetch('/api/contact', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(data)
})
```

## LocalStorage: Client-Side State

React + Redux for storing a theme preference:
```jsx
// action creators, reducers, store setup...
// 50 lines later...

const theme = useSelector(state => state.theme)
const dispatch = useDispatch()
```

Vanilla:
```javascript
// Save
localStorage.setItem('theme', 'dark')

// Load
const theme = localStorage.getItem('theme') || 'light'

// Apply
document.body.classList.toggle('dark-theme', theme === 'dark')
```

Need reactivity? Dispatch a custom event:

```javascript
function setTheme(theme) {
  localStorage.setItem('theme', theme)
  window.dispatchEvent(new CustomEvent('theme-changed', { detail: theme }))
}

window.addEventListener('theme-changed', (e) => {
  document.body.classList.toggle('dark-theme', e.detail === 'dark')
})
```

## Intersection Observer: Lazy Loading Without Libraries

React lazy loading:
```jsx
import { useEffect, useRef, useState } from 'react'
import { useInView } from 'react-intersection-observer' // npm install

function LazyImage({ src }) {
  const { ref, inView } = useInView({ triggerOnce: true })

  return (
    <div ref={ref}>
      {inView && <img src={src} />}
    </div>
  )
}
```

Vanilla:
```javascript
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const img = entry.target
      img.src = img.dataset.src
      observer.unobserve(img)
    }
  })
})

document.querySelectorAll('img[data-src]').forEach(img => {
  observer.observe(img)
})
```

```html
<img data-src="/large-image.jpg" alt="Lazy loaded">
```

Built into browsers. No library needed.

## CSS Variables: Dynamic Styling

React CSS-in-JS:
```jsx
import styled from 'styled-components'

const Button = styled.button`
  background: ${props => props.theme.primary};
  color: ${props => props.theme.text};
`
```

Vanilla CSS Variables:
```css
:root {
  --color-primary: #007bff;
  --color-text: #333;
}

button {
  background: var(--color-primary);
  color: var(--color-text);
}
```

Change them with JavaScript:
```javascript
document.documentElement.style.setProperty('--color-primary', '#ff0000')
```

Theme switching:
```javascript
function setTheme(theme) {
  const root = document.documentElement

  if (theme === 'dark') {
    root.style.setProperty('--color-primary', '#64b5f6')
    root.style.setProperty('--color-text', '#ffffff')
    root.style.setProperty('--color-bg', '#1e1e1e')
  } else {
    root.style.setProperty('--color-primary', '#007bff')
    root.style.setProperty('--color-text', '#333333')
    root.style.setProperty('--color-bg', '#ffffff')
  }
}
```

No CSS-in-JS library. Just web standards.

## Modules: Code Splitting Without Webpack

React code splitting:
```jsx
const Dashboard = React.lazy(() => import('./Dashboard'))

<Suspense fallback={<Loading />}>
  <Dashboard />
</Suspense>
```

Vanilla ES Modules:
```javascript
// main.js
const button = document.querySelector('#load-dashboard')

button.addEventListener('click', async () => {
  const { Dashboard } = await import('./dashboard.js')
  const dashboard = new Dashboard()
  document.body.appendChild(dashboard.render())
})
```

```javascript
// dashboard.js
export class Dashboard {
  render() {
    const div = document.createElement('div')
    div.innerHTML = '<h1>Dashboard</h1>'
    return div
  }
}
```

The browser handles the code splitting. No bundler required.

## History API: Routing Without React Router

```javascript
// Simple router
class Router {
  constructor(routes) {
    this.routes = routes

    window.addEventListener('popstate', () => this.handleRoute())

    document.addEventListener('click', (e) => {
      if (e.target.matches('[data-link]')) {
        e.preventDefault()
        this.navigate(e.target.href)
      }
    })

    this.handleRoute()
  }

  navigate(url) {
    history.pushState(null, null, url)
    this.handleRoute()
  }

  handleRoute() {
    const path = window.location.pathname
    const route = this.routes[path] || this.routes['/404']
    route()
  }
}

// Usage
const router = new Router({
  '/': () => {
    document.querySelector('#app').innerHTML = '<h1>Home</h1>'
  },
  '/about': () => {
    document.querySelector('#app').innerHTML = '<h1>About</h1>'
  },
  '/404': () => {
    document.querySelector('#app').innerHTML = '<h1>Not Found</h1>'
  }
})
```

```html
<nav>
  <a href="/" data-link>Home</a>
  <a href="/about" data-link>About</a>
</nav>

<div id="app"></div>
```

Single-page routing. No React Router. 20 lines of code.

## Real-World Example: Todo App

The classic. No framework edition.

```html
<!DOCTYPE html>
<html>
<head>
  <style>
    .completed { text-decoration: line-through; opacity: 0.6; }
  </style>
</head>
<body>
  <h1>Todos</h1>

  <form id="add-todo">
    <input name="text" placeholder="What needs to be done?" required>
    <button type="submit">Add</button>
  </form>

  <ul id="todo-list"></ul>

  <template id="todo-template">
    <li>
      <input type="checkbox" class="toggle">
      <span class="text"></span>
      <button class="delete">×</button>
    </li>
  </template>

  <script>
    class TodoApp {
      constructor() {
        this.todos = JSON.parse(localStorage.getItem('todos') || '[]')
        this.render()

        document.getElementById('add-todo').addEventListener('submit', (e) => {
          e.preventDefault()
          this.addTodo(new FormData(e.target).get('text'))
          e.target.reset()
        })

        document.getElementById('todo-list').addEventListener('click', (e) => {
          if (e.target.matches('.delete')) {
            const id = e.target.closest('li').dataset.id
            this.deleteTodo(id)
          }
        })

        document.getElementById('todo-list').addEventListener('change', (e) => {
          if (e.target.matches('.toggle')) {
            const id = e.target.closest('li').dataset.id
            this.toggleTodo(id)
          }
        })
      }

      addTodo(text) {
        this.todos.push({ id: Date.now(), text, completed: false })
        this.save()
        this.render()
      }

      deleteTodo(id) {
        this.todos = this.todos.filter(t => t.id != id)
        this.save()
        this.render()
      }

      toggleTodo(id) {
        const todo = this.todos.find(t => t.id == id)
        todo.completed = !todo.completed
        this.save()
        this.render()
      }

      save() {
        localStorage.setItem('todos', JSON.stringify(this.todos))
      }

      render() {
        const list = document.getElementById('todo-list')
        const template = document.getElementById('todo-template')

        list.innerHTML = ''

        this.todos.forEach(todo => {
          const clone = template.content.cloneNode(true)
          const li = clone.querySelector('li')

          li.dataset.id = todo.id
          li.querySelector('.text').textContent = todo.text
          li.querySelector('.toggle').checked = todo.completed

          if (todo.completed) {
            li.classList.add('completed')
          }

          list.appendChild(clone)
        })
      }
    }

    new TodoApp()
  </script>
</body>
</html>
```

Full todo app. Add, delete, toggle, persist to localStorage. Zero dependencies. One HTML file.

Try building that in React without npm install.

## When Vanilla Makes Sense

Vanilla JavaScript is perfect for:

- **Small projects** - The framework overhead isn't worth it
- **Learning** - Understand how things actually work
- **Performance-critical apps** - No framework overhead
- **Long-term projects** - No framework upgrades, no dependencies
- **Progressive enhancement** - Start with HTML, enhance with JS
- **Environments without build tools** - Just works

Vanilla might not be ideal for:

- **Large, complex SPAs** - Frameworks provide helpful structure
- **Teams** - Shared patterns and conventions help
- **Rapid prototyping** - Frameworks can be faster initially
- **When you need ecosystem libraries** - React has more pre-built components

## The Performance Story

Let's compare a simple interactive widget:

**React:**
- React runtime: 42KB (gzipped)
- ReactDOM: 130KB (gzipped)
- Your component: 2KB
- Total: 174KB

**Vanilla:**
- Your code: 2KB
- Total: 2KB

That's 87x smaller.

First load time:
- React: Download, parse, compile, execute framework, then your code
- Vanilla: Execute your code

Vanilla wins every time for simple interactions.

## Real Talk: When to Use Vanilla

The web platform has come a long way. Modern browsers support:
- Custom Elements (Web Components)
- ES Modules
- Fetch API
- Async/Await
- Intersection Observer
- Mutation Observer
- CSS Variables
- Template elements
- FormData
- LocalStorage
- History API

You don't always need a framework.

Sometimes, the framework was inside you (well, inside your browser) all along.

---

*"I spent a month learning React. Then I learned vanilla JavaScript could do most of it in 20 lines. I felt betrayed and enlightened simultaneously."*
— A Developer Who Read MDN

**Up Next:** [Chapter 10: Astro - Content Sites Don't Need Virtual DOMs](10-astro.md)

