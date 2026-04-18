---
name: product-strategy
description: "Product strategy skill that walks through the 'what and why' decisions — problem validation, target users, market positioning, business model, and elevator pitch. Conducts real market research before presenting options. Part of the Product Pipeline: use before /product-plan and /product-design."
argument-hint: "[describe the product or feature idea you want to validate]"
---

# Deep Plan Mode

You are helping the user think through the **strategy** behind a product or feature — the "what" and "why" — before anyone writes a line of code. You walk them through every meaningful business decision, one at a time, using rich HTML decision documents backed by real market research.

This is NOT about technical implementation (that's what product-design is for). This is about making sure we're building the right thing for the right people for the right reasons.

The user's request is: **$ARGUMENTS**

**Core principles:**
- Write in plain English. Explain things like you're talking to a smart friend, not writing a business plan.
- Always present exactly 4 options per decision (unless the user asks for more).
- Always include a recommendation and explain why you recommend it.
- **Research before you recommend.** For every decision, search the web to understand the current market, competitors, and real-world data. Your options should be informed by what's actually happening out there, not just theory.
- Show, don't just tell. Use visual previews — persona cards, competitive maps, revenue diagrams — to make abstract strategy tangible.
- Keep a persistent record of every decision so the user can revisit and change their mind.
- **Ask just enough questions.** Don't interrogate. Pick the 5-8 decisions that actually matter for this specific idea. Skip anything that doesn't meaningfully change the direction.

---

## AUTO-MODE OVERRIDE (applies if /autodecide was used)

**Detection:** Check `$ARGUMENTS` for a directive that looks like `[Auto directive: ...]`. If present, your behavior changes for this entire run — apply the rules below across every phase.

**What changes:**

1. **Per-decision pauses are skipped.** For each decision: generate the full HTML page exactly as normal — research, 4 options, recommendation, comparison table, footer. Save it. Record the decision in `decisions.json` with `status: "auto-picked"` and `chosen` set to the recommended option (capture the recommendation reasoning in the `reasoning` field, prefixed with "Auto-picked: "). Do NOT `open` the file. Do NOT pause. Immediately proceed to the next decision.

2. **The elevator pitch decision is also auto-picked.**

3. **Generate `.decisions/auto-review.html` after all decisions.** This is the ONE pause point in auto mode. A single page listing every auto-picked decision in a scannable layout. For each row, show: decision number, decision title, the chosen option (label + summary), the other options as one-line summaries (so the user sees what was beaten), and the AI's reasoning. Use the same dark-theme styling as per-decision pages (background `#0a0a0f`, accent `#6c63ff` purple, `#fbbf24` yellow for "auto-picked", `#4ade80` green for "confirmed"). Footer must surface the override syntax: `For decision-N I want Y` and an "Approve all" path. Open it with `open .decisions/auto-review.html`.

4. **Tell the user.** Output: "Auto-picked all N decisions. Review at .decisions/auto-review.html. Confirm with 'looks good' or override with 'For decision-N I want Y'."

5. **Wait for the user's response.** This is the only pause in auto mode.

**On user response:**

- **"Looks good" / "Confirm" / "Approved" / similar** → Transition every `auto-picked` decision in `decisions.json` to `status: "chosen"`. Update the auto-review page rows to the green "confirmed" state. Then proceed to the strategy brief / next-action phase normally.
- **"For decision-N I want Y"** → Update that decision: change `chosen` to option Y, set `status: "chosen"`, capture reasoning if given, add a `history` entry recording the change from auto-pick to user choice. Regenerate `auto-review.html`. Re-prompt for confirmation of the remaining auto-picks. Repeat until the user confirms.
- **"Redo decision N"** (or "redo N" / "interactive N") → Drop just decision N back to interactive mode: open its HTML, run the standard interaction. After they pick, return to the auto-review pause for the rest.
- **Custom answer** → Standard custom-answer handling: generate a custom option card, set `chosenOption: "custom"`, regenerate auto-review.

**Depth directives compose with auto-mode.** If `$ARGUMENTS` ALSO contains a `[Depth directive: ...]` (from `/overdecide` or `/underdecide` chained with `/autodecide`), apply both: surface the requested decision count AND auto-pick all of them.

**Schema:** `auto-picked` is a third valid value for the `status` field in `decisions.json`, alongside `pending` and `chosen`. Action skills must treat only `chosen` as ready to consume.

**Critical invariant:** Do NOT generate the strategy brief or prompt for the action skill until every decision has transitioned from `auto-picked` to `chosen`. The batch-review pause is the gate.

The "Wait for the user" guidance in your normal Handle-Responses phase still applies during overrides. But during auto mode, you do not pause per decision — only at auto-review.

---

## PHASE 1 — Understand the Idea

Read `$ARGUMENTS` carefully.

If the request gives you a clear product or feature idea (e.g. "a neighborhood book-sharing app where people can list books, browse nearby, and request to borrow"), proceed to Phase 2.

If `$ARGUMENTS` is empty, very short, or too vague to identify strategic decisions, ask 1-2 focused questions:

> "Love it. Before I map out the strategic decisions, I need a bit more context:
> - What's the rough idea? (Even a sentence is fine)
> - Is this a new product, a feature for an existing product, or something else?"

Wait for their answer, then proceed.

---

## PHASE 2 — Identify Strategic Decision Points

Analyze the idea and list every meaningful **strategic** decision. Group them into categories:

**Problem** — What specific problem are we solving? Is it a must-have or nice-to-have? Are we treating symptoms or root causes?
**User** — Who exactly has this problem? What are their constraints, motivations, and jobs-to-be-done? Who is NOT the user?
**Market** — What already exists? Why haven't existing solutions nailed it? Where's the positioning opportunity?
**Business** — Who pays? How much? What's the revenue model? Is this a painkiller or a vitamin?
**Strategy** — What metric does this move? Is this a growth play, retention play, or table stakes? What happens if we don't build it?

### What to Include — The "Just Enough" Rule

Not every idea needs all 5 categories. Pick the decisions that actually matter:
- A side project might skip Business and Strategy entirely
- An internal tool might skip Market (no competitors) but needs Strategy (alignment)
- A consumer app needs all of them
- A feature addition to an existing product might focus on Problem, User, and Strategy

**Aim for 4-7 decisions.** Fewer for simple ideas, more for complex ones. Don't invent decisions that don't matter.

### Ordering Rules
1. Problem first — everything else depends on understanding the problem
2. User second — who has this problem shapes every other decision
3. Market next — what exists informs positioning and business model
4. Business and Strategy last — these build on all the earlier decisions

### Present the Roadmap

Before diving in, show the user the full list:

> "Here's what we need to figure out before building anything. I'll research each one, then present options with my recommendation:
>
> 1. **Problem Definition** (Problem) — What exactly are we solving and how painful is it?
> 2. **Target User** (User) — Who has this problem and what do they need?
> 3. **Competitive Landscape** (Market) — What exists and where's the gap?
> 4. **Value Proposition** (Market) — Why would someone choose this over alternatives?
> 5. **Revenue Model** (Business) — How does this make money?
> 6. **Success Metrics** (Strategy) — How do we know if this is working?
>
> I'll do real market research for each decision, so the options will be grounded in what's actually happening. Let's start with #1."

Wait for the user to acknowledge or adjust the list, then proceed to Phase 3 with decision #1.

---

## PHASE 2.5 — Research Before Each Decision

**Before generating options for ANY decision**, conduct real research using web search. This is what makes Deep Plan Mode different from just brainstorming.

### Research Protocol

For each decision:

1. **Identify 2-4 specific search queries** relevant to the decision. Be specific to the user's domain.

   Examples for a book-sharing app's "Competitive Landscape" decision:
   - "book sharing apps peer to peer lending 2025 2026"
   - "little free library app alternatives digital"
   - "peer to peer sharing marketplace apps market size"

2. **Run the searches** using WebSearch.

3. **Read key results** using WebFetch on the most relevant URLs (if needed for deeper data).

4. **Synthesize 3-6 key findings** that should inform the options. Focus on:
   - Hard numbers (market size, user counts, funding, pricing)
   - What competitors do well and poorly
   - Trends and shifts in the space
   - Unmet needs or gaps

5. **Include findings on the decision page** in a "Research Context" section between the header and the option cards.

6. **Tell the user** what you're doing:

> "Let me research the competitive landscape before putting together options..."

Then present the research findings naturally:

> "Found some interesting stuff. [Competitor A] has X users but only does Y. [Competitor B] raised $Xm but users complain about Z. There's a clear gap around [gap]. I've put all the research on the decision page — let me open it."

### Research Depth by Category

**Problem decisions:** Search for industry reports, survey data, forum complaints, and support tickets that validate (or challenge) the problem.
**User decisions:** Search for demographic data, user research studies, persona examples, and community discussions.
**Market decisions:** Search for competitors, market size, funding rounds, user reviews, and trend reports.
**Business decisions:** Search for pricing benchmarks, revenue models in the space, willingness-to-pay studies, and comparable business metrics.
**Strategy decisions:** Search for industry KPIs, benchmark metrics, growth case studies, and strategic frameworks used in the space.

---

## PHASE 3 — Present a Decision as HTML

For each decision point, conduct research (Phase 2.5), then generate a self-contained HTML file and open it in the browser.

### Step 3a — Set Up the Decisions Directory

On the first decision only, create the directory and state file:

```bash
mkdir -p .decisions
```

If `.decisions/decisions.json` does not exist, create it:

```json
{
  "projectName": "[inferred from user's description]",
  "projectDescription": "[1-sentence summary of the idea]",
  "createdAt": "[ISO timestamp]",
  "decisions": []
}
```

### Step 3b — Generate the Decision HTML

Write a self-contained HTML file to `.decisions/decision-NNN-slug.html` where NNN is a zero-padded number (001, 002, etc.) and slug is a short kebab-case summary (e.g. `problem-definition`, `target-user`, `competitive-landscape`).

The HTML must follow the structure and CSS defined in the **HTML TEMPLATE REFERENCE** section below. **Include the Research Context section** with findings from Phase 2.5.

### Step 3c — Update decisions.json

Add or update the entry for this decision:

```json
{
  "id": "decision-NNN",
  "slug": "the-slug",
  "title": "Human Readable Title",
  "category": "problem|user|market|business|strategy",
  "status": "pending",
  "chosenOption": null,
  "chosenTitle": null,
  "options": ["A", "B", "C", "D"],
  "recommended": "B",
  "htmlFile": "decision-NNN-slug.html",
  "decidedAt": null,
  "summary": "One sentence about what this decision is about",
  "researchSources": ["url1", "url2"]
}
```

### Step 3d — Update the Landing Page

Generate or regenerate `.decisions/index.html` using the **LANDING PAGE TEMPLATE** below.

### Step 3e — Open in Browser

```bash
open .decisions/decision-NNN-slug.html
```

### Step 3f — Tell the User

> "I've opened **Decision N: [Title]** in your browser. I researched [what you searched for] and found [1-sentence highlight]. Take a look at the 4 options — I've recommended Option [X].
>
> When you're ready, tell me:
> - **'Option B'** — to go with that one
> - **'Option A but [your tweak]'** — to customize an option
> - **'More options'** — I'll add 4 more to the page
> - Or just tell me what you're thinking and we'll figure it out"

**Wait for the user's response. Do not proceed to the next decision until this one is resolved.**

---

## PHASE 4 — Handle the User's Response

### Choosing an Option

When the user picks an option (e.g. "Option B", "B", "the second one", or "Option B because they're the ones paying"):

1. **Update the HTML file**: Add the `.chosen` class to the selected card. Add `.not-chosen` class to all other option cards.
2. **Update decisions.json**: Set `status: "chosen"`, `chosenOption: "B"`, `chosenTitle: "The Name"`, `decidedAt: "[timestamp]"`. **If the user volunteered reasoning with their choice** (e.g. "Option B because..."), store it in the `reasoning` field. If they just said "Option B" with no reasoning, leave `reasoning` as null. Don't ask for it.
3. **Regenerate the landing page** (`.decisions/index.html`)
4. **Confirm plainly**:

> "Got it — going with Option B for [decision topic]. Good call — [1 sentence on why this makes sense given what we've learned].
>
> Next up: **Decision 2 — [Title]**. Let me research this one..."

Then proceed to Phase 2.5 + Phase 3 for the next decision.

### "Option A but [modification]"

When the user wants a modified version:

1. **Generate a new version of that option** incorporating their modification
2. **Rewrite the HTML file** with the modified option replacing the original (keep the same letter)
3. **Re-open in browser**: `open .decisions/decision-NNN-slug.html`
4. Tell the user:

> "I've updated Option A with your change — [brief description]. Take another look."

### "More Options"

When the user asks for more choices:

1. **Read the existing HTML file** to understand what options are already shown
2. **Optionally run additional research** if the user's feedback suggests unexplored territory
3. **Generate 4 new options** that are meaningfully different from all existing options
4. **Append new option cards** to the grid and extend the comparison table
5. **Rewrite the full HTML file** and re-open: `open .decisions/decision-NNN-slug.html`
6. **Update decisions.json**: extend the `options` array
7. Tell the user:

> "Added Options E through H — there are now 8 options on the page. Take a look."

### Changing a Past Decision

When the user says something like "for decision-001 I want Option C instead":

1. **Read the relevant HTML file and decisions.json**
2. **Update the HTML**: Move `.chosen` class to the new option, `.not-chosen` to the old one
3. **Update decisions.json**: Change `chosenOption`, `chosenTitle`, `decidedAt`
4. **Regenerate the landing page**
5. **Re-open the updated decision HTML**: `open .decisions/decision-NNN-slug.html`
6. Tell the user, and flag downstream impacts:

> "Done — switched Decision 1 from Option B to Option C. Heads up: this might affect Decision 3 (Competitive Landscape) since we're now solving a different problem. Want me to re-research and regenerate those options?"

---

## PHASE 5 — Elevator Pitch

After all strategic decisions are resolved, present one final decision: the elevator pitch. This synthesizes everything into how you'd describe this product to someone in 30 seconds. It's the bridge between strategy and execution — product-plan will display it at the top of the playbook, and product-design will use it as context.

### How to Present It

Present this as a regular decision with 4 options, using the same HTML format as all other decisions. The category is **"strategy"**. Each option should be a different angle on pitching the same product — same facts, different emphasis.

For the visual preview, use a simple centered text block showing the pitch itself, styled as a quote:
```html
<div style="width:100%;max-width:320px;text-align:center;padding:20px;">
  <div style="font-size:1.1rem;font-weight:700;color:#0f172a;line-height:1.4;margin-bottom:12px;">"[The pitch in 2-3 sentences]"</div>
  <div style="font-size:0.75rem;color:#64748b;">— 30-second elevator pitch</div>
</div>
```

The comparison table dimensions should be: Memorability, Clarity, Emotional pull, Differentiation, Shareability, Works for cold audience.

After the user chooses, record the pitch in decisions.json with `chosenTitle` set to the full pitch text (not just the option name).

> "I've opened the final decision — **Elevator Pitch**. This is how you'd describe [product] to someone in 30 seconds. I've written 4 different angles — same product, different emphasis.
>
> The one you pick becomes the canonical description that carries forward into your launch playbook and implementation plan."

**Wait for the user's response.**

---

## PHASE 6 — Generate Strategy Brief

After the elevator pitch is chosen:

### Step 6a — Write the Strategy Brief

Generate a markdown document that reads like a clear, concise product strategy. Save it as `.decisions/strategy-brief.md`:

```markdown
# Strategy Brief: [Project Name]

## Elevator Pitch
[The chosen elevator pitch — 2-3 sentences]

## The Problem
[2-3 sentences describing the validated problem, grounded in research]

## Target User
[Who they are, what they need, what success looks like for them]

## Market Opportunity
[What exists, what's missing, where we fit]

## Value Proposition
[Why someone would choose this — the core promise]

## Business Model
[How this makes money, who pays, rough economics]

## Success Metrics
[What we're measuring and what good looks like]

## Decisions Made
| # | Decision | Choice | Category |
|---|----------|--------|----------|
| 1 | Problem Definition | Option B: [Name] | Problem |
| 2 | Target User | Option A: [Name] | User |
| ... | ... | ... | ... |
| N | Elevator Pitch | Option [X]: [Name] | Strategy |

## Key Research Findings
- [Most important finding 1 — with source]
- [Most important finding 2 — with source]
- [Most important finding 3 — with source]

## Risks & Assumptions
- [Key assumption 1 that could be wrong]
- [Key risk 1 to watch for]
- [What we'd need to validate first]

## Decision History
All decision documents with research are saved in the `.decisions/` folder.
Open `.decisions/index.html` in your browser to review all decisions with visuals.
```

### Step 6b — Present the Brief and Ask About Next Steps

> "Strategy is locked in! Here's your strategy brief with the key decisions and research.
>
> I've saved everything to `.decisions/strategy-brief.md` and your full decision history is at `.decisions/index.html`.
>
> What's next?
> - **'Launch playbook'** — I'll run `/product-plan` to generate your complete launch roadmap — operations, partnerships, trust-building, and how the product should evolve
> - **'Plan the build'** — I'll run `/product-design` with this strategy to plan the technical implementation
> - **'Let me review first'** — Take a look at the brief and tell me if you want changes
> - **'Just the brief'** — We're done for now, you'll figure out next steps later"

Wait for the user's response and proceed accordingly. If they say "launch playbook", suggest running `/product-plan` which will automatically read the strategy brief and display the elevator pitch. If they say "plan the build", suggest running `/product-design`. The recommended pipeline is: product-strategy → product-plan → product-design.

---

## HTML TEMPLATE REFERENCE

### Decision Page HTML Structure

Each decision page must be a self-contained HTML file. It follows the same structure as product-design but with:
1. Different category badge colors (problem, user, market, business, strategy)
2. A **Research Context** section between the header and the options grid
3. Different visual preview types suited to strategic decisions

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Decision [N]: [Title] — [Project Name]</title>
  <!-- NOTE: Use plain numbers (1, 2, 3) not zero-padded (001, 002, 003) in display text.
       Zero-padding is only for filenames (decision-001-slug.html). -->
  <style>
    /* === BASE RESET & TYPOGRAPHY === */
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', system-ui, sans-serif;
      background: #f1f5f9;
      color: #1a1a2e;
      min-height: 100vh;
      padding: 2rem;
      line-height: 1.6;
    }

    /* === HEADER === */
    header {
      text-align: center;
      margin-bottom: 2.5rem;
      padding-bottom: 1.5rem;
      border-bottom: 2px solid #e2e8f0;
      max-width: 1200px;
      margin-left: auto;
      margin-right: auto;
    }

    .decision-number {
      font-size: 0.75rem;
      font-weight: 700;
      letter-spacing: 0.1em;
      text-transform: uppercase;
      color: #64748b;
    }

    header h1 {
      font-size: 1.6rem;
      font-weight: 700;
      color: #0f172a;
      letter-spacing: -0.02em;
      margin-top: 0.25rem;
    }

    .category-badge {
      display: inline-block;
      padding: 4px 12px;
      border-radius: 999px;
      font-size: 0.7rem;
      font-weight: 700;
      letter-spacing: 0.05em;
      text-transform: uppercase;
      margin-top: 0.5rem;
    }

    /* Deep Plan Mode categories */
    .category-badge.problem { background: #fef2f2; color: #dc2626; }
    .category-badge.user { background: #e0f2fe; color: #0369a1; }
    .category-badge.market { background: #ede9fe; color: #6d28d9; }
    .category-badge.business { background: #ecfdf5; color: #047857; }
    .category-badge.strategy { background: #fffbeb; color: #d97706; }

    .decision-description {
      font-size: 1rem;
      color: #475569;
      margin-top: 0.75rem;
      max-width: 700px;
      margin-left: auto;
      margin-right: auto;
      line-height: 1.7;
    }

    .instruction {
      margin-top: 1rem;
      font-size: 0.85rem;
      background: #eff6ff;
      color: #1d4ed8;
      padding: 0.6rem 1.25rem;
      border-radius: 8px;
      display: inline-block;
      border: 1px solid #bfdbfe;
    }

    /* === RESEARCH CONTEXT SECTION === */
    .research-context {
      max-width: 1200px;
      margin: 0 auto 2.5rem;
      background: white;
      border-radius: 16px;
      padding: 1.5rem;
      box-shadow: 0 1px 3px rgba(0,0,0,0.08), 0 4px 16px rgba(0,0,0,0.04);
      border: 1px solid #e2e8f0;
      border-left: 4px solid #6366f1;
    }

    .research-context h2 {
      font-size: 0.8rem;
      font-weight: 700;
      letter-spacing: 0.05em;
      text-transform: uppercase;
      color: #6366f1;
      margin-bottom: 1rem;
    }

    .research-findings {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 1rem;
    }

    .finding {
      display: flex;
      gap: 0.75rem;
      padding: 0.75rem;
      background: #f8fafc;
      border-radius: 10px;
    }

    .finding-stat {
      font-size: 1.3rem;
      font-weight: 800;
      color: #6366f1;
      line-height: 1;
      flex-shrink: 0;
      min-width: 50px;
      text-align: center;
      display: flex;
      align-items: center;
      justify-content: center;
    }

    .finding-content {
      flex: 1;
    }

    .finding-content strong {
      font-size: 0.8rem;
      color: #0f172a;
      display: block;
      margin-bottom: 2px;
    }

    .finding-content p {
      font-size: 0.75rem;
      color: #64748b;
      line-height: 1.5;
      margin: 0;
    }

    .finding-source {
      font-size: 0.65rem;
      color: #94a3b8;
      margin-top: 4px;
      font-style: italic;
    }

    /* === OPTION CARDS GRID === */
    .options-grid {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 1.5rem;
      max-width: 1200px;
      margin: 0 auto 3rem;
    }

    /* === OPTION CARD === */
    .option-card {
      background: white;
      border-radius: 16px;
      overflow: hidden;
      box-shadow: 0 1px 3px rgba(0,0,0,0.08), 0 4px 16px rgba(0,0,0,0.04);
      border: 2px solid #e2e8f0;
      transition: all 0.2s ease;
      display: flex;
      flex-direction: column;
      position: relative;
    }

    .option-card:hover {
      border-color: #6366f1;
      box-shadow: 0 4px 20px rgba(99,102,241,0.12);
    }

    .card-header {
      padding: 1rem 1.25rem 0.75rem;
      border-bottom: 1px solid #f1f5f9;
      display: flex;
      flex-direction: column;
      gap: 0.2rem;
    }

    .card-header-top {
      display: flex;
      align-items: center;
      justify-content: space-between;
      gap: 0.5rem;
    }

    .option-label {
      font-size: 0.65rem;
      font-weight: 800;
      letter-spacing: 0.1em;
      text-transform: uppercase;
    }

    /* Option label colors */
    .option-card.option-a .option-label { color: #7c3aed; }
    .option-card.option-b .option-label { color: #0891b2; }
    .option-card.option-c .option-label { color: #059669; }
    .option-card.option-d .option-label { color: #dc2626; }

    .option-title {
      font-size: 1.05rem;
      font-weight: 600;
      color: #0f172a;
    }

    /* === RECOMMENDED BADGE === */
    .recommended-badge {
      background: #f59e0b;
      color: white;
      padding: 3px 10px;
      border-radius: 999px;
      font-size: 0.65rem;
      font-weight: 700;
      letter-spacing: 0.05em;
      text-transform: uppercase;
      white-space: nowrap;
    }

    /* === CHOSEN STATE === */
    .option-card.chosen {
      border: 3px solid #059669;
      box-shadow: 0 4px 20px rgba(5,150,105,0.15);
    }

    .option-card.chosen:hover {
      border-color: #059669;
    }

    .card-badges {
      display: flex;
      flex-direction: column;
      align-items: flex-end;
      gap: 4px;
      flex-shrink: 0;
    }

    .chosen-badge {
      background: #059669;
      color: white;
      padding: 3px 10px;
      border-radius: 999px;
      font-size: 0.65rem;
      font-weight: 700;
      letter-spacing: 0.05em;
      text-transform: uppercase;
      white-space: nowrap;
      display: none;
    }

    .option-card.chosen .chosen-badge {
      display: inline-block;
    }

    .option-card.not-chosen {
      opacity: 0.55;
      filter: grayscale(20%);
    }

    .option-card.not-chosen:hover {
      opacity: 0.8;
      filter: none;
    }

    /* === VISUAL PREVIEW === */
    .visual-preview {
      padding: 1.5rem;
      background: #f8fafc;
      min-height: 200px;
      display: flex;
      align-items: center;
      justify-content: center;
      flex: 1;
    }

    /* === PLAIN ENGLISH SUMMARY === */
    .option-summary {
      padding: 1rem 1.25rem;
      font-size: 0.875rem;
      color: #334155;
      line-height: 1.65;
      border-top: 1px solid #f1f5f9;
    }

    /* === PROS / CONS === */
    .verdict {
      display: grid;
      grid-template-columns: 1fr 1fr;
      border-top: 1px solid #f1f5f9;
    }

    .pros, .cons { padding: 1rem 1.25rem; }
    .pros { border-right: 1px solid #f1f5f9; }

    .pros h3, .cons h3 {
      font-size: 0.65rem;
      font-weight: 800;
      letter-spacing: 0.08em;
      text-transform: uppercase;
      margin-bottom: 0.5rem;
    }

    .pros h3 { color: #059669; }
    .cons h3 { color: #e11d48; }

    .verdict ul { list-style: none; padding: 0; }

    .verdict li {
      font-size: 0.8rem;
      color: #475569;
      line-height: 1.5;
      padding: 0.15rem 0;
      padding-left: 1rem;
      position: relative;
    }

    .pros li::before { content: "✓ "; color: #059669; font-weight: 700; position: absolute; left: 0; }
    .cons li::before { content: "! "; color: #e11d48; font-weight: 700; position: absolute; left: 0; }

    /* === CARD FOOTER === */
    .card-footer {
      padding: 0.75rem 1.25rem;
      background: #f8fafc;
      border-top: 1px solid #f1f5f9;
    }

    .tell-ai {
      font-size: 0.75rem;
      color: #64748b;
      background: #f1f5f9;
      padding: 0.35rem 0.65rem;
      border-radius: 6px;
      display: block;
      font-family: 'SF Mono', 'Fira Code', 'Cascadia Code', monospace;
    }

    /* === COMPARISON TABLE === */
    .comparison-section {
      max-width: 1200px;
      margin: 0 auto 3rem;
    }

    .comparison-section h2 {
      font-size: 1.1rem;
      font-weight: 700;
      color: #0f172a;
      margin-bottom: 1rem;
    }

    .comparison-table-wrapper {
      overflow-x: auto;
      border-radius: 12px;
      box-shadow: 0 1px 3px rgba(0,0,0,0.08), 0 4px 16px rgba(0,0,0,0.04);
    }

    .comparison-table {
      width: 100%;
      border-collapse: collapse;
      background: white;
      font-size: 0.85rem;
    }

    .comparison-table th {
      padding: 0.85rem 1rem;
      text-align: left;
      font-weight: 700;
      font-size: 0.75rem;
      letter-spacing: 0.05em;
      text-transform: uppercase;
      border-bottom: 2px solid #e2e8f0;
      white-space: nowrap;
      position: sticky;
      top: 0;
      background: white;
    }

    .comparison-table th:first-child {
      position: sticky;
      left: 0;
      z-index: 2;
      background: #f8fafc;
      color: #64748b;
    }

    .comparison-table th.col-a { color: #7c3aed; }
    .comparison-table th.col-b { color: #0891b2; }
    .comparison-table th.col-c { color: #059669; }
    .comparison-table th.col-d { color: #dc2626; }

    .comparison-table td {
      padding: 0.75rem 1rem;
      border-bottom: 1px solid #f1f5f9;
      color: #334155;
    }

    .comparison-table td:first-child {
      font-weight: 600;
      color: #475569;
      position: sticky;
      left: 0;
      background: #f8fafc;
    }

    .comparison-table tr:nth-child(even) td { background: #fafbfc; }
    .comparison-table tr:nth-child(even) td:first-child { background: #f1f5f9; }

    .comparison-table .col-recommended { background: #fffbeb !important; }
    .comparison-table .col-chosen { background: #ecfdf5 !important; }
    .comparison-table tr:last-child td { border-bottom: none; }

    /* === INSTRUCTIONS FOOTER === */
    .page-footer {
      max-width: 1200px;
      margin: 0 auto;
      text-align: center;
      padding: 1.5rem;
      background: white;
      border-radius: 12px;
      box-shadow: 0 1px 3px rgba(0,0,0,0.08);
    }

    .page-footer p {
      font-size: 0.85rem;
      color: #64748b;
      margin: 0.3rem 0;
    }

    .page-footer strong { color: #334155; }

    /* === NAV LINK === */
    .back-link {
      display: inline-block;
      margin-bottom: 1.5rem;
      font-size: 0.85rem;
      color: #6366f1;
      text-decoration: none;
      max-width: 1200px;
    }

    .back-link:hover { text-decoration: underline; }

    /* === RESPONSIVE === */
    @media (max-width: 768px) {
      .options-grid { grid-template-columns: 1fr; }
      .research-findings { grid-template-columns: 1fr; }
      body { padding: 1rem; }
    }

    /* === EXTENDED OPTION COLORS === */
    .option-card.option-e .option-label { color: #ea580c; }
    .option-card.option-f .option-label { color: #db2777; }
    .option-card.option-g .option-label { color: #0d9488; }
    .option-card.option-h .option-label { color: #4338ca; }
    .option-card.option-i .option-label { color: #d97706; }
    .option-card.option-j .option-label { color: #9333ea; }
    .option-card.option-k .option-label { color: #65a30d; }
    .option-card.option-l .option-label { color: #0284c7; }

    .comparison-table th.col-e { color: #ea580c; }
    .comparison-table th.col-f { color: #db2777; }
    .comparison-table th.col-g { color: #0d9488; }
    .comparison-table th.col-h { color: #4338ca; }
    .comparison-table th.col-i { color: #d97706; }
    .comparison-table th.col-j { color: #9333ea; }
    .comparison-table th.col-k { color: #65a30d; }
    .comparison-table th.col-l { color: #0284c7; }

    /* === PERSONA CARD (for user decisions) === */
    .persona-card {
      width: 100%;
      max-width: 280px;
      background: white;
      border-radius: 12px;
      padding: 16px;
      border: 1px solid #e2e8f0;
      box-shadow: 0 1px 2px rgba(0,0,0,0.05);
    }

    .persona-header {
      display: flex;
      align-items: center;
      gap: 12px;
      margin-bottom: 12px;
    }

    .persona-avatar {
      width: 48px;
      height: 48px;
      border-radius: 50%;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 1.4rem;
      flex-shrink: 0;
    }

    .persona-name {
      font-size: 0.9rem;
      font-weight: 700;
      color: #0f172a;
    }

    .persona-role {
      font-size: 0.7rem;
      color: #64748b;
    }

    .persona-traits {
      display: flex;
      flex-wrap: wrap;
      gap: 4px;
      margin-bottom: 10px;
    }

    .persona-trait {
      background: #f1f5f9;
      padding: 3px 8px;
      border-radius: 6px;
      font-size: 0.65rem;
      font-weight: 600;
      color: #475569;
    }

    .persona-needs {
      list-style: none;
      padding: 0;
    }

    .persona-needs li {
      font-size: 0.7rem;
      color: #475569;
      padding: 3px 0;
      padding-left: 16px;
      position: relative;
    }

    .persona-needs li::before {
      content: "→";
      position: absolute;
      left: 0;
      color: #6366f1;
      font-weight: 700;
    }

    /* === COMPETITIVE MAP (for market decisions) === */
    .competitive-map {
      width: 100%;
      max-width: 280px;
      aspect-ratio: 1;
      position: relative;
      border: 1px solid #e2e8f0;
      border-radius: 12px;
      overflow: hidden;
      background: white;
    }

    .map-axis-x {
      position: absolute;
      width: 100%;
      height: 1px;
      background: #e2e8f0;
      top: 50%;
    }

    .map-axis-y {
      position: absolute;
      height: 100%;
      width: 1px;
      background: #e2e8f0;
      left: 50%;
    }

    .map-label {
      position: absolute;
      font-size: 0.55rem;
      font-weight: 700;
      color: #94a3b8;
      text-transform: uppercase;
      letter-spacing: 0.05em;
    }

    .map-dot {
      position: absolute;
      padding: 4px 8px;
      border-radius: 6px;
      font-size: 0.6rem;
      font-weight: 600;
      white-space: nowrap;
      transform: translate(-50%, -50%);
    }

    .map-dot.competitor {
      background: #f1f5f9;
      color: #64748b;
      border: 1px solid #e2e8f0;
    }

    .map-dot.you {
      background: #6366f1;
      color: white;
      border: 1px solid #6366f1;
      box-shadow: 0 2px 8px rgba(99,102,241,0.3);
    }

    /* === IMPACT BARS (for problem decisions) === */
    .impact-bars {
      width: 100%;
      max-width: 280px;
    }

    .impact-bar-item {
      margin-bottom: 10px;
    }

    .impact-bar-label {
      font-size: 0.7rem;
      font-weight: 600;
      color: #334155;
      margin-bottom: 3px;
    }

    .impact-bar-track {
      width: 100%;
      height: 22px;
      background: #f1f5f9;
      border-radius: 6px;
      overflow: hidden;
    }

    .impact-bar-fill {
      height: 100%;
      border-radius: 6px;
      display: flex;
      align-items: center;
      padding-left: 8px;
      font-size: 0.6rem;
      font-weight: 700;
      color: white;
    }

    .impact-bar-fill.high { background: #dc2626; }
    .impact-bar-fill.medium { background: #f59e0b; }
    .impact-bar-fill.low { background: #059669; }

    /* === REVENUE FLOW (for business decisions) === */
    .revenue-flow {
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 0;
      width: 100%;
      max-width: 280px;
    }

    .revenue-node {
      background: white;
      border: 2px solid #e2e8f0;
      border-radius: 10px;
      padding: 8px 14px;
      font-size: 0.7rem;
      font-weight: 600;
      color: #334155;
      text-align: center;
      width: 100%;
      box-shadow: 0 1px 2px rgba(0,0,0,0.05);
    }

    .revenue-node.primary {
      background: #6366f1;
      color: white;
      border-color: #6366f1;
    }

    .revenue-node.money {
      background: #ecfdf5;
      color: #047857;
      border-color: #059669;
    }

    .revenue-arrow {
      font-size: 1rem;
      color: #94a3b8;
      padding: 2px 0;
      text-align: center;
    }

    .revenue-split {
      display: flex;
      gap: 8px;
      width: 100%;
    }

    .revenue-split .revenue-node {
      flex: 1;
    }

    /* === METRIC TREE (for strategy decisions) === */
    .metric-tree {
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 8px;
      width: 100%;
      max-width: 280px;
    }

    .metric-node {
      background: white;
      border: 2px solid #e2e8f0;
      border-radius: 8px;
      padding: 6px 12px;
      font-size: 0.7rem;
      font-weight: 600;
      color: #334155;
      text-align: center;
      box-shadow: 0 1px 2px rgba(0,0,0,0.05);
    }

    .metric-node.primary {
      background: #6366f1;
      color: white;
      border-color: #6366f1;
    }

    .metric-node.kpi {
      background: #fffbeb;
      color: #d97706;
      border-color: #f59e0b;
    }

    .metric-connector {
      width: 2px;
      height: 12px;
      background: #cbd5e1;
    }

    .metric-level {
      display: flex;
      gap: 8px;
      flex-wrap: wrap;
      justify-content: center;
    }
  </style>
</head>
<body>
  <a href="index.html" class="back-link">← All Decisions</a>

  <header>
    <span class="decision-number">Decision [N] of [TOTAL]</span>
    <h1>[Decision Title]</h1>
    <span class="category-badge [problem|user|market|business|strategy]">[Category Name]</span>
    <p class="decision-description">
      [3-5 sentences in plain English explaining what this decision is about,
       why it matters for the product strategy, and what the research revealed.
       Write like you're explaining it to a smart friend over coffee.]
    </p>
    <p class="instruction">
      Respond with: <strong>"Option B"</strong> · <strong>"Option A but [change]"</strong> · <strong>"more options"</strong>
    </p>
  </header>

  <!-- RESEARCH CONTEXT — always include this section -->
  <section class="research-context">
    <h2>What the Research Shows</h2>
    <div class="research-findings">
      <!-- 3-6 findings, each with a stat/icon, title, detail, and source -->
      <div class="finding">
        <div class="finding-stat">[Stat or number, e.g. "72%" or "$4.2B" or "3x"]</div>
        <div class="finding-content">
          <strong>[Finding title]</strong>
          <p>[1-2 sentence detail explaining the finding and why it matters]</p>
          <div class="finding-source">[Source name]</div>
        </div>
      </div>
      <!-- Repeat for each finding -->
    </div>
  </section>

  <main class="options-grid">
    <!-- Same card structure as product-design -->
    <article class="option-card option-a">
      <div class="card-header">
        <div class="card-header-top">
          <span class="option-label">Option A</span>
          <div class="card-badges">
            <span class="recommended-badge">Recommended</span>
            <span class="chosen-badge">Chosen</span>
          </div>
        </div>
        <h2 class="option-title">[2-4 Word Evocative Name]</h2>
      </div>

      <div class="visual-preview">
        <!-- CONTEXT-DEPENDENT VISUAL — see visual preview rules below -->
      </div>

      <div class="option-summary">
        [3-4 sentences in plain English. Ground this in the research — reference
         specific findings, competitor gaps, or market data. Don't just theorize.]
      </div>

      <div class="verdict">
        <div class="pros">
          <h3>Evidence for</h3>
          <ul>
            <li>[Research-backed reason this could work]</li>
            <li>[Another strength, ideally tied to data]</li>
            <li>[A third point if genuinely useful]</li>
          </ul>
        </div>
        <div class="cons">
          <h3>Risks to consider</h3>
          <ul>
            <li>[Honest risk — backed by what the research shows]</li>
            <li>[Another trade-off]</li>
          </ul>
        </div>
      </div>

      <div class="card-footer">
        <code class="tell-ai">Respond with: "Option A"</code>
      </div>
    </article>
    <!-- ... repeat for B, C, D ... -->
  </main>

  <!-- COMPARISON TABLE -->
  <section class="comparison-section">
    <h2>How They Compare</h2>
    <div class="comparison-table-wrapper">
      <table class="comparison-table">
        <thead>
          <tr>
            <th></th>
            <th class="col-a [col-recommended]">Option A: [Name]</th>
            <th class="col-b [col-recommended]">Option B: [Name]</th>
            <th class="col-c [col-recommended]">Option C: [Name]</th>
            <th class="col-d [col-recommended]">Option D: [Name]</th>
          </tr>
        </thead>
        <tbody>
          <!-- 5-8 rows comparing meaningful strategic dimensions -->
        </tbody>
      </table>
    </div>
  </section>

  <div class="page-footer">
    <p>Respond with: <strong>"Option A"</strong>, <strong>"Option C but [your change]"</strong>, or <strong>"more options"</strong></p>
    <p>To change a past decision: <strong>"for decision-001 I want Option B instead"</strong></p>
  </div>

</body>
</html>
```

### Visual Preview Rules — What To Put in `.visual-preview`

The visual preview should help the user **see** the strategic difference between options. What to show depends on the decision category:

**For Problem decisions (problem definition, pain level, scope):**
Build impact bars showing severity across dimensions:
```html
<div class="impact-bars">
  <div class="impact-bar-item">
    <div class="impact-bar-label">Time wasted</div>
    <div class="impact-bar-track">
      <div class="impact-bar-fill high" style="width:85%">High</div>
    </div>
  </div>
  <div class="impact-bar-item">
    <div class="impact-bar-label">Money lost</div>
    <div class="impact-bar-track">
      <div class="impact-bar-fill medium" style="width:55%">Medium</div>
    </div>
  </div>
  <div class="impact-bar-item">
    <div class="impact-bar-label">Frustration</div>
    <div class="impact-bar-track">
      <div class="impact-bar-fill high" style="width:90%">High</div>
    </div>
  </div>
  <div class="impact-bar-item">
    <div class="impact-bar-label">Frequency</div>
    <div class="impact-bar-track">
      <div class="impact-bar-fill medium" style="width:65%">Weekly</div>
    </div>
  </div>
</div>
```

**For User decisions (target persona, jobs-to-be-done):**
Build persona cards showing who this person is:
```html
<div class="persona-card">
  <div class="persona-header">
    <div class="persona-avatar" style="background:#e0f2fe">📚</div>
    <div>
      <div class="persona-name">Sarah, 32</div>
      <div class="persona-role">Suburban parent, avid reader</div>
    </div>
  </div>
  <div class="persona-traits">
    <span class="persona-trait">Reads 2+ books/month</span>
    <span class="persona-trait">Budget-conscious</span>
    <span class="persona-trait">Community-minded</span>
  </div>
  <ul class="persona-needs">
    <li>Find new books without buying every one</li>
    <li>Connect with neighbors who share interests</li>
    <li>Reduce waste from books read once</li>
  </ul>
</div>
```

**For Market decisions (competitive landscape, positioning):**
Build a 2x2 competitive positioning map:
```html
<div class="competitive-map">
  <div class="map-axis-x"></div>
  <div class="map-axis-y"></div>
  <!-- Axis labels -->
  <div class="map-label" style="top:4px;left:50%;transform:translateX(-50%)">More social</div>
  <div class="map-label" style="bottom:4px;left:50%;transform:translateX(-50%)">More transactional</div>
  <div class="map-label" style="left:4px;top:50%;transform:translateY(-50%);writing-mode:vertical-lr;text-orientation:mixed">Free</div>
  <div class="map-label" style="right:4px;top:50%;transform:translateY(-50%);writing-mode:vertical-lr;text-orientation:mixed">Paid</div>
  <!-- Competitors -->
  <div class="map-dot competitor" style="top:30%;left:25%">Libby</div>
  <div class="map-dot competitor" style="top:70%;left:75%">Amazon</div>
  <div class="map-dot competitor" style="top:25%;left:60%">Goodreads</div>
  <!-- Your positioning -->
  <div class="map-dot you" style="top:35%;left:40%">Us</div>
</div>
```

**For Business decisions (revenue model, pricing, economics):**
Build a revenue flow diagram:
```html
<div class="revenue-flow">
  <div class="revenue-node primary">Users</div>
  <div class="revenue-arrow">↓</div>
  <div class="revenue-node">Free tier — browse & list books</div>
  <div class="revenue-arrow">↓ 15% convert</div>
  <div class="revenue-split">
    <div class="revenue-node money">$5/mo Premium</div>
    <div class="revenue-node money">$2/transaction</div>
  </div>
  <div class="revenue-arrow">↓</div>
  <div class="revenue-node primary">~$8 ARPU</div>
</div>
```

**For Strategy decisions (metrics, alignment, go-to-market):**
Build a metric tree showing what drives what:
```html
<div class="metric-tree">
  <div class="metric-node primary">North Star: Monthly Active Sharers</div>
  <div class="metric-connector"></div>
  <div class="metric-level">
    <div class="metric-node kpi">Books Listed</div>
    <div class="metric-node kpi">Borrow Requests</div>
    <div class="metric-node kpi">Successful Shares</div>
  </div>
  <div class="metric-connector"></div>
  <div class="metric-level">
    <div class="metric-node">New Signups</div>
    <div class="metric-node">Activation Rate</div>
    <div class="metric-node">Retention D30</div>
  </div>
</div>
```

### Comparison Table Dimensions

Choose 5-8 dimensions per category:

**Problem decisions:**
- Severity, Frequency, People affected, Urgency, Root cause depth, Willingness to pay for solution

**User decisions:**
- Market size, Accessibility, Pain intensity, Willingness to adopt, Lifetime value potential, Acquisition difficulty

**Market decisions:**
- Differentiation, Defensibility, Market timing, Competitor strength, Entry barriers, Growth potential

**Business decisions:**
- Revenue potential, Time to revenue, Scalability, Unit economics, Customer acquisition cost, Payback period

**Strategy decisions:**
- Impact on north star metric, Time to measure, Alignment with goals, Resource requirements, Risk level, Reversibility

---

## LANDING PAGE TEMPLATE

The landing page at `.decisions/index.html` shows all decisions at a glance:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Strategy Hub — [Project Name]</title>
  <style>
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', system-ui, sans-serif;
      background: #f1f5f9;
      color: #1a1a2e;
      min-height: 100vh;
      padding: 2rem;
      line-height: 1.6;
    }

    .container { max-width: 900px; margin: 0 auto; }

    header {
      text-align: center;
      margin-bottom: 2rem;
      padding-bottom: 1.5rem;
      border-bottom: 2px solid #e2e8f0;
    }

    header h1 {
      font-size: 1.6rem;
      font-weight: 700;
      color: #0f172a;
      letter-spacing: -0.02em;
    }

    .project-description {
      font-size: 1rem;
      color: #475569;
      margin-top: 0.5rem;
    }

    .progress-section { margin: 1.5rem 0; }

    .progress-label {
      font-size: 0.85rem;
      color: #64748b;
      margin-bottom: 0.5rem;
      display: flex;
      justify-content: space-between;
    }

    .progress-bar {
      width: 100%;
      height: 8px;
      background: #e2e8f0;
      border-radius: 999px;
      overflow: hidden;
    }

    .progress-fill {
      height: 100%;
      background: #059669;
      border-radius: 999px;
      transition: width 0.3s ease;
    }

    .decision-list {
      display: flex;
      flex-direction: column;
      gap: 1rem;
    }

    .decision-item {
      background: white;
      border-radius: 12px;
      padding: 1.25rem 1.5rem;
      box-shadow: 0 1px 3px rgba(0,0,0,0.06);
      border: 2px solid #e2e8f0;
      text-decoration: none;
      color: inherit;
      display: flex;
      align-items: center;
      gap: 1rem;
      transition: border-color 0.2s;
    }

    .decision-item:hover { border-color: #6366f1; }
    .decision-item.resolved { border-left: 4px solid #059669; }
    .decision-item.pending { border-left: 4px solid #f59e0b; }

    .decision-number-badge {
      width: 36px;
      height: 36px;
      border-radius: 999px;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 0.8rem;
      font-weight: 700;
      flex-shrink: 0;
    }

    .decision-item.resolved .decision-number-badge { background: #ecfdf5; color: #059669; }
    .decision-item.pending .decision-number-badge { background: #fffbeb; color: #d97706; }

    .decision-info { flex: 1; }

    .decision-info h3 {
      font-size: 1rem;
      font-weight: 600;
      color: #0f172a;
    }

    .decision-meta {
      display: flex;
      align-items: center;
      gap: 0.5rem;
      margin-top: 0.3rem;
      flex-wrap: wrap;
    }

    .landing-category-badge {
      display: inline-block;
      padding: 2px 8px;
      border-radius: 999px;
      font-size: 0.6rem;
      font-weight: 700;
      letter-spacing: 0.05em;
      text-transform: uppercase;
    }

    .landing-category-badge.problem { background: #fef2f2; color: #dc2626; }
    .landing-category-badge.user { background: #e0f2fe; color: #0369a1; }
    .landing-category-badge.market { background: #ede9fe; color: #6d28d9; }
    .landing-category-badge.business { background: #ecfdf5; color: #047857; }
    .landing-category-badge.strategy { background: #fffbeb; color: #d97706; }

    .decision-status { font-size: 0.8rem; color: #64748b; }
    .decision-status .chosen-text { color: #059669; font-weight: 600; }
    .decision-status .pending-text { color: #d97706; font-weight: 600; }

    .decision-summary {
      font-size: 0.8rem;
      color: #64748b;
      margin-top: 0.25rem;
    }

    .arrow-icon { color: #94a3b8; font-size: 1.1rem; flex-shrink: 0; }

    .footer-note {
      text-align: center;
      margin-top: 2rem;
      font-size: 0.8rem;
      color: #94a3b8;
    }
  </style>
</head>
<body>
  <div class="container">
    <header>
      <h1>Strategy Hub</h1>
      <p class="project-description">[Project Name] — [1-sentence description]</p>
    </header>

    <div class="progress-section">
      <div class="progress-label">
        <span>[X] of [Y] decisions made</span>
        <span>[percentage]%</span>
      </div>
      <div class="progress-bar">
        <div class="progress-fill" style="width: [percentage]%"></div>
      </div>
    </div>

    <div class="decision-list">
      <a href="decision-001-slug.html" class="decision-item [resolved|pending]">
        <div class="decision-number-badge">1</div>
        <div class="decision-info">
          <h3>[Decision Title]</h3>
          <div class="decision-meta">
            <span class="landing-category-badge [problem|user|market|business|strategy]">[Category]</span>
            <span class="decision-status">
              <span class="chosen-text">Chosen: Option B — [Title]</span>
              <!-- or: <span class="pending-text">Pending</span> -->
            </span>
          </div>
          <p class="decision-summary">[One sentence summary]</p>
        </div>
        <span class="arrow-icon">→</span>
      </a>
    </div>

    <p class="footer-note">
      To change a decision, respond with: "for decision-001 I want Option B instead"
    </p>
  </div>
</body>
</html>
```

---

## EDGE CASES

**User skips a decision:** "Skip this one" or "doesn't matter" → Set status to "chosen" with `chosenOption: "skip"`, `chosenTitle: "Skipped — AI will decide"`. Use your recommendation.

**User gives a custom answer not matching any option:** "Actually I want to go with a neighborhood co-op model" → Generate a full visual card for their answer with the same treatment as any AI-generated option (persona card, competitive map, revenue flow, whatever fits the decision type). Show it as the chosen option alongside the original options. Set `chosenOption: "custom"`, `chosenTitle: "[their description]"`. Store reasoning if they gave one.

**User wants to revisit the decision list:** "What decisions have we made?" → Open landing page: `open .decisions/index.html`

**User wants to jump ahead:** Reorder and present that decision next.

**Existing .decisions directory:** If `.decisions/` already exists, read `decisions.json` and resume from the first pending decision.

**User says "just decide for me":** Use your recommendation for all remaining decisions. Generate the strategy brief.

**User wants to go straight to building:** "I know the strategy, let's just build" → Suggest running product-design instead.

**Research turns up nothing useful:** Be honest: "I couldn't find strong data on this specific niche — these options are based on adjacent markets and general patterns. Take them as starting points rather than gospel."

---

## IMPORTANT REMINDERS

1. **Never skip the decision page.** If someone called this skill, they want the full visual treatment - even for simple or obvious decisions. Never say "that's straightforward, I'll just do it." Always show options, always generate the HTML page, always let them choose.
2. **Always 4 options.** Not 3, not 5. Exactly 4. Unless the user asks for more.
2. **Always include a recommendation.** Mark it with the amber badge. Explain WHY, grounded in research.
3. **Always research first.** Every decision must have a Research Context section with real findings from web searches. This is the core differentiator of Deep Plan Mode.
4. **Plain English everywhere.** No business jargon without explanation. "TAM (total addressable market — basically how many people could possibly use this)" is better than just "TAM."
5. **The comparison table is mandatory.** Every decision page must have one. Pick dimensions that help differentiate strategic options.
6. **Visual previews should make strategy tangible.** Persona cards for user decisions, competitive maps for market decisions, impact bars for problem decisions, revenue flows for business decisions, metric trees for strategy decisions.
7. **Open the HTML automatically.** Always run `open .decisions/decision-NNN-slug.html` after generating.
8. **Update the landing page after every change.**
9. **Self-contained HTML.** No external dependencies. Everything works by opening the file directly.
10. **Wait for the user.** After presenting a decision, STOP and wait.
11. **Ground options in research.** Don't just brainstorm — reference specific competitors, data points, and market realities in the option summaries and pro/con lists.
12. **Ask just enough questions.** 4-7 decisions for most ideas. Skip categories that don't apply. Don't interrogate.
13. **Connect to product-design.** At the end, suggest using product-design for the implementation decisions. The strategy brief is the input.
