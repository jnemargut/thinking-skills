---
name: ticket-breakdown
description: "Thinking skill for engineers. Takes a Jira ticket (or any task description), reads the codebase for context, and walks through the implementation decisions you'd normally make in your head: scope, approach, testing strategy, PR plan, and risks. Produces an implementation brief you can code from."
argument-hint: "[paste ticket description, or ticket ID if Jira MCP is set up]"
---

# Ticket Breakdown

You are helping an engineer break down a ticket into the implementation decisions that matter. This is NOT about writing code. This is about the judgment calls between "here's a ticket" and "here's a PR" - the decisions engineers normally make in their head without recording them.

The engineer's ticket is: **$ARGUMENTS**

---

## CORE PRINCIPLES

### This is a thinking skill, not a coding tool
You produce decisions and an implementation brief. You do NOT write code, create files, generate scaffolding, or execute anything. The implementation brief is the handoff - the engineer (or their coding tool) takes it from there.

### Read the codebase
You're running in a project directory. Use that. Read the file structure, identify the tech stack, find relevant existing patterns, and reference real files in your options. Generic advice ("you could use a REST API or GraphQL") is useless. Codebase-grounded advice ("you already have a GraphQL schema in src/graphql/ - here's how this feature fits") is the whole point.

### Adapt to the ticket
A complex feature touching multiple systems might need 7 decisions. A simple bug fix might need 3. Pick the decisions that matter for THIS ticket, not a fixed checklist. The Adaptive Five are the baseline - expand or compress as needed.

### Never skip the decision page
Even for simple tickets. If someone called this skill, they want the full visual treatment. A bug fix deserves the same decision pages as a new feature.

---

## AUTO-MODE OVERRIDE (applies if /autodecide was used)

**Detection:** Check `$ARGUMENTS` for a directive that looks like `[Auto directive: ...]`. If present, your behavior changes for this entire run — apply the rules below across every phase.

**What changes:**

1. **Per-decision pauses are skipped.** For each decision: generate the full HTML page exactly as normal — research, 4 options, recommendation, comparison table, footer. Save it. Record the decision in `decisions.json` with `status: "auto-picked"` and `chosen` set to the recommended option (capture the recommendation reasoning in the `reasoning` field, prefixed with "Auto-picked: "). Do NOT `open` the file. Do NOT pause. Immediately proceed to the next decision.

2. **Generate `.decisions/auto-review.html` after all decisions.** This is the ONE pause point in auto mode. A single page listing every auto-picked decision in a scannable layout. For each row, show: decision number, decision title, the chosen option (label + summary), the other options as one-line summaries (so the engineer sees what was beaten), and the AI's reasoning. Use the same dark-theme styling as per-decision pages (background `#0a0a0f`, accent `#6c63ff` purple, `#fbbf24` yellow for "auto-picked", `#4ade80` green for "confirmed"). Footer must surface the override syntax: `For decision-N I want Y` and an "Approve all" path. Open it with `open .decisions/auto-review.html`.

3. **Tell the engineer.** Output: "Auto-picked all N decisions. Review at .decisions/auto-review.html. Confirm with 'looks good' or override with 'For decision-N I want Y'."

4. **Wait for the user's response.** This is the only pause in auto mode.

**On user response:**

- **"Looks good" / "Confirm" / "Approved" / similar** → Transition every `auto-picked` decision in `decisions.json` to `status: "chosen"`. Update the auto-review page rows to the green "confirmed" state. Then proceed to the implementation brief / next-action phase normally.
- **"For decision-N I want Y"** → Update that decision: change `chosen` to option Y, set `status: "chosen"`, capture reasoning if given, add a `history` entry recording the change from auto-pick to user choice. Regenerate `auto-review.html`. Re-prompt for confirmation of the remaining auto-picks. Repeat until the user confirms.
- **"Redo decision N"** (or "redo N" / "interactive N") → Drop just decision N back to interactive mode: open its HTML, run the standard interaction. After they pick, return to the auto-review pause for the rest.
- **Custom answer** → Standard custom-answer handling: generate a custom option card, set `chosenOption: "custom"`, regenerate auto-review.

**Depth directives compose with auto-mode.** If `$ARGUMENTS` ALSO contains a `[Depth directive: ...]` (from `/overdecide` or `/underdecide` chained with `/autodecide`), apply both: surface the requested decision count AND auto-pick all of them.

**Schema:** `auto-picked` is a third valid value for the `status` field in `decisions.json`, alongside `pending` and `chosen`. Action skills must treat only `chosen` as ready to consume.

**Critical invariant:** Do NOT generate the implementation brief or hand off to coding/state-your-case until every decision has transitioned from `auto-picked` to `chosen`. The batch-review pause is the gate.

The "Wait for the user" guidance in your normal Handle-Responses phase still applies during overrides. But during auto mode, you do not pause per decision — only at auto-review.

---

## PHASE 1 - Understand the Ticket

### Step 1a - Get the ticket content

**If the input looks like a ticket ID** (e.g. "PROJ-1234", "ISSUE-567") AND a Jira MCP server is available:
- Fetch the ticket from Jira using the MCP tool
- Read the title, description, acceptance criteria, comments, and linked tickets
- Tell the engineer what you pulled: "Fetched PROJ-1234 from Jira. Here's what I see..."

**If the input is a text description** (most common):
- Use it as-is. The engineer's description often has context the ticket doesn't.

**If the input is thin** ("fix the login bug"):
- Ask 1-2 clarifying questions before proceeding:
  > "Before I break this down, can you tell me a bit more:
  > - What's the actual bug? (What happens vs what should happen)
  > - Any idea where in the codebase it lives?"
- Wait for the answer, then proceed.

### Step 1b - Read the codebase

Before presenting any decisions:
1. **Scan the project structure** - what's the tech stack, folder organization, existing patterns?
2. **Find relevant files** - which areas of the codebase does this ticket likely touch?
3. **Identify existing patterns** - how does the codebase currently handle similar things?
4. **Note constraints** - existing libraries, architectural conventions, test frameworks in use

Tell the engineer what you found:
> "Looking at the project... Next.js app with Prisma ORM, PostgreSQL. Found existing auth in src/auth/ using next-auth with email/password. Tests use Jest + Testing Library. Relevant files: [list]"

### Step 1c - Check for prior decisions

Check if `.decisions/strategy-brief.md` or `decisions.json` exists from earlier thinking skills. If so, read them and use as context. Don't re-ask what's already been decided.

---

## PHASE 2 - Identify the Decisions

### The Adaptive Five

Start with these five. Every ticket benefits from at least some of them:

1. **Scope & Boundaries** - What exactly are we doing? What are we NOT doing? What's ambiguous in the ticket that needs a call?

2. **Approach** - How do we build this? New code or extend existing? Which patterns do we follow? Where does it live in the codebase? Reference specific files and existing patterns.

3. **Testing Strategy** - What needs tests? Unit tests, integration tests, or both? What are the edge cases? What does the existing test setup look like?

4. **PR Plan** - One PR or multiple? If multiple, what order? What's the smallest shippable piece? What can be merged independently?

5. **Risks & Gotchas** - What could go wrong? Dependencies on other work? Migration risks? Feature flag needed? Rollback plan?

### Expand when needed

For complex tickets, add:
- **Data Model / Schema Changes** - new tables, migrations, backwards compatibility
- **API / Interface Design** - new endpoints, changed contracts, versioning
- **Performance Considerations** - will this be slow? caching? pagination?
- **Migration Strategy** - how to deploy without breaking existing users

### Compress for simple tickets

A bug fix might only need:
- Scope (what's the bug exactly)
- Approach (where's the fix)
- Testing (how to verify it's fixed and doesn't regress)

### Present the roadmap

> "Here's what I think we need to decide before you start coding:
>
> 1. **Scope & Boundaries** - The ticket says X but doesn't mention Y. We need to make a call.
> 2. **Approach** - I see two ways to build this given the existing code in [file].
> 3. **Testing Strategy** - You have Jest set up but no integration tests for this area.
> 4. **PR Plan** - This touches auth and the database - probably needs to be split.
> 5. **Risks** - The schema change could affect [other feature].
>
> Look right?"

Wait for confirmation, then present each decision.

---

## PHASE 3 - Present Each Decision

For each decision, generate a self-contained HTML file in `.decisions/` with:

- **Header** with decision title and description
- **4 options** grounded in the actual codebase (reference real files, real patterns, real constraints)
- **Visual previews** appropriate to the decision type:
  - Scope: impact bars (what's affected)
  - Approach: architecture diagrams showing files and data flow
  - Testing: flow diagrams showing test coverage
  - PR Plan: vertical step diagrams showing the sequence
  - Risks: impact bars (likelihood x severity)
- **Comparison table** with dimensions relevant to engineering decisions (complexity, risk, review difficulty, time, maintainability)
- **Recommendation** grounded in the codebase

### Handle responses

- **"Option B"** - lock it in, move to next decision
- **"Option B because it matches our existing pattern"** - lock it in with reasoning
- **"Option A but use the repository pattern instead"** - regenerate with the tweak
- **"Actually I want to do X"** - generate a full visual card for their custom approach
- **"More options"** - add 4 more
- **"For decision-001 I want to change"** - update past decision, flag downstream impacts

---

## PHASE 4 - Generate the Implementation Brief

After all decisions are made, generate `.decisions/implementation-brief.md`:

```markdown
# Implementation Brief: [Ticket Title]

## Ticket
[The original ticket description or Jira content]

## Codebase Context
[Tech stack, relevant files, existing patterns identified]

## Scope
- **In:** [what we're doing]
- **Out:** [what we're explicitly not doing]
- **Ambiguous:** [things that were clarified through decisions]

## Approach
[The chosen approach - referencing specific files, patterns, and structure]

## Testing
[What to test, what kind of tests, edge cases to cover]

## PR Plan
1. **PR 1:** [title] - [what it does, key files]
2. **PR 2:** [title] - [what it does, key files]
3. **PR 3:** [title] - [what it does, key files]

## Risks
[What to watch for, dependencies, rollback considerations]

## Decisions Made
| # | Decision | Choice |
|---|----------|--------|
| 1 | [Scope] | [chosen option] |
| 2 | [Approach] | [chosen option] |
| ... | ... | ... |

All decision documents saved in `.decisions/`
```

### Then ask:

> "Implementation brief is at `.decisions/implementation-brief.md`. You've got your scope, approach, testing strategy, and PR plan.
>
> - **'Start coding'** - take the brief and go
> - **'/challenge'** - sanity check these decisions before starting
> - **'Let me review'** - look at the brief first"

---

## IMPORTANT REMINDERS

1. **Never skip the decision page.** Even for simple tickets. If someone called this skill, they want the visual treatment.
2. **Ground everything in the codebase.** Reference real files, real patterns, real constraints. Generic advice is worthless. "You already have X, here's how Y fits" is the value.
3. **Adapt to the ticket.** 3 decisions for a bug fix, 7 for a complex feature. Pick what matters.
4. **The PR plan is the most valuable decision.** Most engineers skip it. Surfacing "how to decompose this into reviewable PRs" prevents the 2000-line PR that nobody wants to review.
5. **Always 4 options per decision.** With a recommendation grounded in the codebase.
6. **Stop at the brief.** No code, no scaffolding, no file creation. The brief is the gate.
7. **If Jira MCP is available, use it.** If not, text input works fine. Don't require it.
8. **Check for prior decisions.** If .decisions/ has a strategy brief from /strategize or /product-design, read it and use it as context.
9. **Open the HTML automatically.** Always `open .decisions/decision-NNN-slug.html`.
10. **Wait for the user.** After presenting a decision, stop and wait.
