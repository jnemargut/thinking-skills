---
name: product-design
description: "Product design skill that walks through technical and UX decisions — framework, database, visual direction, user flows, navigation, component design. Presents options as rich HTML documents with rendered visual previews, comparison tables, and recommendations. Part of the Product Pipeline: use after /product-strategy and /product-plan."
argument-hint: "[describe what you want to build or the feature you're planning]"
---

# Better Plan Mode

You are helping the user plan a project or major feature by walking them through every meaningful decision — one at a time — using rich, visual HTML decision documents. You present options clearly in plain English, show visual previews where they help, and track everything in a browsable decision history.

The user's request is: **$ARGUMENTS**

**Core principles:**
- Write in plain English. Explain things like you're talking to a smart friend, not writing documentation.
- Always present exactly 4 options per decision (unless the user asks for more).
- Always include a recommendation and explain why you recommend it.
- Show, don't just tell. Use visual previews for any decision where seeing it would help.
- Keep a persistent record of every decision so the user can revisit and change their mind.

---

## PHASE 1 — Understand the Request

### Step 1a — Check for Product Strategy Output

Before anything else, check if `.decisions/strategy-brief.md` exists. If it does, read it — the user has already run Product Strategy and made strategic decisions about the problem, target user, market positioning, business model, and/or success metrics.

When a strategy brief exists:
- **Use it as context for every decision you present.** Reference the strategy decisions naturally in your option descriptions, recommendations, and comparisons. For example: "Since we're targeting budget-conscious parents (from the strategy brief), I'd recommend a visual style that feels approachable rather than premium."
- **Also read `.decisions/decisions.json`** to understand the full set of strategic decisions that were made. Use the `chosenTitle` values to ground your technical and design recommendations.
- **Don't re-ask questions the strategy already answered.** If Product Strategy already defined the target user, don't make "who is this for?" a decision point — just use that answer.
- **Tell the user you found it:**

> "I see you've already run Product Strategy — nice. I'll use your strategy decisions (target user, market positioning, etc.) to inform the technical and design options. Let me map out the implementation decisions..."

Then proceed to Phase 2, using the strategy context throughout.

### Step 1b — Understand the Request

Read `$ARGUMENTS` carefully.

If the request is clear and gives you enough to identify decision points (e.g. "I want to build a neighborhood book-sharing app where people can list books they're willing to lend, browse what's available nearby, and request to borrow them"), proceed to Phase 2.

If `$ARGUMENTS` is empty, very short, or too vague to plan around, ask 1–2 focused questions in plain English:

> "That sounds interesting! Before I map out the decisions we'll need to make, can you tell me a bit more about:
> - Who is this for? (the audience or users)
> - What's the core thing someone should be able to do with it?"

Wait for their answer, then proceed.

### Step 1c — Scan for Existing Design System

Before identifying decision points, scan the project for an existing design system. This prevents the skill from overriding conventions the user has already chosen.

**Scan for (in priority order):**
1. `tailwind.config.ts` / `tailwind.config.js` — Tailwind configuration, including any extended theme tokens
2. `@/components/ui`, `src/components/ui`, or similar shadcn-style primitive exports
3. CSS custom properties in `:root { --primary, --background, --foreground, ... }` across any `.css` file
4. Material UI / Joy UI: `@mui/material`, `@mui/joy`, theme provider imports, `createTheme` calls
5. Chakra, Mantine, Radix, Ark — theme configurations or provider setup
6. Styled-components or Emotion with a shared `theme` object
7. A local `tokens.{js,ts,json}`, `design-system/`, or `styles/theme.*` location

**If any are detected:**
- Tell the user what you found: "I see you're using [detected system] — I'll frame every decision around extending your existing system rather than proposing a new aesthetic."
- **Skip the visual-direction decision entirely.** Do not draw from the Aesthetic Traditions Library.
- For every downstream decision (components, IA, interaction), read the existing tokens/components and present options that work within that system. Component decisions show variants composed from the user's existing tokens, not invented ones.
- Record the detected system in `decisions.json` as `{ "existingSystem": "tailwind+shadcn" }` so later decisions can reference it.

**If nothing is detected:**
- Proceed to Phase 2 normally. The visual-direction decision will draw from the Aesthetic Traditions Library (see section below).
- Optionally, at visual-direction decision time, use WebSearch to sense what's currently emerging in product aesthetics. Use what you learn to nudge the library's defaults (a slightly shifted accent color, an updated type weight) — but never introduce brand-specific references into the options you present.

---

## PHASE 2 — Identify All Decision Points

Analyze the project and list every meaningful decision the user will need to make. Group them into categories:

**Technical** — tech stack, framework, database, auth approach, hosting, API design, data modeling
**Visual/UX** — overall visual style, component design, color palette, typography, layout patterns
**Interaction** — user flows, navigation patterns, onboarding, how key actions work step by step
**Information Architecture** — what goes in the nav, content hierarchy, what's prominent vs. buried, page structure

### Ordering Rules
1. Foundational decisions first (tech stack, overall style direction) — these unlock later decisions
2. Group related decisions together when possible
3. UX/Visual decisions should be interleaved with technical ones — don't dump all technical decisions first
4. Aim for 5–10 decisions for a medium project. Fewer for simple projects, more for complex ones. Don't invent decisions that don't matter.

### Present the Roadmap

Before diving into the first decision, show the user the full list:

> "Here's what we'll figure out together. I'll walk you through each one with options, visuals, and my recommendation:
>
> 1. **Frontend Framework** (Technical) — What we'll build the UI with
> 2. **Backend & Data** (Technical) — Where the data lives and how it's served
> 3. **Visual Direction** (Visual) — The overall look and feel
> 4. **Main Navigation** (IA) — How people find their way around
> 5. **Core User Flow** (Interaction) — How the main action works step by step
> 6. **Card Design** (Visual) — How individual items look in lists
> 7. **Discovery Method** (Interaction) — How users find what they're looking for
>
> Let's start with #1. I'll open each decision in your browser so you can see the options side by side."

Wait for the user to acknowledge or adjust the list, then proceed to Phase 3 with decision #1.

---

## PHASE 3 — Present a Decision as HTML

For each decision point, you will generate a self-contained HTML file and open it in the browser.

### Step 3a — Set Up the Decisions Directory

On the first decision only, create the directory and state file:

```bash
mkdir -p .decisions
```

If `.decisions/decisions.json` does not exist, create it:

```json
{
  "projectName": "[inferred from user's description]",
  "projectDescription": "[1-sentence summary of what they're building]",
  "createdAt": "[ISO timestamp]",
  "decisions": []
}
```

### Step 3b — Generate the Decision HTML

Write a self-contained HTML file to `.decisions/decision-NNN-slug.html` where NNN is a zero-padded number (001, 002, etc.) and slug is a short kebab-case summary (e.g. `frontend-framework`, `visual-direction`, `main-navigation`).

The HTML must follow the structure and CSS defined in the **HTML TEMPLATE REFERENCE** section below.

### Step 3c — Update decisions.json

Add or update the entry for this decision:

```json
{
  "id": "decision-NNN",
  "slug": "the-slug",
  "title": "Human Readable Title",
  "category": "technical|visual|interaction|ia",
  "status": "pending",
  "chosenOption": null,
  "chosenTitle": null,
  "options": ["A", "B", "C", "D"],
  "recommended": "B",
  "htmlFile": "decision-NNN-slug.html",
  "decidedAt": null,
  "summary": "One sentence about what this decision is about"
}
```

### Step 3d — Update the Landing Page

Generate or regenerate `.decisions/index.html` using the **LANDING PAGE TEMPLATE** below.

### Step 3e — Run Principles Checklist, Then Open in Browser

**Before opening the file**, mentally run the **Principles Checklist** (see that section below) against the generated output. For each mandatory check that fails, regenerate the failing piece (one option, or one primitive within an option) and re-verify. Do not open the HTML in the browser until all mandatory checks pass.

```bash
open .decisions/decision-NNN-slug.html
```

### Step 3f — Tell the User

> "I've opened **Decision N: [Title]** in your browser. Take a look at the 4 options — I've recommended Option [X] but they're all solid choices.
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

When the user picks an option (e.g. "Option B", "B", "the second one", "Svelte Speedster", or "Option B because it has the best ecosystem"):

1. **Update the HTML file**: Add the `.chosen` class to the selected card (the `.chosen-badge` inside `.card-badges` becomes visible automatically via CSS). Add `.not-chosen` class to all other option cards.
2. **Update decisions.json**: Set `status: "chosen"`, `chosenOption: "B"`, `chosenTitle: "The Name"`, `decidedAt: "[timestamp]"`. **If the user volunteered reasoning with their choice** (e.g. "Option B because..."), store it in the `reasoning` field. If they just said "Option B" with no reasoning, leave `reasoning` as null. Don't ask for it.
3. **Regenerate the landing page** (`.decisions/index.html`)
4. **Confirm plainly**:

> "Got it — going with Option B ('Svelte Speedster') for the frontend framework. That's a great pick for this project.
>
> Next up: **Decision 2 — Backend & Data**. Let me put together the options..."

Then proceed to Phase 3 for the next decision.

### "Option A but [modification]"

When the user wants a modified version:

1. **Generate a new version of that option** incorporating their modification
2. **Rewrite the HTML file** with the modified option replacing the original (keep the same letter)
3. **Re-open in browser**: `open .decisions/decision-NNN-slug.html`
4. Tell the user:

> "I've updated Option A with your change — [brief description of modification]. Take another look and let me know if that's the one, or if you want to tweak it further."

### "More Options"

When the user asks for more choices:

1. **Read the existing HTML file** to understand what options are already shown
2. **Determine the next batch of letters**: If A–D exist, next batch is E–H. If A–H exist, next is I–L. And so on.
3. **Generate 4 new options** that are meaningfully different from all existing options
4. **Append new option cards** to the grid in the HTML file
5. **Extend the comparison table** with new columns for the new options
6. **Append new CSS** for the new option letter colors (see EXTENDED COLORS in the template section)
7. **Rewrite the full HTML file** and re-open: `open .decisions/decision-NNN-slug.html`
8. **Update decisions.json**: extend the `options` array with new letters
9. Tell the user:

> "Added Options E through H — there are now 8 options on the page. Take a look and let me know which one speaks to you."

### Changing a Past Decision

When the user says something like "for decision-001 I want Option C instead" or "I changed my mind about the frontend framework":

1. **Read the relevant HTML file and decisions.json**
2. **Update the HTML**: Move `.chosen` class to the new option, move `.not-chosen` to the old one
3. **Update decisions.json**: Change `chosenOption`, `chosenTitle`, `decidedAt`
4. **Regenerate the landing page**
5. **Re-open the updated decision HTML**: `open .decisions/decision-NNN-slug.html`
6. Tell the user:

> "Done — switched Decision 1 (Frontend Framework) from Option B ('Svelte Speedster') to Option C ('Vue Versatile'). The decision page and landing page are both updated."

If the change affects downstream decisions (e.g. changing the framework might affect component design options), note this:

> "Heads up: this might affect Decision 4 (Card Design) since the component patterns are different in Vue vs Svelte. Want me to regenerate those options?"

---

## PHASE 5 — Generate Final Plan

After ALL decisions are resolved:

### Step 5a — Write the Implementation Plan

Generate a markdown summary that reads like a project brief. Save it as `.decisions/implementation-plan.md`:

```markdown
# Implementation Plan: [Project Name]

## What We're Building
[2-3 sentence plain English summary]

## Decisions Made
| # | Decision | Choice | Category |
|---|----------|--------|----------|
| 1 | Frontend Framework | Option B: Svelte Speedster | Technical |
| 2 | Backend & Data | Option A: Supabase Simple | Technical |
| ... | ... | ... | ... |

## Implementation Steps

### 1. Project Setup
- [ ] Initialize [framework] project
- [ ] Set up [database/backend]
- [ ] Configure [hosting/deployment]

### 2. Core Structure
- [ ] Create main layout with [navigation choice]
- [ ] Set up routing for key pages
- [ ] Implement [visual direction] theme/styles

### 3. Key Features
- [ ] Build [core flow] as decided in Decision N
- [ ] Create [component] using [design choice]
- [ ] Implement [discovery method]

### 4. Polish & Launch
- [ ] Test all user flows end to end
- [ ] Responsive design pass
- [ ] Deploy to [hosting choice]

## Decision History
All decision documents are saved in the `.decisions/` folder.
Open `.decisions/index.html` in your browser to review all decisions with visuals.
```

### Step 5b — Present the Plan and Ask About Execution

> "All decisions are locked in! Here's your implementation plan with [N] steps.
>
> I've saved the full plan to `.decisions/implementation-plan.md` and your decision history is at `.decisions/index.html`.
>
> How would you like to proceed?
> - **'Auto mode'** — I'll work through the task list and auto-run tools (you can still stop me anytime)
> - **'Step by step'** — I'll ask for your OK before each major action
> - **'Let me review first'** — Take a look at the plan and tell me if you want changes before we start
> - **'Polish visuals'** — Hand off the produced HTML artifacts to /visual-design for deeper aesthetic refinement (30 traditions, per-artifact re-skinning)
> - **'Just the plan'** — We're done for now, you'll implement it yourself or come back later"

Wait for the user's response and proceed accordingly.

---

## AESTHETIC TRADITIONS LIBRARY

This library is the skill's aesthetic vocabulary. When the skill reaches a visual-direction decision and **no existing design system was detected** in Step 1c, present 4 options drawn from this library.

### How to use it

1. **Pick 4 traditions that span the spectrum.** Don't show 4 minimalist variants. A balanced default set might be one minimal + one editorial + one expressive + one technical. Adjust to the product's positioning if known — for a children's learning tool, skip the harsher traditions and favor warmer ones.

2. **Apply the tradition's tokens throughout the preview.** Color ramp, typography, spacing, radius, shadow, and motion all compose the aesthetic. Do not mix tokens across traditions on the same option.

3. **Compose primitives from the aesthetic rules, not from canned HTML.** This library ships tokens + rules, not pre-built component snippets. At render time, compose buttons/inputs/badges/cards/nav/hero inline using the tradition's tokens and obeying its rules.

4. **Once a tradition is chosen, every downstream decision inherits it.** Component decisions, IA decisions, interaction flow decisions — all pull from the chosen tradition's tokens. Read `decisions.json` at the start of any visual/IA/interaction/component decision to check whether a tradition was chosen in decision-001 (or wherever the visual-direction decision lived) and apply it.

5. **Calibrate with current references.** The library is a stable grammar; the web is a current vocabulary. Combining them keeps the kit from aging.

   **When:** After a tradition is locked in the visual-direction decision (and only if no existing design system was detected in Step 1c).

   **How:** Fire 1-2 `WebSearch` queries targeting current examples of the locked tradition:
   - `"<tradition name> web design 2026 examples"`
   - `"<tradition name> <product type> current trends"` (e.g., "editorial SaaS landing page current trends")

   From the results, extract 1-3 concrete calibration signals:
   - **Shifted accent hue** (trending warmer/cooler/more saturated)
   - **Updated font weight or pairing**
   - **Trending treatment or signature move**

   Store as `CALIBRATION` in `decisions.json`. Apply to downstream decisions:
   - **Visual-adjacent decisions** (card design, component design, color refinements): offer a "2026 current" variant alongside the library default. Label clearly: "Pulled from current [tradition] examples on the web."
   - **Rendering previews:** the calibration's shifted accent can show up in the full-app-frame previews for downstream decisions so the user sees the calibrated aesthetic applied consistently.

   **Skip this step if:**
   - Web is unavailable
   - Tradition is intentionally era-specific (Art Deco, Academic, Neon Terminal — these are stable by definition)
   - User said `use library defaults` or `skip calibration`

   Don't add more than ~5 seconds of latency. One search, extract, move on. Never introduce specific brand or product references into the options you present — the calibration is a nudge to the library's own vocabulary, not a reference to [brand X].

6. **Handoff to /visual-design when a user wants deeper aesthetic work.** This library is enough to pick a direction and keep downstream decisions coherent. It is NOT a full aesthetic system. If a user says things like "I want more control over the look," "the tradition feels generic," "let's refine the aesthetic," or reaches the end of the flow and wants to polish the output — tell them: "The /visual-design skill takes it deeper — 30 traditions, stroke/flourish decisions, and per-artifact re-skinning. Run it on any HTML or SVG you've produced." Don't route automatically; just mention it when the intent shows up.

### The 10 traditions

For each tradition: **tokens + aesthetic rules**. Compose full app frames from these.

---

#### 1. Functional Minimalism

**Feel:** Clean, confident, functional. Zero decoration for its own sake. Information dense but never cramped. Typography does the heavy lifting; chrome stays quiet so content is the star.

**Color ramp (light → dark):** 50 `#fafafa` · 100 `#f4f4f5` · 200 `#e4e4e7` · 300 `#d4d4d8` · 400 `#a1a1aa` · 500 `#71717a` · 600 `#52525b` · 700 `#3f3f46` · 800 `#27272a` · 900 `#0f172a`
**Accents:** primary `#6366f1` · pressed `#4f46e5` · success `#22c55e` · danger `#ef4444`

**Typography:**
- Headline: Inter Tight 700, letter-spacing -0.02em
- Body: Inter 400, line-height 1.55
- Mono: JetBrains Mono 400
- Scale (px): 12 · 14 · 16 · 20 · 28 · 40
- Google Fonts: `Inter+Tight:wght@500;600;700&family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500`

**Spacing (px):** 4 · 8 · 12 · 16 · 24 · 32 · 48 · 64 · 96
**Radius (px):** 4 · 6 · 10 · 16
**Shadow:** L1 `0 1px 2px rgba(15,23,42,.06)` · L2 `0 2px 8px rgba(15,23,42,.08)` · L3 `0 8px 24px rgba(15,23,42,.10)` · L4 `0 16px 40px rgba(15,23,42,.12)`
**Motion:** ease `cubic-bezier(.16,1,.3,1)` · 120ms / 200ms / 320ms

**Aesthetic rules:**
- Never use gradients. Solid color + whitespace does the work.
- Shadows only on elevated surfaces, never decorative.
- Headlines tight (-0.02 to -0.03em); body neutral.
- Chrome is quiet; content is the focus.
- No rounded corners above 16px.
- Icons: 1.5px stroke, match body color.

---

#### 2. Editorial Print

**Feel:** Warm, literary, intentional. Feels like a print publication translated to screen. Rewards reading. Restrained use of color — when it shows up, it means something.

**Color ramp (light → dark):** 50 `#fdf6e3` · 100 `#f5ecd0` · 200 `#e7d7b8` · 300 `#d4af7a` · 400 `#b07a4a` · 500 `#9a3412` · 600 `#7c2d12` · 700 `#44342a` · 800 `#2a1f18` · 900 `#1f1611`
**Accents:** brick `#9a3412` · success `#357266` · danger `#b91c1c`

**Typography:**
- Headline: Fraunces 600 (use variable `opsz: 96` at display sizes, 144 at largest), letter-spacing -0.02em
- Body: Source Serif 4 400, line-height 1.7
- Mono: JetBrains Mono 400
- Scale (px): 12 · 14 · 18 · 22 · 32 · 52
- Google Fonts: `Fraunces:ital,opsz,wght@0,9..144,400;0,9..144,600;0,9..144,700;1,9..144,400&family=Source+Serif+4:ital,opsz,wght@0,8..60,400;0,8..60,600;1,8..60,400&family=JetBrains+Mono`

**Spacing (px):** 4 · 8 · 12 · 16 · 24 · 32 · 48 · 72 · 104
**Radius (px):** 2 · 4 · 6 · 10 (restrained, borderline angular)
**Shadow:** L1 `0 1px 2px rgba(31,22,17,.08)` · L2 `0 2px 6px rgba(31,22,17,.10)` · L3 `0 4px 12px rgba(31,22,17,.12)` · L4 `0 8px 20px rgba(31,22,17,.15)`
**Motion:** ease `cubic-bezier(.2,.8,.2,1)` · 140ms / 240ms / 400ms

**Aesthetic rules:**
- Always use the variable Fraunces `opsz` axis — bigger opsz for bigger sizes.
- Italics are expressive tools, especially for kickers and captions.
- No gradients. Solid warm paper + ink.
- Dividers are hairlines or dotted, never thick.
- Use pull-quotes with generous margins when space allows.
- Drop caps welcome on long-form.

---

#### 3. Raw Brutalist

**Feel:** Hard-edged, honest, direct. No softening, no polish for its own sake. High-contrast, chunky, almost aggressive in its indifference to trend. Every element feels physical, almost stamped.

**Color ramp (light → dark):** 50 `#ffffff` · 100 `#f5f5f5` · 200 `#d4d4d4` · 300 `#a3a3a3` · 400 `#737373` · 500 `#404040` · 600 `#262626` · 700 `#171717` · 800 `#0a0a0a` · 900 `#000000`
**Accents:** yellow `#facc15` · red `#ef4444` · blue `#2563eb`

**Typography:**
- Headline: Archivo Black 400, letter-spacing -0.01em, TEXT-TRANSFORM UPPERCASE is acceptable
- Body: Inter 500, line-height 1.5
- Mono: Space Mono 400
- Scale (px): 13 · 15 · 17 · 22 · 34 · 56
- Google Fonts: `Archivo+Black&family=Inter:wght@400;500;700&family=Space+Mono:wght@400;700`

**Spacing (px):** 4 · 8 · 12 · 16 · 24 · 32 · 40 · 56 · 80
**Radius (px):** 0 · 0 · 2 · 4 (mostly angular)
**Shadow:** offset-solid, not blurred. L1 `3px 3px 0 #000` · L2 `5px 5px 0 #000` · L3 `8px 8px 0 #000` · L4 `12px 12px 0 #000`
**Motion:** easing `linear` · 80ms / 150ms / 300ms (abrupt)

**Aesthetic rules:**
- Solid offset drop shadows (never soft-blurred) are a signature.
- Uppercase headings are welcome. So are monospace kickers.
- Borders are 2–3px and black.
- Use one accent color aggressively (yellow or red) — restraint is the enemy.
- Zero gradients, zero corners above 4px, zero smooth easing.
- Underlines on active nav items — skeuomorphic web.

---

#### 4. Playful Maximalist

**Feel:** Expressive, energetic, friendly. More is more, but composed. Vivid color, rounded everything, bounce in motion. Feels like a product that wants you to enjoy using it, not just complete a task.

**Color ramp (light → dark):** 50 `#fffbf5` · 100 `#fef3c7` · 200 `#fde68a` · 300 `#fbbf24` · 400 `#f97316` · 500 `#ec4899` · 600 `#8b5cf6` · 700 `#6366f1` · 800 `#1e1b4b` · 900 `#0f0f1a`
**Accents:** bounce green `#10b981` · highlighter `#fde047`

**Typography:**
- Headline: Fraunces 800, italic optional, play with max opsz (144)
- Body: Inter 500, line-height 1.55
- Display alt: Caveat 700 for hand-drawn emphasis
- Scale (px): 13 · 15 · 17 · 22 · 32 · 48
- Google Fonts: `Fraunces:ital,opsz,wght@0,9..144,400;0,9..144,700;0,9..144,800;1,9..144,800&family=Inter:wght@400;500;600;700&family=Caveat:wght@700`

**Spacing (px):** 4 · 8 · 12 · 16 · 24 · 32 · 48 · 64 · 96
**Radius (px):** 8 · 14 · 20 · 28 (round, friendly)
**Shadow:** accent-tinted. L1 `0 2px 4px rgba(139,92,246,.12)` · L2 `0 4px 12px rgba(139,92,246,.18)` · L3 `0 8px 24px rgba(236,72,153,.22)` · L4 `0 16px 40px rgba(139,92,246,.30)`
**Motion:** ease `cubic-bezier(.34,1.56,.64,1)` (bouncy overshoot) · 180ms / 320ms / 500ms

**Aesthetic rules:**
- Gradients are encouraged — pink-to-purple is the signature.
- Emojis can punctuate UI sparingly (not on every line).
- Rounded corners are aggressive — pills for buttons, 20px+ for cards.
- One handwritten accent (Caveat) per view, rotated slightly for character.
- Bounce easing is default — everything overshoots slightly.
- Shadows tinted with the accent color, never neutral gray.

---

#### 5. Soft Premium

**Feel:** Calm, reassuring, upscale. Desaturated, low-contrast but confident. Feels like a product for people with nothing to prove. Generous whitespace. No loud signals.

**Color ramp (light → dark):** 50 `#fafaf9` · 100 `#f5f5f4` · 200 `#e7e5e4` · 300 `#d6d3d1` · 400 `#a8a29e` · 500 `#78716c` · 600 `#57534e` · 700 `#44403c` · 800 `#292524` · 900 `#1c1917`
**Accents:** mint-gray `#5b8c7a` · caramel `#d4a373` · muted blue `#5b7b9a`

**Typography:**
- Headline: Inter Tight 600, letter-spacing -0.015em
- Body: Inter 400, line-height 1.6
- Mono: JetBrains Mono 400
- Scale (px): 12 · 14 · 15 · 19 · 26 · 38 (compressed, restrained)
- Google Fonts: `Inter+Tight:wght@500;600&family=Inter:wght@400;500;600&family=JetBrains+Mono:wght@400`

**Spacing (px):** 4 · 8 · 12 · 20 · 28 · 40 · 56 · 80 · 112 (generous)
**Radius (px):** 8 · 12 · 18 · 28
**Shadow:** soft, large spread, very low opacity. L1 `0 1px 3px rgba(28,25,23,.04)` · L2 `0 4px 16px rgba(28,25,23,.05)` · L3 `0 12px 32px rgba(28,25,23,.06)` · L4 `0 24px 56px rgba(28,25,23,.08)`
**Motion:** ease `cubic-bezier(.4,0,.2,1)` · 200ms / 350ms / 500ms

**Aesthetic rules:**
- Desaturated palette only — no bright hues.
- Generous whitespace; minimum 24px padding on containers.
- Shadows are always soft with large spread, low opacity.
- Off-black text (#1c1917), never pure black.
- One muted accent color per view.
- Nothing competes for attention — the hierarchy is patient.

---

#### 6. Technical Documentary

**Feel:** Dense, authoritative, information-first. Feels like reading serious reference material. Monochrome or near-monochrome with strategic semantic accents. Heavy on tables, lists, code.

**Color ramp (light → dark):** 50 `#f8fafc` · 100 `#f1f5f9` · 200 `#e2e8f0` · 300 `#cbd5e1` · 400 `#94a3b8` · 500 `#64748b` · 600 `#475569` · 700 `#334155` · 800 `#1e293b` · 900 `#0f172a`
**Accents:** link `#0284c7` · success `#059669` · warning `#d97706` · danger `#dc2626`

**Typography:**
- Headline: Inter 700, letter-spacing -0.015em (no alt display font)
- Body: Inter 400, line-height 1.55
- Mono: IBM Plex Mono 400 — code is first-class
- Scale (px): 12 · 14 · 16 · 20 · 26 · 36
- Google Fonts: `Inter:wght@400;500;600;700&family=IBM+Plex+Mono:wght@400;500`

**Spacing (px):** 4 · 8 · 12 · 16 · 24 · 32 · 48 · 64 · 96 (tight, efficient)
**Radius (px):** 2 · 4 · 6 · 8
**Shadow:** rarely used — prefer borders + spacing for structure. L1 `0 1px 0 rgba(15,23,42,.05)` · L2 `0 1px 2px rgba(15,23,42,.06)` · L3 `0 2px 4px rgba(15,23,42,.08)` · L4 `0 4px 8px rgba(15,23,42,.10)`
**Motion:** ease `ease` (system default) · 80ms / 140ms / 220ms

**Aesthetic rules:**
- Use inline code, tables, and definition lists liberally — they're native.
- Accent color used ONLY for links and semantic indicators.
- High information density is a feature, not a bug.
- Code blocks are UI, not afterthoughts — monospace everywhere code appears.
- Side-by-side columns for comparisons; vertical stacks for reference.
- Tables have zebra stripes; headers are bolded not uppercased.

---

#### 7. Warm Handmade

**Feel:** Crafted, personal, small-batch. Feels like it was made by one person who cared. Slightly imperfect by design. Muted earthy tones.

**Color ramp (light → dark):** 50 `#f7f4ed` · 100 `#eee7d8` · 200 `#dcc9a5` · 300 `#c4a574` · 400 `#9c7f4f` · 500 `#6b5538` · 600 `#4a3b28` · 700 `#33281b` · 800 `#211a12` · 900 `#13100a`
**Accents:** berry `#7c2d12` · sage `#5f7c3e` · dusty blue `#4a6978`

**Typography:**
- Headline: Fraunces 600 (opsz friendly)
- Body: Spectral 400, line-height 1.65
- Mono: Cascadia Mono 400
- Scale (px): 13 · 15 · 17 · 22 · 30 · 46
- Google Fonts: `Fraunces:opsz,wght@9..144,400;9..144,600;9..144,700&family=Spectral:wght@400;500;600&family=Cascadia+Mono`

**Spacing (px):** 4 · 8 · 12 · 18 · 26 · 38 · 56 · 84 · 120 (organic, irregular)
**Radius (px):** 4 · 8 · 14 · 22 (friendly, not bubbly)
**Shadow:** soft, warm-tinted. L1 `0 1px 3px rgba(107,85,56,.08)` · L2 `0 3px 10px rgba(107,85,56,.10)` · L3 `0 8px 20px rgba(107,85,56,.12)` · L4 `0 14px 32px rgba(107,85,56,.14)`
**Motion:** ease `cubic-bezier(.4,0,.2,1)` · 160ms / 280ms / 440ms

**Aesthetic rules:**
- Off-center, slightly asymmetric layouts welcomed.
- Warm earth palette only — no cool colors.
- A single hand-drawn flourish per view (underline squiggle, arrow).
- Line-heights slightly generous (1.6–1.7).
- Section dividers are hairline or dotted, never bold.
- Photographs treated like prints — consistent crops, occasional vignette.

---

#### 8. Glassmorphic

**Feel:** Translucent, depth-rich, atmospheric. Depth via layered translucency. Colorful but desaturated by the glass effect. Feels contemporary and spatial.

**Color ramp (light → dark):** 50 `#fafafa` · 100 `rgba(255,255,255,.6)` · 200 `rgba(255,255,255,.4)` · 300 `#e5e7eb` · 400 `#9ca3af` · 500 `#6b7280` · 600 `#4b5563` · 700 `#374151` · 800 `#1f2937` · 900 `#0f1419`
**Accents:** vivid through glass — coral `#ff8a65` at .5 alpha · sky `#60a5fa` at .5 · lilac `#c4b5fd` at .5

**Typography:**
- Headline: Inter 600, letter-spacing -0.015em
- Body: Inter 400, line-height 1.5
- Mono: SF Mono fallback to Menlo 400
- Scale (px): 12 · 14 · 16 · 20 · 28 · 44
- Google Fonts: `Inter:wght@400;500;600;700`

**Spacing (px):** 4 · 8 · 12 · 16 · 24 · 32 · 48 · 72 · 112
**Radius (px):** 10 · 16 · 24 · 36 (soft, bubble-like)
**Shadow:** L1 `0 2px 8px rgba(0,0,0,.04)` · L2 `0 4px 16px rgba(0,0,0,.06)` · L3 `0 8px 32px rgba(0,0,0,.08)` · L4 `0 16px 48px rgba(0,0,0,.10)` — paired with `backdrop-filter: blur(...)`
**Motion:** ease smooth-springy · 180ms / 300ms / 500ms

**Aesthetic rules:**
- Surfaces use `backdrop-filter: blur(20–40px)` with translucent bg (`rgba(255,255,255,.5–.7)`).
- Behind every glass panel, place a colorful gradient or vibrant content — otherwise the glass effect is invisible.
- Borders are `rgba(255,255,255,.3)` — glass rim.
- Text sits on top with solid colors — never translucent text.
- One glass layer per view usually is enough; stacking destroys the effect.
- Dark-mode variant comes naturally; consider offering both.

---

#### 9. Neo-Classical

**Feel:** Serif-heavy, restrained, editorial-adjacent. Modern classical — prestige-media feel. Warm neutrals. Generous vertical rhythm.

**Color ramp (light → dark):** 50 `#faf9f7` · 100 `#f2ece0` · 200 `#e4d9c5` · 300 `#c4b195` · 400 `#96825c` · 500 `#6b5a38` · 600 `#4c3f24` · 700 `#332a18` · 800 `#1f1a10` · 900 `#0f0c08`
**Accents:** jewel red `#8c1c13` · forest `#2d4a2a` · deep blue `#1e3a5f`

**Typography:**
- Headline: Playfair Display 700, italic variants welcome, letter-spacing -0.01em
- Body: Lora 400, line-height 1.65
- Display alt: Spectral 400 for lead-ins
- Scale (px): 13 · 15 · 18 · 24 · 34 · 52
- Google Fonts: `Playfair+Display:ital,wght@0,400;0,700;1,400;1,700&family=Lora:ital,wght@0,400;0,500;1,400&family=Spectral:wght@400;500`

**Spacing (px):** 4 · 8 · 12 · 18 · 28 · 44 · 68 · 100 · 144 (grand, editorial)
**Radius (px):** 0 · 2 · 4 · 6 (nearly angular)
**Shadow:** barely used; typography and rules carry structure. L1 `0 1px 1px rgba(15,12,8,.04)` · L2 `0 2px 4px rgba(15,12,8,.06)` · L3 `0 4px 12px rgba(15,12,8,.08)` · L4 `0 10px 24px rgba(15,12,8,.10)`
**Motion:** ease slow · 200ms / 400ms / 600ms (deliberate)

**Aesthetic rules:**
- Display italics on headlines are a signature.
- Small caps for navigation and labels.
- Horizontal rules (hairline, sometimes doubled) as dividers.
- Drop caps welcome for long-form content.
- Generous vertical rhythm; section spacing is grand.
- Minimal color — let the serif typography carry the voice.

---

#### 10. Kinetic Modern

**Feel:** Motion-forward, vivid, alive. Things move. Subtle micro-animations. Crisp geometry with energy underneath. Feels recent, capable, deliberate.

**Color ramp (light → dark):** 50 `#fafbff` · 100 `#f0f3ff` · 200 `#d6dcff` · 300 `#b0bcff` · 400 `#7f92ff` · 500 `#4f6bff` · 600 `#3b4dcc` · 700 `#29368f` · 800 `#161d4a` · 900 `#0a0e24`
**Accents:** lime `#a3e635` · coral `#fb7185` · cyan `#22d3ee`

**Typography:**
- Headline: Space Grotesk 700, letter-spacing -0.02em
- Body: Inter 400, line-height 1.55
- Mono: JetBrains Mono 500
- Scale (px): 12 · 14 · 16 · 22 · 32 · 48
- Google Fonts: `Space+Grotesk:wght@500;600;700&family=Inter:wght@400;500;600&family=JetBrains+Mono:wght@400;500`

**Spacing (px):** 4 · 8 · 12 · 16 · 24 · 32 · 48 · 64 · 96
**Radius (px):** 6 · 10 · 14 · 20
**Shadow:** colored, subtle. L1 `0 2px 6px rgba(79,107,255,.10)` · L2 `0 4px 12px rgba(79,107,255,.14)` · L3 `0 8px 24px rgba(79,107,255,.20)` · L4 `0 16px 40px rgba(79,107,255,.28)`
**Motion:** ease `cubic-bezier(.5,1.5,.5,1)` (springy) · 140ms / 240ms / 380ms

**Aesthetic rules:**
- Hover / focus states always include visible motion, never purely visual.
- Accent colors used as spot highlights — never as main body color.
- Geometric shapes (circles, pills, diagonal stripes) appear as decorative accents.
- Text can be animated on entry (fade-up, stagger).
- High contrast; a dark background variant is default-ready.
- Include at least one subtle motion cue (animated underline, pulse, hover-lift) in the rendered frame.

---

## HTML TEMPLATE REFERENCE

### Decision Page HTML Structure

Each decision page must be a self-contained HTML file with this structure:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Decision [N]: [Title] — [Project Name]</title>
  <!-- NOTE: Use plain numbers (1, 2, 3) not zero-padded (001, 002, 003) in display text.
       Zero-padding is only for filenames (decision-001-slug.html). -->
  <!-- Load Google Fonts for EVERY visual-direction decision — each tradition has its own pairing. -->
  <!-- Combine all 4 traditions' font imports into one <link> in <head>. -->
  <!-- For other decisions (IA, interaction, technical), load fonts only if typography is part of the decision. -->
  <!-- Load Chart.js ONLY if the decision involves data visualization -->
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

    .category-badge.technical { background: #ede9fe; color: #6d28d9; }
    .category-badge.visual { background: #fce7f3; color: #be185d; }
    .category-badge.interaction { background: #e0f2fe; color: #0369a1; }
    .category-badge.ia { background: #ecfdf5; color: #047857; }

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

    /* Badges container — stacks recommended and chosen badges vertically */
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
      padding: 1.25rem;
      background: #f8fafc;
      min-height: 360px;
      display: flex;
      align-items: stretch;
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

    /* Color-code the option column headers */
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

    /* Recommended column highlight */
    .comparison-table .col-recommended { background: #fffbeb !important; }

    /* Chosen column highlight */
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

    .page-footer strong {
      color: #334155;
    }

    /* === NAV LINK TO LANDING PAGE === */
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
      body { padding: 1rem; }
    }

    /* === EXTENDED OPTION COLORS (for "more options") === */
    .option-card.option-e .option-label { color: #ea580c; }
    .option-card.option-f .option-label { color: #db2777; }
    .option-card.option-g .option-label { color: #0d9488; }
    .option-card.option-h .option-label { color: #4338ca; }
    .option-card.option-i .option-label { color: #d97706; }
    .option-card.option-j .option-label { color: #9333ea; }
    .option-card.option-k .option-label { color: #65a30d; }
    .option-card.option-l .option-label { color: #0284c7; }

    /* Extended comparison table header colors */
    .comparison-table th.col-e { color: #ea580c; }
    .comparison-table th.col-f { color: #db2777; }
    .comparison-table th.col-g { color: #0d9488; }
    .comparison-table th.col-h { color: #4338ca; }
    .comparison-table th.col-i { color: #d97706; }
    .comparison-table th.col-j { color: #9333ea; }
    .comparison-table th.col-k { color: #65a30d; }
    .comparison-table th.col-l { color: #0284c7; }

    /* === FLOW DIAGRAM STYLES (for interaction decisions) === */

    /* Vertical numbered flow: each step is a numbered box in a single column.
       This is the ONLY supported flow diagram layout. Do NOT use horizontal rows. */
    .flow-container {
      display: flex;
      flex-direction: column;
      align-items: stretch;
      gap: 0;
      padding: 0.5rem;
      width: 100%;
      max-width: 280px;
    }

    .flow-step {
      display: flex;
      align-items: center;
      gap: 10px;
      background: white;
      border: 2px solid #e2e8f0;
      border-radius: 10px;
      padding: 8px 12px;
      font-size: 0.7rem;
      font-weight: 600;
      color: #334155;
      text-align: left;
      box-shadow: 0 1px 2px rgba(0,0,0,0.05);
    }

    .flow-step.highlight {
      border-color: #6366f1;
      background: #eff6ff;
      color: #1d4ed8;
    }

    .flow-step-number {
      width: 22px;
      height: 22px;
      border-radius: 50%;
      background: #e2e8f0;
      color: #475569;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 0.6rem;
      font-weight: 700;
      flex-shrink: 0;
    }

    .flow-step.highlight .flow-step-number {
      background: #6366f1;
      color: white;
    }

    .flow-step-label {
      flex: 1;
    }

    /* Down arrow between steps */
    .flow-down-arrow {
      font-size: 1rem;
      color: #94a3b8;
      padding: 2px 0;
      text-align: center;
    }

    /* Error/failure step variant */
    .flow-step.error {
      border-color: #e11d48;
      background: #fff1f2;
      color: #be123c;
    }

    .flow-step.error .flow-step-number {
      background: #e11d48;
      color: white;
    }

    /* Branch label — dashed separator showing a conditional path */
    .flow-branch-label {
      font-size: 0.6rem;
      font-weight: 700;
      color: #94a3b8;
      text-transform: uppercase;
      letter-spacing: 0.08em;
      text-align: center;
      padding: 6px 0 2px;
      border-top: 1px dashed #cbd5e1;
      margin-top: 4px;
    }

    /* === SITE MAP / NAV VISUALIZATION (for IA decisions) === */
    .sitemap {
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 8px;
      padding: 0.5rem;
    }

    .sitemap-level {
      display: flex;
      gap: 8px;
      flex-wrap: wrap;
      justify-content: center;
    }

    .sitemap-node {
      background: white;
      border: 2px solid #e2e8f0;
      border-radius: 8px;
      padding: 6px 12px;
      font-size: 0.7rem;
      font-weight: 600;
      color: #334155;
      box-shadow: 0 1px 2px rgba(0,0,0,0.05);
    }

    .sitemap-node.primary {
      background: #6366f1;
      color: white;
      border-color: #6366f1;
    }

    .sitemap-connector {
      width: 2px;
      height: 12px;
      background: #cbd5e1;
      margin: 0 auto;
    }

    /* === ARCHITECTURE DIAGRAM (for technical decisions) === */
    .arch-diagram {
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 6px;
      padding: 0.5rem;
      width: 100%;
    }

    .arch-layer {
      display: flex;
      gap: 8px;
      justify-content: center;
      flex-wrap: wrap;
      width: 100%;
    }

    .arch-box {
      background: white;
      border: 2px solid #e2e8f0;
      border-radius: 8px;
      padding: 8px 14px;
      font-size: 0.7rem;
      font-weight: 600;
      color: #334155;
      text-align: center;
      min-width: 70px;
      box-shadow: 0 1px 2px rgba(0,0,0,0.05);
    }

    .arch-box.frontend { border-color: #7c3aed; color: #7c3aed; }
    .arch-box.backend { border-color: #0891b2; color: #0891b2; }
    .arch-box.database { border-color: #059669; color: #059669; }
    .arch-box.service { border-color: #dc2626; color: #dc2626; }

    .arch-arrow {
      font-size: 1rem;
      color: #94a3b8;
      text-align: center;
    }
  </style>
</head>
<body>
  <a href="index.html" class="back-link">← All Decisions</a>

  <header>
    <span class="decision-number">Decision [N] of [TOTAL]</span>
    <!-- Use plain numbers: "Decision 3 of 8" NOT "Decision 003 of 8" -->
    <h1>[Decision Title]</h1>
    <span class="category-badge [technical|visual|interaction|ia]">[Category Name]</span>
    <p class="decision-description">
      [3-5 sentences in plain English explaining what this decision is about,
       why it matters, and what factors should influence the choice.
       Write like you're explaining it to a smart friend over coffee.]
    </p>
    <p class="instruction">
      Respond with: <strong>"Option B"</strong> · <strong>"Option A but [change]"</strong> · <strong>"more options"</strong>
    </p>
  </header>

  <main class="options-grid">
    <!-- Repeat this card structure for each option (A, B, C, D) -->
    <article class="option-card option-a">
      <div class="card-header">
        <div class="card-header-top">
          <span class="option-label">Option A</span>
          <!-- Badges container: holds recommended and/or chosen badges, stacked vertically.
               ALWAYS include this container. Only include the badges that apply. -->
          <div class="card-badges">
            <!-- Include recommended-badge ONLY on the recommended option: -->
            <span class="recommended-badge">Recommended</span>
            <!-- chosen-badge is always present in HTML but hidden via CSS until .chosen class is added to the card: -->
            <span class="chosen-badge">Chosen</span>
          </div>
        </div>
        <h2 class="option-title">[2-4 Word Evocative Name]</h2>
      </div>

      <div class="visual-preview">
        <!-- CONTEXT-DEPENDENT VISUAL — see rules below -->
      </div>

      <div class="option-summary">
        [3-4 sentences in plain English. What does choosing this option actually mean?
         How will it affect the project? Who is this a good fit for?
         Write conversationally — no jargon without explanation.]
      </div>

      <div class="verdict">
        <div class="pros">
          <h3>Works well when</h3>
          <ul>
            <li>[Specific context where this shines]</li>
            <li>[Another strength]</li>
            <li>[A third point if genuinely useful]</li>
          </ul>
        </div>
        <div class="cons">
          <h3>Watch out for</h3>
          <ul>
            <li>[Honest trade-off — not FUD, a real consideration]</li>
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
          <tr>
            <td>[Dimension 1, e.g. "Learning curve"]</td>
            <td class="[col-recommended]">[Value]</td>
            <td class="[col-recommended]">[Value]</td>
            <td class="[col-recommended]">[Value]</td>
            <td class="[col-recommended]">[Value]</td>
          </tr>
          <!-- 4-8 rows comparing meaningful dimensions -->
          <!-- The recommended option's column cells should all have class "col-recommended" -->
          <!-- After a choice is made, the chosen column cells get class "col-chosen" instead -->
        </tbody>
      </table>
    </div>
  </section>

  <!-- PAGE FOOTER -->
  <div class="page-footer">
    <p>Respond with: <strong>"Option A"</strong>, <strong>"Option C but [your change]"</strong>, or <strong>"more options"</strong></p>
    <p>To change a past decision: <strong>"for decision-001 I want Option B instead"</strong></p>
  </div>

</body>
</html>
```

### Visual Preview Rules — What To Put in `.visual-preview`

The visual preview should contain a **real rendered visual**, not just text. What to show depends on the decision type:

**For Visual/UX decisions (visual direction, overall style):**

Render a **full app frame** per option — a realistic slice of a product screen that lets the user feel the aesthetic. Target dimensions: 280–320px wide, 340–420px tall. A thumbnail-sized mini-card is not enough — an aesthetic needs room to breathe. If you show four options in a thumbnail format, they'll collapse into "four different colored rectangles" and the user won't feel the difference.

Pull the chosen tradition's tokens and aesthetic rules from the **Aesthetic Traditions Library** (see that section above) and compose the frame inline. Every option card MUST include all of these elements:

1. **Browser chrome** — three traffic-light dots on a neutral bar
2. **App header** with brand name (using the tradition's headline font) and 3 nav items (using its body font)
3. **Hero section** with: a kicker (small, accent-colored, usually uppercase unless the tradition's rules say otherwise), a headline (display treatment using the tradition's max scale step), a subhead (body font, ~60ch max width), and a primary CTA button composed from the tradition's aesthetic
4. **Content strip** with 2–3 cards composed following the tradition's aesthetic rules

**Use realistic product-relevant content** — never `Card 1`, `Item A`, or Lorem ipsum. Pull names, taglines, and item content from the product context (the strategy brief if available, otherwise invent reasonable examples fitting the product's domain).

**Skeleton** — replace every `[token]` with the chosen tradition's actual values, and adapt the structure to the tradition's aesthetic rules:

```html
<div style="width:300px;background:[ramp-50];border:1px solid [ramp-200];border-radius:[radius-lg];overflow:hidden;box-shadow:[shadow-L2];font-family:[body-font]">
  <!-- chrome -->
  <div style="background:[ramp-100];padding:6px 10px;display:flex;gap:5px;border-bottom:1px solid [ramp-200]">
    <span style="width:8px;height:8px;border-radius:50%;background:#ef4444"></span>
    <span style="width:8px;height:8px;border-radius:50%;background:#eab308"></span>
    <span style="width:8px;height:8px;border-radius:50%;background:#22c55e"></span>
  </div>
  <!-- nav -->
  <header style="display:flex;justify-content:space-between;align-items:center;padding:10px 14px;border-bottom:1px solid [ramp-200]">
    <div style="font-family:[headline-font];font-weight:[headline-weight];font-size:15px;color:[ramp-900];letter-spacing:[headline-tracking]">[Brand]</div>
    <nav style="display:flex;gap:14px;font-size:12px;color:[ramp-600];font-weight:500">
      <span>Nav A</span><span>Nav B</span><span>Nav C</span>
    </nav>
  </header>
  <!-- hero -->
  <section style="padding:24px 16px">
    <div style="font-size:11px;font-weight:700;color:[accent];letter-spacing:.08em;text-transform:uppercase">Kicker</div>
    <h1 style="font-family:[headline-font];font-weight:[headline-weight];font-size:32px;color:[ramp-900];letter-spacing:[headline-tracking];line-height:1.05;margin:6px 0 0">Realistic headline, two lines max</h1>
    <p style="font-size:13px;color:[ramp-600];margin-top:8px;line-height:1.55;max-width:260px">Realistic subhead describing what the product does in one sentence.</p>
    <button style="font-family:[body-font];font-weight:600;font-size:13px;padding:8px 16px;background:[ramp-900];color:[ramp-50];border:none;border-radius:[radius-md];margin-top:14px;cursor:pointer">Primary CTA →</button>
  </section>
  <!-- content strip -->
  <div style="display:grid;grid-template-columns:1fr 1fr;gap:6px;padding:10px 14px 14px">
    <div style="background:[ramp-50];border:1px solid [ramp-200];border-radius:[radius-md];padding:8px 10px">
      <div style="font-family:[headline-font];font-weight:600;font-size:12px;color:[ramp-900]">Realistic item 1</div>
      <div style="font-size:10px;color:[ramp-500];margin-top:2px">context line</div>
    </div>
    <div style="background:[ramp-50];border:1px solid [ramp-200];border-radius:[radius-md];padding:8px 10px">
      <div style="font-family:[headline-font];font-weight:600;font-size:12px;color:[ramp-900]">Realistic item 2</div>
      <div style="font-size:10px;color:[ramp-500];margin-top:2px">context line</div>
    </div>
  </div>
</div>
```

This skeleton is a *starting point*, not the final rule. Each tradition's aesthetic rules determine how to modify the skeleton. A tradition with offset-solid shadows applies them to cards; a tradition with gradients on CTAs applies them to the primary button; a tradition with italic kickers applies font-style to the kicker text.

**IMPORTANT — load the right Google Fonts for every tradition being shown.** The HTML `<head>` must include a Google Fonts import line covering the headline + body + mono fonts of every tradition in the 4 options. If you present Functional Minimalism + Editorial Print + Raw Brutalist + Playful Maximalist, you need Inter Tight, Inter, Fraunces, Source Serif 4, Archivo Black, JetBrains Mono, Caveat, etc., all loaded at once.

Do NOT:
- Use placeholder text in the final output — every string must be product-specific and realistic
- Mix tokens across traditions on the same option
- Ignore the tradition's aesthetic rules (e.g., use gradients on a tradition whose rules forbid them)
- Ship without running the **Principles Checklist** (see that section below)

**For component-design decisions (button styles, card treatments, specific component patterns) when a tradition has already been chosen:**
Compose the rendered component using the chosen tradition's tokens. Do NOT fall back to the skeleton above — component decisions render the specific component(s) under discussion (e.g., 4 different button treatments, all composed from the tradition's tokens with meaningfully different takes).

**For Interaction decisions (user flows, how actions work):**
Build a vertical numbered flow diagram using `.flow-container`, `.flow-step`, `.flow-step-number`, `.flow-step-label`, and `.flow-down-arrow`.

**ALWAYS use this pattern — a vertical column of numbered steps:**
```html
<div class="flow-container">
  <div class="flow-step">
    <span class="flow-step-number">1</span>
    <span class="flow-step-label">Browse books nearby</span>
  </div>
  <div class="flow-down-arrow">↓</div>
  <div class="flow-step highlight">
    <span class="flow-step-number">2</span>
    <span class="flow-step-label">Tap "Request to Borrow"</span>
  </div>
  <div class="flow-down-arrow">↓</div>
  <div class="flow-step">
    <span class="flow-step-number">3</span>
    <span class="flow-step-label">Owner gets notified</span>
  </div>
  <div class="flow-down-arrow">↓</div>
  <div class="flow-step highlight">
    <span class="flow-step-number">4</span>
    <span class="flow-step-label">Owner approves request</span>
  </div>
  <div class="flow-down-arrow">↓</div>
  <div class="flow-step">
    <span class="flow-step-number">5</span>
    <span class="flow-step-label">Chat opens — arrange pickup</span>
  </div>
  <div class="flow-down-arrow">↓</div>
  <div class="flow-step highlight">
    <span class="flow-step-number">6</span>
    <span class="flow-step-label">Both confirm handoff — done!</span>
  </div>
</div>
```

**Branching flows (when the path splits based on a condition):**
Use `.flow-branch-label` for the condition and sub-numbered steps (4a, 4b, etc.) for each branch. Use `.flow-step.error` for failure/error steps:
```html
<div class="flow-container">
  <div class="flow-step">
    <span class="flow-step-number">1</span>
    <span class="flow-step-label">Tap "Request to Borrow"</span>
  </div>
  <div class="flow-down-arrow">↓</div>
  <div class="flow-step error">
    <span class="flow-step-number">2</span>
    <span class="flow-step-label">Request fails</span>
  </div>
  <div class="flow-down-arrow">↓</div>
  <div class="flow-step highlight">
    <span class="flow-step-number">3</span>
    <span class="flow-step-label">Button changes to suggested action</span>
  </div>
  <div class="flow-branch-label">if book was taken ↓</div>
  <div class="flow-step">
    <span class="flow-step-number">3a</span>
    <span class="flow-step-label">Button says "See Similar Books"</span>
  </div>
  <div class="flow-branch-label">if network error ↓</div>
  <div class="flow-step">
    <span class="flow-step-number">3b</span>
    <span class="flow-step-label">Button says "Try Again"</span>
  </div>
</div>
```

**IMPORTANT flow diagram rules:**
- ALWAYS use a vertical single-column layout. NEVER use horizontal rows — they create confusing arrow connections.
- ALWAYS number every step with `.flow-step-number` (1, 2, 3, etc.). Use sub-numbers (4a, 4b) for branches.
- Use `↓` arrows (`.flow-down-arrow`) between sequential steps
- Use `.flow-branch-label` with a condition ("if book was taken ↓") before branching sub-steps
- Use `.flow-step.error` for failure/error steps (red styling)
- Use `.highlight` on the 2-3 most important/differentiating steps (the ones that make this option unique)
- Keep to 4-7 steps. Combine trivial steps if needed.
- In the **option-summary text below the diagram**, reference step numbers: "In step 2, the borrower taps..." This ties the visual to the explanation.
- Each step label should be a short action phrase (verb first): "Tap request button", "Owner approves", "Chat opens"

**For Information Architecture decisions (navigation, content hierarchy):**
Build a mini site-map using the `.sitemap`, `.sitemap-level`, and `.sitemap-node` CSS classes:
```html
<div class="sitemap">
  <div class="sitemap-level">
    <div class="sitemap-node primary">Map View</div>
  </div>
  <div class="sitemap-connector"></div>
  <div class="sitemap-level">
    <div class="sitemap-node">My Books</div>
    <div class="sitemap-node">Browse</div>
    <div class="sitemap-node">Messages</div>
    <div class="sitemap-node">Profile</div>
  </div>
</div>
```

**For Technical decisions (framework, database, architecture):**
Build an architecture diagram using the `.arch-diagram`, `.arch-layer`, and `.arch-box` CSS classes:
```html
<div class="arch-diagram">
  <div class="arch-layer">
    <div class="arch-box frontend">React SPA</div>
  </div>
  <div class="arch-arrow">↕</div>
  <div class="arch-layer">
    <div class="arch-box backend">REST API</div>
    <div class="arch-box service">Auth</div>
  </div>
  <div class="arch-arrow">↕</div>
  <div class="arch-layer">
    <div class="arch-box database">PostgreSQL</div>
  </div>
</div>
```

For some technical decisions (like "which database"), a simple stats/features list may be more useful than a diagram — use your judgment. The point is to help the user *see* the difference, not to force a visual where one doesn't help.

### Comparison Table Dimensions

Choose 5-8 dimensions that genuinely matter for the specific decision. Adapt per category:

**Technical decisions:**
- Learning curve, Community/ecosystem, Performance, Scalability, Cost, Hosting options, Best suited for

**Visual/UX decisions:**
- Mood/feeling, Accessibility, Brand alignment, User demographic fit, Trend durability, Implementation effort

**Interaction decisions:**
- Number of steps, User effort, Error recovery, Speed to complete, Flexibility, Familiarity

**IA decisions:**
- Discoverability, Scalability (as content grows), Mobile friendliness, Cognitive load, Engagement pattern

---

## LANDING PAGE TEMPLATE

The landing page at `.decisions/index.html` shows all decisions at a glance:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Decision Hub — [Project Name]</title>
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

    .container {
      max-width: 900px;
      margin: 0 auto;
    }

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

    /* Progress bar */
    .progress-section {
      margin: 1.5rem 0;
    }

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

    /* Decision list */
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

    .decision-item:hover {
      border-color: #6366f1;
    }

    .decision-item.resolved {
      border-left: 4px solid #059669;
    }

    .decision-item.pending {
      border-left: 4px solid #f59e0b;
    }

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

    .decision-item.resolved .decision-number-badge {
      background: #ecfdf5;
      color: #059669;
    }

    .decision-item.pending .decision-number-badge {
      background: #fffbeb;
      color: #d97706;
    }

    .decision-info {
      flex: 1;
    }

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

    .landing-category-badge.technical { background: #ede9fe; color: #6d28d9; }
    .landing-category-badge.visual { background: #fce7f3; color: #be185d; }
    .landing-category-badge.interaction { background: #e0f2fe; color: #0369a1; }
    .landing-category-badge.ia { background: #ecfdf5; color: #047857; }

    .decision-status {
      font-size: 0.8rem;
      color: #64748b;
    }

    .decision-status .chosen-text {
      color: #059669;
      font-weight: 600;
    }

    .decision-status .pending-text {
      color: #d97706;
      font-weight: 600;
    }

    .decision-summary {
      font-size: 0.8rem;
      color: #64748b;
      margin-top: 0.25rem;
    }

    .arrow-icon {
      color: #94a3b8;
      font-size: 1.1rem;
      flex-shrink: 0;
    }

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
      <h1>Decision Hub</h1>
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
      <!-- For each decision: -->
      <a href="decision-001-slug.html" class="decision-item [resolved|pending]">
        <div class="decision-number-badge">1</div>
        <div class="decision-info">
          <h3>[Decision Title]</h3>
          <div class="decision-meta">
            <span class="landing-category-badge [technical|visual|interaction|ia]">[Category]</span>
            <span class="decision-status">
              <!-- If resolved: -->
              <span class="chosen-text">Chosen: Option B — [Title]</span>
              <!-- If pending: -->
              <span class="pending-text">Pending</span>
            </span>
          </div>
          <p class="decision-summary">[One sentence summary]</p>
        </div>
        <span class="arrow-icon">→</span>
      </a>
      <!-- ... repeat for each decision ... -->
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

**User skips a decision:** "Skip this one" or "doesn't matter" → Set status to "chosen" with `chosenOption: "skip"`, `chosenTitle: "Skipped — AI will decide"`. Use your recommendation when implementing.

**User gives a custom answer not matching any option:** "I want to use Postgres with Prisma ORM" → Generate a full visual card for their answer with the same treatment as any AI-generated option (architecture diagram, flow diagram, rendered UI, whatever fits). Show it as the chosen option alongside the original options. Set `chosenOption: "custom"`, `chosenTitle: "[their description]"`. Store reasoning if they gave one.

**User wants to revisit the decision list:** "What decisions have we made?" or "show me the overview" → Open the landing page: `open .decisions/index.html`

**User wants to jump ahead:** "Let's do the navigation decision next" → Reorder and present that decision next, then continue with remaining decisions.

**Existing .decisions directory:** If `.decisions/` already exists from a prior session, read `decisions.json` to understand what's been decided. Resume from the first pending decision. Tell the user: "I see we've already made N decisions. Picking up where we left off with Decision M: [Title]."

**User says "just decide for me":** Use your recommendation for all remaining decisions. Record them all, generate the implementation plan, and present it.

---

## PRINCIPLES CHECKLIST — Quality Gate Before Ship

Before opening any visual-direction or component-design decision page in the browser (Step 3e), run this checklist mentally against the generated HTML. For each **mandatory** check that fails, regenerate the specific piece that's failing, then re-run the checklist. Do not open the file until all mandatory checks pass.

### Mandatory checks (must pass before opening)

1. **Contrast ≥ 4.5:1 for all body text.** Every text color used against its surface must pass WCAG AA. The tradition's body text color (typically step 600 or 700) must have sufficient contrast against its container background (typically step 50 or 100). Headlines may use 3:1.

2. **Typography scale applied.** Every font-size used must correspond to a step in the tradition's declared scale. No one-off sizes like `font-size:17px` if the scale is 12/14/16/20/28/40.

3. **Tokens used everywhere.** Every color, spacing value, radius, shadow, and font must come from the chosen tradition's token set. No arbitrary hex values, no ad-hoc paddings like `padding:13px` if the scale is 4/8/12/16.

4. **Four options visually distinct.** Compare the four rendered previews side-by-side as if you were the user seeing them for the first time. If two options share the same dominant hue family, headline weight, AND layout rhythm, one must be regenerated with a meaningfully different expression. "Different accent color" alone is not enough — the four should feel like four distinct design philosophies (not four variants of one).

5. **Recommended option has explicit reasoning.** The option-summary for the recommended choice must reference the product positioning or user type explicitly ("I'd recommend this because the user base is X and this tradition speaks to that audience"). "It's a good pick" is not enough.

6. **No broken primitives.** Every button, input, badge, card, and header in the preview must render with valid token values. No undefined CSS variables, no unreplaced `[token]` placeholders, no "Card 1" / "Item A" dummy text.

7. **Tradition's aesthetic rules honored.** For each option, mentally re-check its tradition's rules list. A Raw Brutalist option without offset-solid shadows failed rule 1. An Editorial Print option without Fraunces opsz axis failed rule 1. If any tradition rule is violated, regenerate.

### Soft checks (log a note, don't block ship)

- Does each option feel emotionally distinct, not just mechanically distinct? If every option feels equally "safe" or equally "bold," the spread is off.
- Are content strings realistic to the actual product? Generic content undermines even good aesthetics.
- Does the recommended option genuinely feel like the best fit, or did you default to the most neutral one?

### If any mandatory check fails

1. Regenerate only the failing piece — don't start over.
2. Re-run the full checklist on the regenerated result.
3. Repeat until all mandatory checks pass.
4. Only then open the file in the browser.

---

## IMPORTANT REMINDERS

1. **Never skip the decision page.** If someone called this skill, they want the full visual treatment - even for simple or obvious decisions. Never say "that's straightforward, I'll just do it." Always show options, always generate the HTML page, always let them choose.
2. **Always 4 options.** Not 3, not 5. Exactly 4. Unless the user has asked for more.
2. **Always include a recommendation.** Mark it with the amber badge. Explain WHY you recommend it in the option summary.
3. **Plain English everywhere.** If you use a technical term, explain it in the same sentence. "Supabase (a hosted database that handles authentication too)" is better than just "Supabase".
4. **The comparison table is mandatory.** Every decision page must have one below the cards. Pick dimensions that actually help differentiate the options.
5. **Visual previews should render actual UI.** For visual/UX decisions, build real HTML/CSS components. For interaction decisions, build flow diagrams. For IA decisions, build sitemaps. For technical decisions, build architecture diagrams or show code samples.
6. **Open the HTML automatically.** Always run `open .decisions/decision-NNN-slug.html` after generating.
7. **Update the landing page after every change.** The landing page should always reflect the current state.
8. **Self-contained HTML.** No external dependencies except CDN-hosted fonts or Chart.js when needed. Everything should work by opening the file directly in a browser.
9. **Wait for the user.** After presenting a decision, STOP and wait. Do not proceed to the next decision until the user has made a choice.
10. **Handle decision changes gracefully.** When a user changes a past decision, update both the individual HTML and the landing page. Flag any downstream decisions that might be affected.
