---
name: draft-pull-request
description: >
  Write a pull request title and description for the changes in the current
  coding session. Use when the user says "write a PR", "draft a pull request",
  "PR description", "prepare a PR", "write PR title and description", or wants
  to document their changes for review before opening a pull request. Also use
  when the user asks to summarize changes for a PR, even if they don't
  explicitly say "pull request".
---

# Write Pull Request

Write a PR title and description that helps reviewers understand **why** the changes were made, not just what changed.

## Procedure

### 1. Ask about the ticket number

Before gathering context, ask the user:

> Is there an issue tracker ticket associated with this PR? If so, what's the ticket number?

Wait for the user's response. The ticket number is optional.

### 2. Gather context

Collect the information needed to write the PR:

- Run `git diff main...HEAD` (or the appropriate base branch) to see what changed.
- Run `git log main...HEAD --oneline` to see the commit history.
- Review the current conversation for context about *why* these changes were made — the problem being solved, the motivation, any design decisions discussed.

### 3. Determine the output file path

- Run `git branch --show-current` to get the current branch name.
- If the branch is a main branch (`main`, `master`, `develop`, `development`), choose a short descriptive kebab-case file name that summarizes the PR instead.
- Otherwise, use the branch name as the file name.
- Prefix the file name with the current date in ISO format: `yyyy-MM-dd-<name>.md`.
- The file goes in the `docs/pull-requests/` directory at the project root. Create the directory if it doesn't exist.

### 4. Write the PR title

Format: `<title>` or `<ticket_number>: <title>` if a ticket number was provided.

The title should be a concise imperative sentence describing the change (e.g., "Add retry logic to payment webhook handler"). Keep it under 72 characters when possible.

### 5. Write the PR description

The description prioritizes **why over what**. Structure it as follows:

```markdown
## Motivation

Why this change exists. What problem it solves, what need it addresses, or what
opportunity it captures. This is the most important section — a reviewer who
reads only this section should understand the purpose of the PR.

For bug fixes, describe the root cause of the bug here.

## Changes

A judicious summary of what changed. Do NOT exhaustively list every modified
file or mechanically repeat what the diff shows. Instead, highlight:
- Non-obvious design decisions and why they were made
- Anything a reviewer should pay special attention to
- Relationships between changes that aren't obvious from the diff alone

Omit this section entirely for trivial changes where the title and motivation
say everything needed.

## Notes for reviewers

Optional. Include only when there's something specific the reviewer should know:
- Areas where you'd like focused feedback
- Known limitations or follow-up work
- Testing instructions if non-obvious
```

### 6. Write the file

Write the PR title as an H1 heading followed by the description body to the determined file path.

## Gotchas

- The "Changes" section is the most common place to go wrong. Resist the urge to list every file touched or describe changes mechanically. The reviewer has the diff — they need context the diff doesn't provide.
- When the user discussed the *why* during the coding session, use that conversation as the primary source for the Motivation section rather than trying to infer motivation from the code alone.
- If the diff is large, focus the Changes section on the high-level approach and key decisions, not on covering every detail.
- For bug fixes, always describe the root cause in the Motivation section. "Fixed the bug" is not useful; "Requests were retried without resetting the timeout counter, causing cascading failures under load" is.
