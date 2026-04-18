---
name: core-principles
description: "Derive tension-based principles (tenets) for a strategy, product, or initiative. Amazon-style 'X over Y' format where both sides are genuinely good — you're declaring which one wins when they conflict. Surfaces the real moral, business, and user tensions hiding in your idea and forces you to take a stance."
argument-hint: "[describe your strategy, product idea, or initiative — what you're building and who it's for]"
---

# Core Principles

You are helping the user discover and articulate the core principles (tenets) that should guide their strategy, product, or initiative. These are NOT generic values. They are tension-based declarations in the Amazon tenet tradition: **"X over Y"** where both X and Y are genuinely good things, but you're declaring which one wins when they conflict.

The user's idea is: **$ARGUMENTS**

## What Makes a Good Tenet

A good tenet has three qualities:

1. **Real tension.** Both sides must be things a reasonable person would want. "Quality over sloppiness" is not a tenet — nobody advocates for sloppiness. "Depth of experience over breadth of features" IS a tenet — both are legitimately valuable.

2. **Takes a stance.** It tells people what to do when two good things conflict. It's a pre-made decision. When a team member faces a tradeoff, they can point to the tenet and say "we already decided this."

3. **Grounded in the specific situation.** Generic principles like "move fast" could apply to anything. Good tenets emerge from the particular tensions in THIS product, THIS market, THIS mission.

The three forces that create tension in any initiative:
- **User needs** -- what people actually want and benefit from
- **Business needs** -- what makes the initiative sustainable
- **Humanity needs** -- what's right for the world, the community, the long-term

Most interesting tenets live at the intersection where two or three of these forces pull in different directions. The skill's job is to surface those tensions and help the user decide which way to lean.

## Format of a Tenet

Each tenet has three parts:
- **Name:** A short phrase in "X over Y" format (e.g., "Community safety over growth velocity")
- **Stance:** 1-2 sentences explaining why you're choosing X, written as a truth about the world — not a corporate mission statement. It should sound like a belief, something you'd say in a conversation.
- **The tension:** A brief acknowledgment of why Y is also good and when you might revisit this.

Example:
> **Transparency of process over speed of resolution**
> People trust systems where they can see what's happening, even if it takes longer. When you hide the mechanism to move faster, you create anxiety that costs more than the time you saved.
> *We're choosing this over speed of resolution, which matters too — but trust compounds and speed doesn't.*

---

## CORE PRINCIPLES (for this skill itself)

- **Surface real tensions, not platitudes.** If someone wouldn't reasonably argue the other side, it's not a tenet.
- **Research the domain.** Find real examples of companies/products that chose each side of the tension. Show what happened.
- **Write like a person, not a corporation.** Tenets should sound like beliefs, not mission statements. No buzzwords. No "leverage synergies."
- **The user decides.** Present the tension clearly, recommend a side, but never assume. The whole point is that reasonable people can disagree.
- **Connect tenets to each other.** Good principle sets have internal coherence — they tell a story about what kind of thing this is.
- Never skip the decision page, even if a tenet seems obvious.
- Always present exactly 4 options per tenet.
- Always include a recommendation with reasoning.

---

## AUTO-MODE OVERRIDE (applies if /autodecide was used)

**Detection:** Check `$ARGUMENTS` for a directive that looks like `[Auto directive: ...]`. If present, your behavior changes for this entire run — apply the rules below across every phase.

**What changes:**

1. **Per-tenet pauses are skipped.** For each tenet decision: generate the full HTML page exactly as normal — research, 4 options, recommendation, comparison table, footer. Save it. Record the decision in `decisions.json` with `status: "auto-picked"` and `chosen` set to the recommended tenet (capture the recommendation reasoning in the `reasoning` field, prefixed with "Auto-picked: "). Do NOT `open` the file. Do NOT pause. Immediately proceed to the next tenet.

2. **Generate `.decisions/auto-review.html` after all tenets.** This is the ONE pause point in auto mode. A single page listing every auto-picked tenet in a scannable layout. For each row, show: tenet number, the tension being resolved, the chosen "X over Y" stance (label + summary), the other options as one-line summaries, and the AI's reasoning. Use the same dark-theme styling as per-decision pages (background `#0a0a0f`, accent `#6c63ff` purple, `#fbbf24` yellow for "auto-picked", `#4ade80` green for "confirmed"). Footer must surface the override syntax: `For decision-N I want Y` and an "Approve all" path. Open it with `open .decisions/auto-review.html`.

3. **Tell the user.** Output: "Auto-picked all N tenets. Review at .decisions/auto-review.html. Confirm with 'looks good' or override with 'For decision-N I want Y'."

4. **Wait for the user's response.** This is the only pause in auto mode.

**On user response:**

- **"Looks good" / "Confirm" / "Approved" / similar** → Transition every `auto-picked` tenet in `decisions.json` to `status: "chosen"`. Update the auto-review page rows to the green "confirmed" state. Then proceed to the principles document / next-action phase normally.
- **"For decision-N I want Y"** → Update that tenet: change `chosen` to option Y, set `status: "chosen"`, capture reasoning if given, add a `history` entry recording the change from auto-pick to user choice. Regenerate `auto-review.html`. Re-prompt for confirmation of the remaining auto-picks. Repeat until the user confirms.
- **"Redo decision N"** → Drop just decision N back to interactive mode: open its HTML, run the standard interaction. After they pick, return to the auto-review pause for the rest.
- **Custom answer** → Standard custom-answer handling: generate a custom option card, set `chosenOption: "custom"`, regenerate auto-review.

**Depth directives compose with auto-mode.** If `$ARGUMENTS` ALSO contains a `[Depth directive: ...]` (from `/overdecide` or `/underdecide` chained with `/autodecide`), apply both: surface the requested tenet count AND auto-pick all of them.

**Schema:** `auto-picked` is a third valid value for the `status` field in `decisions.json`, alongside `pending` and `chosen`. Action skills must treat only `chosen` as ready to consume.

**Critical invariant:** Do NOT finalize the principles document or hand off to a downstream skill until every tenet has transitioned from `auto-picked` to `chosen`. The batch-review pause is the gate.

The "Wait for the user" guidance in your normal Handle-Responses phase still applies during overrides. But during auto mode, you do not pause per tenet — only at auto-review.

---

## PHASE 1 -- Understand the Idea

Read `$ARGUMENTS` carefully.

Check if `.decisions/strategy-brief.md` or `decisions.json` exists. If prior decisions exist from another skill (like /strategize or /product-strategy), read them. Those decisions are context — they tell you what the user has already decided about WHAT they're building. Your job is to figure out the principles that should guide HOW they build it.

If the input is clear enough to identify tensions, proceed to Phase 2.

If it's vague, ask 1-2 focused questions:

> "I want to help you find the principles hiding in this idea. Before I dig in:
> - Who benefits from this, and who might not?
> - What's the hardest tradeoff you've already felt but haven't resolved?"

Wait for their answer.

---

## PHASE 2 -- Research and Surface Tensions

This is the heart of the skill. Before identifying specific tenets, you need to understand where the real tensions live.

### Step 1: Research the Domain

Run 3-5 web searches targeting:
- **Ethical tensions** in this space. Search for controversies, criticism, unintended consequences of similar products/initiatives.
- **Business model tensions.** How do similar products balance revenue with user welfare? What happens when growth conflicts with quality?
- **User trust tensions.** What do users in this space complain about? Where do they feel exploited vs. served?
- **Community/societal impact.** What are the second-order effects? Who are the stakeholders beyond direct users?
- **Existing principles/tenets** from companies in similar spaces. What stances have others taken?

### Step 2: Map the Tension Landscape

From your research, identify exactly 3 genuine tensions — the 3 that matter most. Fewer principles means each one carries more weight and is easier to remember. Each tension should sit at the intersection of at least two of the three forces (user needs, business needs, humanity needs).

Common tension categories (adapt to the situation — these are starting points, not a checklist):

- **Growth vs. Safety** -- scaling fast vs. protecting users from harm
- **Revenue vs. Trust** -- monetization approaches that help the business but might erode user trust
- **Personalization vs. Privacy** -- better experiences vs. data collection
- **Engagement vs. Wellbeing** -- keeping people coming back vs. respecting their time/attention
- **Speed vs. Thoroughness** -- shipping fast vs. getting it right
- **Access vs. Quality** -- serving everyone vs. serving fewer people well
- **Automation vs. Human Touch** -- efficiency vs. genuine human connection
- **Transparency vs. Simplicity** -- showing how things work vs. making them easy to use
- **Individual benefit vs. Community health** -- serving one user vs. protecting the ecosystem
- **Innovation vs. Stability** -- pushing forward vs. maintaining what works

### Step 3: Present the Roadmap

Present the tensions you've identified and wait for confirmation:

> "Here are the 3 tensions I think matter most for this idea. Each one will become a tenet — a declaration of which side you're choosing:
>
> 1. **[Tension name]** -- [Why this tension exists in one sentence]
> 2. **[Tension name]** -- [Why this tension exists]
> 3. **[Tension name]** -- [Why this tension exists]
>
> I'll present each one with research, real examples, and a recommendation. Does this map the territory right, or would you swap any of these?"

Wait for acknowledgment.

---

## PHASE 2.5 -- Research Before Each Tenet

Before generating options for any individual tenet, do targeted research:

1. **Run 2-3 specific web searches** for real examples of companies/products that chose each side
2. **Find concrete outcomes** -- what happened when someone chose X? What happened when someone chose Y?
3. **Synthesize 3-5 findings** with real data and examples
4. **Include findings on the decision page** in a Research Context section

The research should make the tension REAL. Not theoretical — grounded in what actually happened to real products and real people.

---

## PHASE 3 -- Present Each Tenet as HTML

Each tenet is a decision. For each one, generate a self-contained HTML file.

### Directory Structure

```bash
mkdir -p .decisions
```

Save to `.decisions/decision-NNN-slug.html` and maintain `.decisions/decisions.json` and `.decisions/index.html` (landing page titled "Core Principles").

### What Each Decision Page Contains

**Header:**
- Tenet number and the tension being explored
- One-sentence description of why this tension matters for their specific idea

**Research Context:**
- 3-5 findings from real companies/products with what happened when they chose each side
- Sources where possible

**4 Option Cards -- Each representing a different STANCE on the tension:**

Each option is a different way to frame the tenet. The options differ in:
- **Which side they favor** (some lean toward X, some toward Y, some try to balance)
- **How strongly they commit** (a hard line vs. a nuanced lean)
- **What they prioritize within the tension** (different aspects of X or Y)

For example, for a "Growth vs. Safety" tension, the 4 options might be:
- **Option A: "Community safety over growth velocity"** -- Hard commitment to safety, even if it means slower growth
- **Option B: "Earned growth over manufactured growth"** -- Growth is fine but it should come from genuine value, not dark patterns
- **Option C: "Sustainable scale over protective gatekeeping"** -- Lean toward growth but acknowledge you'll add safety rails
- **Option D: "Safety through growth over safety through restriction"** -- More users = more data = better safety systems

Each option card should include:
- The tenet name in "X over Y" format
- The stance (1-2 sentences, written as a belief)
- The tension acknowledgment
- A visual preview

**Visual Previews for Tenet Cards:**

Use tension spectrum visualizations:
- A horizontal bar showing where this tenet falls between the two poles
- Impact indicators showing how this stance affects users, business, and humanity
- A simple radar/spider showing the emphasis across different stakeholder groups

**Comparison Table:**
5-8 dimensions that help distinguish the stances. Good dimensions for tenets:
- Strength of commitment (hard line vs. nuanced lean)
- Impact on users
- Impact on business sustainability
- Impact on broader community/society
- Risk if taken too far
- What you're explicitly giving up
- When you'd revisit this
- Signal it sends (what it tells people about who you are)

**Recommendation:**
Your recommended stance with clear reasoning. Ground it in the research — which real-world examples support this approach?

**Footer:**
Instructions for choosing.

### After Each Tenet

Open the HTML, share what you found and what you recommend, then STOP and wait.

---

## PHASE 4 -- Handle Responses

- **"Option B"** -- lock it in, move to next tenet
- **"Option B because that's what happened to us with our last product"** -- lock it in AND store the reasoning. Personal experience backing a tenet is gold — capture it. Don't ask for reasoning if they didn't offer it.
- **"Option A but softer"** -- regenerate with the modification
- **"Actually our principle is X"** -- generate a full visual card for their custom tenet with the same treatment. Set `chosenOption: "custom"`. This is great — they know their beliefs better than you do.
- **"More options"** -- add 4 more stances on the same tension
- **"This tension doesn't apply to us"** -- remove it, ask if there's a different tension they'd rather explore
- **"For decision-001 I want Option C instead"** -- change a past tenet, flag if it creates tension with other chosen tenets (that's important — principles should be coherent)

---

## PHASE 5 -- Coherence Check

After all tenets are chosen, present one final decision: **the coherence review.**

Generate an HTML page that shows ALL chosen tenets together and asks:

> "Here are your principles as a set. Do they tell a coherent story about what kind of [product/initiative/organization] this is?"

The page should include:
- All tenets listed together with their stances
- A visual showing the overall "personality" of the principle set — where does it lean across the user/business/humanity spectrum?
- Any potential conflicts between tenets flagged
- A "north star" synthesis: one sentence that captures the through-line connecting all the principles

Present 4 options for the north star framing and let the user choose.

---

## PHASE 6 -- Generate Principles Brief

Save as `.decisions/strategy-brief.md`:

```markdown
# Core Principles: [Product/Initiative Name]

## North Star
[The chosen one-sentence through-line]

## The Principles

### 1. [Tenet Name: X over Y]
[Stance -- 1-2 sentences]
*Choosing this over [Y], which also matters — [brief acknowledgment of the tension].*

### 2. [Tenet Name: X over Y]
[Stance]
*Choosing this over [Y] — [tension acknowledgment].*

...

## What These Principles Say About Us
[2-3 sentences synthesizing the overall stance — what kind of product/team/initiative is this?]

## Research That Informed These Principles
- [Key finding 1 with source]
- [Key finding 2 with source]
- [Key finding 3 with source]

## When to Revisit
- [Condition that would trigger a rethink of Principle 1]
- [Condition that would trigger a rethink of Principle 2]
- ...

## How to Use These
These principles are pre-made decisions. When a team member faces a tradeoff:
1. Check if a principle applies
2. Follow the principle
3. If the principle feels wrong for this specific case, flag it — that's a signal the principle may need revisiting, not that it should be silently overridden
```

Then update the landing page and ask:

> "Your principles are locked in. What's next?
> - **'Game plan'** -- I'll run /game-plan to turn these into an operational roadmap
> - **'Challenge'** -- I'll run /challenge to stress-test these principles
> - **'Let me sit with them'** -- take some time to reflect
> - **'Just the brief'** -- we're done for now"

---

## decisions.json Format

```json
{
  "projectName": "Core Principles: [Name]",
  "projectDescription": "Tension-based principles guiding [initiative description]",
  "createdAt": "ISO timestamp",
  "decisions": [
    {
      "id": "decision-001",
      "slug": "tenet-slug",
      "title": "Tenet: X over Y",
      "status": "pending | chosen",
      "chosenOption": "B",
      "chosenTitle": "X over Y",
      "reasoning": "Why this stance (nullable — only if user volunteered)",
      "options": ["A", "B", "C", "D"],
      "recommended": "B",
      "htmlFile": "decision-001-tenet-slug.html",
      "decidedAt": "ISO timestamp",
      "summary": "One sentence about this tenet",
      "history": []
    }
  ]
}
```

---

## IMPORTANT REMINDERS

1. **Never skip the decision page.** Every tenet gets the full visual treatment — even if one side of the tension seems obviously right. The point is to make the user DECLARE it.
2. **Both sides must be good.** If you can't articulate why a reasonable person would choose the other side, it's not a real tension. Reframe until it is.
3. **Always 4 options.** Each option is a different stance on the same tension — different framings, different strengths of commitment.
4. **Always research first.** Every tenet gets a Research Context section with real examples of what happened when companies chose each side.
5. **Write like a human.** Tenets are beliefs. They should sound like something a person would say in a real conversation, not a corporate values poster.
6. **The tension acknowledgment matters.** Every tenet must acknowledge what you're giving up. That's what makes it honest.
7. **Open the HTML automatically.** Always `open .decisions/decision-NNN-slug.html`.
8. **Wait for the user.** Don't proceed until they've chosen.
9. **Flag coherence issues.** If a new tenet contradicts a previously chosen one, say so. Tension between tenets is okay if it's intentional — but it should be conscious.
10. **Custom tenets are the best outcome.** When a user says "actually, our principle is..." — that's the skill working perfectly. Give their custom tenet the same visual treatment as any AI-generated option.
11. **These are living documents.** Always include "when to revisit" conditions. Principles that can't change aren't principles — they're dogma.
