---
layout: default
---

# Chapter 8: HTMX - The Intervention You Didn't Know You Needed

## What If Most of Your JavaScript Was a Mistake?

Let me tell you about a journey:

2010: "JavaScript in the browser makes websites dynamic!"

2015: "SPAs with React make everything better!"

2020: "Wait, why is my blog 500KB of JavaScript?"

2024: "What if the server just sent HTML?"

Welcome to HTMX. Welcome to the realization that we may have over-complicated everything.

## Hypermedia-Driven Applications

React's model:
1. Server sends JSON
2. Client-side JavaScript parses JSON
3. Client-side JavaScript builds HTML
4. Client-side JavaScript updates DOM
5. Ship megabytes of framework

HTMX's model:
1. Server sends HTML
2. Done

"But how do you handle state? How do you handle—"

The server handles it. Like we did for 20 years before we decided to rebuild the entire application in the browser.

## HTMX Basics: HTML with Superpowers

```html
<!-- Include HTMX -->
<script src="https://unpkg.com/htmx.org@1.9.10"></script>

<!-- A button that fetches HTML -->
<button hx-get="/click-data" hx-target="#result">
  Click Me
</button>

<div id="result"></div>
```

When clicked, HTMX:
1. Makes GET request to `/click-data`
2. Takes the HTML response
3. Puts it in `#result`

No JavaScript written. No state management. No framework. Just attributes.

The server returns:

```html
<p>You clicked the button at 3:42 PM!</p>
```

That's it. HTML goes in. HTML comes out. No JSON parsing. No templating in the browser. The server did the work.

## Every HTTP Verb, Any Event

React requires JavaScript for everything:

```jsx
function DeleteButton({ id }) {
  const [deleting, setDeleting] = useState(false)

  const handleDelete = async () => {
    setDeleting(true)
    try {
      await fetch(`/api/items/${id}`, { method: 'DELETE' })
      // Now manually update UI state
    } catch (err) {
      // Handle error
    }
  }

  return (
    <button onClick={handleDelete} disabled={deleting}>
      {deleting ? 'Deleting...' : 'Delete'}
    </button>
  )
}
```

HTMX:

```html
<button
  hx-delete="/items/123"
  hx-target="closest tr"
  hx-swap="outerHTML"
  hx-confirm="Are you sure?">
  Delete
</button>
```

When clicked:
1. Confirms with user
2. Sends DELETE request
3. Replaces the entire table row with response (probably empty)

The server handles the deletion and returns the new HTML state. No client-side state. No useState. No JavaScript.

### Trigger on Any Event

```html
<!-- Load on page load -->
<div hx-get="/news" hx-trigger="load"></div>

<!-- Search as you type -->
<input
  type="search"
  hx-get="/search"
  hx-trigger="keyup changed delay:500ms"
  hx-target="#results">

<!-- Infinite scroll -->
<div hx-get="/more-items" hx-trigger="revealed">
  Loading more...
</div>

<!-- Poll every 2 seconds -->
<div hx-get="/server-status" hx-trigger="every 2s">
  Status: Loading...
</div>
```

All without writing JavaScript. The attributes describe the behavior. HTMX does the rest.

## Swap Strategies

HTMX gives you control over how HTML gets inserted:

```html
<!-- Replace inner HTML (default) -->
<div hx-get="/content" hx-swap="innerHTML"></div>

<!-- Replace entire element -->
<div hx-get="/content" hx-swap="outerHTML"></div>

<!-- Insert before -->
<div hx-get="/content" hx-swap="beforebegin"></div>

<!-- Insert after -->
<div hx-get="/content" hx-swap="afterend"></div>

<!-- Prepend -->
<div hx-get="/content" hx-swap="afterbegin"></div>

<!-- Append -->
<div hx-get="/content" hx-swap="beforeend"></div>

<!-- Delete (don't insert anything) -->
<div hx-delete="/items/1" hx-swap="delete"></div>
```

## Real-World Patterns

### Inline Editing

**React version:**
```jsx
function EditableField({ id, value }) {
  const [editing, setEditing] = useState(false)
  const [currentValue, setCurrentValue] = useState(value)
  const [saving, setSaving] = useState(false)

  const save = async () => {
    setSaving(true)
    await fetch(`/api/items/${id}`, {
      method: 'PATCH',
      body: JSON.stringify({ value: currentValue })
    })
    setSaving(false)
    setEditing(false)
  }

  if (editing) {
    return (
      <div>
        <input
          value={currentValue}
          onChange={e => setCurrentValue(e.target.value)}
        />
        <button onClick={save} disabled={saving}>Save</button>
        <button onClick={() => setEditing(false)}>Cancel</button>
      </div>
    )
  }

  return (
    <div onClick={() => setEditing(true)}>
      {currentValue}
    </div>
  )
}
```

**HTMX version:**

View mode (initial server response):
```html
<div hx-get="/edit/123" hx-target="this" hx-swap="outerHTML">
  <p>Click to edit: Current Value</p>
</div>
```

Edit mode (server returns this when clicked):
```html
<form hx-put="/update/123" hx-target="this" hx-swap="outerHTML">
  <input name="value" value="Current Value">
  <button type="submit">Save</button>
  <button hx-get="/cancel/123" hx-target="closest div">Cancel</button>
</form>
```

After saving (server returns view mode again):
```html
<div hx-get="/edit/123" hx-target="this" hx-swap="outerHTML">
  <p>Click to edit: New Value</p>
</div>
```

No client-side state. No React. Just HTML swapping.

### Active Search

```html
<input
  type="search"
  name="query"
  hx-get="/search"
  hx-trigger="keyup changed delay:300ms"
  hx-target="#results"
  hx-indicator="#spinner"
  placeholder="Search...">

<img id="spinner" class="htmx-indicator" src="/spinner.gif">

<div id="results"></div>
```

Server endpoint:

```python
@app.get("/search")
def search(query: str):
    results = db.search(query)
    return render_template("results.html", results=results)
```

Returns:

```html
<ul>
  {% for result in results %}
    <li>{{ result.name }}</li>
  {% endfor %}
</ul>
```

Debounced search with loading indicator. No React. No useState. No useEffect. Just attributes.

### Infinite Scroll

```html
<div id="items">
  {% for item in items %}
    <div class="item">{{ item.name }}</div>
  {% endfor %}

  <div
    hx-get="/items?page=2"
    hx-trigger="revealed"
    hx-swap="afterend">
    <p>Loading more...</p>
  </div>
</div>
```

When the loading div is revealed (scrolled into view), HTMX requests the next page and appends it. The server response includes the next loading div for page 3, and so on.

No intersection observer code. No state management. Just HTML.

### Optimistic UI

```html
<button
  hx-post="/like"
  hx-swap="outerHTML"
  hx-target="this"
  hx-swap-oob="true">
  <span class="count">42</span> ❤️
</button>
```

Server immediately returns:

```html
<button
  hx-post="/unlike"
  hx-swap="outerHTML"
  hx-target="this">
  <span class="count">43</span> 💔
</button>
```

Feels instant. Server handles the actual logic. If it fails, server returns the original state.

## Out of Band Swaps

HTMX can update multiple parts of the page from one response:

```html
<button hx-post="/buy-item">Buy Now</button>

<div id="cart-count">3 items</div>
<div id="cart-total">$45.00</div>
```

Server response:

```html
<!-- Main response goes to the button's target -->
<div>Thank you for your purchase!</div>

<!-- Out-of-band updates -->
<div id="cart-count" hx-swap-oob="true">4 items</div>
<div id="cart-total" hx-swap-oob="true">$65.00</div>
```

One request updates three parts of the page. Try doing that in React without Redux or Context nonsense.

## When the Backend Was Right All Along

Remember when we said "the backend should be a dumb JSON API"?

What if that was wrong?

With HTMX, the backend is smart. It:
- Owns the state
- Renders the HTML
- Handles validation
- Manages sessions
- Controls authorization

The frontend is simple. It:
- Shows HTML
- Sends requests
- Swaps HTML

This is how websites worked in 2005. Except now it feels like a SPA because HTMX makes the requests async and swaps the content smoothly.

## Compared to React SPAs

**React SPA Architecture:**
```
Browser                          Server
--------                         ------
React App (300KB)     ←JSON→     API Endpoint
├─ State Management              (Just sends data)
├─ Routing
├─ Validation
├─ UI Logic
├─ Template Rendering
└─ API Calls
```

- Most logic in the browser
- Server is a dumb data API
- Duplicate validation (client and server)
- Large bundle size
- Complex state management

**HTMX Architecture:**
```
Browser                    Server
--------                   ------
HTMX (14KB)    ←HTML→      Full App
├─ Swap HTML              ├─ State
└─ Make requests          ├─ Validation
                          ├─ Templates
                          ├─ Business Logic
                          └─ Database
```

- Most logic on the server
- Client is thin
- Validation once (server)
- Tiny bundle
- State lives on server

Different paradigms. Different trade-offs.

## Migration from React: The Simplification

Moving from React to HTMX isn't just a framework migration—it's an architectural shift.

### Step 1: Identify Read-Heavy Pages

Pages that mostly display data are perfect candidates:

**Before (React):**
```jsx
function UserProfile() {
  const [user, setUser] = useState(null)
  const { id } = useParams()

  useEffect(() => {
    fetch(`/api/users/${id}`)
      .then(r => r.json())
      .then(setUser)
  }, [id])

  if (!user) return <Spinner />

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
      <p>{user.bio}</p>
    </div>
  )
}
```

**After (HTMX):**
```html
<!-- Server renders this directly -->
<div>
  <h1>{{ user.name }}</h1>
  <p>{{ user.email }}</p>
  <p>{{ user.bio }}</p>
</div>
```

No fetch. No loading state. No useEffect. The server rendered it. Done.

### Step 2: Convert Interactive Pieces

**Before (React):**
```jsx
function LikeButton({ postId, initialLikes, initialLiked }) {
  const [likes, setLikes] = useState(initialLikes)
  const [liked, setLiked] = useState(initialLiked)

  const toggle = async () => {
    const newLiked = !liked
    setLiked(newLiked)
    setLikes(likes + (newLiked ? 1 : -1))

    await fetch(`/api/posts/${postId}/like`, {
      method: newLiked ? 'POST' : 'DELETE'
    })
  }

  return (
    <button onClick={toggle}>
      {liked ? '❤️' : '🤍'} {likes}
    </button>
  )
}
```

**After (HTMX):**
```html
<!-- Server renders this -->
<button
  hx-post="/posts/{{ post.id }}/like"
  hx-swap="outerHTML">
  {{ '❤️' if liked else '🤍' }} {{ likes }}
</button>
```

Server handles the toggle and returns the new button HTML.

### Step 3: Forms

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

  const handleSubmit = async (e) => {
    e.preventDefault()
    setSubmitting(true)
    setErrors({})

    const res = await fetch('/api/contact', {
      method: 'POST',
      body: JSON.stringify(formData)
    })

    if (res.ok) {
      alert('Sent!')
      setFormData({ name: '', email: '', message: '' })
    } else {
      const data = await res.json()
      setErrors(data.errors)
    }

    setSubmitting(false)
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={formData.name}
        onChange={e => setFormData({...formData, name: e.target.value})}
      />
      {errors.name && <span>{errors.name}</span>}
      {/* Repeat for each field */}
    </form>
  )
}
```

**After (HTMX):**
```html
<form
  hx-post="/contact"
  hx-target="this"
  hx-swap="outerHTML">

  <input name="name" value="{{ form.name or '' }}">
  {% if errors.name %}
    <span class="error">{{ errors.name }}</span>
  {% endif %}

  <input name="email" value="{{ form.email or '' }}">
  {% if errors.email %}
    <span class="error">{{ errors.email }}</span>
  {% endif %}

  <textarea name="message">{{ form.message or '' }}</textarea>
  {% if errors.message %}
    <span class="error">{{ errors.message }}</span>
  {% endif %}

  <button type="submit">Send</button>
</form>
```

Server validates, and either:
- Returns success message
- Returns the form again with errors filled in

No client-side validation logic. No state management. Just HTML.

## When HTMX Makes Sense

HTMX is perfect for:

- **Server-rendered apps** - Django, Rails, Laravel, Flask, Express
- **Content-heavy sites** - Blogs, documentation, marketing
- **CRUD apps** - Admin panels, dashboards
- **Teams with strong backend developers** - Let the backend do the work
- **Projects that want simple architecture** - Less JavaScript = fewer problems

HTMX might not be ideal for:

- **Highly interactive apps** - Figma, Google Docs, games
- **Offline-first apps** - Need client-side state
- **Real-time collaborative editing** - Better with operational transforms
- **Apps with complex client-side logic** - Sometimes you do need the client to be smart

## The Philosophy Shift

React says: "Move everything to the client. The server is just an API."

HTMX says: "The server is the application. The client is a thin view layer."

Both can work. But one requires 300KB of JavaScript and the other requires 14KB.

One duplicates validation logic in client and server. The other validates once.

One requires complex state management. The other asks the server "what should I show now?"

Different approaches for different needs.

## Real Talk: HTMX vs React

HTMX won't replace React for everything. Google Docs needs client-side state. Figma needs client-side rendering. Complex dashboards with real-time collaboration need smart clients.

But your blog? Your marketing site? Your CRUD app? Your admin panel?

Those don't need React. They never did. We just convinced ourselves that SPAs were always better.

HTMX is the reminder that simple solutions exist. And sometimes, the server was right all along.

---

*"I replaced 50KB of React with 14KB of HTMX and realized most of my 'state management' was just asking the server for HTML."*
— A Developer Who Simplified

**Up Next:** [Chapter 9: Astro - Content Sites Don't Need Virtual DOMs](09-astro.md)
