---
name: ticket-writer
description: >
  Generate a detailed, developer-ready ticket title and description from a vague user issue report.
  Use this skill whenever the user mentions a bug, feature request, task, or problem they want
  tracked — even if they say "create a ticket", "log this issue", "write up a ticket", "make a
  Jira/Plane/Linear/GitHub issue", or casually describe something broken. Also trigger when the user
  describes unexpected behavior, a missing feature, a performance problem, or anything that sounds
  like it belongs in an issue tracker. The skill searches the codebase for relevant context and asks
  targeted clarifying questions before producing the final ticket. Do NOT use for support chat
  replies, general code review, or documentation generation unless the user explicitly wants a ticket.
---

# Ticket Writer Skill

Generate a detailed, developer-ready ticket title and description by combining:
1. **Codebase search** — find relevant files, functions, configs, and error patterns
2. **Targeted clarifying questions** — ask only what's essential, never more than 3 at once
3. **Structured ticket output** — title + description ready to paste into any issue tracker

---

## Workflow

### Phase 1 — Parse the Issue

Extract from the user's message:
- **What**: the symptom or desired behaviour
- **Where**: component, page, API route, or service (if mentioned)
- **Impact**: who is affected, how often, severity signals

If the user's message is fewer than 20 words or lacks a clear "what", ask one open question before proceeding: *"Can you tell me a bit more about what's going wrong / what you need?"*

---

### Phase 2 — Search the Codebase

Run targeted searches to ground the ticket in real code. Use `bash_tool` (or whatever file/search tool is available).

**Search strategies (pick the ones relevant to the issue):**

```bash
# 1. Keyword / symptom search
grep -rn "keyword" --include="*.ts" --include="*.tsx" --include="*.py" --include="*.js" . | head -40

# 2. File/component search by name
find . -type f -name "*ComponentName*" | head -20

# 3. Error message search
grep -rn "error text or exception name" . | head -20

# 4. API route / endpoint search
grep -rn "/api/route-name\|routeName\|handler" --include="*.ts" --include="*.py" . | head -20

# 5. Config / env variable search
grep -rn "ENV_VAR\|config key" . | head -20

# 6. Related test files (understand expected behaviour)
find . -type f -name "*.test.*" | xargs grep -l "keyword" 2>/dev/null | head -10
```

**What to capture from search results:**
- File paths and line numbers of relevant code
- Function/class names involved
- Related config keys or environment variables
- Existing error handling or TODO comments that relate to the issue
- Any `FIXME`, `HACK`, or `@deprecated` annotations near the affected code

If no codebase is accessible (no filesystem tools available), skip to Phase 3 and note in the ticket that file references could not be verified.

---

### Phase 3 — Identify Knowledge Gaps → Ask Clarifying Questions

After parsing + searching, identify what's still unknown. Ask **at most 3 questions**, prioritising by impact on ticket quality. Use `ask_user_input_v0` when options are enumerable; otherwise ask inline.

**Question bank — pick only what's missing:**

| Gap | Example question |
|-----|-----------------|
| Reproduction steps unclear | "What's the exact sequence of steps to reproduce this?" |
| Expected vs actual behaviour | "What did you expect to happen, and what happened instead?" |
| Environment / scope | "Does this happen in all environments (dev/staging/prod) or just one?" |
| Frequency | "Is this always broken, intermittent, or did it just start?" |
| User impact | "Who is affected — all users, specific roles, or specific accounts?" |
| Priority / urgency | "Is this blocking anyone right now, or is it a nice-to-have fix?" |
| Acceptance criteria | "How will we know the ticket is done? Any specific tests or behaviours to verify?" |

**Rules:**
- Never ask something that was already answered in the user's message
- Never ask something you found the answer to in the codebase
- Batch multiple questions into one `ask_user_input_v0` call if possible
- If you truly have everything needed, skip this phase entirely

---

### Phase 4 — Generate the Ticket

Produce the ticket using the template below. Fill every section — if information is genuinely unknown after searching and asking, write `_To be confirmed_` rather than guessing.

---

## Output Template

````markdown
## 🎫 Ticket

**Title:** `[TYPE] Short, specific, action-oriented summary (≤10 words)`

> Title types: `[BUG]`, `[FEAT]`, `[TASK]`, `[CHORE]`, `[PERF]`, `[SECURITY]`

---

### Summary
One or two sentences describing the problem or goal in plain language. Written for someone who wasn't in the conversation.

---

### Background / Context
- Why does this matter? What system or feature is affected?
- Reference relevant code locations found during search:
  - `src/path/to/file.ts:42` — brief note on what this code does
  - `src/path/to/other.ts:88` — brief note

---

### Steps to Reproduce *(for bugs)*
1. Step one
2. Step two
3. Step three

**Expected:** What should happen  
**Actual:** What happens instead

---

### Proposed Solution / Approach *(if known)*
- High-level implementation idea
- Specific files/functions likely to change:
  - `src/path/to/file.ts` — what needs to change and why

---

### Acceptance Criteria
- [ ] Criterion 1
- [ ] Criterion 2
- [ ] All existing tests pass
- [ ] New test(s) added for the changed behaviour

---

### Additional Notes
- Environment: `dev` / `staging` / `prod` / `all`
- Frequency: `always` / `intermittent` / `once`
- Affected users: _e.g. all users / admin role / specific tenant_
- Priority signals: _e.g. blocking release, regression since v2.3_
- Related issues / PRs: _if known_
````

---

## Quality Rules

**Title must:**
- Start with a type tag: `[BUG]`, `[FEAT]`, `[TASK]`, `[CHORE]`, `[PERF]`, `[SECURITY]`
- Be ≤10 words after the tag
- Be specific: mention the component, route, or behaviour — not generic ("Fix bug in system")

**Description must:**
- Include at least one real file path from the codebase search (or note why it's unavailable)
- Have measurable acceptance criteria — not "it works"
- Separate bug reproduction steps from feature acceptance criteria

**Never:**
- Hallucinate file paths or function names not found in the search
- Leave the "Steps to Reproduce" section empty for a bug
- Skip acceptance criteria

---

## Handling Edge Cases

| Situation | Action |
|-----------|--------|
| No codebase access | Generate ticket from conversation only; note "File references unverified — confirm paths before assigning" |
| Issue is too vague after one clarifying round | Add a `## Open Questions` section at the bottom listing what the assignee should clarify before starting |
| Multiple issues bundled together | Point out the bundle: *"It looks like there are 2 separate issues here. Should I write one ticket or two?"* |
| User provides a stack trace | Include the key exception class and message verbatim in the Background section; search for the throw site in the codebase |
| Feature request with no acceptance criteria | Draft 3–5 acceptance criteria based on the description and ask the user to confirm or adjust |