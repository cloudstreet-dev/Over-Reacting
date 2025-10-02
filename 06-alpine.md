---
layout: default
---

# Chapter 6: Alpine.js - When You Just Need to Sprinkle Some Magic

## jQuery's Cool Nephew Who Went to College

Remember jQuery?

```javascript
$(document).ready(function() {
  $('.button').on('click', function() {
    $(this).toggleClass('active')
    $('.menu').slideToggle()
  })
})
```

Simple. Direct. No build step. Just include a script tag and go.

Then React came along and said "you need to npm install 47 packages, configure webpack, set up babel, learn JSX, understand the virtual DOM, and restructure your entire application to toggle a class."

For a dropdown menu.

Alpine.js looked at this and said "what if we kept the simplicity of jQuery but made it reactive and modern?"

```html
<div x-data="{ open: false }">
  <button @click="open = !open">Toggle</button>
  <div x-show="open" class="menu">
    Menu content
  </div>
</div>
```

No build step. No npm. No webpack. Just a script tag and attributes. Like the good old days, but not terrible.

## No Build Step, No npm install, No Tears

**React project setup:**
```bash
npx create-react-app my-app
cd my-app
npm install
# Wait 5 minutes
# Get coffee
# 200MB of node_modules later
npm start
# Wait for webpack
# Finally ready
```

**Alpine project setup:**
```html
<!DOCTYPE html>
<html>
<head>
  <script defer src="https://cdn.jsdelivr.net/npm/alpinejs@3.x.x/dist/cdn.min.js"></script>
</head>
<body>
  <!-- You're ready -->
</body>
</html>
```

That's it. You're done. Start coding.

15KB. No build step. No dependency hell. Just a script tag like it's 2010, except actually good.

## Reactivity in HTML Attributes

**React approach:**
```jsx
function Dropdown() {
  const [open, setOpen] = useState(false)

  return (
    <div>
      <button onClick={() => setOpen(!open)}>
        Toggle
      </button>
      {open && (
        <div className="menu">
          <a href="/profile">Profile</a>
          <a href="/settings">Settings</a>
          <a href="/logout">Logout</a>
        </div>
      )}
    </div>
  )
}

// Now configure babel, webpack, the runtime...
```

**Alpine approach:**
```html
<div x-data="{ open: false }">
  <button @click="open = !open">
    Toggle
  </button>

  <div x-show="open" class="menu">
    <a href="/profile">Profile</a>
    <a href="/settings">Settings</a>
    <a href="/logout">Logout</a>
  </div>
</div>
```

Same functionality. Zero JavaScript files. Zero build configuration. Just HTML with superpowers.

## The Directives

Alpine uses Vue-inspired directives:

### x-data: Component Scope

```html
<div x-data="{ count: 0, name: 'Alice' }">
  <!-- count and name are available here -->
</div>
```

Everything inside has access to that data. It's like a tiny component.

### x-show / x-if: Conditional Rendering

```html
<!-- x-show: toggles visibility (display: none) -->
<div x-data="{ visible: true }">
  <div x-show="visible">I can be toggled</div>
</div>

<!-- x-if: removes from DOM entirely -->
<template x-if="user.loggedIn">
  <div>Welcome back!</div>
</template>
```

### x-for: Loops

```html
<div x-data="{ items: ['one', 'two', 'three'] }">
  <template x-for="item in items">
    <li x-text="item"></li>
  </template>
</div>
```

### x-model: Two-Way Binding

```html
<div x-data="{ message: '' }">
  <input type="text" x-model="message">
  <p>You typed: <span x-text="message"></span></p>
</div>
```

No onChange handlers. No controlled components. Just works.

### x-on (@): Event Handling

```html
<button @click="count++">Increment</button>
<button @click="doSomething($event)">Click me</button>

<!-- Modifiers work too -->
<form @submit.prevent="handleSubmit">
  <input @keydown.enter="search">
  <input @click.outside="close">
</form>
```

### x-bind (:): Attribute Binding

```html
<div x-data="{ color: 'red' }">
  <div :class="color">Colored div</div>
  <input :disabled="loading">
  <img :src="imageUrl">
</div>
```

## When React Is 300KB of Overkill

You know that project where you used React for:
- A modal
- A dropdown
- A form with validation
- An accordion

And it felt like using a nuclear reactor to make toast?

Alpine is the toaster.

### Example: Modal

**React version:**
```jsx
// Modal.jsx
import { useState } from 'react'
import ReactDOM from 'react-dom'

function Modal({ children, trigger }) {
  const [isOpen, setIsOpen] = useState(false)

  return (
    <>
      <button onClick={() => setIsOpen(true)}>
        {trigger}
      </button>

      {isOpen && ReactDOM.createPortal(
        <div className="modal-overlay" onClick={() => setIsOpen(false)}>
          <div className="modal" onClick={e => e.stopPropagation()}>
            {children}
            <button onClick={() => setIsOpen(false)}>Close</button>
          </div>
        </div>,
        document.body
      )}
    </>
  )
}

// Now bundle it, ship it, pray it doesn't break
```

**Alpine version:**
```html
<div x-data="{ open: false }">
  <button @click="open = true">Open Modal</button>

  <div x-show="open"
       @click="open = false"
       class="modal-overlay">
    <div @click.stop class="modal">
      <h2>Modal Content</h2>
      <button @click="open = false">Close</button>
    </div>
  </div>
</div>
```

No components. No portals. No build step. Just HTML that works.

### Example: Tabs

```html
<div x-data="{ tab: 'home' }">
  <nav>
    <button @click="tab = 'home'" :class="{ active: tab === 'home' }">
      Home
    </button>
    <button @click="tab = 'profile'" :class="{ active: tab === 'profile' }">
      Profile
    </button>
    <button @click="tab = 'settings'" :class="{ active: tab === 'settings' }">
      Settings
    </button>
  </nav>

  <div x-show="tab === 'home'">Home content</div>
  <div x-show="tab === 'profile'">Profile content</div>
  <div x-show="tab === 'settings'">Settings content</div>
</div>
```

Try building this in React without wanting to cry about component architecture.

### Example: Live Search

```html
<div x-data="{
  query: '',
  results: [],
  async search() {
    const response = await fetch(`/api/search?q=${this.query}`)
    this.results = await response.json()
  }
}">
  <input
    type="search"
    x-model="query"
    @input.debounce.500ms="search"
    placeholder="Search...">

  <ul>
    <template x-for="result in results">
      <li x-text="result.name"></li>
    </template>
  </ul>
</div>
```

Built-in debouncing. Async handling. No hooks. No useEffect. No external dependencies.

## Magic Properties

Alpine has some delightful shortcuts:

### $el

```html
<div x-data @click="$el.style.backgroundColor = 'red'">
  Click me
</div>
```

`$el` is the element itself. No refs needed.

### $refs

```html
<div x-data>
  <input x-ref="email" type="email">
  <button @click="$refs.email.focus()">Focus input</button>
</div>
```

Like React refs but without the ceremony.

### $watch

```html
<div x-data="{ count: 0 }" x-init="$watch('count', value => {
  console.log('Count changed to:', value)
})">
  <button @click="count++">{{ count }}</button>
</div>
```

Watch for changes. Like useEffect but you don't have to fight with dependency arrays.

### $dispatch

```html
<div @notify="alert($event.detail.message)">
  <button @click="$dispatch('notify', { message: 'Hello!' })">
    Notify
  </button>
</div>
```

Custom events. Simple communication without context hell.

## Stores: Shared State Without the Drama

Need global state? Alpine has that:

```html
<script>
  document.addEventListener('alpine:init', () => {
    Alpine.store('user', {
      name: 'Alice',
      loggedIn: false,

      login(name) {
        this.name = name
        this.loggedIn = true
      }
    })
  })
</script>

<!-- Use it anywhere -->
<div x-data>
  <div x-show="$store.user.loggedIn">
    Welcome, <span x-text="$store.user.name"></span>!
  </div>

  <button @click="$store.user.login('Bob')">
    Login
  </button>
</div>
```

No Context.Provider. No Redux. No anything. Just a global reactive store.

## Plugins

Alpine's plugin system is delightful:

### Focus Plugin

```html
<script src="https://cdn.jsdelivr.net/npm/@alpinejs/focus@3.x.x/dist/cdn.min.js"></script>

<div x-data="{ open: false }">
  <button @click="open = true">Open Dialog</button>

  <div x-show="open" x-trap.inert.noscroll="open">
    <!-- Focus is trapped here when open -->
    <input type="text">
    <button @click="open = false">Close</button>
  </div>
</div>
```

Accessibility handled. No fighting with focus management.

### Mask Plugin

```html
<script src="https://cdn.jsdelivr.net/npm/@alpinejs/mask@3.x.x/dist/cdn.min.js"></script>

<input x-mask="(999) 999-9999" placeholder="(555) 555-5555">
<input x-mask="99/99/9999" placeholder="MM/DD/YYYY">
```

Input masking without a library.

## Migration from React: The De-escalation

Moving from React to Alpine isn't really migration—it's de-escalation. You're choosing the right tool for the job.

### Pattern 1: Progressive Enhancement

Start with HTML that works without JavaScript:

```html
<!-- Works without JS -->
<details>
  <summary>Click to expand</summary>
  <div>Content here</div>
</details>

<!-- Enhanced with Alpine -->
<div x-data="{ open: false }">
  <button @click="open = !open">
    Click to expand
  </button>
  <div x-show="open" x-collapse>
    Content here with smooth animation
  </div>
</div>
```

### Pattern 2: Replace React Islands

That React component you loaded for one interactive widget? Alpine can do it:

**Before:**
```jsx
// Heavy React component for a simple interaction
ReactDOM.render(<LikeButton postId={123} />, el)
```

**After:**
```html
<div x-data="{ liked: false, count: 42 }">
  <button @click="liked = !liked; count += liked ? 1 : -1"
          :class="{ liked }">
    ❤️ <span x-text="count"></span>
  </button>
</div>
```

No JavaScript file. No build step. Just works.

### Pattern 3: Forms

React forms are verbose. Alpine forms are not:

**Before (React):**
```jsx
function ContactForm() {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    message: ''
  })
  const [errors, setErrors] = useState({})
  const [submitting, setSubmitting] = useState(false)

  const handleChange = (e) => {
    setFormData({
      ...formData,
      [e.target.name]: e.target.value
    })
  }

  const handleSubmit = async (e) => {
    e.preventDefault()
    setSubmitting(true)

    try {
      await fetch('/api/contact', {
        method: 'POST',
        body: JSON.stringify(formData)
      })
      alert('Success!')
    } catch (err) {
      setErrors({ submit: err.message })
    } finally {
      setSubmitting(false)
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="name"
        value={formData.name}
        onChange={handleChange}
      />
      {/* ... */}
    </form>
  )
}
```

**After (Alpine):**
```html
<form
  x-data="{
    formData: { name: '', email: '', message: '' },
    submitting: false,
    async submit() {
      this.submitting = true
      try {
        await fetch('/api/contact', {
          method: 'POST',
          body: JSON.stringify(this.formData)
        })
        alert('Success!')
      } catch (err) {
        alert('Error: ' + err.message)
      } finally {
        this.submitting = false
      }
    }
  }"
  @submit.prevent="submit">

  <input type="text" x-model="formData.name">
  <input type="email" x-model="formData.email">
  <textarea x-model="formData.message"></textarea>

  <button :disabled="submitting">
    <span x-show="!submitting">Send</span>
    <span x-show="submitting">Sending...</span>
  </button>
</form>
```

Same functionality. No controlled component dance. No hooks.

## When Alpine Is Perfect

Alpine excels at:

- **Server-rendered sites** - Django, Rails, Laravel, PHP
- **Progressive enhancement** - Start with HTML, add interactivity
- **Marketing sites** - Modals, dropdowns, simple forms
- **Prototypes** - No setup time
- **Teaching** - Beginners can be productive in 20 minutes
- **Projects allergic to build steps** - Just include a script

Alpine is not ideal for:

- **SPAs** - Use a real framework
- **Complex state management** - You'll want something heavier
- **Team collaboration on large components** - Lack of files makes code review harder
- **TypeScript lovers** - It's possible but not idiomatic

## The Performance Conversation

"But isn't putting JavaScript in HTML slow?"

Compared to what?

**Alpine:**
- 15KB runtime
- No virtual DOM
- No re-rendering
- Direct DOM updates

**React:**
- 140KB runtime (React + ReactDOM)
- Virtual DOM reconciliation
- Re-renders on every state change
- Plus your component code

For simple interactions, Alpine is significantly faster.

For complex SPAs, React's architecture pays off.

Use the right tool.

## Real Talk: Alpine vs React

**React:** "We're a library for building user interfaces. You'll need to configure a build system, choose state management, set up routing, handle forms, manage side effects, optimize re-renders..."

**Alpine:** "Here's a script tag. Make things interactive."

Different problems. Different solutions.

React is for building applications.
Alpine is for adding interactivity.

If your "app" is really a website with some interactive bits, you don't need React. You need Alpine.

And your users will thank you when the page loads in 100ms instead of waiting for 300KB of JavaScript to download, parse, and execute.

---

*"I replaced a 50KB React bundle with 15KB of Alpine and users thought I fixed performance bugs. I just stopped over-engineering."*
— A Developer Who Chose Simplicity

**Up Next:** [Chapter 7: Lit - Web Components for Adults](07-lit.md)
