# Datastar Reference

## Attributes

### `data-signals`
Patches signals into the reactive store.
```html
<div data-signals:foo="1"></div>
<div data-signals:foo.bar="1"></div>          <!-- nested -->
<div data-signals="{foo: 1, bar: 'hi'}"></div> <!-- object syntax -->
<div data-signals="{foo: null}"></div>         <!-- removes signal -->
```
Modifiers: `__case` (.camel/.kebab/.snake/.pascal), `__ifmissing` (only set if not exists).

### `data-bind`
Two-way binding between signal and form element.
```html
<input data-bind:search />
<input data-bind="search" />
<select data-bind:choice>...</select>
<input type="file" data-bind:files multiple />  <!-- auto base64 -->
```
Modifiers: `__case`, `__prop` (bind to specific property), `__event` (which events trigger sync).

### `data-text`
Sets element text content to expression result.
```html
<span data-text="$count + ' items'"></span>
```

### `data-show`
Toggles element visibility.
```html
<div data-show="$isOpen" style="display: none"></div>
```

### `data-class`
Adds/removes classes conditionally.
```html
<div data-class:active="$selected"></div>
<div data-class="{active: $sel, 'font-bold': $strong}"></div>
```
Modifier: `__case`.

### `data-attr`
Sets any HTML attribute reactively.
```html
<button data-attr:disabled="$loading"></button>
<div data-attr="{'aria-label': $label, hidden: $hide}"></div>
```

### `data-style`
Sets inline CSS properties reactively.
```html
<div data-style:background-color="$active ? 'green' : 'red'"></div>
<div data-style="{opacity: $fade, display: $hide && 'none'}"></div>
```

### `data-computed`
Read-only derived signal. Must be pure (no side effects).
```html
<div data-computed:full-name="$first + ' ' + $last"></div>
```
Modifier: `__case`.

### `data-effect`
Runs expression on load and whenever referenced signals change. For side effects.
```html
<div data-effect="$total = $price * $qty"></div>
```

### `data-on`
Event listener with expression.
```html
<button data-on:click="@post('/save')">Save</button>
<div data-on:keydown__window="evt.key === 'Escape' && ($open = false)"></div>
```
Modifiers: `__once`, `__passive`, `__capture`, `__case`, `__delay` (.500ms/.1s), `__debounce` (.Xms/.leading/.notrailing), `__throttle` (.Xms/.noleading/.trailing), `__viewtransition`, `__window`, `__document`, `__outside`, `__prevent`, `__stop`.

Variable `evt` is the event object. `data-on:submit` prevents default automatically.

### `data-on-intersect`
Fires when element enters/exits viewport.
```html
<div data-on-intersect__once="@get('/lazy')"></div>
```
Modifiers: `__once`, `__exit`, `__half`, `__full`, `__threshold` (.25/.75), plus timing modifiers.

### `data-on-interval`
Fires at regular intervals (default 1s).
```html
<div data-on-interval__duration.5s="@get('/poll')"></div>
```
Modifiers: `__duration` (.Xms/.Xs/.leading), `__viewtransition`.

### `data-indicator`
Creates boolean signal that's `true` during fetch, `false` otherwise.
```html
<button data-on:click="@get('/data')"
        data-indicator:loading>Load</button>
<div data-show="$loading">Spinner</div>
```
Must precede `data-init` on same element.

### `data-init`
Runs expression on attribute initialization (page load, DOM patch, attribute change).
```html
<div data-init="@get('/initial-data')"></div>
```
Modifiers: `__delay`, `__viewtransition`.

### `data-ref`
Creates signal referencing the DOM element.
```html
<canvas data-ref:my-canvas></canvas>
<!-- $myCanvas is now the <canvas> element -->
```

### `data-ignore`
Skips Datastar processing for element and descendants.
```html
<div data-ignore>...</div>
```
Modifier: `__self` (ignore element only, process children).

### `data-ignore-morph`
Skips morphing for element during patch.

### `data-preserve-attr`
Preserves specific attributes during morph.
```html
<details open data-preserve-attr="open"></details>
```

### `data-json-signals`
Debug helper: renders signals as JSON in element text.
```html
<pre data-json-signals></pre>
<pre data-json-signals="{include: /user/}"></pre>
```

### `data-on-signal-patch`
Fires when any signal is patched. Variable `patch` available.
```html
<div data-on-signal-patch="console.log(patch)"></div>
```

### `data-on-signal-patch-filter`
Filters which signals trigger `data-on-signal-patch`.
```html
<div data-on-signal-patch-filter="{include: /^counter$/}"></div>
```

## Actions

### Frontend actions

**`@peek(callable)`** — Access signals without subscribing to changes.
```html
<div data-text="$a + @peek(() => $b)"></div>
<!-- Re-evaluates when $a changes, NOT when $b changes -->
```

**`@setAll(value, filter?)`** — Set all matching signals to a value.
```html
<button data-on:click="@setAll('', {include: /^form\./})">Reset</button>
```

**`@toggleAll(filter?)`** — Toggle boolean signals.
```html
<button data-on:click="@toggleAll({include: /^item\./})">Toggle All</button>
```

### Backend actions

All share the signature: `@method(url, options?)`.

**`@get(url, opts?)`** — GET request. Signals sent as query param.  
**`@post(url, opts?)`** — POST request. Signals sent as JSON body.  
**`@put(url, opts?)`** — PUT request.  
**`@patch(url, opts?)`** — PATCH request.  
**`@delete(url, opts?)`** — DELETE request.

Options object:
- `contentType`: `'json'` (default) | `'form'`
- `filterSignals`: `{include: /regex/, exclude?: /regex/}`
- `selector`: CSS selector for form (when contentType is 'form')
- `headers`: `{key: value}` object
- `openWhenHidden`: boolean (default false for GET, true for others)
- `payload`: custom fetch payload override
- `retry`: `'auto'` | `'error'` | `'always'` | `'never'`
- `retryInterval`: ms (default 1000)
- `retryScaler`: multiplier (default 2)
- `retryMaxWait`: ms (default 30000)
- `retryMaxCount`: number (default 10)
- `requestCancellation`: `'auto'` | `'cleanup'` | `'disabled'` | AbortController

Request cancellation: by default, new requests to the same URL+method cancel in-flight ones.

### Response content types

Backend can respond with:
- `text/event-stream` — SSE events (standard Datastar flow)
- `text/html` — patches elements directly (headers: `datastar-selector`, `datastar-mode`)
- `application/json` — patches signals (header: `datastar-only-if-missing`)
- `text/javascript` — executes script in browser

### Fetch lifecycle events

`datastar-fetch` custom event with `evt.detail.type`: `started`, `finished`, `error`, `retrying`, `retries-failed`.

## SSE Events

### `datastar-patch-elements`

```
event: datastar-patch-elements
data: elements <div id="target">New content</div>

```

Data lines:
- `data: selector <css>` — target selector (optional if using outer mode with ID)
- `data: mode <mode>` — outer (default), inner, replace, prepend, append, before, after, remove
- `data: namespace <ns>` — svg, mathml
- `data: useViewTransition true`
- `data: elements <html>` — the HTML (can span multiple `data: elements` lines)

### `datastar-patch-signals`

```
event: datastar-patch-signals
data: signals {key: value, nested: {a: 1}}

```

Data lines:
- `data: onlyIfMissing true` — only set signals that don't exist yet
- `data: signals <expression>` — signal values to merge

Set to `null` to remove: `data: signals {foo: null}`.

## Expressions

- `$signalName` in expressions resolves to the signal's value
- `el` variable references the current element
- `evt` variable available in `data-on` expressions
- Standard JS operators, ternary, object/array literals all work
- Expressions are sandboxed — no access to global scope except through signals and actions

## Attribute evaluation order

Attributes are evaluated in DOM order (depth-first). On the same element, they apply in source order. This matters for `data-indicator` before `data-init`.

## Casing rules

Signal-defining attributes convert keys to **camelCase**: `data-signals:my-signal` → `$mySignal`.

Other attributes convert keys to **kebab-case**: `data-class:font-bold` → class `font-bold`, `data-on:my-event` → event `my-event`.

Use `__case` modifier to override: `.camel`, `.kebab`, `.snake`, `.pascal`.
