---
name: autodecide
description: "Like /decide but auto-picks the recommended option for every decision. Same research, same options, same recommendations — skip the per-decision pause and review everything at the end before the action skill fires. Hidden power-user variant of /decide."
argument-hint: "[just say what's on your mind, anything goes]"
---

# Autodecide (Orchestrator + Auto-Pick)

You are the entry point when the user wants the rigor of a thinking skill but doesn't want to be interrogated decision-by-decision. Same routing logic as `/decide`, but you inject a directive into the downstream skill telling it to **auto-pick the recommended option for every decision** and pause once at the end for batch review.

The user said: **$ARGUMENTS**

---

## How This Works

This skill behaves exactly like `/decide`:

1. Read the user's input
2. Classify it against the same skill profiles that `/decide` uses
3. If confidence is HIGH → route immediately
4. If confidence is LOW → present 2-3 plain-English interpretations, let the user pick, then route

**The only difference from `/decide`:** when you invoke the downstream skill, append this auto directive to the args:

```
[Auto directive: the user invoked /autodecide. Run all decisions end-to-end without pausing per decision. For each decision, generate the full HTML page (research, options, recommendation) but DO NOT open it and DO NOT wait for input — record the recommended option as the choice with status "auto-picked" in decisions.json, then immediately move to the next decision. After all decisions and the elevator pitch are auto-picked, generate .decisions/auto-review.html (a single batch-review page listing every auto-pick with chosen option, alternatives, and reasoning), open it, and pause ONCE for confirm/override. Use the standard "For decision-N I want Y" syntax for overrides. On confirm, transition every "auto-picked" decision to "chosen" before proceeding to the brief and the action-skill prompt.]
```

---

## CHAIN HANDLING

This orchestrator can be chained with `/overdecide` or `/underdecide`. The user might write:

- `/autodecide /overdecide [topic]` — auto-pick AND extra depth (8-12 decisions, all auto-picked)
- `/overdecide /autodecide [topic]` — same, order-agnostic
- `/autodecide /underdecide [topic]` — auto-pick AND fewer decisions (2-3, all auto-picked)
- `/underdecide /autodecide [topic]` — same, order-agnostic

**If `$ARGUMENTS` begins with another orchestrator's slash command from this family (`/decide`, `/underdecide`, `/overdecide`, `/autodecide`):**

1. Strip that leading slash command from the args.
2. Treat it as if both orchestrators were invoked. When routing to the downstream skill, append BOTH directives — your auto directive AND the chained orchestrator's depth directive.
3. In your "routing to..." note, mention both modes (e.g., "routing to /strategize with extra depth AND auto-pick").

**Mutually exclusive:** `/underdecide` and `/overdecide` can't be combined. If the user chains both, prefer the FIRST one mentioned and tell them you're ignoring the second.

The depth directives to merge if chained:

- From `/overdecide`: `[Depth directive: the user invoked /overdecide. Surface 8-12 decision points instead of the usual 4-7. Be thorough. Include decisions that might normally be skipped for brevity. Cover edge cases, secondary concerns, and downstream implications. Don't add filler — if you genuinely can't find 8 meaningful decisions, it's fine to present fewer. But try to find them.]`
- From `/underdecide`: `[Depth directive: the user invoked /underdecide. Surface only 2-3 decision points instead of the usual 4-7. Focus on the decisions that matter most — the ones that would genuinely break or make the whole thing. Skip secondary decisions, edge cases, and anything that can be reasonably defaulted. The goal is speed and signal, not thoroughness.]`

---

## Step 1: Classify the Input

Use the exact same skill profiles as `/decide`. The routing logic is identical. Only the args sent to the downstream skill change.

**Thinking skills** (these are the skills that walk decisions, so auto-mode matters here most):

- `/strategize` — broad strategy for any complex situation
- `/shape` — design and implementation planning
- `/product-strategy` — product "what and why"
- `/product-design` — product "how" (tech + UX)
- `/ticket-breakdown` — ticket to implementation plan
- `/core-principles` — tension-based principles
- `/visual-design` — aesthetic re-skin (auto-picks tradition/color/type/mood/flourish)

**Other skills** (auto-mode doesn't apply, but route them like `/decide` would):

- `/self-code-review`, `/journal`, `/state-your-case`, `/excavate` — still route, but omit the auto directive since they don't walk through a per-decision pause loop in the same way
- Action skills (`/game-plan`, `/product-plan`, `/brief`, `/challenge`, `/observe`, `/investigate`) — route, but they don't present decision points, so omit the directive

---

## Step 2: High Confidence → Fast Path

If one skill clearly fits, output a brief one-line note:

```
This sounds like [what it is] → routing to /[skill-name] with auto-pick (review at the end)
```

Then invoke the Skill tool:
- `skill`: the chosen skill name
- `args`: the user's original input + the auto directive (and any merged depth directive from chain handling)

Example:
> User: "I want to launch a food truck"
> You: "This sounds like a strategic situation → routing to /strategize with auto-pick (review at the end)"
> [Invoke Skill tool with skill="strategize", args="I want to launch a food truck\n\n[Auto directive: ...]"]

Chained example:
> User: `/autodecide /overdecide` "I want to launch a food truck"
> You: "Strategic situation → routing to /strategize with extra depth AND auto-pick (8-12 decisions, all auto-picked, review at end)"
> [Invoke Skill tool with skill="strategize", args="I want to launch a food truck\n\n[Auto directive: ...]\n\n[Depth directive: ...]"]

---

## Step 3: Low Confidence → Disambiguation

Same pattern as `/decide`: present 2-3 plain-English interpretations, let the user pick, then route with the auto directive (and any merged depth directive) injected.

---

## Step 4: Edge Cases

Same as `/decide`. The only difference is that when you route to a thinking skill that walks decisions, you append the auto directive.

---

## Important Rules

1. **You are /decide with an auto-pick knob.** Classification is identical. Only the args change.
2. **Only inject the auto directive for thinking skills that walk decisions.** /strategize, /shape, /product-strategy, /product-design, /ticket-breakdown, /core-principles, /visual-design. For everything else, route normally.
3. **The auto-pick is the *recommended* option, not a random pick.** Downstream skills already produce a recommendation per decision; auto-mode just commits to it.
4. **Batch review is non-negotiable.** The auto directive REQUIRES the downstream skill to pause once at the end with `auto-review.html`. The action skill must not run until the user confirms.
5. **Preserve the user's words.** Put the auto directive AFTER their input, not mixed into it.
6. **Reveal the skill name and the auto mode.** Tell the user you're routing to /X "with auto-pick" so they understand what changed.
7. **Chain support is order-agnostic.** Whether the user writes `/autodecide /overdecide` or `/overdecide /autodecide`, both directives merge.
