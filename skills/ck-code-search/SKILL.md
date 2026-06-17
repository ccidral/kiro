---
name: ck-code-search
description: >
  Search code using the ck CLI tool. Use when searching for text, patterns, or
  concepts in the codebase. Supports semantic search (by meaning), lexical search
  (BM25 ranked keywords), hybrid search (regex + semantic), and regex search
  (exact patterns). Prefer this over the built-in grepSearch tool for all code
  search tasks. Triggers on: "search for", "find code", "where is", "grep",
  "look for", "search the codebase", or any request to locate code by name or
  concept.
compatibility: Requires the ck CLI tool installed and available on PATH.
---

# ck Code Search

Use the `ck` CLI tool for all code search tasks instead of the built-in `grepSearch` tool.

## Search Modes

Pick the mode that fits the query:

| Mode | Flag | When to Use | Example |
|------|------|-------------|---------|
| Regex | `--regex` (default) | Exact name, pattern, or literal string | `ck "fn main" src/`, `ck -i "TODO\|FIXME" .` |
| Semantic | `--sem` | Conceptual search — you describe behavior, not a symbol | `ck --sem "error handling" src/` |
| Lexical | `--lex` | Keyword search ranked by relevance (BM25) | `ck --lex "session manager" .` |
| Hybrid | `--hybrid` | Mix of keyword precision and semantic understanding | `ck --hybrid "JWT token validation" src/` |

### Decision Guide

- **You know the exact symbol name or pattern** → `--regex` (or plain `ck "pattern"`)
- **You're searching by concept, not by name** → `--sem`
- **You have keywords but want ranked results** → `--lex`
- **You're unsure which mode fits, or the query mixes terms with intent** → `--hybrid`

## Defaults and Flags

Always use these flags for manageable output:

```
--limit 20       # Cap results (alias for --topk)
--threshold 0.7  # For --sem and --lex (scores range 0.0–1.0)
--threshold 0.025 # For --hybrid only (RRF scores range ~0.01–0.05)
-C 3             # Context lines for regex mode
-q               # Quiet mode — suppress progress indicators
```

For structured output suitable for parsing, add `--jsonl`:

```
ck --jsonl --sem "error handling" src/
```

Use `--no-snippet` with `--jsonl` when you only need file paths and line numbers.

## Index Availability

Before using `--sem`, `--lex`, or `--hybrid` modes, check if an index exists:

```bash
ck --status-json | jq .index_exists
```

- If `true`: proceed with semantic/lexical/hybrid searches.
- If `false`: fall back to `--regex` mode (which needs no index), or use the built-in search tools.

Regex mode (`ck "pattern"`) always works regardless of index status.

## Indexing

Semantic, lexical, and hybrid modes auto-index on first use. You do not need to
manually index before searching. If you suspect the index is stale after bulk
changes:

```bash
ck --status .          # Check index status
ck --index .           # Rebuild index (only if needed)
```

Do not re-index routinely. Let ck handle incremental updates.

## Threshold Tuning

Semantic and hybrid searches use different score scales. Never mix them up.

- **Semantic / Lexical**: scores 0.0–1.0. Start at `0.7`. Lower to `0.5` if too few results.
- **Hybrid**: RRF scores ~0.01–0.05. Start at `0.025`. Using `0.7` returns nothing.

If a search returns no results, check for near-miss hints in the output. ck will
suggest a lower threshold when the closest match fell just below your cutoff.
Retry with the suggested threshold.

## Common Patterns

```bash
# Find a specific function by name
ck "fn process_request" src/

# Find all TODO/FIXME comments
ck -i "TODO|FIXME" .

# Find error handling logic (conceptual)
ck -q --sem --limit 20 --threshold 0.7 "error handling" src/

# Find authentication code (conceptual)
ck -q --sem --limit 20 --threshold 0.7 "authentication logic" src/

# Keyword search ranked by relevance
ck -q --lex --limit 20 "database connection" .

# Hybrid when unsure
ck -q --hybrid --limit 20 --threshold 0.025 "retry with backoff" src/

# Get full function/class bodies around matches (tree-sitter)
ck -q --sem --full-section --limit 10 "input validation" src/

# Structured output for parsing
ck -q --jsonl --sem --limit 20 "caching strategy" src/
```

## Gotchas

- `--threshold 0.7` is for semantic/lexical only. For hybrid, use `0.025`.
- `--limit` and `--topk` are aliases. Use `--limit` for clarity.
- Regex mode is the default when no mode flag is given. It needs no index.
- Use `-q` (quiet) to suppress progress bars and status messages that clutter output.
- Use `--full-section` to get complete function/class bodies instead of just matching lines. Useful when you need surrounding context. Supported for Python, JavaScript, TypeScript, Haskell, Rust, Ruby.
