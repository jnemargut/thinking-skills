---
name: visual-design
description: Re-skin any existing HTML artifact OR SVG asset with a chosen aesthetic. Post-step aesthetic thinking. For HTML: walks through 5 decisions (tradition, color, type, mood, signature flourish) and rewrites the style block. For SVG icons/illustrations: walks through 3 decisions (tradition, stroke weight, color) and rewrites the stroke/fill attributes. Produces a side-by-side styled file + tokens.json sidecar. Use AFTER producing an artifact that looks visually generic — output from another thinking skill (strategize, game-plan, better-proposal, launch-playbook, product-design), an existing HTML page, or a raw SVG icon.
---

# Visual Design

You are helping the user re-skin an existing artifact with a distinct aesthetic. This is a post-step skill: something else produced the artifact; your job is to make it feel like *something*, not a generic template.

The skill supports two modes based on the input file:
- **HTML mode** (`.html` input) — 5-step flow: **Tradition → Color → Type → Mood → Signature Flourish**. Rewrites the `<style>` block.
- **SVG mode** (`.svg` input) — 3-step flow: **Tradition → Stroke → Color**. Rewrites stroke/fill attributes on paths. For icons, logos, and illustrations.

The user invokes `/visual-design [optional path to .html or .svg]`. Original is preserved; a new `<name>.styled.html` or `<name>.styled.svg` is written alongside, and `.visual-design/tokens.json` captures the decisions for reuse in the project.

**Core principles:**
- Write in plain English. Talk like a designer who cares, not a form-builder.
- Each decision presents exactly 4 options *except* Tradition (top 3 matched + 20+ catalog) and Flourish (top 3 curated + library) — those use name-based picking.
- Always include a recommendation.
- Show, don't just tell — every option renders a preview using the actual tradition's tokens.
- The skill's job ends when the styled file and `tokens.json` are on disk and opened for the user.

---

## PHASE 1 — Invocation

### Step 1a — Parse the argument + detect mode

The user invokes `/visual-design [path]` where `[path]` is optional.

**If a path is provided:**
- Verify the file exists.
- Branch on extension:
  - `.html` → set `MODE = "html"`, `TARGET = path`, proceed to Phase 2.
  - `.svg` → set `MODE = "svg"`, `TARGET = path`, proceed to Phase 2.
  - `.png` / `.jpg` / `.jpeg` / `.webp` → tell the user rasters aren't supported (the skill rewrites vector/style, not pixels). Suggest they convert to SVG or provide an HTML wrapper. Stop.
  - Any other extension → tell the user and fall through to the picker path below.

**If no path is provided:**
- Glob for recent styleable artifacts in priority order:
  1. `.decisions/*.html` (most common — decision-kit outputs)
  2. `*.{html,svg}` in cwd
  3. `**/*.{html,svg}` up to 2 levels deep (excluding `node_modules`, `dist`, `.git`)
- Sort results by modification time (newest first).
- If 0 matches: tell the user no artifacts found, suggest running a thinking skill first or passing a path. Stop.
- If 1 match: use it, but confirm with user first:
  > "Found **[filename]** (modified [time ago]). Re-skin this one? Reply `yes` or pass a different path."
- If 2+ matches: show the top 5 with the newest marked ✓:
  > "Found these artifacts. Which one do you want to re-skin?
  >   1. ✓ strategy-brief.html   (2 min ago)
  >   2. icon-download.svg        (1 hr ago)
  >   3. proposal.html            (yesterday)
  > Reply with a number, a filename, or paste a different path."

Once confirmed, set `MODE` from the extension. Wait for confirmation before proceeding.

### Step 1b — Check for project memory

Once `TARGET` is set, check for `.visual-design/tokens.json` at the project root (walk up from the target file to find the nearest one).

- **If present:** parse it. Note the previous tradition + flourish. On the Phase 3 starting screen, surface this as suggestion #1 in the top 3 with the label "your project aesthetic."
- **If absent:** normal flow — skill will create the `.visual-design/` directory later.

### Step 1c — Create `.decisions/` working directory for this run

Create `.decisions/visual-design/` at the project root. Per-step decision pages for this invocation go here.

**HTML mode output:**
- `01-tradition.html`
- `02-color.html`
- `03-type.html`
- `04-mood.html`
- `05-flourish.html`
- `index.html` (run summary)

**SVG mode output:**
- `01-tradition.html`
- `02-stroke.html`
- `03-color.html`
- `index.html` (run summary)

These let the user revisit their aesthetic decisions later.

---

## PHASE 2 — Artifact Analysis

Analysis branches on `MODE`.

### Phase 2 — HTML mode

Read the target HTML file. Extract:

1. **Structural selectors** — every class and ID used on elements. Note the semantic regions (`header`, `footer`, `nav`, `main`, `section`, etc.).
2. **Current style block** — the contents of `<style>...</style>` if present. Note what tokens it already defines (e.g., `:root { --ink: ... }`).
3. **Artifact type hints:**
   - Word count (rough estimate from stripped body text)
   - Heading structure (h1-h6 count)
   - Presence of tables, lists, code blocks, forms
   - Inline font stacks used
4. **Infer artifact type** — one of:
   - `brief` — word-heavy, 1-3 headings, long prose paragraphs
   - `landing` — short hero text, multiple sections, CTAs
   - `doc` — many headings, tables, code blocks
   - `proposal` — mixed — hero + sections + tables + CTAs
   - `dashboard` — low text, many small cards
   - `slide` / `one-pager` — small body, large display text

Keep this as a data structure you reference throughout the run. Example:

```json
{
  "target": ".decisions/strategy-brief.html",
  "mode": "html",
  "type": "brief",
  "wordCount": 2400,
  "headings": { "h1": 1, "h2": 6, "h3": 3 },
  "hasTables": true,
  "hasCode": false,
  "selectors": [".page", ".option", ".footer", ".research", "h1.title", ".deck", "..."]
}
```

### Phase 2 — SVG mode

Read the target SVG file. Extract:

1. **ViewBox and dimensions** — `viewBox`, `width`, `height` attributes on the root `<svg>` element.
2. **Shape inventory** — count of `<path>`, `<circle>`, `<rect>`, `<line>`, `<polygon>`, `<polyline>`, `<ellipse>`. Note which primitives dominate.
3. **Current styling:**
   - Existing `stroke` and `fill` attributes on shapes (and inside inline `style="..."`)
   - Existing `stroke-width` values
   - Any embedded `<style>` tag inside the SVG
   - Any `<defs>` (gradients, filters, patterns) that will need to be updated or preserved
4. **Color count** — how many distinct colors are used? (1 = single-color, 2-3 = duotone/limited, 4+ = multi-color illustration)
5. **Infer asset type** — one of:
   - `icon` — small viewBox (≤64), 1-2 colors, outline-style (no fill or single fill)
   - `logo` — small-medium viewBox, branded colors, mixed stroke/fill
   - `illustration` — larger viewBox, 4+ colors, complex shapes
   - `glyph` — path-only, single color, no stroke (text-like)

Example:

```json
{
  "target": "icons/download.svg",
  "mode": "svg",
  "type": "icon",
  "viewBox": "0 0 24 24",
  "shapes": { "path": 3, "circle": 0, "rect": 0 },
  "colorCount": 1,
  "currentStroke": "currentColor",
  "currentFill": "none",
  "currentStrokeWidth": "2"
}
```

**Warn the user** if `type === "illustration"` and they've picked a tradition with single-color rules — multi-color illustrations may lose detail when flattened to a tradition's palette.

---

## PHASE 3 — Tradition Selection (Decision 1)

This is the first and biggest decision. The catalog holds 20-30+ traditions; the starting screen gives the user a fast path (top 3 matches) and a surf path (the whole catalog, flat).

### Step 3a — Score the traditions for this artifact

For each tradition in the library (see AESTHETIC TRADITIONS LIBRARY below), compute a fit score from the artifact type:

| Artifact type | Tradition affinities (highest fit first) |
|---------------|------------------------------------------|
| `brief` (HTML) | Editorial Print, Warm Minimal, Academic, Newsprint, Japandi, Neo-Classical, Botanical Herbarium, Swiss Modern |
| `landing` (HTML) | Neo-Brutalist, Kinetic Modern, Glassmorphic, Playful Maximalist, Y2K Maximalist, Memphis Revival, Retro Futurism, Swiss Modern |
| `doc` (HTML) | Technical Documentary, Swiss Modern, Monochrome, Academic, Dashboard Operator, Newsprint |
| `proposal` (HTML) | Editorial Print, Neo-Classical, Swiss Modern, Warm Minimal, Luxury Serif, Midnight Marine, Art Deco |
| `dashboard` (HTML) | Swiss Modern, Technical Documentary, Dashboard Operator, Kinetic Modern, Monochrome, Neon Terminal |
| `slide` (HTML) | Editorial Print, Neo-Brutalist, Luxury Serif, Kinetic Modern, Art Deco, Bauhaus Grid, Zine |
| `icon` (SVG) | Swiss Modern, Monochrome, Neo-Brutalist, Technical Documentary, Neon Terminal, Cyberpunk Neon, Dashboard Operator, Bauhaus Grid |
| `logo` (SVG) | Swiss Modern, Neo-Brutalist, Art Deco, Luxury Serif, Bauhaus Grid, Retro Futurism, Monochrome |
| `illustration` (SVG) | Warm Handmade, Sketchbook, Botanical Herbarium, Editorial Print, Memphis Revival, Kraft Paper, Playful Maximalist |
| `glyph` (SVG) | Monochrome, Swiss Modern, Neo-Brutalist, Art Deco, Luxury Serif, Academic |

**SVG-incompatible traditions** (warn user if they pick one for an SVG):
- **Glassmorphic** — requires backdrop-filter + layered translucency, meaningless on single-shape icons
- **Playful Maximalist** — gradients-as-default need careful per-shape handling, best for illustrations only
- **Anti-Design** — clashing-font premise doesn't apply to vectors without text

If the user picks one of these in SVG mode, offer a gentle "are you sure? here's what will change" note rather than blocking. The tradition will still resolve (color palette at least).

Top 3 scores become the featured matches. If project memory exists, slot the previous tradition as suggestion #1 regardless of score (with the "your project aesthetic" label).

### Step 3b — Render the tradition decision page

Write `.decisions/visual-design/01-tradition.html` following the HTML TEMPLATE REFERENCE below. The page has two sections:

1. **Matched for your artifact** (3 large tiles, rendered with each tradition's real tokens — type-forward thumbnail style, see below)
2. **Browse all** (flat grid of remaining traditions, ~48px tiles)

Each tile renders the tradition's name using the tradition's **own headline font**, **primary text color**, and **base background**. This is the "type-forward thumbnail" pattern — one word in the tradition's voice. Readable at small sizes, conveys feel instantly.

Open the file with `open .decisions/visual-design/01-tradition.html` and wait for the user's response.

### Step 3c — Handle the user's pick

User's response shapes:
- Name: `Editorial`, `Editorial Print`, `warm minimal`, `Swiss`, `neo-brutal` → fuzzy-match the library (case-insensitive, partial prefix match).
- Shortcut: `top pick` / `first` / `best match` / `A` / `B` / `C` → resolves to the 1st, 2nd, or 3rd matched tile.
- `recommended` → resolves to the highest-scored tradition (usually match #1).
- `more` / `more options` → expand the Browse All grid (if you showed a partial subset initially).
- `surprise me` / `skill's pick` → pick the highest-scored tradition, move on.

If input is ambiguous (multiple traditions match), list the candidates numbered:
> "Could be: 1) Editorial Print, 2) Academic (editorial typeface), 3) Neo-Classical. Reply with a number or a more specific name."

Once locked: set `TRADITION` in state, proceed to Step 2 (Color).

---

## PHASE 4 — Remaining Steps

The flow branches on `MODE`:
- **HTML mode** → 4 remaining steps (Color, Type, Mood, Signature Flourish), all A/B/C/D
- **SVG mode** → 2 remaining steps (Stroke, Color), all A/B/C/D

---

## PHASE 4 (HTML mode) — Steps 2-5

All four of these steps use the **A/B/C/D pattern** (standard thinking-skill convention). Each step:
1. Generates a decision HTML page with 4 options
2. Opens it in the browser
3. Waits for a letter-based response

### Step 2 — Color

Given `TRADITION`, offer 4 color variants that stay within the tradition's ramp but shift emphasis:

- **Option A: Faithful** — the tradition's default ramp + accent as specified
- **Option B: Warmer** — shift accents toward warmer hues (wood, brick, amber)
- **Option C: Cooler** — shift accents toward cooler hues (navy, forest, slate)
- **Option D: Higher contrast** — pump the contrast between text and background one step

Render each option as a mini-frame with the tradition's structure (browser chrome, header, hero headline, CTA button) using the shifted palette. Write `.decisions/visual-design/02-color.html`. Open. Wait.

### Step 3 — Type

Given `TRADITION` + `COLOR`, offer 4 type treatments:

- **Option A: Tradition default** — the headline + body pair specified by the tradition (recommended)
- **Option B: Bigger headlines** — same fonts, one scale step larger on h1/h2
- **Option C: Alternate pair** — the tradition's alternate typeface suggestion (e.g., for Editorial, swap Fraunces display for Source Serif display)
- **Option D: Mono everything** — replace body with a monospace stack (good for technical feel)

Each preview shows a styled headline + two lines of body text at realistic sizes. Write, open, wait.

### Step 4 — Mood

Given prior choices, offer 4 mood combos (shape × shadow bundled):

- **Option A: Sharp + flat** — tight radius, no shadows
- **Option B: Sharp + elevated** — tight radius, strong shadows
- **Option C: Soft + flat** — generous radius, no shadows
- **Option D: Soft + elevated** — generous radius, soft shadows

Each preview shows a CTA button + a card + a divider to demonstrate the combo. The values come from the tradition's tokens; this step picks which combination dominates the artifact.

### Step 5 — Signature Flourish

Given prior choices, load the flourish candidates from the tradition's entry in the SIGNATURE FLOURISH LIBRARY.

- **Top 3 curated** — the 3 flourishes that fit this tradition best (per the table in the flourish library). Surface with the label "✦ fits [tradition] best."
- **Full library below** — all 8-10 flourishes available. Browse flat.

This is the novel step — the anti-generic move. Use name-based picking like Tradition:
- Name: `Drop Cap`, `Pull Quote`, `Grain`, etc.
- `A` / `B` / `C` hits the top 3 tiles
- `none` / `skip` is a valid choice (no flourish)

Each flourish preview renders the flourish rendered inside a small paragraph using the chosen tradition.

Write `.decisions/visual-design/05-flourish.html`. Open. Wait.

---

## PHASE 4 (SVG mode) — Steps 2-3

Two remaining steps after Tradition. Both use A/B/C/D.

### Step 2 — Stroke Weight

Pull the tradition's default stroke weight preference and offer 4 options calibrated around it. For a tradition whose rule specifies "1.5px stroke" (Swiss), the range might be 1 / 1.5 / 2 / 3. For Neo-Brutalist, 2 / 3 / 4 / 6.

General pattern:

- **Option A: Thin** — hairline, precise. Good for dense icon sets and reading-context icons.
- **Option B: Regular** — the tradition's default stroke width (recommended).
- **Option C: Bold** — stepped up one level, more presence.
- **Option D: Very Bold** — aggressive, chunky. Sets the icon apart. Default for Neo-Brutalist, Y2K, Anti-Design.

Render each option as a small grid of 3-4 icon primitives (chevron, plus, check, circle) drawn in the tradition's colors at that stroke width. User sees how each weight reads at typical icon size.

Also apply the tradition's stroke-linecap and stroke-linejoin preference:
- Most traditions → `round` for linecap + linejoin
- Swiss Modern, Bauhaus, Monochrome, Technical, Dashboard → `square` / `miter`
- Neo-Brutalist → `square` + thick miter join
- Neon Terminal, Cyberpunk → `square` with glow filter

Write `.decisions/visual-design/02-stroke.html`. Open. Wait.

### Step 3 — Color

Given `TRADITION` + `STROKE`, offer 4 color treatments:

- **Option A: Faithful palette** — apply the tradition's accent color as stroke/fill (e.g., Editorial's `#9a3412` brick, Cyberpunk's `#ff006e` neon).
- **Option B: Monochrome** — single color, typically the tradition's ink (e.g., Editorial's `#1f1611`, Swiss's `#0f172a`). Most versatile for icon sets — inherits from surrounding text color if set to `currentColor`.
- **Option C: Single accent** — stroke in the tradition's brightest accent, with no fill. Best for call-to-action icons.
- **Option D: Duotone** — two colors on different paths/shapes. Stroke in accent, fill in a secondary tone. Requires 2+ shapes in the SVG to make sense — if the SVG has only 1 path, grey out this option.

Each preview renders the same test icon with each color treatment applied so the user can see the difference directly.

**Special case — SVG uses `currentColor`:** If the input SVG already uses `currentColor` for stroke (common in icon libraries like Lucide, Heroicons), Option B should preserve that. Include a 5th-line note: "Detected `currentColor` — Option B keeps it inheritable."

Write `.decisions/visual-design/03-color.html`. Open. Wait.

---

## PHASE 5 — Rewrite & Output

Branches on `MODE`.

**HTML mode** — Steps 5a–5e below (existing flow).
**SVG mode** — Steps 5s-a through 5s-d (new, after the HTML section).

---

### Phase 5 (HTML mode)

Once all 5 decisions are locked, you have:

- `TRADITION` — tokens + aesthetic rules
- `COLOR` — palette shift applied to ramp
- `TYPE` — headline + body pair + scale
- `MOOD` — shape + shadow combo
- `FLOURISH` — the signature element to add

### Step 5a — Resolve final tokens

Combine everything into a single tokens object:

```json
{
  "tradition": "Editorial Print",
  "variant": {
    "color": "faithful",
    "type": "tradition-default",
    "mood": "sharp-flat",
    "flourish": "drop-cap"
  },
  "tokens": {
    "color": {
      "ramp": ["#fdf6e3", "#f5ecd0", "#e7d7b8", "...", "#1f1611"],
      "accent": "#9a3412",
      "ink": "#1f1611",
      "bg": "#fdf6e3"
    },
    "type": {
      "headline": { "family": "'Iowan Old Style', Georgia, serif", "weights": [400, 600, 700] },
      "body": { "family": "'Source Serif 4', Georgia, serif", "weights": [400, 600] },
      "mono": { "family": "'JetBrains Mono', ui-monospace, monospace" },
      "scale": [12, 14, 18, 22, 32, 52]
    },
    "spacing": [4, 8, 12, 16, 24, 32, 48, 72, 104],
    "radius": [2, 4, 6, 10],
    "shadow": { "l1": "...", "l2": "...", "l3": "...", "l4": "..." },
    "motion": { "ease": "cubic-bezier(.2,.8,.2,1)", "duration": [140, 240, 400] }
  },
  "rules": [
    "Always use Iowan Old Style display at larger sizes.",
    "Italics are expressive tools for kickers and captions.",
    "No gradients — solid warm paper + ink.",
    "..."
  ],
  "flourish": {
    "type": "drop-cap",
    "css": ".drop { float: left; font-family: var(--headline); font-size: 3.5em; line-height: 0.85; padding: 0.1em 0.1em 0 0; color: var(--accent); }",
    "htmlHook": "add <span class=\"drop\">[first letter]</span> to the first paragraph after each h1"
  }
}
```

### Step 5b — Generate the new `<style>` block

Using the artifact's selectors (from Phase 2) and the resolved tokens, compose a complete CSS stylesheet:

1. Add Google Fonts `<link>` if needed (check the tradition's type stack)
2. Add `:root { --... }` with the resolved tokens
3. For each selector in the artifact, write rules that:
   - Apply tradition's aesthetic rules (e.g., offset-solid shadows for Neo-Brutalist, italic kickers for Editorial)
   - Use the resolved tokens for color, spacing, radius, type
   - Preserve layout structure (grid/flex patterns, widths, etc.) — only aesthetic properties change
4. Append the flourish CSS
5. Responsive breakpoints mirror the original (preserve the artifact's mobile behavior)

### Step 5c — Write output files

1. Copy the original artifact HTML structure (body, semantics) into a new file named `<original-name-without-ext>.styled.html` in the same directory as the original.
2. Replace its `<style>...</style>` block with the freshly generated CSS.
3. If `FLOURISH.htmlHook` specifies DOM insertions (e.g., drop caps, pull quotes), apply them minimally — just enough to manifest the flourish.
4. Ensure the `<head>` has the needed Google Fonts `<link>` tags.

### Step 5d — Write tokens sidecar

Write the resolved tokens object (from Step 5a) to `.visual-design/tokens.json` at the project root. Also write `.visual-design/run.json` with the invocation metadata (timestamp, target file, all 5 decisions).

If `.gitignore` exists and doesn't include `.visual-design/`, append a line:
```
# visual-design skill state
.visual-design/
```

### Step 5e — Open + report

Open both files side by side (so user can compare):
```bash
open <original>
open <original-basename>.styled.html
```

Report:
> "Re-skinned! **[basename].styled.html** is open alongside the original.
>
> Applied **[Tradition]** with **[Flourish]**. Decisions saved to `.visual-design/tokens.json` — next run in this project will surface this aesthetic as suggestion #1.
>
> Reply:
> - `love it` → nothing to do, done
> - `redo` → rerun the flow from scratch
> - `change [tradition|color|type|mood|flourish]` → rerun just that step
> - `different flourish` → return to step 5
> - `more contrast` / `warmer` / etc → I'll interpret and adjust"

---

### Phase 5 (SVG mode)

Once all 3 decisions are locked, you have:

- `TRADITION` — aesthetic rules (stroke-linecap, linejoin, filter needs)
- `STROKE` — stroke-width value
- `COLOR` — resolved stroke + fill values (faithful / mono / accent / duotone)

### Step 5s-a — Resolve SVG attributes

Derive the exact attribute values to apply to each shape:

```json
{
  "mode": "svg",
  "tradition": "Editorial Print",
  "variant": { "stroke": "regular", "color": "monochrome" },
  "svg": {
    "strokeWidth": "1.5",
    "stroke": "#1f1611",
    "fill": "none",
    "strokeLinecap": "round",
    "strokeLinejoin": "round",
    "filter": null
  },
  "rootAttributes": {
    "width": "24",
    "height": "24",
    "viewBox": "0 0 24 24",
    "fill": "none",
    "stroke": "currentColor",
    "stroke-width": "1.5",
    "stroke-linecap": "round",
    "stroke-linejoin": "round"
  }
}
```

For traditions with a signature filter (Cyberpunk Neon, Neon Terminal), generate an SVG `<filter>` element to embed under `<defs>`. Example for Neon Terminal's phosphor glow:

```xml
<defs>
  <filter id="vd-glow" x="-50%" y="-50%" width="200%" height="200%">
    <feGaussianBlur stdDeviation="1.5" result="blur"/>
    <feMerge>
      <feMergeNode in="blur"/>
      <feMergeNode in="SourceGraphic"/>
    </feMerge>
  </filter>
</defs>
```

Then apply `filter="url(#vd-glow)"` to grouped shapes.

### Step 5s-b — Rewrite the SVG

The strategy: **hoist common attributes to the root `<svg>` element**, then **remove redundant per-shape attributes**. This keeps the output clean and makes the SVG easy to re-theme later.

1. **Set root attributes** on `<svg>` itself:
   - `stroke`, `fill`, `stroke-width`, `stroke-linecap`, `stroke-linejoin`
   - Keep existing `viewBox`, `width`, `height`
   - Preserve `xmlns`

2. **Strip old styling** from every shape (`<path>`, `<circle>`, etc.):
   - Remove `style="..."` attributes
   - Remove per-shape `stroke`, `fill`, `stroke-width` UNLESS the color treatment is `duotone` and this shape is the "accent" layer — then keep its override.

3. **Duotone handling**:
   - For multi-shape SVGs with duotone: the first half of shapes (or all "filled" shapes) get `fill="var(--accent)"` + `stroke="none"`; the second half get `stroke="var(--ink)"` + `fill="none"`. Tune per SVG inspection.
   - Single-path SVGs can't express duotone — fall back to accent.

4. **Filter injection** (if tradition requires one):
   - Add `<defs>` with the filter after `<svg>` opening
   - Add `filter="url(#vd-glow)"` to either the root `<g>` wrapper OR individual shapes

5. **Add a CSS comment header** inside `<svg>` as a text annotation for discoverability:
   ```xml
   <!-- /visual-design · Editorial Print · 1.5px · monochrome -->
   ```

6. **Preserve everything else** untouched — `<title>`, `<desc>`, `<metadata>`, comments, id attributes, aria attributes.

### Step 5s-c — Write output files

1. Write the rewritten SVG to `<original-basename>.styled.svg` in the same directory as the original.
2. Write tokens to `.visual-design/tokens.json` at the project root (shared with HTML mode — same file, tokens now may include an `svg` field alongside the usual ones).
3. Write run metadata to `.visual-design/run.json` (target, mode, decisions, timestamp).
4. Add `.visual-design/` to `.gitignore` if missing.

### Step 5s-d — Open + report

Open both files side by side (browsers render SVGs directly):
```bash
open <original>
open <original-basename>.styled.svg
```

Report:
> "Re-skinned! **[basename].styled.svg** is open alongside the original.
>
> Applied **[Tradition]** at **[Stroke]px** in **[Color treatment]**. Decisions saved to `.visual-design/tokens.json`.
>
> Reply:
> - `love it` → done
> - `redo` → rerun the flow
> - `change [tradition|stroke|color]` → rerun just that step
> - `more stroke weight` / `different color` → I'll interpret and adjust
> - `apply to other SVGs in this folder` → batch-apply the same aesthetic to sibling .svg files"

**Batch mode (bonus):** if the user invokes `/visual-design` again in the same project and another `.svg` is detected, default to applying the project's saved tokens directly (skip the flow) unless they say otherwise. This makes consistent icon sets trivial.

---

## RESPONSE PARSING — How Users Pick

The five steps don't all have 4 options, so input parsing varies per step:

### Steps 2, 3, 4 (Color, Type, Mood) — standard A/B/C/D
- Exactly 4 options each. Respond like any thinking skill.
- Accept: `Option A`, `A`, `the first one`, `Option B because [reason]`, `Option A but [modification]`, `more options`.

### Steps 1 and 5 (Tradition, Flourish) — name-based picking
- 10-30+ options, letters don't scale. Pick by name.
- **Primary:** `Editorial`, `Warm Minimal`, `Neo-Brutalist`, `Drop Cap`, `Rule Line`, etc.
- **Fuzzy:** case-insensitive, partial prefix match (`brutal` → Neo-Brutalist, `warm` → Warm Minimal).
- **Top-3 shortcuts:** `A` / `B` / `C` hit the top 3 tiles. Also: `top pick`, `first`, `best match`.
- **Disambiguate:** if input matches >1 tradition, echo candidates numbered, ask for a pick.
- **Recommended:** the highest-scored tradition earns a ★. User can say `recommended` to pick it.

### Catalog-wide commands (any step)
- `more` / `more options` — expand visible set.
- `surprise me` / `skill's pick` — take the highest-scored option, move on.
- `back` / `previous` — step back without rerunning.
- `skip` — at step 5, valid meaning "no flourish." At other steps, use recommended.

Match input against (a) letter regex `^([A-Za-z])\b` at steps 2-4, (b) shortcut keywords, (c) fuzzy name match at steps 1 and 5. Show numbered candidates when ambiguous.

---

## AESTHETIC TRADITIONS LIBRARY

30 starter traditions, grouped by feel. Each entry provides the tokens and aesthetic rules needed to render previews AND to generate the final rewrite.

**Feel groups** (used only for internal organization — all traditions render in the flat grid):
- **Structural:** Swiss Modern, Technical Documentary, Monochrome, Newsprint, Academic, Bauhaus Grid, Dashboard Operator
- **Warm:** Editorial Print, Warm Minimal, Warm Handmade, Kraft Paper, Japandi
- **Expressive:** Neo-Brutalist, Playful Maximalist, Y2K Maximalist, Memphis Revival, Anti-Design
- **Quiet:** Soft Premium, Neo-Classical, Botanical Herbarium, Midnight Marine
- **Raw:** Neon Terminal, Zine, Sketchbook
- **Kinetic / Premium:** Kinetic Modern, Glassmorphic, Luxury Serif, Art Deco, Retro Futurism, Cyberpunk Neon

Below are the 30 seed traditions. Add more over time by appending new entries in the same format.

---

### 1. Swiss Modern

**Feel:** Clean, confident, functional. Zero decoration. Information dense but never cramped. Typography does the heavy lifting.

**Color ramp:** `#fafafa` · `#f4f4f5` · `#e4e4e7` · `#d4d4d8` · `#a1a1aa` · `#71717a` · `#52525b` · `#3f3f46` · `#27272a` · `#0f172a`
**Accents:** primary `#6366f1` · pressed `#4f46e5` · success `#22c55e` · danger `#ef4444`

**Type:**
- Headline: `"Inter Tight"` 700, letter-spacing -0.02em
- Body: `Inter` 400, line-height 1.55
- Mono: `"JetBrains Mono"` 400
- Scale: 12 · 14 · 16 · 20 · 28 · 40
- Google Fonts: `Inter+Tight:wght@500;600;700&family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500`

**Spacing:** 4 · 8 · 12 · 16 · 24 · 32 · 48 · 64 · 96
**Radius:** 4 · 6 · 10 · 16
**Shadow:** L1 `0 1px 2px rgba(15,23,42,.06)` · L2 `0 2px 8px rgba(15,23,42,.08)` · L3 `0 8px 24px rgba(15,23,42,.10)` · L4 `0 16px 40px rgba(15,23,42,.12)`
**Motion:** `cubic-bezier(.16,1,.3,1)` · 120 / 200 / 320ms

**Rules:**
- No gradients. Solid color + whitespace.
- Shadows only on elevated surfaces.
- Headlines tight (-0.02 to -0.03em); body neutral.
- No rounded corners above 16px.
- Icons: 1.5px stroke, match body color.

**Flourish picks:** Kicker · Rule Line · Small-Caps Label

---

### 2. Editorial Print

**Feel:** Warm, literary, intentional. Print publication translated to screen. Rewards reading. Restrained color — when it shows up, it means something.

**Color ramp:** `#fdf6e3` · `#f5ecd0` · `#e7d7b8` · `#d4af7a` · `#b07a4a` · `#9a3412` · `#7c2d12` · `#44342a` · `#2a1f18` · `#1f1611`
**Accents:** brick `#9a3412` · success `#357266` · danger `#b91c1c`

**Type:**
- Headline: `Fraunces` 600 with `opsz` 96-144, letter-spacing -0.02em
- Body: `"Source Serif 4"` 400, line-height 1.7
- Mono: `"JetBrains Mono"` 400
- Scale: 12 · 14 · 18 · 22 · 32 · 52
- Google Fonts: `Fraunces:ital,opsz,wght@0,9..144,400;0,9..144,600;0,9..144,700;1,9..144,400&family=Source+Serif+4:ital,opsz,wght@0,8..60,400;0,8..60,600;1,8..60,400&family=JetBrains+Mono`

**Spacing:** 4 · 8 · 12 · 16 · 24 · 32 · 48 · 72 · 104
**Radius:** 2 · 4 · 6 · 10
**Shadow:** L1 `0 1px 2px rgba(31,22,17,.08)` · L2 `0 2px 6px rgba(31,22,17,.10)` · L3 `0 4px 12px rgba(31,22,17,.12)` · L4 `0 8px 20px rgba(31,22,17,.15)`
**Motion:** `cubic-bezier(.2,.8,.2,1)` · 140 / 240 / 400ms

**Rules:**
- Always use Fraunces `opsz` axis — bigger opsz for bigger sizes.
- Italics are expressive tools, especially for kickers and captions.
- No gradients. Solid warm paper + ink.
- Dividers hairline or dotted, never thick.
- Use pull-quotes with generous margins when space allows.
- Drop caps welcome on long-form.

**Flourish picks:** Drop Cap · Kicker · Pull Quote

---

### 3. Neo-Brutalist

**Feel:** Hard-edged, honest, direct. No softening. High-contrast, chunky, almost aggressive in its indifference to trend.

**Color ramp:** `#ffffff` · `#f5f5f5` · `#d4d4d4` · `#a3a3a3` · `#737373` · `#404040` · `#262626` · `#171717` · `#0a0a0a` · `#000000`
**Accents:** yellow `#facc15` · red `#ef4444` · blue `#2563eb`

**Type:**
- Headline: `"Archivo Black"` 400, letter-spacing -0.01em, UPPERCASE acceptable
- Body: `Inter` 500, line-height 1.5
- Mono: `"Space Mono"` 400
- Scale: 13 · 15 · 17 · 22 · 34 · 56
- Google Fonts: `Archivo+Black&family=Inter:wght@400;500;700&family=Space+Mono:wght@400;700`

**Spacing:** 4 · 8 · 12 · 16 · 24 · 32 · 40 · 56 · 80
**Radius:** 0 · 0 · 2 · 4 (mostly angular)
**Shadow:** offset-solid, not blurred. L1 `3px 3px 0 #000` · L2 `5px 5px 0 #000` · L3 `8px 8px 0 #000` · L4 `12px 12px 0 #000`
**Motion:** `linear` · 80 / 150 / 300ms (abrupt)

**Rules:**
- Solid offset drop shadows (never soft-blurred) are a signature.
- Uppercase headings welcome. Monospace kickers welcome.
- Borders are 2-3px and black.
- One accent color used aggressively (yellow or red) — restraint is the enemy.
- Zero gradients. Zero corners above 4px. Zero smooth easing.
- Underlines on active nav items — skeuomorphic web.

**Flourish picks:** Offset Box Shadow · Uppercase Kicker · Thick Rule

---

### 4. Warm Minimal

**Feel:** Muted earth tones, generous whitespace, serif revival. Calm, upscale, nothing loud.

**Color ramp:** `#fafaf9` · `#f5f5f4` · `#e7e5e4` · `#d6d3d1` · `#a8a29e` · `#78716c` · `#57534e` · `#44403c` · `#292524` · `#1c1917`
**Accents:** mint-gray `#5b8c7a` · caramel `#d4a373` · muted blue `#5b7b9a`

**Type:**
- Headline: `"Inter Tight"` 600, letter-spacing -0.015em
- Body: `Inter` 400, line-height 1.6
- Mono: `"JetBrains Mono"` 400
- Scale: 12 · 14 · 15 · 19 · 26 · 38
- Google Fonts: `Inter+Tight:wght@500;600&family=Inter:wght@400;500;600&family=JetBrains+Mono:wght@400`

**Spacing:** 4 · 8 · 12 · 20 · 28 · 40 · 56 · 80 · 112 (generous)
**Radius:** 8 · 12 · 18 · 28
**Shadow:** soft, large spread. L1 `0 1px 3px rgba(28,25,23,.04)` · L2 `0 4px 16px rgba(28,25,23,.05)` · L3 `0 12px 32px rgba(28,25,23,.06)` · L4 `0 24px 56px rgba(28,25,23,.08)`
**Motion:** `cubic-bezier(.4,0,.2,1)` · 200 / 350 / 500ms

**Rules:**
- Desaturated palette only.
- Generous whitespace — minimum 24px padding on containers.
- Soft shadows with large spread, low opacity.
- Off-black text (#1c1917), never pure black.
- One muted accent color per view.

**Flourish picks:** Kicker · Rule Line · Small-Caps Label

---

### 5. Technical Documentary

**Feel:** Dense, authoritative, information-first. Feels like serious reference material. Heavy on tables, lists, code.

**Color ramp:** `#f8fafc` · `#f1f5f9` · `#e2e8f0` · `#cbd5e1` · `#94a3b8` · `#64748b` · `#475569` · `#334155` · `#1e293b` · `#0f172a`
**Accents:** link `#0284c7` · success `#059669` · warning `#d97706` · danger `#dc2626`

**Type:**
- Headline: `Inter` 700, letter-spacing -0.015em
- Body: `Inter` 400, line-height 1.55
- Mono: `"IBM Plex Mono"` 400 — code is first-class
- Scale: 12 · 14 · 16 · 20 · 26 · 36
- Google Fonts: `Inter:wght@400;500;600;700&family=IBM+Plex+Mono:wght@400;500`

**Spacing:** 4 · 8 · 12 · 16 · 24 · 32 · 48 · 64 · 96
**Radius:** 2 · 4 · 6 · 8
**Shadow:** rarely used — prefer borders + spacing. L1 `0 1px 0 rgba(15,23,42,.05)` · L2 `0 1px 2px rgba(15,23,42,.06)` · L3 `0 2px 4px rgba(15,23,42,.08)` · L4 `0 4px 8px rgba(15,23,42,.10)`
**Motion:** `ease` · 80 / 140 / 220ms

**Rules:**
- Inline code, tables, and definition lists are native.
- Accent color used ONLY for links and semantic indicators.
- High information density is a feature, not a bug.
- Code blocks are UI, not afterthoughts.
- Tables have zebra stripes; headers are bolded not uppercased.

**Flourish picks:** Inline Code · Rule Line · Small-Caps Label

---

### 6. Monochrome

**Feel:** Pure black and white. Uncompromising. Type is everything.

**Color ramp:** `#ffffff` · `#fafafa` · `#e5e5e5` · `#d4d4d4` · `#a3a3a3` · `#737373` · `#404040` · `#262626` · `#171717` · `#000000`
**Accents:** none — mono stays mono. Optional single accent at user request.

**Type:**
- Headline: `"SF Mono"`/`"JetBrains Mono"` 500, letter-spacing 0
- Body: `"SF Mono"`/`"JetBrains Mono"` 400, line-height 1.55
- Mono: same
- Scale: 12 · 14 · 16 · 20 · 28 · 40
- Google Fonts: `JetBrains+Mono:wght@400;500;700`

**Spacing:** 4 · 8 · 12 · 16 · 24 · 32 · 48 · 64 · 96
**Radius:** 0 · 0 · 2 · 4
**Shadow:** L1 `0 1px 0 rgba(0,0,0,.1)` · L2 `0 2px 0 rgba(0,0,0,.15)` · L3 `0 4px 0 rgba(0,0,0,.2)` · L4 `0 8px 0 rgba(0,0,0,.25)`
**Motion:** `linear` · 100 / 200 / 400ms

**Rules:**
- Pure mono — no color accents.
- All text in a monospace family.
- Borders are 1px solid #000 or #e5e5e5.
- Headings differentiated by weight, not color.
- Use `_` or `-` as visual separators in labels.

**Flourish picks:** ASCII Divider · Small-Caps Label · Inline Code

---

### 7. Glassmorphic

**Feel:** Translucent, depth-rich, atmospheric. Depth via layered translucency.

**Color ramp:** `#fafafa` · `rgba(255,255,255,.6)` · `rgba(255,255,255,.4)` · `#e5e7eb` · `#9ca3af` · `#6b7280` · `#4b5563` · `#374151` · `#1f2937` · `#0f1419`
**Accents:** coral `#ff8a65` · sky `#60a5fa` · lilac `#c4b5fd` (all at .5 alpha)

**Type:**
- Headline: `Inter` 600, letter-spacing -0.015em
- Body: `Inter` 400, line-height 1.5
- Mono: `"SF Mono"` fallback to `Menlo` 400
- Scale: 12 · 14 · 16 · 20 · 28 · 44
- Google Fonts: `Inter:wght@400;500;600;700`

**Spacing:** 4 · 8 · 12 · 16 · 24 · 32 · 48 · 72 · 112
**Radius:** 10 · 16 · 24 · 36 (soft, bubble-like)
**Shadow:** L1 `0 2px 8px rgba(0,0,0,.04)` · L2 `0 4px 16px rgba(0,0,0,.06)` · L3 `0 8px 32px rgba(0,0,0,.08)` · L4 `0 16px 48px rgba(0,0,0,.10)` — paired with `backdrop-filter: blur(...)`
**Motion:** smooth-springy · 180 / 300 / 500ms

**Rules:**
- Surfaces use `backdrop-filter: blur(20-40px)` with translucent bg.
- Behind every glass panel: a colorful gradient or vibrant content.
- Borders are `rgba(255,255,255,.3)` — glass rim.
- Text solid colors — never translucent text.
- One glass layer per view — stacking destroys the effect.

**Flourish picks:** Gradient Backdrop · Glass Rim · Soft Glow

---

### 8. Neo-Classical

**Feel:** Serif-heavy, restrained, editorial-adjacent. Prestige-media feel. Warm neutrals, grand vertical rhythm.

**Color ramp:** `#faf9f7` · `#f2ece0` · `#e4d9c5` · `#c4b195` · `#96825c` · `#6b5a38` · `#4c3f24` · `#332a18` · `#1f1a10` · `#0f0c08`
**Accents:** jewel red `#8c1c13` · forest `#2d4a2a` · deep blue `#1e3a5f`

**Type:**
- Headline: `"Playfair Display"` 700, italic variants welcome
- Body: `Lora` 400, line-height 1.65
- Display alt: `Spectral` 400 for lead-ins
- Scale: 13 · 15 · 18 · 24 · 34 · 52
- Google Fonts: `Playfair+Display:ital,wght@0,400;0,700;1,400;1,700&family=Lora:ital,wght@0,400;0,500;1,400&family=Spectral:wght@400;500`

**Spacing:** 4 · 8 · 12 · 18 · 28 · 44 · 68 · 100 · 144 (grand)
**Radius:** 0 · 2 · 4 · 6 (nearly angular)
**Shadow:** barely used. L1 `0 1px 1px rgba(15,12,8,.04)` · L2 `0 2px 4px rgba(15,12,8,.06)` · L3 `0 4px 12px rgba(15,12,8,.08)` · L4 `0 10px 24px rgba(15,12,8,.10)`
**Motion:** slow · 200 / 400 / 600ms (deliberate)

**Rules:**
- Display italics on headlines are a signature.
- Small caps for navigation and labels.
- Horizontal rules (hairline, sometimes doubled) as dividers.
- Drop caps welcome for long-form.
- Generous vertical rhythm; section spacing is grand.
- Minimal color — serif typography carries voice.

**Flourish picks:** Drop Cap · Small-Caps Label · Doubled Rule

---

### 9. Warm Handmade

**Feel:** Crafted, personal, small-batch. Made by one person who cared. Slightly imperfect by design.

**Color ramp:** `#f7f4ed` · `#eee7d8` · `#dcc9a5` · `#c4a574` · `#9c7f4f` · `#6b5538` · `#4a3b28` · `#33281b` · `#211a12` · `#13100a`
**Accents:** berry `#7c2d12` · sage `#5f7c3e` · dusty blue `#4a6978`

**Type:**
- Headline: `Fraunces` 600 (opsz friendly)
- Body: `Spectral` 400, line-height 1.65
- Mono: `"Cascadia Mono"` 400
- Scale: 13 · 15 · 17 · 22 · 30 · 46
- Google Fonts: `Fraunces:opsz,wght@9..144,400;9..144,600;9..144,700&family=Spectral:wght@400;500;600`

**Spacing:** 4 · 8 · 12 · 18 · 26 · 38 · 56 · 84 · 120 (organic)
**Radius:** 4 · 8 · 14 · 22 (friendly, not bubbly)
**Shadow:** warm-tinted. L1 `0 1px 3px rgba(107,85,56,.08)` · L2 `0 3px 10px rgba(107,85,56,.10)` · L3 `0 8px 20px rgba(107,85,56,.12)` · L4 `0 14px 32px rgba(107,85,56,.14)`
**Motion:** `cubic-bezier(.4,0,.2,1)` · 160 / 280 / 440ms

**Rules:**
- Off-center, slightly asymmetric layouts welcome.
- Warm earth palette only — no cool colors.
- A single hand-drawn flourish per view (squiggle underline, arrow).
- Line-heights generous (1.6-1.7).
- Section dividers hairline or dotted, never bold.

**Flourish picks:** Grain Texture · Squiggle Underline · Pull Quote

---

### 10. Kinetic Modern

**Feel:** Motion-forward, vivid, alive. Crisp geometry with energy underneath. Feels recent, capable, deliberate.

**Color ramp:** `#fafbff` · `#f0f3ff` · `#d6dcff` · `#b0bcff` · `#7f92ff` · `#4f6bff` · `#3b4dcc` · `#29368f` · `#161d4a` · `#0a0e24`
**Accents:** lime `#a3e635` · coral `#fb7185` · cyan `#22d3ee`

**Type:**
- Headline: `"Space Grotesk"` 700, letter-spacing -0.02em
- Body: `Inter` 400, line-height 1.55
- Mono: `"JetBrains Mono"` 500
- Scale: 12 · 14 · 16 · 22 · 32 · 48
- Google Fonts: `Space+Grotesk:wght@500;600;700&family=Inter:wght@400;500;600&family=JetBrains+Mono:wght@400;500`

**Spacing:** 4 · 8 · 12 · 16 · 24 · 32 · 48 · 64 · 96
**Radius:** 6 · 10 · 14 · 20
**Shadow:** colored, subtle. L1 `0 2px 6px rgba(79,107,255,.10)` · L2 `0 4px 12px rgba(79,107,255,.14)` · L3 `0 8px 24px rgba(79,107,255,.20)` · L4 `0 16px 40px rgba(79,107,255,.28)`
**Motion:** `cubic-bezier(.5,1.5,.5,1)` (springy) · 140 / 240 / 380ms

**Rules:**
- Hover / focus states include visible motion.
- Accent colors as spot highlights — never main body color.
- Geometric shapes (circles, pills, diagonal stripes) as decorative accents.
- High contrast; dark background variant default-ready.

**Flourish picks:** Animated Underline · Geometric Accent · Colored Glow

---

### 11. Academic

**Feel:** Old-textbook, scholarly. Classic serif, disciplined layout, functional decoration.

**Color ramp:** `#fbfaf6` · `#f2ede0` · `#e2d9c2` · `#bfa981` · `#8a7547` · `#5a4a2a` · `#3f331c` · `#2a2114` · `#1a140c` · `#0d0a07`
**Accents:** ink blue `#1e3a5f` · burgundy `#7c1d2e` · forest `#2d4a2a`

**Type:**
- Headline: `Georgia`/`"Iowan Old Style"` serif 700
- Body: `Georgia` 400, line-height 1.7
- Mono: `"Courier New"` 400
- Scale: 12 · 14 · 16 · 20 · 26 · 38
- Google Fonts: (uses system serifs, optionally load `Lora`)

**Spacing:** 4 · 8 · 12 · 16 · 24 · 36 · 56 · 84 · 120
**Radius:** 0 · 2 · 4 · 4
**Shadow:** not used.
**Motion:** slow · 200 / 400 / 600ms

**Rules:**
- Traditional page layout — centered columns, wide margins.
- Small caps for section labels.
- Footnote-style indicators (superscript numbers).
- No gradients, no shadows, no rounded corners above 4px.
- Hairline rules between sections.

**Flourish picks:** Drop Cap · Footnote Marker · Small-Caps Label

---

### 12. Luxury Serif

**Feel:** Dark, upscale, gold-accented. Fashion-editorial. High-contrast, serif display.

**Color ramp:** `#1a1a1a` · `#232323` · `#2e2e2e` · `#3f3f3f` · `#5e5e5e` · `#8a8a8a` · `#b4b4b4` · `#d6d6d6` · `#ededed` · `#ffffff`
**Accents:** gold `#d4af37` · champagne `#e8c87a` · deep red `#8b1a1a`

**Type:**
- Headline: `"Bodoni Moda"`/`"Playfair Display"` 700, letter-spacing -0.01em, ALL CAPS welcome
- Body: `"Cormorant Garamond"` 400, line-height 1.65
- Scale: 12 · 14 · 16 · 22 · 34 · 60
- Google Fonts: `Bodoni+Moda:ital,wght@0,400;0,700;0,900;1,400;1,700&family=Cormorant+Garamond:ital,wght@0,300;0,400;0,500;1,400`

**Spacing:** 4 · 8 · 12 · 20 · 32 · 48 · 72 · 108 · 160 (grand)
**Radius:** 0 · 0 · 0 · 2 (angular)
**Shadow:** gold-tinted. L1 `0 1px 2px rgba(212,175,55,.08)` · L2 `0 2px 8px rgba(0,0,0,.25)` · L3 `0 4px 16px rgba(0,0,0,.35)` · L4 `0 8px 32px rgba(0,0,0,.45)`
**Motion:** slow elegant · 240 / 480 / 720ms

**Rules:**
- Dark background; gold accent used sparingly.
- All-caps display headlines with wide tracking.
- High-contrast serifs (Bodoni) for drama.
- Minimal chrome — let type + gold do the work.
- Hairline gold rules as dividers.

**Flourish picks:** Gold Hairline · Ornament Divider · All-Caps Kicker

---

### 13. Playful Maximalist

**Feel:** Expressive, energetic, friendly. More is more, but composed. Vivid color, rounded everything, bounce in motion.

**Color ramp:** `#fffbf5` · `#fef3c7` · `#fde68a` · `#fbbf24` · `#f97316` · `#ec4899` · `#8b5cf6` · `#6366f1` · `#1e1b4b` · `#0f0f1a`
**Accents:** bounce green `#10b981` · highlighter `#fde047`

**Type:**
- Headline: `Fraunces` 800, italic optional, opsz 144
- Body: `Inter` 500, line-height 1.55
- Display alt: `Caveat` 700 for hand-drawn emphasis
- Scale: 13 · 15 · 17 · 22 · 32 · 48
- Google Fonts: `Fraunces:ital,opsz,wght@0,9..144,700;0,9..144,800;1,9..144,800&family=Inter:wght@400;500;600;700&family=Caveat:wght@700`

**Spacing:** 4 · 8 · 12 · 16 · 24 · 32 · 48 · 64 · 96
**Radius:** 8 · 14 · 20 · 28 (round, friendly)
**Shadow:** accent-tinted. L1 `0 2px 4px rgba(139,92,246,.12)` · L2 `0 4px 12px rgba(139,92,246,.18)` · L3 `0 8px 24px rgba(236,72,153,.22)` · L4 `0 16px 40px rgba(139,92,246,.30)`
**Motion:** `cubic-bezier(.34,1.56,.64,1)` (bouncy overshoot) · 180 / 320 / 500ms

**Rules:**
- Gradients encouraged — pink-to-purple is the signature.
- Rounded corners aggressive — pills for buttons, 20px+ for cards.
- One handwritten accent (Caveat) per view, rotated slightly.
- Bounce easing default — everything overshoots.
- Shadows tinted with accent, never neutral gray.

**Flourish picks:** Hand-Drawn Squiggle · Pill Button · Tinted Shadow

---

### 14. Soft Premium

**Feel:** Calm, reassuring, upscale. Desaturated, low-contrast but confident. Nothing to prove.

**Color ramp:** `#fafaf9` · `#f5f5f4` · `#e7e5e4` · `#d6d3d1` · `#a8a29e` · `#78716c` · `#57534e` · `#44403c` · `#292524` · `#1c1917`
**Accents:** mint-gray `#5b8c7a` · caramel `#d4a373` · muted blue `#5b7b9a`

**Type:**
- Headline: `"Inter Tight"` 600, letter-spacing -0.015em
- Body: `Inter` 400, line-height 1.6
- Mono: `"JetBrains Mono"` 400
- Scale: 12 · 14 · 15 · 19 · 26 · 38 (compressed)
- Google Fonts: `Inter+Tight:wght@500;600&family=Inter:wght@400;500;600&family=JetBrains+Mono:wght@400`

**Spacing:** 4 · 8 · 12 · 20 · 28 · 40 · 56 · 80 · 112 (generous)
**Radius:** 8 · 12 · 18 · 28
**Shadow:** soft, large spread. L1 `0 1px 3px rgba(28,25,23,.04)` · L2 `0 4px 16px rgba(28,25,23,.05)` · L3 `0 12px 32px rgba(28,25,23,.06)` · L4 `0 24px 56px rgba(28,25,23,.08)`
**Motion:** `cubic-bezier(.4,0,.2,1)` · 200 / 350 / 500ms

**Rules:**
- Desaturated palette only — no bright hues.
- Generous whitespace, minimum 24px padding.
- Shadows always soft with large spread, low opacity.
- Off-black text (#1c1917), never pure black.
- One muted accent per view.
- Nothing competes for attention.

**Flourish picks:** Hairline Rule · Small-Caps Label · Soft Shadow

---

### 15. Newsprint

**Feel:** Newspaper condensed sans on cream paper. Black ink + red accent splash. Dense columns, thick rules.

**Color ramp:** `#faf7f0` · `#f2ece0` · `#e5dcc8` · `#c9beaa` · `#8a7f6a` · `#52483a` · `#332a1e` · `#1f1912` · `#14100a` · `#0a0805`
**Accents:** red splash `#c1272d` · ink black `#0f0f0f`

**Type:**
- Headline: `Oswald` 700 condensed, ALL CAPS welcome
- Body: `Merriweather` 400, line-height 1.55
- Mono: `"IBM Plex Mono"` 400
- Scale: 11 · 13 · 15 · 20 · 30 · 48
- Google Fonts: `Oswald:wght@400;500;600;700&family=Merriweather:ital,wght@0,300;0,400;0,700;1,400&family=IBM+Plex+Mono:wght@400`

**Spacing:** 4 · 6 · 10 · 14 · 20 · 28 · 40 · 56 · 80 (dense)
**Radius:** 0 · 2 · 4 · 4 (mostly angular)
**Shadow:** rarely. L1 `0 1px 0 rgba(0,0,0,.1)` · L2 `0 2px 4px rgba(0,0,0,.08)` · L3 `0 4px 8px rgba(0,0,0,.1)` · L4 `0 8px 16px rgba(0,0,0,.12)`
**Motion:** `ease` · 100 / 180 / 300ms

**Rules:**
- Condensed sans for headlines; serif body.
- Red reserved for kickers, dates, alerts.
- Thick black rules (3-4px) between sections.
- Multi-column body layout if space allows.
- Datelines and bylines in italics, small caps.

**Flourish picks:** Thick Rule · Red Kicker · Byline Italic

---

### 16. Y2K Maximalist

**Feel:** Chrome gradients, bubble text, holographic accents. Early 2000s web aesthetic revived and refined.

**Color ramp:** `#f5f3ff` · `#e9e5ff` · `#c4bbff` · `#918aff` · `#6b65ff` · `#4f4af0` · `#3d38c8` · `#2b2798` · `#1a186e` · `#0a094a`
**Accents:** magenta `#ec4899` · cyan `#22d3ee` · lime `#a3e635` · chrome silver gradient

**Type:**
- Headline: `Poppins` 800, rounded geometric
- Body: `Nunito` 500, line-height 1.55
- Mono: `"Space Mono"` 400
- Scale: 13 · 15 · 17 · 22 · 32 · 52
- Google Fonts: `Poppins:wght@400;600;700;800&family=Nunito:wght@400;500;700&family=Space+Mono:wght@400;700`

**Spacing:** 4 · 8 · 12 · 16 · 24 · 32 · 48 · 64 · 96
**Radius:** 16 · 24 · 32 · 999 (pills everywhere)
**Shadow:** colored + holographic. L1 `0 2px 8px rgba(139,92,246,.3)` · L2 `0 4px 16px rgba(236,72,153,.3)` · L3 `0 8px 32px rgba(34,211,238,.3)` · L4 `0 16px 48px rgba(163,230,53,.3)`
**Motion:** `cubic-bezier(.25,.1,.25,1)` · 200 / 320 / 500ms

**Rules:**
- Gradients everywhere — magenta-cyan, chrome silver.
- Bubble pill buttons (999px radius).
- Holographic accents on CTAs (multi-color gradient).
- Rounded geometric sans for everything.
- Subtle glossy overlays on cards (top-to-transparent white gradient).

**Flourish picks:** Chrome Gradient · Pill Button · Holographic Accent

---

### 17. Zine / Photocopied

**Feel:** Cut-and-paste, punk, xerox aesthetic. Deliberately crooked, noisy, hand-made.

**Color ramp:** `#fffefb` · `#f5f1e6` · `#d8cfbb` · `#b0a384` · `#7a6f55` · `#4a4131` · `#2e2817` · `#1a1508` · `#0f0b04` · `#000000`
**Accents:** toner black `#0a0a0a` · photocopy red `#c1272d`

**Type:**
- Headline: `Anton` condensed, UPPERCASE
- Body: `"Courier Prime"` 400, line-height 1.5
- Display alt: `"Permanent Marker"` for hand-scrawled
- Scale: 12 · 14 · 16 · 22 · 36 · 64
- Google Fonts: `Anton&family=Courier+Prime:ital,wght@0,400;0,700;1,400&family=Permanent+Marker`

**Spacing:** 3 · 6 · 10 · 14 · 20 · 28 · 40 · 56 · 80 (tight, uneven feel)
**Radius:** 0 · 0 · 2 · 2 (mostly none)
**Shadow:** solid offset + xerox smudge. L1 `2px 2px 0 #000` · L2 `4px 4px 0 #000` · L3 `6px 6px 0 #000` · L4 `0 0 8px rgba(0,0,0,.4)` (smudge)
**Motion:** `linear` · 80 / 150 / 300ms (abrupt)

**Rules:**
- Elements rotated 0.5-2deg for paste-up feel.
- Grain/noise overlay on backgrounds.
- Mix UPPERCASE condensed sans + typewriter + marker.
- Black borders 2-3px, occasional taped-down edges.
- High-contrast only — no gradients, no subtle colors.
- Photocopy smudge shadows.

**Flourish picks:** Paste-Up Rotation · Grain Overlay · Marker Scrawl

---

### 18. Neon Terminal

**Feel:** CRT terminal, phosphor glow, hacker aesthetic. Green on black. Monospace everything.

**Color ramp:** `#000000` · `#0a0a0a` · `#111111` · `#1a1a1a` · `#2a2a2a` · `#4a4a4a` · `#6a6a6a` · `#8a8a8a` · `#b0b0b0` · `#d0d0d0`
**Accents:** phosphor green `#00ff66` · amber `#ffb000` · cyan `#00ffff` · danger red `#ff0040`

**Type:**
- Headline: `"VT323"` 400, pixelated monospace
- Body: `"IBM Plex Mono"` 400, line-height 1.5
- Display alt: `"Major Mono Display"` for headers
- Scale: 13 · 15 · 17 · 20 · 28 · 40
- Google Fonts: `VT323&family=IBM+Plex+Mono:wght@400;500;700&family=Major+Mono+Display`

**Spacing:** 4 · 8 · 12 · 16 · 24 · 32 · 48 · 64 · 96
**Radius:** 0 · 0 · 0 · 2 (sharp)
**Shadow:** phosphor glow. L1 `0 0 4px rgba(0,255,102,.4)` · L2 `0 0 8px rgba(0,255,102,.5)` · L3 `0 0 16px rgba(0,255,102,.6)` · L4 `0 0 24px rgba(0,255,102,.7)`
**Motion:** `steps(4)` · 80 / 160 / 320ms (stepped)

**Rules:**
- Monospace everywhere; no proportional fonts.
- Green phosphor glow on interactive elements.
- Scanline overlay (`repeating-linear-gradient`, 2-4px).
- Terminal prompts (`$ `, `> `) as visual anchors.
- Blinking cursor on active inputs.
- Zero color outside the accent palette.

**Flourish picks:** Phosphor Glow · Scanline Overlay · Terminal Prompt

---

### 19. Japandi

**Feel:** Japanese minimalism + Scandinavian warmth. Generous negative space, thin strokes, mixed serif + sans.

**Color ramp:** `#faf8f3` · `#f0ebe0` · `#e0d8c5` · `#c4b8a0` · `#9a8e75` · `#6d634e` · `#483f2f` · `#2f2918` · `#1c1810` · `#0a0805`
**Accents:** tea green `#7a8f6c` · sumi ink `#1c1810` · persimmon `#c96e3e`

**Type:**
- Headline: `"Shippori Mincho"` 700 or `"Noto Serif"` 600
- Body: `Inter` 400, line-height 1.75
- Display alt: `"Noto Serif JP"` 400 italic
- Scale: 12 · 14 · 16 · 20 · 28 · 42
- Google Fonts: `Shippori+Mincho:wght@400;500;700&family=Noto+Serif:wght@400;500;600&family=Inter:wght@400;500;600`

**Spacing:** 4 · 8 · 16 · 24 · 40 · 64 · 96 · 144 · 200 (very generous)
**Radius:** 0 · 2 · 4 · 6 (restrained)
**Shadow:** barely. L1 `0 1px 1px rgba(0,0,0,.03)` · L2 `0 1px 3px rgba(0,0,0,.04)` · L3 `0 2px 6px rgba(0,0,0,.05)` · L4 `0 4px 12px rgba(0,0,0,.06)`
**Motion:** slow · 280 / 500 / 800ms (meditative)

**Rules:**
- Negative space is the primary design element.
- Thin 1px hairlines for dividers; no thick rules.
- Mix mincho serif for headings + geometric sans body.
- Single accent per view (tea green or persimmon).
- Asymmetric layouts with intentional empty quadrants.
- No gradients, no shadows deeper than L2.

**Flourish picks:** Negative Space · Hairline Rule · Vertical Text

---

### 20. Bauhaus Grid

**Feel:** Primary colors, geometric shapes, Futura-style sans. Strict grid. Form follows function.

**Color ramp:** `#ffffff` · `#f5f5f5` · `#e0e0e0` · `#b0b0b0` · `#707070` · `#404040` · `#202020` · `#101010` · `#080808` · `#000000`
**Accents:** red `#e63946` · blue `#1d4ed8` · yellow `#fbbf24`

**Type:**
- Headline: `"Josefin Sans"` 700, geometric
- Body: `Inter` 400, line-height 1.55
- Mono: `"Space Mono"` 400
- Scale: 12 · 14 · 16 · 20 · 28 · 44
- Google Fonts: `Josefin+Sans:wght@400;500;600;700&family=Inter:wght@400;500;600&family=Space+Mono`

**Spacing:** 4 · 8 · 16 · 24 · 32 · 48 · 64 · 96 · 128 (grid-aligned)
**Radius:** 0 · 0 · 0 · 0 (zero — angular only)
**Shadow:** not used.
**Motion:** `linear` · 100 / 200 / 400ms

**Rules:**
- Strict modular grid; elements align to 8px multiples.
- Primary colors only — no tints, no gradients.
- Geometric primitives (circles, squares, triangles) as decoration.
- Sans-serif everything, zero serifs.
- No rounded corners anywhere.
- Hairline black rules as dividers.

**Flourish picks:** Primary Color Block · Geometric Primitive · Hairline Rule

---

### 21. Memphis Revival

**Feel:** 80s geometric, zigzags, terrazzo. Playful clashing pastels + primaries + black outlines.

**Color ramp:** `#fff8f3` · `#ffd9e8` · `#ffb9d1` · `#ff7bac` · `#f43e8a` · `#d61e6a` · `#9b1650` · `#5f0e38` · `#330620` · `#1a0310`
**Accents:** teal `#14b8a6` · mustard `#eab308` · black `#000000`

**Type:**
- Headline: `Fraunces` 800, italic welcome
- Body: `"Space Grotesk"` 500, line-height 1.55
- Mono: `"Space Mono"` 400
- Scale: 14 · 16 · 18 · 24 · 36 · 56
- Google Fonts: `Fraunces:ital,opsz,wght@0,9..144,700;0,9..144,800;1,9..144,700&family=Space+Grotesk:wght@400;500;700&family=Space+Mono`

**Spacing:** 4 · 8 · 12 · 16 · 24 · 32 · 48 · 72 · 104
**Radius:** 4 · 12 · 24 · 999 (mix sharp + round)
**Shadow:** solid offset in contrast color. L1 `3px 3px 0 #000` · L2 `5px 5px 0 #14b8a6` · L3 `8px 8px 0 #eab308` · L4 `12px 12px 0 #000`
**Motion:** `cubic-bezier(.34,1.56,.64,1)` · 160 / 280 / 440ms

**Rules:**
- Clashing pastel + primary accents.
- Black outlines on cards and buttons (2-3px).
- Terrazzo / zigzag / squiggle decorative patterns.
- Mix rounded corners and sharp corners on same view.
- Solid colored drop shadows (not blurred).

**Flourish picks:** Terrazzo Pattern · Squiggle Underline · Colored Offset Shadow

---

### 22. Sketchbook

**Feel:** Hand-drawn, pencil, loose. Feels like a designer's notebook page.

**Color ramp:** `#fdfbf6` · `#f5f0e3` · `#e4dcc6` · `#c4b89a` · `#9a8b65` · `#6b5f42` · `#44392a` · `#2b2317` · `#18130c` · `#0c0906`
**Accents:** pencil graphite `#2b2317` · sepia `#8a5a2b` · dusty blue `#4a6978`

**Type:**
- Headline: `"Caveat"` 700 or `"Architects Daughter"` 400
- Body: `Nunito` 400, line-height 1.6
- Mono: `"Cascadia Mono"` 400
- Scale: 13 · 15 · 17 · 22 · 32 · 48
- Google Fonts: `Caveat:wght@400;600;700&family=Architects+Daughter&family=Nunito:wght@300;400;600`

**Spacing:** 4 · 8 · 12 · 18 · 26 · 40 · 56 · 84 · 112 (organic)
**Radius:** 4 · 8 · 14 · 22
**Shadow:** pencil-smudge soft. L1 `0 1px 2px rgba(43,35,23,.08)` · L2 `0 2px 6px rgba(43,35,23,.10)` · L3 `0 4px 12px rgba(43,35,23,.12)` · L4 `0 8px 20px rgba(43,35,23,.15)`
**Motion:** `cubic-bezier(.4,0,.2,1)` · 180 / 320 / 500ms

**Rules:**
- Hand-drawn fonts for headlines + display.
- Slight element rotation (0.5-1.5deg) for sketchy feel.
- Dashed or dotted borders (hand-drawn effect).
- Paper-texture backgrounds welcome.
- Sketchy arrows and squiggle dividers.

**Flourish picks:** Squiggle Underline · Hand-Drawn Arrow · Pencil Margin Note

---

### 23. Retro Futurism

**Feel:** 70s sci-fi, chrome edges, wide stretched type, burnt orange + teal. Bladerunner-meets-Apollo.

**Color ramp:** `#fef6e8` · `#fde4b8` · `#f9c878` · `#ec9a3f` · `#c9691e` · `#8a3a12` · `#5a2810` · `#3b1a0b` · `#231007` · `#120803`
**Accents:** teal `#0891b2` · mustard `#eab308` · cream `#fef6e8` · chrome silver gradient

**Type:**
- Headline: `"Righteous"` 400 or `"Bungee"` 400 wide display
- Body: `Inter` 500, line-height 1.5
- Mono: `"Space Mono"` 500
- Scale: 14 · 16 · 18 · 24 · 38 · 60
- Google Fonts: `Righteous&family=Bungee&family=Inter:wght@400;500;600;700&family=Space+Mono:wght@400;700`

**Spacing:** 4 · 8 · 12 · 16 · 24 · 32 · 48 · 72 · 112
**Radius:** 0 · 2 · 4 · 16 (mix sharp + occasional curve)
**Shadow:** chrome-tinted. L1 `0 1px 0 rgba(255,255,255,.3), 0 2px 4px rgba(0,0,0,.15)` · L2 same + blur 4 · L3 same + blur 8 · L4 same + blur 16
**Motion:** `cubic-bezier(.25,.46,.45,.94)` · 200 / 400 / 640ms

**Rules:**
- Wide-set type for headlines (letter-spacing 0.05-0.1em).
- Burnt orange + teal is the signature combo.
- Chrome gradient (silver) on buttons and CTA borders.
- Sunburst / starburst motifs as decorative accents.
- Dark-mode variant natural.

**Flourish picks:** Chrome Border · Sunburst Motif · Wide Letterspacing

---

### 24. Cyberpunk Neon

**Feel:** Dark + neon magenta/cyan/purple. Glitch effects, hard angles, deliberately oppressive.

**Color ramp:** `#030014` · `#0a0a1e` · `#14142b` · `#1e1e40` · `#2e2e60` · `#4a4a8a` · `#8080bf` · `#b3b3d9` · `#d6d6ed` · `#f5f5ff`
**Accents:** neon magenta `#ff006e` · cyan `#00f5ff` · purple `#8338ec` · yellow-warning `#ffbe0b`

**Type:**
- Headline: `"Rajdhani"` 700 or `"Orbitron"` 800 geometric
- Body: `Inter` 500, line-height 1.5
- Mono: `"Share Tech Mono"` 400
- Scale: 13 · 15 · 17 · 22 · 32 · 52
- Google Fonts: `Rajdhani:wght@400;500;600;700&family=Orbitron:wght@500;700;800;900&family=Inter:wght@400;500;600&family=Share+Tech+Mono`

**Spacing:** 4 · 8 · 12 · 16 · 24 · 32 · 48 · 64 · 96
**Radius:** 0 · 0 · 2 · 4 (sharp)
**Shadow:** neon glow. L1 `0 0 4px rgba(255,0,110,.5)` · L2 `0 0 12px rgba(255,0,110,.4), 0 0 24px rgba(131,56,236,.3)` · L3 `0 0 20px rgba(0,245,255,.5)` · L4 `0 0 40px rgba(0,245,255,.6)`
**Motion:** `cubic-bezier(.7,0,.3,1)` · 120 / 240 / 400ms (glitchy)

**Rules:**
- Dark background, neon foreground.
- Hard angles, zero curves on primary elements.
- Neon glow on interactive states.
- Occasional glitch offset (::after translated 2px).
- Uppercase headings with wide letter-spacing.
- Never use neutral color accents — always neon.

**Flourish picks:** Neon Glow · Glitch Offset · Chromatic Split

---

### 25. Art Deco

**Feel:** 1920s luxury, gold accents, fan shapes, vertical symmetry, Gatsby-era.

**Color ramp:** `#fef9f0` · `#f2e4c2` · `#d9c08a` · `#b89a5a` · `#8a6f3a` · `#5a4520` · `#3d2f16` · `#241d0e` · `#141008` · `#0a0704`
**Accents:** gold `#d4af37` · deep navy `#1e3a5f` · burgundy `#7c1d2e`

**Type:**
- Headline: `"Limelight"` 400 display or `"Poiret One"` 400 thin elegant
- Body: `"Cormorant Garamond"` 400, line-height 1.65
- Scale: 12 · 14 · 16 · 22 · 36 · 64
- Google Fonts: `Limelight&family=Poiret+One&family=Cormorant+Garamond:ital,wght@0,300;0,400;0,500;1,400`

**Spacing:** 4 · 8 · 14 · 22 · 36 · 56 · 88 · 136 · 200 (grand)
**Radius:** 0 · 0 · 0 · 2 (angular)
**Shadow:** rarely — use geometric ornament instead.
**Motion:** slow elegant · 240 / 480 / 720ms

**Rules:**
- Gold hairlines as dividers (1px).
- Vertical symmetry on hero sections.
- Fan-shape and sunburst motifs as ornament.
- Tall thin typography with extreme letter-spacing.
- Deep navy + cream + gold is the signature palette.
- No rounded corners, no gradients except gold foil effect.

**Flourish picks:** Gold Hairline · Fan Ornament · Vertical Symmetry

---

### 26. Botanical Herbarium

**Feel:** Muted greens + warm cream. Italic display, botanical illustration adjacency, herbarium-specimen feel.

**Color ramp:** `#fbf9f3` · `#f0ece0` · `#dfd8c2` · `#c2b89a` · `#9a9276` · `#6b6a4c` · `#48472f` · `#2a2b1b` · `#18180f` · `#0b0b06`
**Accents:** sage `#7a8b5c` · moss `#556b3a` · ink brown `#48472f` · burnt sienna `#9a3e1c`

**Type:**
- Headline: `"Cormorant Garamond"` 600 italic display
- Body: `"Source Serif 4"` 400, line-height 1.7
- Mono: `"IBM Plex Mono"` 400
- Scale: 13 · 15 · 17 · 22 · 32 · 48
- Google Fonts: `Cormorant+Garamond:ital,wght@0,400;0,500;0,600;1,400;1,500;1,600&family=Source+Serif+4:ital,opsz,wght@0,8..60,400;0,8..60,500;1,8..60,400`

**Spacing:** 4 · 8 · 12 · 18 · 28 · 44 · 68 · 100 · 144
**Radius:** 2 · 4 · 8 · 12
**Shadow:** soft earth-tinted. L1 `0 1px 2px rgba(72,71,47,.08)` · L2 `0 2px 6px rgba(72,71,47,.10)` · L3 `0 4px 12px rgba(72,71,47,.12)` · L4 `0 8px 20px rgba(72,71,47,.14)`
**Motion:** `cubic-bezier(.4,0,.2,1)` · 200 / 360 / 520ms

**Rules:**
- Italic display type for headlines.
- Sage + moss + ink brown palette.
- Hairline botanical-style borders.
- Latin-style small caps for labels.
- No bright accents — all earth tones.
- Line engravings / botanical illustrations welcome.

**Flourish picks:** Botanical Border · Italic Kicker · Small-Caps Label

---

### 27. Kraft Paper

**Feel:** Brown paper bag texture, rubber stamps, ink-on-cardboard. Hand-made small-batch feel.

**Color ramp:** `#f4ead6` · `#e8d9b3` · `#d4bf8e` · `#b89e68` · `#8e7445` · `#634e2a` · `#3d2e17` · `#241a0d` · `#140d06` · `#0a0603`
**Accents:** rubber-stamp red `#a3281c` · charcoal `#2a2218` · muted teal `#4a6c63`

**Type:**
- Headline: `"Courier Prime"` 700, stenciled feel
- Body: `"Anonymous Pro"` 400, line-height 1.55
- Display alt: `"Special Elite"` for hand-stamped
- Scale: 12 · 14 · 16 · 20 · 28 · 44
- Google Fonts: `Courier+Prime:ital,wght@0,400;0,700;1,400;1,700&family=Anonymous+Pro:ital,wght@0,400;0,700;1,400&family=Special+Elite`

**Spacing:** 4 · 8 · 12 · 16 · 24 · 32 · 48 · 72 · 104
**Radius:** 0 · 2 · 4 · 6
**Shadow:** ink-blot soft. L1 `0 1px 2px rgba(99,78,42,.15)` · L2 `0 2px 6px rgba(99,78,42,.18)` · L3 `0 4px 12px rgba(99,78,42,.22)` · L4 `0 8px 20px rgba(99,78,42,.26)`
**Motion:** `cubic-bezier(.4,0,.2,1)` · 180 / 320 / 500ms

**Rules:**
- Warm brown/kraft backgrounds on cards.
- Rubber-stamp red for emphasis (sparingly).
- Typewriter / stencil typography.
- Slight ink-bleed effect on borders (inset box-shadow).
- Hand-rough uneven edges (no perfect alignment).

**Flourish picks:** Rubber Stamp · Paper Grain · Hand-Rough Border

---

### 28. Dashboard Operator

**Feel:** Data-dense, telemetry-style, serious operator UI. Mono + compact sans, alert colors, table-first.

**Color ramp:** `#0b0f19` · `#111827` · `#1f2937` · `#374151` · `#4b5563` · `#6b7280` · `#9ca3af` · `#d1d5db` · `#e5e7eb` · `#f9fafb`
**Accents:** acid green `#10b981` (OK) · alert red `#ef4444` · amber warning `#f59e0b` · info cyan `#06b6d4`

**Type:**
- Headline: `"JetBrains Mono"` 700 or `Inter` 700
- Body: `Inter` 400, line-height 1.45 (compact)
- Mono: `"JetBrains Mono"` 500
- Scale: 11 · 13 · 14 · 16 · 20 · 28
- Google Fonts: `JetBrains+Mono:wght@400;500;700&family=Inter:wght@400;500;600;700`

**Spacing:** 2 · 4 · 6 · 8 · 12 · 16 · 24 · 32 · 48 (tight)
**Radius:** 2 · 4 · 6 · 6
**Shadow:** none (borders do the work).
**Motion:** `linear` · 60 / 120 / 200ms (snappy)

**Rules:**
- Dark background default; everything high contrast.
- Monospace numerals in all tables/metrics.
- Status colors used ONLY for semantic state (OK / warn / error / info).
- Tables with zebra stripes, 1px hairline borders.
- Small-caps labels for sections.
- Information density > breathing room.

**Flourish picks:** Status Dot · Mono Table · Small-Caps Section Label

---

### 29. Anti-Design

**Feel:** Intentionally ugly. Clashing fonts, broken grid, provocative. Subverts expectations. Used sparingly for effect.

**Color ramp:** `#ffffff` · `#f0f0f0` · `#cccccc` · `#888888` · `#555555` · `#000000` · `#ff00ff` · `#00ff00` · `#ffff00` · `#ff0000`
**Accents:** hot pink `#ff00aa` · lime `#aaff00` · electric yellow `#ffee00`

**Type:**
- Headline: `"Times New Roman"` serif (system, no import)
- Body: `"Comic Sans MS"` or `"Papyrus"` (intentionally)
- Display alt: `"Wingdings"` for chaos
- Scale: 11 · 14 · 19 · 23 · 38 · 70 (deliberately uneven)
- Google Fonts: none (system fonts intentionally)

**Spacing:** 3 · 7 · 11 · 17 · 23 · 31 · 43 · 59 · 89 (primes)
**Radius:** 0 · 30 · 3 · 40 (mix wildly)
**Shadow:** clashing. L1 `7px 7px 0 #ff00aa` · L2 `-4px 4px 0 #aaff00` · L3 `0 10px 0 #ffee00` · L4 `3px -3px 0 #000`
**Motion:** `steps(3)` · 90 / 220 / 370ms

**Rules:**
- Clashing fonts are the point — mix serif + Comic Sans + system.
- Broken grid — elements aligned to nothing.
- Clashing color combos (hot pink on lime).
- Mixed radii — some elements pill-round, others sharp.
- Intentionally misaligned shadows and borders.
- Use only for artifacts meant to provoke.

**Flourish picks:** Clashing Font Pair · Broken Grid · Mismatched Radii

---

### 30. Midnight Marine

**Feel:** Deep navy + aqua + pale gold + cream. Modern nautical. Calm, upscale, considered.

**Color ramp:** `#f5f3ed` · `#e4e0d0` · `#c9c3a8` · `#8a9ba5` · `#4a6b7a` · `#2a4a5f` · `#1a2f40` · `#0f1e2b` · `#08121a` · `#04090d`
**Accents:** aqua `#6ec1c9` · pale gold `#c9a867` · cream `#f5f3ed` · signal red `#c13f3f`

**Type:**
- Headline: `"Cormorant Garamond"` 600, italic welcome
- Body: `Inter` 400, line-height 1.65
- Mono: `"IBM Plex Mono"` 400
- Scale: 13 · 15 · 17 · 22 · 32 · 52
- Google Fonts: `Cormorant+Garamond:ital,wght@0,400;0,500;0,600;1,400;1,500&family=Inter:wght@400;500;600&family=IBM+Plex+Mono:wght@400`

**Spacing:** 4 · 8 · 12 · 18 · 28 · 44 · 68 · 100 · 144 (grand)
**Radius:** 0 · 2 · 4 · 6 (restrained)
**Shadow:** deep, quiet. L1 `0 1px 3px rgba(8,18,26,.15)` · L2 `0 2px 8px rgba(8,18,26,.20)` · L3 `0 4px 16px rgba(8,18,26,.25)` · L4 `0 8px 32px rgba(8,18,26,.30)`
**Motion:** `cubic-bezier(.4,0,.2,1)` · 240 / 420 / 640ms (deliberate)

**Rules:**
- Dark navy backgrounds with cream text preferred.
- Pale gold for accents only — never dominant.
- Hairline gold rules (1px) as dividers.
- Italic display type for editorial feel.
- Compass-rose or nautical ornament sparingly.
- Aqua used for interactive states only.

**Flourish picks:** Gold Hairline · Italic Display · Compass Ornament

---

## SIGNATURE FLOURISH LIBRARY

Ten flourish types. Each has: default CSS + per-tradition variant if needed + HTML insertion hook.

### Drop Cap
```css
.vd-dropcap { float: left; font-family: var(--headline); font-size: 3.5em; line-height: 0.85; padding: 0.12em 0.15em 0 0; color: var(--accent); }
```
**Insertion:** wrap first letter of the first `<p>` after each `<h1>` in `<span class="vd-dropcap">`.
**Fits:** Editorial, Neo-Classical, Warm Handmade, Academic, Luxury Serif.

### Kicker
```css
.vd-kicker { font-family: var(--body); font-size: .75em; font-weight: 700; letter-spacing: .15em; text-transform: uppercase; color: var(--accent); margin-bottom: .4em; }
/* Editorial variant: italic instead of uppercase */
.vd-kicker--editorial { font-style: italic; font-weight: 600; text-transform: none; }
```
**Insertion:** prepend `<span class="vd-kicker">[section number or label]</span>` to each h2.
**Fits:** everything. Default flourish.

### Rule Line
```css
.vd-rule { border: none; border-top: 1px solid var(--ink-3); margin: 1.5em 0; }
/* Doubled (Neo-Classical): */
.vd-rule--double { border-top: 1px solid var(--ink-3); border-bottom: 1px solid var(--ink-3); height: 3px; background: transparent; }
/* Thick (Neo-Brutalist): */
.vd-rule--thick { border-top: 3px solid var(--ink-900); }
```
**Insertion:** add `<hr class="vd-rule">` between major sections.
**Fits:** Swiss, Technical, Academic, Neo-Classical, Neo-Brutalist.

### Grain Texture
```css
.vd-grain { background-image: radial-gradient(rgba(0,0,0,.12) 0.5px, transparent 0.5px); background-size: 3px 3px; }
```
**Insertion:** apply `.vd-grain` to the body or main container as an overlay.
**Fits:** Warm Handmade, Editorial, Warm Minimal.

### Pull Quote
```css
.vd-pullquote { border-left: 4px solid var(--accent); padding: .5em 1em; font-family: var(--headline); font-style: italic; font-size: 1.2em; color: var(--ink-2); margin: 1.5em 0; }
```
**Insertion:** wrap selected long `<p>` in `<blockquote class="vd-pullquote">`.
**Fits:** Editorial, Neo-Classical, Warm Handmade, Luxury Serif.

### Ornament Divider
```css
.vd-ornament { text-align: center; color: var(--accent); letter-spacing: .3em; font-size: 1.2em; margin: 2em 0; }
.vd-ornament::before { content: "✦  ❋  ✦"; }
/* Luxury variant: */
.vd-ornament--luxury::before { content: "◆  ◆  ◆"; color: var(--accent-gold); }
```
**Insertion:** replace `<hr>` or insert between major sections.
**Fits:** Warm Handmade, Luxury Serif, Neo-Classical, Editorial.

### Small-Caps Label
```css
.vd-smcp { font-variant: small-caps; letter-spacing: .1em; font-weight: 700; color: var(--accent); }
```
**Insertion:** wrap nav items, metadata, bylines in `<span class="vd-smcp">`.
**Fits:** Swiss, Technical, Academic, Neo-Classical, Luxury Serif, Warm Minimal.

### Offset Box Shadow
```css
.vd-offset { box-shadow: 5px 5px 0 var(--ink-900); border: 2px solid var(--ink-900); }
```
**Insertion:** apply to cards, buttons, callout blocks.
**Fits:** Neo-Brutalist exclusively — signature of that tradition.

### ASCII Divider
```css
.vd-ascii { font-family: var(--mono); color: var(--ink-3); white-space: pre; text-align: center; margin: 1.5em 0; }
.vd-ascii::before { content: "- - - - - - - - - -"; }
```
**Insertion:** between sections, replacing `<hr>`.
**Fits:** Monochrome, Technical, Neon Terminal.

### Inline Code Accent
```css
code { font-family: var(--mono); background: var(--ramp-100); padding: .15em .4em; border-radius: 3px; font-size: .9em; color: var(--accent); }
```
**Insertion:** style any existing `<code>` tags — no DOM insertion needed.
**Fits:** Technical, Monochrome, Neo-Brutalist.

---

## HTML DECISION PAGE TEMPLATE

Each step generates a decision HTML file. Use a consistent shell with per-step variations.

### Shell skeleton (applies to all 5 step pages)

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Visual Design · Step [N] · [StepName]</title>
<style>
  :root {
    --page-bg: #faf8f4;
    --page-ink: #1a1714;
    --page-ink-2: #4a433b;
    --page-ink-3: #7a7066;
    --page-rule: #d8cfbf;
    --page-accent: #9a3412;
    --page-accent-soft: #fef3e8;
  }
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body { font-family: -apple-system, BlinkMacSystemFont, "Inter", sans-serif; background: var(--page-bg); color: var(--page-ink); padding: 48px 32px; line-height: 1.6; }
  .page { max-width: 1100px; margin: 0 auto; }
  .eyebrow { font-family: ui-monospace, "SF Mono", monospace; font-size: 11px; letter-spacing: 0.18em; text-transform: uppercase; color: var(--page-accent); margin-bottom: 20px; }
  h1 { font-family: "Iowan Old Style", Palatino, Georgia, serif; font-size: 44px; font-weight: 400; letter-spacing: -0.015em; margin-bottom: 16px; }
  .deck { font-family: "Iowan Old Style", Palatino, Georgia, serif; font-style: italic; font-size: 18px; color: var(--page-ink-2); max-width: 720px; margin-bottom: 40px; }
  /* Step-specific styles follow here */
</style>
</head>
<body>
<div class="page">
  <div class="eyebrow">Step [N] of 5 · [StepName]</div>
  <h1>[Decision question]</h1>
  <p class="deck">[Plain-English framing — 1 sentence]</p>
  [STEP-SPECIFIC CONTENT]
  <div class="footer">[instruction on how to respond]</div>
</div>
</body>
</html>
```

### Step 1 (Tradition) — structure

- Section A: "Matched for your artifact" — 3 large tiles (160px × 100px each), horizontal row. Each renders the tradition name in the tradition's own type + color + bg.
- Section B: "Browse all" — flat grid, 5 columns × N rows of small tiles (56-64px). Each tile = name in the tradition's type-forward thumbnail style.
- Footer: "Reply with a tradition name (e.g., `Editorial`) or `A`/`B`/`C` for one of the top 3 matches."

### Step 2/3/4 (Color/Type/Mood) — structure

- 4 option cards, 2×2 grid. Each card has:
  - Letter badge (A/B/C/D)
  - Short title ("Faithful", "Warmer", "Higher contrast", etc.)
  - Visual preview rendered with the variant's tokens
  - 1-line description
- Recommended badge on one card (usually A or the tradition-default).
- Footer: "Reply with `Option A`, `A`, or `Option A but [modification]`."

### Step 5 (Flourish) — structure

- Section A: "✦ Fits [Tradition] best" — 3 curated flourish tiles. Each tile renders the flourish *applied to the tradition*, with the flourish name labeled.
- Section B: "Full library" — all ~10 flourishes as smaller tiles, flat grid.
- Include a "None" tile at the end of both sections for users who want no flourish.
- Footer: "Reply with a flourish name (e.g., `Drop Cap`) or `none`."

### Run summary page (`index.html`)

Mirror the decision-hub style used elsewhere in the decision-kit (serif display, mono kickers, warm paper bg). Shows:
- The 5 locked choices
- The final resolved aesthetic summary
- Links to each decision page
- Links to the original + styled artifact

---

## EDGE CASES

**Target file has no `<style>` block:** Inject one inside `<head>` with the full resolved stylesheet. All selectors still match.

**Target file uses external CSS (`<link rel="stylesheet">`):** Convert to inline — fetch the external CSS, rewrite selectors, inline into the `.styled.html`. If external CSS is unreachable (remote, 404), tell the user and skip the file.

**Target file is HTML with embedded `<script>`:** Preserve all scripts. The skill only touches styles and minimal DOM for flourish hooks.

**Target file is already a `.styled.html` / `.styled.svg`:** Ask if the user wants to re-skin it (changing aesthetic) or start over (forgetting the prior aesthetic). If re-skin: use the existing styled file as the source, write a new `<name>.styled.*` overwriting prior styled version, update tokens.json.

**Target file is a PNG/JPG raster:** The skill can't re-style pixels. Tell the user: "I can't restyle raster images — this skill rewrites vector/style. If this is an icon, see if you have the source SVG. If it's a photo, it's out of scope."

**SVG is a multi-color illustration (4+ colors):** Warn the user before Phase 4 starts: "This looks like a multi-color illustration. If you pick a tradition with a single-color rule (most), colors will flatten. Traditions built for illustrations: Warm Handmade, Sketchbook, Botanical Herbarium, Editorial Print. Continue anyway?"

**SVG has `<defs>` with gradients/patterns:** Preserve them. The rewrite only touches stroke/fill on rendered shapes, not referenced `url(#foo)` paint servers — unless the chosen tradition explicitly replaces them (e.g., Y2K Maximalist overriding gradients with its own chrome gradient).

**SVG uses `currentColor`:** Honor it. Monochrome color treatment (Option B in SVG Step 3) should preserve `currentColor` so the icon inherits from its context. Note this on the decision page.

**SVG has only one path:** Grey out the Duotone color option (D) since it needs 2+ shapes.

**Tradition not found (typo):** Show 3 closest matches numbered, ask which.

**User says `redo` after the output:** Delete the `.styled.*` file, re-run from Phase 1 with the same target.

**User says `change [step]`:** Jump to that step's decision page, re-run from there. Downstream choices stay unless they conflict.

**No artifact type detected confidently:** Use `brief` (HTML) or `icon` (SVG) as the fallback for type inference.

---

## IMPORTANT REMINDERS

1. **Never skip a decision step.** The user invoked `/visual-design` because they want to *think through* the aesthetic. Fast ≠ skipped. Each step gets its own HTML page with options.
2. **Always open the file after writing.** `open .decisions/visual-design/0N-<step>.html`. User needs to see what you've generated.
3. **Wait for the user at every step.** Don't proceed until they've picked.
4. **Preserve the original.** Never overwrite the target. Always write `.styled.html` alongside.
5. **Plain English.** Option descriptions talk about feel and audience, not tokens. Save the JSON for the tokens file.
6. **Tokens.json is sacred.** It's what makes the skill compound across artifacts. Write it every run. Respect existing ones as suggestion #1.
7. **Flourish is the hero step.** Step 5 is what separates generic from characterful. Don't treat it as a throwaway.
8. **Fuzzy match is friendly.** Accept typos, partial names, abbreviations (`brutal` → Neo-Brutalist). If truly ambiguous, show numbered candidates.
9. **Respect existing aesthetic.** If the target artifact already uses a well-known tradition (detect by font family hints), suggest that tradition as match #1.
10. **Add to `.gitignore`.** First time `.visual-design/` is created in a project, append a line to `.gitignore` (create if missing).
