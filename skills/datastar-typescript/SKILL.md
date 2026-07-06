---
description: "Use the Datastar TypeScript SDK to stream SSE patches from Node/Deno/Bun backends. Use when implementing SSE handlers with this SDK, reading client signals, patching DOM or signals, or choosing the right import for a runtime."
---

# Datastar TypeScript SDK

Stream `datastar-patch-elements` and `datastar-patch-signals` SSE events to Datastar frontends from Node.js, Deno, or Bun.

## Import paths

| Runtime | Import |
|---------|--------|
| Node.js | `@starfederation/datastar-sdk/node` |
| Deno | `npm:@starfederation/datastar-sdk/web` |
| Bun | `@starfederation/datastar-sdk/web` |

## Reading signals

`ServerSentEventGenerator.readSignals(request)` — static, async.

- **GET/DELETE**: parses JSON from `?datastar=` query param.
- **POST/PUT/PATCH**: parses JSON body.
- Returns `{ success: true, signals }` or `{ success: false, error }`.

```typescript
const reader = await ServerSentEventGenerator.readSignals(req);
if (!reader.success) { res.writeHead(400); res.end(); return; }
const { signals } = reader;
```

## Opening a stream

`ServerSentEventGenerator.stream(req, res, callback, options?)` — static, async.

Sends SSE headers, calls `callback(stream)`, then closes the response.

Options: `onError`, `onAbort`, `keepalive`.

When `keepalive: true`, the stream stays open after the callback returns — you must call `stream.close()` yourself.

```typescript
ServerSentEventGenerator.stream(req, res, (stream) => {
  stream.patchElements(`<div id="output">Hello</div>`);
});
```

## Stream methods

| Method | Purpose | Key options |
|--------|---------|-------------|
| `patchElements(html, opts?)` | Patch DOM elements | `mode`, `selector`, `namespace`, `useViewTransition` |
| `patchSignals(json, opts?)` | Patch signal store | `onlyIfMissing` |
| `removeElements(selector?, html?, opts?)` | Remove elements | Provide either a CSS selector or html-with-IDs |
| `removeSignals(keys, opts?)` | Remove signals from store | `onlyIfMissing` |
| `executeScript(code, opts?)` | Run JS on client | `autoRemove` (default true), `attributes` |
| `close()` | End the stream | — |

All methods accept `eventId` and `retryDuration` in options.

## Patch modes

Default is `outer` (morph entire element, preserving state). Override with `mode` option:

`outer` (morph, preserve state) · `inner` (morph children) · `replace` (reset state) · `prepend` · `append` · `before` · `after` · `remove`

## Rules

- Every patched element **must have an `id`** unless targeting by `selector`.
- `patchSignals` takes a JSON **string**, not an object.
- Signals use RFC 7386 merge-patch: set a key to `null` to remove it (`removeSignals` does this for you).
- Valid namespaces: `html`, `svg`, `mathml`.

## Full example (Node.js)

```typescript
import { ServerSentEventGenerator } from "@starfederation/datastar-sdk/node";

// In your route handler:
const reader = await ServerSentEventGenerator.readSignals(req);
if (!reader.success) { res.writeHead(400); res.end(); return; }

ServerSentEventGenerator.stream(req, res, (stream) => {
  stream.patchSignals(JSON.stringify({ count: reader.signals.count + 1 }));
  stream.patchElements(`<div id="counter">Count: ${reader.signals.count + 1}</div>`);
});
```
