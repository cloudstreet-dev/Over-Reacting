---
layout: default
---

# Chapter 7: Lit - Web Components for Adults

## Standards-Based and Future-Proof

Pop quiz: How many JavaScript frameworks will exist in 10 years?

Trick question. The answer is "who knows, probably 47 new ones."

Better question: What will browsers support in 10 years?

Web Components. Because they're a web standard, not a framework.

Lit is a thin layer over Web Components that makes them actually pleasant to use. It's betting on the platform instead of fighting it.

## What Are Web Components, Really?

Web Components are built into browsers. They're real, native custom elements:

```html
<!-- This is a real HTML element -->
<my-counter></my-counter>

<script>
  class MyCounter extends HTMLElement {
    constructor() {
      super()
      this.count = 0
      this.attachShadow({ mode: 'open' })
      this.shadowRoot.innerHTML = `
        <button>Count: ${this.count}</button>
      `
    }
  }

  customElements.define('my-counter', MyCounter)
</script>
```

No framework. No compiler. Just the browser.

But vanilla Web Components are verbose and painful. That's where Lit comes in.

## Lit: Web Components Without the Pain

Same component with Lit:

```javascript
import { LitElement, html, css } from 'lit'
import { customElement, property } from 'lit/decorators.js'

@customElement('my-counter')
class MyCounter extends LitElement {
  @property({ type: Number })
  count = 0

  static styles = css`
    button {
      padding: 8px 16px;
      background: blue;
      color: white;
      border: none;
      border-radius: 4px;
    }
  `

  render() {
    return html`
      <button @click=${() => this.count++}>
        Count: ${this.count}
      </button>
    `
  }
}
```

Tagged template literals for HTML. Reactive properties. Scoped styles. Lifecycle methods. All the good parts of frameworks, but producing actual web standards.

## No Virtual DOM, Just Efficient Updates

Like Svelte and Solid, Lit doesn't use a virtual DOM:

```javascript
@customElement('user-profile')
class UserProfile extends LitElement {
  @property()
  user = null

  render() {
    return html`
      <div>
        <h1>Hello, ${this.user.name}</h1>
        <p>Email: ${this.user.email}</p>
      </div>
    `
  }
}
```

When `this.user.name` changes, Lit updates just that text node. No diffing. No reconciliation. Just direct, surgical updates.

The tagged template literal acts like a template. Lit figures out the dynamic parts at compile time and only updates those.

## Shadow DOM: CSS Scoping That Actually Works

React's CSS story:
- CSS-in-JS (styled-components, emotion)
- CSS Modules
- Tailwind
- BEM naming conventions
- Just giving up and using global styles

None of these are standards. All require tools, libraries, or discipline.

Lit uses Shadow DOM:

```javascript
@customElement('my-card')
class MyCard extends LitElement {
  static styles = css`
    :host {
      display: block;
      border: 1px solid #ccc;
      border-radius: 8px;
      padding: 16px;
    }

    h2 {
      color: blue;
      margin: 0;
    }

    /* These styles CANNOT leak out */
    /* Outside styles CANNOT leak in */
  `

  render() {
    return html`
      <h2><slot name="title"></slot></h2>
      <div><slot></slot></div>
    `
  }
}
```

That `h2 { color: blue }` won't affect any other `h2` on the page. And no other styles can affect this component's `h2`. True encapsulation. No CSS-in-JS library needed. It's a browser feature.

## Properties and Attributes

Web Components have both properties (JavaScript) and attributes (HTML):

```javascript
@customElement('user-badge')
class UserBadge extends LitElement {
  @property({ type: String })
  name = 'Guest'

  @property({ type: Boolean })
  admin = false

  render() {
    return html`
      <div class="badge">
        ${this.name}
        ${this.admin ? html`<span class="admin-star">⭐</span>` : ''}
      </div>
    `
  }
}
```

Usage:

```html
<!-- As attributes (strings) -->
<user-badge name="Alice" admin></user-badge>

<!-- As properties (JavaScript) -->
<script>
  const badge = document.querySelector('user-badge')
  badge.name = 'Bob'
  badge.admin = true
</script>
```

Lit handles the synchronization automatically.

## Events: Real DOM Events

React's events are synthetic:

```jsx
<button onClick={handleClick}>Click</button>
```

Behind the scenes, React creates a synthetic event system because reasons (historical reasons, mostly).

Lit uses real DOM events:

```javascript
@customElement('my-button')
class MyButton extends LitElement {
  _handleClick() {
    // Dispatch a real DOM event
    this.dispatchEvent(new CustomEvent('my-click', {
      detail: { message: 'Button clicked!' },
      bubbles: true,
      composed: true
    }))
  }

  render() {
    return html`
      <button @click=${this._handleClick}>
        <slot></slot>
      </button>
    `
  }
}
```

Usage:

```html
<my-button @my-click=${(e) => console.log(e.detail.message)}>
  Click me
</my-button>

<!-- Or with vanilla JS -->
<script>
  document.querySelector('my-button')
    .addEventListener('my-click', (e) => {
      console.log(e.detail.message)
    })
</script>
```

Real events that work with any framework. Or no framework.

## Composition with Slots

React's composition:

```jsx
function Card({ title, children }) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <div className="card-body">
        {children}
      </div>
    </div>
  )
}
```

Lit's composition uses standard slots:

```javascript
@customElement('my-card')
class MyCard extends LitElement {
  render() {
    return html`
      <div class="card">
        <h2><slot name="title">Default Title</slot></h2>
        <div class="card-body">
          <slot></slot>
        </div>
      </div>
    `
  }
}
```

Usage:

```html
<my-card>
  <span slot="title">Custom Title</span>
  <p>This goes in the default slot</p>
  <p>So does this</p>
</my-card>
```

Named slots, default content, and it's all a web standard.

## Lifecycle

React has useEffect and a million gotchas.

Lit has standard Web Component lifecycle:

```javascript
@customElement('my-component')
class MyComponent extends LitElement {
  connectedCallback() {
    super.connectedCallback()
    console.log('Component added to DOM')
    // Like componentDidMount
  }

  disconnectedCallback() {
    super.disconnectedCallback()
    console.log('Component removed from DOM')
    // Cleanup subscriptions, timers, etc.
  }

  updated(changedProperties) {
    super.updated(changedProperties)
    if (changedProperties.has('userId')) {
      console.log('userId changed!')
      this.loadUserData()
    }
  }

  firstUpdated() {
    // Called after first render
    // Good for focusing inputs, measuring DOM, etc.
  }
}
```

Clear, predictable lifecycle hooks. No dependency arrays. No stale closures.

## Reactive Controllers: Composition Without Hooks Drama

React hooks are great until they're not:

```jsx
// Can't use hooks conditionally
// Must follow Rules of Hooks™
// Dependency arrays everywhere
// Stale closures waiting to bite you
```

Lit has Reactive Controllers:

```javascript
class MouseController {
  host

  constructor(host) {
    this.host = host
    host.addController(this)
  }

  _onMouseMove = (e) => {
    this.x = e.clientX
    this.y = e.clientY
    this.host.requestUpdate()
  }

  hostConnected() {
    window.addEventListener('mousemove', this._onMouseMove)
  }

  hostDisconnected() {
    window.removeEventListener('mousemove', this._onMouseMove)
  }

  x = 0
  y = 0
}

@customElement('mouse-tracker')
class MouseTracker extends LitElement {
  mouse = new MouseController(this)

  render() {
    return html`
      <p>Mouse at: ${this.mouse.x}, ${this.mouse.y}</p>
    `
  }
}
```

Reusable. Composable. No rules. Works conditionally. Share logic without hooks gymnastics.

## Framework Interop: The Secret Weapon

React components only work in React.
Vue components only work in Vue.
Lit components work **everywhere**:

```html
<!-- In vanilla HTML -->
<my-button>Click</my-button>

<!-- In React -->
<my-button onClick={handleClick}>Click</my-button>

<!-- In Vue -->
<my-button @click="handleClick">Click</my-button>

<!-- In Angular -->
<my-button (click)="handleClick()">Click</my-button>

<!-- In Svelte -->
<my-button on:click={handleClick}>Click</my-button>
```

Build once, use anywhere. Because they're web standards, not framework abstractions.

## Migration from React: The Long Game

Migrating to Lit isn't about replacing React tomorrow. It's about betting on standards for tomorrow.

### Strategy 1: Design System Components

Build your design system in Lit:

```javascript
// button.js - Works in any framework
@customElement('ds-button')
class DSButton extends LitElement {
  @property({ type: String })
  variant = 'primary'

  @property({ type: Boolean })
  disabled = false

  static styles = css`
    /* Your design system styles */
  `

  render() {
    return html`
      <button
        class=${this.variant}
        ?disabled=${this.disabled}
        @click=${this._handleClick}>
        <slot></slot>
      </button>
    `
  }

  _handleClick() {
    this.dispatchEvent(new Event('click', { bubbles: true }))
  }
}
```

Use it in your React app:

```jsx
function App() {
  return (
    <div>
      <ds-button variant="primary" onClick={handleClick}>
        Click me
      </ds-button>
    </div>
  )
}
```

When you migrate away from React, your design system stays.

### Strategy 2: Leaf Components First

Convert simple components to Lit:

**Before (React):**
```jsx
function Avatar({ src, name, size = 40 }) {
  return (
    <img
      src={src}
      alt={name}
      width={size}
      height={size}
      className="avatar"
    />
  )
}
```

**After (Lit):**
```javascript
@customElement('ui-avatar')
class Avatar extends LitElement {
  @property() src
  @property() name
  @property({ type: Number }) size = 40

  static styles = css`
    img {
      border-radius: 50%;
      object-fit: cover;
    }
  `

  render() {
    return html`
      <img
        src=${this.src}
        alt=${this.name}
        width=${this.size}
        height=${this.size}>
    `
  }
}
```

Use in React:

```jsx
<ui-avatar src="/alice.jpg" name="Alice" size={50} />
```

### Strategy 3: Micro-Frontends

Different parts of your app can be different frameworks:

```html
<!-- Main React app -->
<div id="react-root"></div>

<!-- Lit components mixed in -->
<shopping-cart></shopping-cart>
<user-menu></user-menu>

<!-- Both work together -->
```

## Build Output: Just JavaScript

Lit components compile to plain JavaScript:

```javascript
// Your component
class MyComponent extends LitElement { /* ... */ }

// Compiles to
class MyComponent extends LitElement { /* ... */ }
```

Okay, that's not much of an example. Point is: there's no magic. No virtual DOM runtime. No framework overhead. Just classes and template literals.

Your bundle includes:
- Lit library (~5KB)
- Your component code
- That's it

Compare to React:
- React runtime (~100KB)
- ReactDOM (~40KB)
- Your component code
- Framework overhead

## When Lit Makes Sense

Lit is perfect for:

- **Design systems** - Build once, use everywhere
- **Long-term projects** - Betting on standards, not frameworks
- **Framework-agnostic components** - Works with React, Vue, anything
- **Teams tired of framework churn** - Web Components aren't going away
- **Progressive enhancement** - Real HTML elements that work without JavaScript

Lit might not be ideal for:

- **Complex SPAs** - A full framework might be easier
- **Teams that need massive ecosystems** - Lit's is smaller
- **Developers who hate classes** - Lit uses class-based components
- **Projects that need IE11** - Web Components require polyfills

## The Standards Bet

Here's the thing about frameworks: they come and go.

Remember Backbone? Knockout? Angular.js? Ember?

Web Components are a browser standard. They're not going anywhere.

React might be dominant today. In 10 years? Who knows.

Lit components you write today will work in 10 years. Because they're built on the platform, not on a framework that might be legacy tech by then.

That's a different kind of insurance.

## Real Talk: Lit vs React

**React:** "We're a library for building user interfaces."

**Lit:** "We're a thin layer over web standards."

React builds its own world. You buy into the React ecosystem.

Lit builds on the browser. You buy into web standards.

Different philosophies. Different trade-offs.

If you want the safety of standards and the ability to use your components anywhere, Lit is compelling.

If you want the largest ecosystem and the most jobs, React is still there.

Choose based on your priorities.

---

*"I built a design system in React. Then we switched frameworks and rebuilt it. Then we switched again. Now I build in Lit. Problem solved."*
— A Developer Who Learned the Hard Way

**Up Next:** [Chapter 8: HTMX - The Intervention You Didn't Know You Needed](08-htmx.md)
