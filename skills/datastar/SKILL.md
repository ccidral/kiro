---
description: "Build hypermedia-driven UIs with Datastar — backend patches DOM/signals via SSE, frontend reacts via data-* attributes. Use when writing Datastar components, backend SSE handlers, or wiring reactivity with data-* attributes."
---

# Datastar

Datastar is a hypermedia-first frontend framework. The backend *drives* the frontend by **patching** HTML elements and signals via SSE events. The frontend declares reactivity through `data-*` attributes. No build step, no npm.

## Mental model

Two loops, one framework:

1. **Backend → Frontend (SSE):** Server sends `datastar-patch-elements` and `datastar-patch-signals` events. The client morphs DOM and merges signal values.
2. **Frontend reactivity:** `data-*` attributes bind signals (`$`-prefixed reactive variables) to DOM, execute expressions, and fire backend actions (`@`-prefixed helpers).

The backend is the single source of truth. Frontend signals exist for transient UI state (open/closed, input values) until the backend confirms them.

## Syntax essentials

- **Signals** — reactive variables, always `$name` in expressions. Created by `data-signals`, `data-bind`, `data-computed`, or patched from backend.
- **Actions** — `@name()` helpers in expressions. `@get`, `@post`, `@put`, `@patch`, `@delete` for backend; `@peek`, `@setAll`, `@toggleAll` for frontend.
- **Modifiers** — double-underscore suffixes on attribute keys: `data-on:click__debounce.300ms`, `data-on:keydown__window`.
- **Casing** — signal-defining attributes (`data-signals`, `data-bind`, `data-computed`) convert hyphenated keys to camelCase. Other attributes (`data-class`, `data-on`) convert to kebab-case.

## Writing frontend HTML

```html
<!-- Declare signals -->
<div data-signals="{search: '', page: 1}">

  <!-- Two-way bind input to signal -->
  <input data-bind:search />

  <!-- Debounced search on input -->
  <input data-bind:search
         data-on:input__debounce.300ms="@get('/search')" />

  <!-- Conditional visibility -->
  <div data-show="$search != ''">Results:</div>

  <!-- Reactive text -->
  <span data-text="$search.length + ' chars'"></span>

  <!-- Computed signal -->
  <div data-computed:upper="$search.toUpperCase()"
       data-text="$upper"></div>

  <!-- Loading indicator -->
  <button data-on:click="@get('/data')"
          data-indicator:loading
          data-attr:disabled="$loading">
    Load
  </button>
  <div data-show="$loading">Loading...</div>

  <!-- Infinite scroll -->
  <div data-on-intersect__once="@get('/more')"></div>

  <!-- Polling -->
  <div data-on-interval__duration.5s="@get('/status')"></div>
</div>
```

## Writing backend SSE responses

Set `Content-Type: text/event-stream`. Send events separated by two newlines.

```
event: datastar-patch-elements
data: elements <div id="results">...new HTML...</div>

event: datastar-patch-signals
data: signals {page: 2, hasMore: true}

```

### Patch modes

Default is `outer` (morph by ID). Override with `data: mode <mode>` and `data: selector <css>`:

- `outer` — morph outer HTML (default, match by element ID)
- `inner` — morph inner HTML of selector target
- `prepend` / `append` — add to target's children
- `before` / `after` — insert as siblings
- `remove` — remove target from DOM
- `replace` — replace without morphing

### Removing signals

```
event: datastar-patch-signals
data: signals {obsolete: null}

```

## Backend action options

All `@get/@post/@put/@patch/@delete` accept a second options argument:

```html
<button data-on:click="@post('/submit', {
  filterSignals: {include: /^form\./},
  headers: {'X-Csrf-Token': 'abc'},
  contentType: 'form',
  retry: 'error',
  retryMaxCount: 3
})">Submit</button>
```

Key options: `filterSignals`, `headers`, `contentType` (`json`|`form`), `openWhenHidden`, `retry` (`auto`|`error`|`always`|`never`), `retryInterval`, `retryMaxCount`, `requestCancellation` (`auto`|`cleanup`|`disabled`).

## Common patterns

| Pattern | Technique |
|---------|-----------|
| Active search | `data-bind` + `data-on:input__debounce.300ms="@get('/search')"` |
| Loading state | `data-indicator:loading` + `data-show="$loading"` |
| Infinite scroll | `data-on-intersect__once="@get('/next-page')"` |
| Polling | `data-on-interval__duration.5s="@get('/status')"` |
| Bulk toggle | `@toggleAll({include: /^item\./})` |
| Click-to-edit | Show form on click, `@put` to save, backend patches display HTML |
| Lazy load | `data-on-intersect__once="@get('/lazy-content')"` |
| File upload | `<input type="file" data-bind:files multiple>` (auto base64) |
| Form submission | `@post('/endpoint', {contentType: 'form'})` on closest `<form>` |
| Keyboard shortcuts | `data-on:keydown__window="evt.key === 'k' && evt.metaKey && ..."` |
| Prevent flicker | `style="display: none"` on `data-show` elements |

## Rules

- Every top-level element patched from backend **must have an `id`**.
- Signals starting with `_` are local — not sent to backend.
- Signal names cannot contain `__` (reserved for modifier delimiter).
- `data-indicator` must appear *before* `data-init` on the same element (attribute evaluation order matters).
- `data-on:submit` automatically prevents default form submission.
- `GET` requests send signals as `?datastar=...` query param; other methods send JSON body.
- `data-computed` expressions must be pure — no side effects. Use `data-effect` for side effects.

## Full reference

See [`REFERENCE.md`](REFERENCE.md) for the complete attribute, action, and SSE event reference with all modifiers.
