---
name: underdecide
description: "Like /decide but dials the decision count DOWN. Routes to the right thinking skill and tells it to surface only 2-3 decision points instead of the usual 4-7. For when you just want the most important calls made and the rest defaulted. Hidden power-user variant of /decide."
argument-hint: "[just say what's on your mind, anything goes]"
---

# Underdecide (Orchestrator + Less Depth)

You are the entry point when the user wants to move fast and not sweat the small stuff. Same routing logic as `/decide`, but you inject a directive into the downstream skill's args telling it to surface only **2-3 decision points** instead of its usual 4-7.

The user said: **$ARGUMENTS**

---

## How This Works

This skill behaves exactly like `/decide`:

1. Read the user's input
2. Classify it against the same skill profiles that `/decide` uses
3. If confidence is HIGH → route immediately
4. If confidence is LOW → present 2-3 plain-English interpretations, let the user pick, then route

**The only difference from `/decide`:** when you invoke the downstream skill, append this depth directive to the args:

```
[Depth directive: the user invoked /underdecide. Surface only 2-3 decision points instead of the usual 4-7. Focus on the decisions that matter most — the ones that would genuinely break or make the whole thing. Skip secondary decisions, edge cases, and anything that can be reasonably defaulted. The goal is speed and signal, not thoroughness.]
```

---

## CHAIN HANDLING

This orchestrator can be chained with `/autodecide` (or `/decide`). The user might write:

- `/underdecide /autodecide [topic]` — fewer decisions AND auto-pick (2-3 decisions, all auto-picked)
- `/autodecide /underdecide [topic]` — same, order-agnostic

**If `$ARGUMENTS` begins with another orchestrator's slash command from this family (`/decide`, `/underdecide`, `/overdecide`, `/autodecide`):**

1. Strip that leading slash command from the args.
2. Treat it as if both orchestrators were invoked. When routing to the downstream skill, append BOTH directives — your depth directive AND the chained orchestrator's directive.
3. In your "routing to..." note, mention both modes (e.g., "routing to /strategize with less depth AND auto-pick").

**Mutually exclusive:** `/underdecide` and `/overdecide` can't be combined. If the user chains both, prefer the FIRST one mentioned and tell them you're ignoring the second.

The other directive to merge if `/autodecide` is chained:

- From `/autodecide`: `[Auto directive: the user invoked /autodecide. Run all decisions end-to-end without pausing per decision. For each decision, generate the full HTML page (research, options, recommendation) but DO NOT open it and DO NOT wait for input — record the recommended option as the choice with status "auto-picked" in decisions.json, then immediately move to the next decision. After all decisions and the elevator pitch are auto-picked, generate .decisions/auto-review.html (a single batch-review page listing every auto-pick with chosen option, alternatives, and reasoning), open it, and pause ONCE for confirm/override. Use the standard "For decision-N I want Y" syntax for overrides. On confirm, transition every "auto-picked" decision to "chosen" before proceeding to the brief and the action-skill prompt.]`

---

## Step 1: Classify the Input

Use the exact same skill profiles as `/decide`. The routing logic is identical. Only the args sent to the downstream skill change.

**Thinking skills** (these are the skills that actually walk through decisions, so underdecide matters here most):

- `/strategize` — broad strategy for any complex situation
- `/shape` — design and implementation planning
- `/product-strategy` — product "what and why"
- `/product-design` — product "how" (tech + UX)
- `/ticket-breakdown` — ticket to implementation plan
- `/core-principles` — tension-based principles

**Other skills** (these don't benefit from a decision count directive, but route them like `/decide` would):

- `/self-code-review`, `/journal`, `/state-your-case`, `/excavate` — still route, but you can omit the depth directive since they don't walk through a variable number of decisions
- `/visual-design` — also route normally, but omit the depth directive: it has a fixed decision count per mode (5 for HTML, 3 for SVG). Auto-mode applies (via `/autodecide`), but depth modifiers don't.
- Action skills (`/game-plan`, `/product-plan`, `/brief`, `/challenge`, `/observe`, `/investigate`) — route, but they don't present decision points, so omit the directive

---

## Step 2: High Confidence → Fast Path

If one skill clearly fits, output a brief one-line note:

```
This sounds like [what it is] → routing to /[skill-name] with less depth (2-3 decisions)
```

Then invoke the Skill tool:
- `skill`: the chosen skill name
- `args`: the user's original input + the depth directive (for thinking skills that walk decisions)

Example:
> User: "I want to launch a food truck"
> You: "This sounds like a strategic situation → routing to /strategize with less depth (2-3 decisions)"
> [Invoke Skill tool with skill="strategize", args="I want to launch a food truck\n\n[Depth directive: the user invoked /underdecide. Surface only 2-3 decision points instead of the usual 4-7. Focus on the decisions that matter most — the ones that would genuinely break or make the whole thing. Skip secondary decisions, edge cases, and anything that can be reasonably defaulted. The goal is speed and signal, not thoroughness.]"]

---

## Step 3: Low Confidence → Disambiguation

Same pattern as `/decide`: present 2-3 plain-English interpretations, let the user pick, then route with the depth directive injected.

---

## Step 4: Edge Cases

Same as `/decide`. The only difference is that when you route, you append the depth directive for thinking skills that walk through variable decision counts.

---

## Important Rules

1. **You are /decide with a depth knob.** Classification is identical. Only the args change.
2. **Only inject the depth directive for thinking skills that walk decisions.** /strategize, /shape, /product-strategy, /product-design, /ticket-breakdown, /core-principles. For everything else, route normally.
3. **The picked decisions should be the highest-stakes ones.** Not random. Not the easiest. The ones that genuinely make or break the outcome.
4. **Preserve the user's words.** Put the depth directive AFTER their input, not mixed into it.
5. **Reveal the skill name and the depth mode.** Tell the user you're routing to /X "with less depth" so they understand what changed.
