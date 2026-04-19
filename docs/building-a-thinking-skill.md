# Building Your Own Thinking Skill

A thinking skill is any SKILL.md that follows three rules: **Show Decisions. Make Decisions. Remember Decisions.** This guide walks you through building one from scratch.

## Before You Start

Read [SPEC.md](../SPEC.md) for the full specification. The short version:

1. **Show Decisions** - visual HTML decision pages with options. Never skip, even for simple requests.
2. **Make Decisions** - the human chooses. Wait for them. Accept custom answers, tweaks, and reasoning.
3. **Remember Decisions** - store every choice in `.decisions/` so the next skill can build on it.

## Step 1: Choose Your Domain

A thinking skill can be about anything:
- Product decisions (strategy, design, architecture)
- Business decisions (pricing, go-to-market, partnerships)
- Creative decisions (visual direction, narrative, scope)
- Life decisions (major purchases, relocations, goals)

The domain defines what kinds of decisions you'll present. Pick something you know well.

## Step 2: Create the SKILL.md

Create a new directory with a `SKILL.md` file:

```
my-thinking-skill/
  SKILL.md
```

Your SKILL.md should include these sections:

### The Preamble

```markdown
# [Skill Name]

You are helping the user [do what] by walking them through every meaningful
decision — one at a time — using rich, visual HTML decision documents.

The user's request is: **[user input placeholder]**
```

### Phase 1: Check for Prior Decisions

This is what makes composition work:

```markdown
## PHASE 1 — Understand the Request

First, check if `.decisions/strategy-brief.md` exists.
If it does, read it — the user has already run a thinking skill.
Use their prior decisions as context. Don't re-ask questions
the strategy already answered.
```

### Phase 2: Identify Decision Points

```markdown
## PHASE 2 — Identify Decision Points

Analyze the request and list 4-7 meaningful decisions.
Present the roadmap to the user and wait for confirmation.
```

### Phase 3: Present Each Decision as HTML

This is the core. Each decision becomes a self-contained HTML page with:
- Header with decision title and description
- 4 option cards with visual previews
- Comparison table
- Footer with instructions

```markdown
## PHASE 3 — Present Each Decision

For each decision:
1. Gather context (web searches, prior decisions, domain knowledge)
2. Generate a self-contained HTML file in `.decisions/`
3. Update `decisions.json`
4. Open the HTML in the browser
5. Wait for the user to choose
```

### Phase 4: Handle Responses

This is where recent changes matter. Your skill should handle all of these:

```markdown
## PHASE 4 — Handle Responses

- "Option B" → lock it in, move to next
- "Option B because X" → lock it in AND store the reasoning.
  Don't ask for reasoning if they didn't offer it.
- "Option A but [modification]" → regenerate with the tweak
- "Actually I want X" (custom answer) → generate a full visual card
  for their answer with the same treatment as any AI option.
  Set chosenOption to "custom".
- "More options" → add 4 more
- "For decision-001 I want Option C" → change a past decision

IMPORTANT: Never skip the decision page. If someone called this skill,
they want the full visual treatment - even for simple or obvious decisions.
```

### Phase 5: Generate Summary

```markdown
## PHASE 5 — Generate Summary

After all decisions, generate `strategy-brief.md` (or whatever
summary document fits your domain) in `.decisions/`.
```

## Step 3: Include the HTML Template

Your SKILL.md should include the HTML structure for decision pages. Look at any existing thinking skill for the full template. The key elements:

- **Self-contained HTML** with inline CSS (no external dependencies)
- **Option cards** with visual previews, summaries, and pros/cons
- **Comparison table** below the cards
- **Instructions** in the footer

## Step 4: Test the Composability

The real test: run an existing thinking skill first (like `/strategize`), then run yours. Does yours:

- [ ] Find and read the prior `strategy-brief.md`?
- [ ] Skip questions already answered?
- [ ] Reference prior decisions in its options?
- [ ] Write its own decisions in the same `.decisions/` format?
- [ ] Handle custom answers with full visual treatment?
- [ ] Capture reasoning when the user volunteers it?
- [ ] Never skip the decision page, even for simple requests?

If yes, your thinking skill composes. It works with every other thinking skill in the ecosystem.

## Step 5: Add Auto-Mode Support (Recommended)

Auto-Mode lets users run your skill end-to-end without per-decision pauses, then review all the AI's picks at once. It's how `/autodecide [topic]` (which routes through your skill) and `/your-skill /autodecide [topic]` (direct inline modifier) work. See [SPEC.md → Auto-Mode](../SPEC.md#auto-mode) for the full spec.

To support Auto-Mode, add a single block to your SKILL.md between your Core Principles and PHASE 1:

````markdown
## AUTO-MODE OVERRIDE (applies if /autodecide was used)

**Detection:** Auto-mode applies if EITHER:

- `$ARGUMENTS` contains a `[Auto directive: ...]` block (injected by the `/autodecide` orchestrator), OR
- `$ARGUMENTS` starts with `/autodecide` (direct inline modifier)

In the second case, strip `/autodecide` from the args before treating the rest as the user's input.

**Inline depth modifiers also work.** If `$ARGUMENTS` starts with (or contains alongside `/autodecide`) `/overdecide` or `/underdecide`, treat them as depth directives too:
- `/overdecide` → surface 8-12 decisions instead of the usual 4-7
- `/underdecide` → surface only 2-3 decisions
- Order doesn't matter; if both depth modifiers appear, the FIRST wins

Strip all leading modifier tokens before treating the rest as the user's input.

If auto-mode is triggered, your behavior changes:

1. **Per-decision pauses are skipped.** Generate every decision page as normal (research, options, recommendation, comparison) but record `status: "auto-picked"` in `decisions.json` with `chosen` set to the recommendation (capture reasoning in the `reasoning` field, prefixed with "Auto-picked: "). Do NOT open the file. Do NOT pause. Move to the next decision.
2. **Generate `.decisions/auto-review.html`** after all decisions: a single batch-review page listing every auto-pick with the chosen option, the alternatives it beat, and the reasoning. Open it.
3. **Pause once** for confirm/override.
4. **On confirm** ("looks good", "approved"): transition every `auto-picked` decision to `status: "chosen"`, then continue to your summary phase normally.
5. **On override** ("For decision-N I want Y"): update that decision (set `chosen` to Y, `status: "chosen"`, capture reasoning if given, add a `history` entry), regenerate `auto-review.html`, re-prompt for confirmation of the rest.

**Critical invariant:** Do not generate the summary brief or hand off to action skills until every decision has transitioned from `auto-picked` to `chosen`. The batch-review pause is the gate.
````

Copy-paste this into your SKILL.md. Tweak the wording ("user's input" → "user's project" / "engineer's ticket" / etc.) to match your domain.

**Test it:** Run `/your-skill /autodecide [example input]`. It should generate every decision page without pausing, then open `auto-review.html`. Reply "looks good" and verify it proceeds to your summary phase. Try `/your-skill /autodecide /overdecide [input]` for the chained case (more decisions, all auto-picked).

**Skip Auto-Mode if:** your skill doesn't follow the "AI recommends, human picks from N options" pattern — for example, skills that surface findings the human verifies (`/excavate`), skills where the human leads (`/journal`), or skills designed to interrupt and ask (`/state-your-case`). Auto-Mode doesn't fit those paradigms.

## Tips

- **Start with 4-7 decisions.** Too few feels shallow. Too many feels tedious.
- **Visual previews matter.** The HTML pages should help people SEE the difference between options. Use rendered UI mockups, flow diagrams, positioning maps - whatever fits the decision.
- **Write in plain English.** If you use a technical term, explain it in the same sentence.
- **The comparison table is mandatory.** Pick 5-8 dimensions that genuinely differentiate the options.
- **Never skip.** Even if the user's request seems simple. The visual is the value.
- **Custom answers are first-class.** When someone says "actually I want X," generate the same quality visual card as any AI option.
- **Capture reasoning silently.** If they say "Option B because X," store the "because X." If they just say "Option B," store null. Never nag.

## Example: A Minimal Thinking Skill

The simplest possible thinking skill might look like:

```markdown
# My Skill

You help the user make decisions about [domain].

## Phase 1: Check for prior decisions
Read .decisions/strategy-brief.md if it exists.

## Phase 2: Identify 4-5 key decisions
Present the list and wait for confirmation.

## Phase 3: For each decision
1. Generate HTML with 4 options + comparison table
2. Save to .decisions/decision-NNN-slug.html
3. Update decisions.json
4. Open in browser
5. Wait for choice
6. If they volunteer reasoning, store it
7. If they bring a custom answer, generate a full visual for it

## Phase 4: Generate summary
Save strategy-brief.md with all choices.
```

That's a thinking skill. It shows decisions, lets humans make them (including custom answers and reasoning), and remembers them.

## Making Your Skill Discoverable via /decide

The `/decide` orchestrator routes users to skills based on natural language input. To make your new thinking skill routable, add a profile for it in the orchestrator's SKILL.md (`orchestrator/decide/SKILL.md`).

Add a profile block in the **Skill Profiles** section:

```
/your-skill-name
Does: [what it does in one sentence]
Signals: [words and patterns that indicate this skill]
Not this if: [when NOT to use this skill — reference the sibling skills that handle nearby intent]
Examples: [2-3 example inputs that should route here]
```

The **"Not this if"** field is the most important. Your skill library has siblings — skills that sound similar but serve different purposes. This field tells the orchestrator when to pick a different skill instead of yours. Without it, ambiguous inputs will misroute.

---

*For the full specification, see [SPEC.md](../SPEC.md).*
*For examples of thinking skills in action, see [examples/](../examples/).*
