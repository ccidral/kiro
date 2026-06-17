---
name: systematic-refactoring
description: >-
  Use when performing code refactoring, restructuring existing code, improving
  code design, cleaning up code smells, extracting classes or methods, renaming
  symbols, or applying any transformation that improves internal quality without
  changing observable behavior.
---

# Systematic Refactoring

Refactoring is performed in baby steps. One small, self-contained refactoring at a time. The project must build without errors and all tests must pass after every single step.

## Goals

Every refactoring should advance one or more of these goals:

- Clarity — code is easy to read and understand
- Simplicity — no unnecessary complexity; the simplest thing that works
- Removing duplication — duplication is a design failure, not a cosmetic issue
- Enabling change — the code becomes easier to modify when the next requirement arrives
- Revealing intent — code expresses *what* it does at the domain level, not *how*
- Reducing complexity — complexity compounds faster than technical debt; fight it continuously
- Improving testability — hard-to-test code is poorly structured; refactoring reveals missing abstractions

If a refactoring doesn't serve at least one of these, question whether it's worth doing.

## The Baby-Step Cycle

```
Identify next refactoring → Check test coverage → Apply it → Run tests → Commit → Repeat
```

Before each baby step, ask: "does the code I'm about to refactor have sufficient unit tests to detect if I break something?" If not, write the necessary tests first, run them green against the current code, and commit them. Only then proceed with the refactoring. Without coverage, a green test run after refactoring proves nothing.

A baby step is the atomic unit of refactoring. Each baby step:
- Is one named refactoring (Extract Function, Rename Variable, Move Method, etc.)
- May touch multiple files — that's fine
- Leaves the project in a green state (compiles, all tests pass)
- Gets its own git commit before the next step begins

## Breaking Down Big Refactorings

When a refactoring task involves multiple changes, decompose it into a sequence of baby steps BEFORE starting. List them out. Then execute them one at a time through the cycle.

Example — "Extract validation logic into a Validator class":
1. Extract validation into a private method in the current class
2. Run tests → commit
3. Move the private method to a new Validator class
4. Run tests → commit
5. Update the original class to inject and call the Validator
6. Run tests → commit

Each step is independently safe. If step 3 breaks something, you revert step 3 only — steps 1-2 are already committed and safe.

## After Each Baby Step

1. Run all tests: `dotnet test src`
2. If tests pass: commit with a message naming the refactoring applied
3. If tests fail: revert the step immediately — do not debug forward on a broken foundation
4. Proceed to the next baby step

## Commit Messages for Refactoring Steps

Each commit names the refactoring and briefly explains why:

```
Extract timezone resolution into private method

The timezone lookup logic was tangled with appointment time
calculation, making both harder to understand independently.
```

```
Move TimezoneResolver to its own class

Timezone resolution is reused by three use cases. Extracting it
eliminates duplication and makes the dependency explicit.
```

## Red Flags — STOP

- Making a second change before committing the first
- "I'll run tests after I finish all the changes"
- "These two changes are related, I'll commit them together"
- "This is too small to deserve its own commit"
- "I'll batch these renames with the extraction"
- Tests are failing and you're making more changes instead of reverting

All of these mean: stop, commit (or revert) the current step, run tests, then continue.

## Common Rationalizations

| Excuse | Reality |
|---|---|
| "These changes are related" | Related ≠ same step. Each named refactoring is its own step. |
| "It's faster to batch them" | It's faster until something breaks and you can't tell which change caused it. |
| "This is too small for a commit" | Small commits are the point. They're individually revertible. |
| "I'll run tests at the end" | Tests after each step catch the exact change that broke things. Tests at the end don't. |
| "The user wants this done quickly" | Baby steps ARE the fast path. Debugging a batched refactoring is the slow path. |
| "I already made multiple changes" | Stop. Run tests. If green, commit what you have. Then resume the cycle. |

## What Counts as One Baby Step

One baby step = one named refactoring from the catalog:

- Extract Function/Method/Class
- Inline Function/Variable
- Rename Variable/Function/Class/Parameter
- Move Function/Field to another class
- Replace Conditional with Polymorphism
- Introduce Parameter Object
- Remove Dead Code
- Change Function Declaration (signature change)

If you can't name it, it's probably not one step — break it down further.

## What Does NOT Count as One Baby Step

- "Extract class and rename its methods" → two steps
- "Move function and update all callers to use new parameter" → two steps (move, then change signature)
- "Remove dead code and clean up the imports" → one step (removing dead code includes its imports)
