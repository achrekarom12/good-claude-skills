---
name: commit-message
description: Generates a conventional commit message and description from staged changes. Use this skill whenever the user asks to write a commit message, describe changes for a commit, summarize staged git changes, or when they say things like "write my commit", "what should my commit say", "generate a commit message", or "commit these changes". Always use this skill when git commits are involved.
---

# Commit Message Generator

Generate a conventional commit message + description from the current staged diff.

## Step 1 — Gather the diff

Run:
```bash
git diff --staged
```

If the output is empty, also try:
```bash
git diff HEAD
```

If still empty, inform the user that there are no staged changes and ask them to stage files first (`git add`).

## Step 2 — Choose a commit type

Pick the **one** type that best matches the primary intent of the changes:

| Type       | When to use |
|------------|-------------|
| `feat`     | Adding a new feature |
| `fix`      | Fixing a bug |
| `docs`     | Documentation updates only |
| `style`    | Formatting, whitespace, no logic changes |
| `refactor` | Code restructuring, no feature or bug change |
| `test`     | Adding or updating tests |
| `chore`    | Maintenance tasks, dependency updates |
| `ci`       | CI/CD pipeline changes |
| `build`    | Build system or tooling changes |
| `perf`     | Performance improvements |
| `revert`   | Reverting a previous commit |

## Step 3 — Write the commit message

**First line (commit title):**
```
<type>: <short imperative description>
```
- Use imperative mood: "add", "fix", "update" — not "added", "fixed", "updated"
- Keep under 72 characters
- Lowercase after the colon
- No emojis anywhere in the output

**Example titles:**
```
feat: add user authentication module
fix: resolve null pointer in payment handler
docs: update API endpoint documentation
refactor: extract database connection logic
```

## Step 4 — Write the full description

Output the full commit block in this format:

```
<type>: <short description>

## What
One sentence explaining what this commit does.

## Why
Brief context on why this change is needed. What problem does it solve or what goal does it achieve?

## Changes
- Bullet points of specific changes made
- Group related changes together
- Mention any files deleted or renamed
- Note any breaking changes if applicable
```

## Output rules

- Present the commit title first in a code block so it's easy to copy
- Then show the full description in a separate code block
- Do not use emojis anywhere in the output
- Do not add any extra commentary unless the diff is ambiguous — just output the two blocks
- If the diff touches many unrelated concerns, note this and suggest splitting into multiple commitsw