---
name: variable-renamer
description: >
  Analyzes code and renames variables, functions, and parameters to meaningful, self-documenting names based on how they are actually used — instead of adding comments. Use this skill whenever the user wants to improve code readability by renaming things, clean up unclear variable names like `x`, `tmp`, `data`, `val`, `d`, `res`, `flag`, `arr`, etc., or when the user says "rename variables", "make this code self-documenting", "improve variable names", "replace comments with better names", "clean up this code", or similar. Also trigger when the user pastes code with single-letter variables, abbreviations, or generic names and asks for a review or cleanup. This skill should be used proactively any time code clarity could be improved through better naming.
---

# Variable Renamer Skill

Rename variables, functions, and parameters to be self-documenting — so the code explains itself without needing comments.

---

## Core Philosophy

> "Good code is its own best documentation." — Steve McConnell

The goal is **semantic clarity**: each name should convey *what the thing is* or *what it does*, so a reader understands the code without looking elsewhere.

- **No comments needed** after renaming — the names speak for themselves
- **Preserve behavior** — only rename, never refactor logic
- **Respect language conventions** — match the style of the surrounding code

---

## Step-by-Step Process

### 1. Understand Before Renaming
Read the entire code block first. Trace data flow, understand what each variable holds at each stage, what each function does end-to-end. Do **not** rename on first pass.

### 2. Build a Rename Map
For each identifier, ask:
- What does this variable *hold*? → name it after its *content* (e.g., `d` → `userBirthDate`)
- What does this function *do*? → use a verb phrase (e.g., `proc` → `calculateMonthlyTax`)
- What does this parameter *represent*? → name after its *role* (e.g., `x` → `targetTemperature`)

### 3. Apply Consistently
Replace **every occurrence** of the old name — declarations, assignments, usage, return statements, and any string references in the same file that reflect the variable name.

### 4. Present the Output
- Show the **fully renamed code** (not a diff)
- Include a **rename table** summarizing changes:

| Original | Renamed | Reason |
|----------|---------|--------|
| `d`      | `orderDeliveryDate` | Holds the delivery date of an order |
| `fn`     | `applyDiscount`     | Applies a discount percentage to a price |

- If any name was ambiguous or could be interpreted multiple ways, briefly note it and explain your choice.

---

## Naming Rules

### Variables & Constants
- Name after **what they store**, not how they're used
- Booleans: use `is`, `has`, `should`, `can` prefix → `isLoggedIn`, `hasPermission`
- Collections: use plural noun → `userIds`, `activeOrders`
- Counters/indices: be explicit → `retryAttemptCount`, `currentPageIndex`
- Avoid: `data`, `info`, `value`, `temp`, `result`, `obj`, `item`, `thing`

### Functions / Methods
- Start with a **verb** → `fetchUserProfile`, `parseCSVRow`, `validateEmailFormat`
- Event handlers: `on` + event → `onSubmitForm`, `onSocketDisconnect`
- Getters: `get` + noun → `getTotalPrice`
- Predicates: `is/has/can` + condition → `isValidToken`

### Parameters
- Name after their **role in the function**, not their type
- `str` → `rawInputText`, `n` → `maxRetryCount`, `cb` → `onComplete`

### Language Conventions
| Language | Convention |
|----------|-----------|
| Python   | `snake_case` for variables/functions, `PascalCase` for classes |
| JavaScript/TypeScript | `camelCase` for variables/functions, `PascalCase` for classes, `UPPER_SNAKE` for constants |
| Java/C#  | `camelCase` for variables/methods, `PascalCase` for classes |
| Go       | `camelCase`, short but meaningful (`buf` → `responseBuffer`) |
| Rust     | `snake_case` for variables/functions |

---

## Edge Cases & Judgment Calls

### Already-good names
If a variable is already clear (e.g., `userId`, `emailAddress`), do **not** rename it. Only rename when clarity would meaningfully improve.

### Loop variables
Short loop variables are acceptable when the loop is trivial:
- `for i in range(3)` → keep `i` (index is obvious)
- But `for i in users: process(i)` → rename `i` to `user`

### External APIs / Libraries
Never rename variables that shadow or mirror external API field names (e.g., `event.target`, `req.body`, `err` in Node callbacks). These are conventional and readers expect them.

### Scope & Context
A short name may be fine in a 3-line function but terrible in a 100-line one. Use length of scope as a signal for how descriptive to be.

### Abbreviations
Expand common abbreviations unless they are industry-standard:
- `usr` → `user` ✅
- `btn` → `button` ✅  
- `HTML`, `URL`, `ID`, `DB` → keep as-is ✅

---

## Output Format

Always return:

1. **The complete renamed code** (syntax-highlighted if possible)
2. **A rename summary table** (only list names that actually changed)
3. **A short note** if any rename was a judgment call with alternatives

Do not add any comments to the renamed code — the names should be sufficient.