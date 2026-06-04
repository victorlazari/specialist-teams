---
name: pretty-slides
description: Create stunning, cinematic scrollytelling landing pages from scratch or from existing content. Use when the user wants to build a product page, landing page, marketing page, or any single-page web experience with premium animations and scroll-driven storytelling. Helps non-designers discover their aesthetic through visual exploration.
---

# Pretty Slides

Create zero-dependency, cinematic scrollytelling landing pages that run entirely in the browser.

## Core Principles

1. **Zero Dependencies** — Single HTML files with inline CSS/JS. No npm, no build tools.
2. **Show, Don't Tell** — Generate visual previews, not abstract choices. People discover what they want by seeing it.
3. **Distinctive Design** — No generic "AI slop." Every page must feel custom-crafted for its brand.
4. **Cinematic Storytelling** — Every page tells a story through scroll. Sections flow like scenes in a film.
5. **Performance First** — Lazy loading, content-visibility, throttled handlers. Premium feel with fast paint.

## Design Aesthetics

You tend to converge toward generic, "on distribution" outputs. In landing page design, this creates what users call the "AI slop" aesthetic. Avoid this: make creative, distinctive pages that surprise and delight.

Focus on:

- Typography: Choose fonts that are beautiful, unique, and interesting. Avoid generic fonts like Arial and Inter; opt instead for distinctive choices that elevate the page's character.
- Color & Theme: Commit to a cohesive aesthetic. Use CSS variables for consistency. Dominant colors with sharp accents outperform timid, evenly-distributed palettes. Draw from cinematic palettes and premium brand references.
- Motion: Use scroll-driven animations for narrative flow. Prioritize CSS-only solutions. Focus on high-impact moments: the hero entrance, a key product reveal, a dramatic stat counter. One well-orchestrated scrollytelling sequence creates more impact than scattered micro-interactions.
- Backgrounds: Create atmosphere and depth. Layer CSS gradient meshes, use grain textures, add subtle grid patterns. Dark themes with selective color highlights make product imagery pop.
- Spacing: Generous padding and whitespace signal premium quality. Don't cram content — let sections breathe.

Avoid generic AI-generated aesthetics:

- Overused font families (Inter, Roboto, Arial, system fonts)
- Cliched color schemes (particularly purple gradients on white backgrounds)
- Predictable layouts where every section looks the same
- Cookie-cutter hero sections with stock photo backgrounds
- Generic card grids without visual hierarchy

Interpret creatively and make unexpected choices. Vary between dark cinematic and light editorial, between serif and sans-serif, between geometric and organic. You still tend to converge on common choices — avoid this by thinking outside the box!

## Section Density Rules

These constraints apply to EVERY section:

| Section Type | Maximum Content |
| --- | --- |
| Hero section | 1 headline + 1 subline + 1 CTA button + optional background media |
| Feature showcase | 1 heading + 3-6 feature cards (icon + title + short description each) |
| Spec grid | 1 heading + up to 8 spec items in 2-4 column grid |
| Scrollytelling sequence | 1 sticky media element + 3-5 text panels triggered on scroll |
| Testimonial | 1 heading + 1-3 quotes with attribution |
| Comparison table | 1 heading + up to 4 columns, 8 rows |
| CTA section | 1 heading + 1 subline + 1-2 buttons |
| Image gallery | 1 heading + 3-6 images in grid or carousel |
| Stats/numbers | 1 heading + 3-4 animated counter stats |

**Content exceeds limits? Split into multiple sections. Never cram — let content breathe.**

---

## Phase 0: Detect Mode

Determine what the user wants:

- **Mode A: New Landing Page** — Create from scratch. Go to Phase 1.
- **Mode B: Content Conversion** — Convert existing markdown, docs, or brief into a landing page. Go to Phase 1 with content pre-filled.
- **Mode C: Enhancement** — Improve an existing HTML landing page. Read it, understand it, enhance. **Follow Mode C modification rules below.**

### Mode C: Modification Rules

When enhancing existing landing pages:

1. **Before adding content:** Count existing elements in the section, check against density limits
2. **Adding images:** Must lazy-load with `data-src`. If section already has max content, split into new section
3. **Adding text:** Check density limits per section type. Exceeds limits? Create a new section
4. **After ANY modification:** Verify new elements use `clamp()`, images lazy-load, sections have `content-visibility: auto`, performance isn't degraded
5. **Proactively reorganize:** If modifications will cause clutter, automatically split content and inform the user

---

## Phase 1: Content Discovery (New Landing Pages)

**Ask ALL questions in a single AskUserQuestion call** so the user fills everything out at once:

**Question 1 — Purpose** (header: "Purpose"):
What is this page for? Options: Product launch / SaaS marketing / Portfolio-showcase / Event-conference

**Question 2 — Length** (header: "Sections"):
How many sections? Options: Compact 4-6 / Standard 8-12 / Epic 12+

**Question 3 — Content** (header: "Content"):
Do you have content ready? Options: All content ready / Rough notes-brief / Topic only

**Question 4 — Media** (header: "Media"):
What media do you have? Options: Product images / Screenshots / Videos / No media (CSS-only visuals)

If user has content or media, ask them to share it.

### Step 1.2: Media Evaluation (if media provided)

If user selected "No media" → skip to Phase 2.

If user provides images or video:

1. **Scan** — List all media files (.png, .jpg, .svg, .webp, .mp4, etc.)
2. **View each image** — Use the Read tool (Claude is multimodal)
3. **Evaluate** — For each: what it shows, USABLE or NOT USABLE (with reason), what concept it represents, dominant colors, suggested section placement
4. **Co-design the structure** — Media informs section structure alongside text. Design around both from the start (e.g., 3 product shots → 3 feature sections, 1 hero image → full-bleed hero background, 1 demo video → scrollytelling sequence)
5. **Confirm via AskUserQuestion** (header: "Structure"): "Does this section outline and media placement look right?" Options: Looks good / Adjust media / Adjust structure

**Logo in previews:** If a usable logo was identified, embed it into each style preview in Phase 2.

---

## Phase 2: Style Discovery

**This is the "show, don't tell" phase.** Most people can't articulate design preferences in words.

### Step 2.0: Style Path

Ask how they want to choose (header: "Style"):

- "Show me options" (recommended) — Generate 3 previews based on mood
- "I know what I want" — Pick from preset list directly

**If direct selection:** Show preset picker and skip to Phase 3. Available presets are defined in [STYLE_PRESETS.md](STYLE_PRESETS.md).

### Step 2.1: Mood Selection (Guided Discovery)

Ask (header: "Vibe", multiSelect: true, max 2):
What feeling should visitors have? Options:

- Premium / Luxurious — Cinematic, dark, high-end
- Bold / Energetic — Vibrant, dynamic, attention-grabbing
- Clean / Professional — Minimal, trustworthy, focused
- Warm / Inviting — Friendly, approachable, human

### Step 2.2: Generate 3 Style Previews

Based on mood, generate 3 distinct HTML previews showing a hero section + one feature section. Read [STYLE_PRESETS.md](STYLE_PRESETS.md) for available presets and their specifications.

| Mood | Suggested Presets |
| --- | --- |
| Premium / Luxurious | Midnight Cinema, Obsidian Product, Dark Editorial |
| Bold / Energetic | Electric Showcase, Neon Product, Split Contrast |
| Clean / Professional | Swiss Landing, Paper Craft, Soft Studio |
| Warm / Inviting | Warm Editorial, Sunset Gradient, Natural Texture |

Save previews to `.claude-design/page-previews/` (style-a.html, style-b.html, style-c.html). Each should be self-contained, ~100-150 lines, showing one animated hero + feature section.

Open each preview automatically for the user.

### Step 2.3: User Picks

Ask (header: "Style"):
Which style preview do you prefer? Options: Style A: [Name] / Style B: [Name] / Style C: [Name] / Mix elements

If "Mix elements", ask for specifics.

---

## Phase 3: Generate Landing Page

Generate the full landing page using content from Phase 1 (text + media) and style from Phase 2.

If media was provided, the section structure already incorporates it from Step 1.2. If not, CSS-generated visuals (gradient meshes, geometric shapes, patterns, grain textures) provide atmosphere — this is a fully supported first-class path.

**Before generating, read these supporting files:**

- [html-template.md](html-template.md) — HTML architecture, JS features, section type patterns
- [viewport-base.css](viewport-base.css) — Mandatory CSS (include in full)
- [scrollytelling-patterns.md](scrollytelling-patterns.md) — Scroll-driven animation reference

**Key requirements:**

- Single self-contained HTML file, all CSS/JS inline
- Include the FULL contents of viewport-base.css in the `<style>` block
- Use fonts from Fontshare or Google Fonts — never system fonts
- Sticky navigation with anchor links to each section
- Scroll progress indicator at the top
- Intersection Observer for scroll-triggered `.reveal` animations
- Lazy loading for all images and videos (`data-src` pattern)
- `content-visibility: auto` on all non-hero sections
- Smooth scroll behavior for anchor links
- `prefers-reduced-motion` support (in viewport-base.css)
- Semantic HTML: `<section>`, `<nav>`, `<header>`, `<footer>`, `<main>`
- Every section needs a clear `/* === SECTION NAME === */` comment block
- Add detailed comments explaining each section and how to modify it

**Scrollytelling sequences (if applicable):**

- Use sticky positioning with scroll-progress tracking
- Map scroll offset to CSS custom property `--scroll-progress`
- Activate/deactivate text panels based on scroll thresholds
- Calculate scroll track height dynamically
- Graceful fallback for `prefers-reduced-motion`

---

## Phase 4: Delivery

1. **Clean up** — Delete `.claude-design/page-previews/` if it exists
2. **Open** — Use `open [filename].html` to launch in browser
3. **Summarize** — Tell the user:
   - File location, style name, section count
   - Navigation: Scroll naturally, click nav links, progress bar shows position
   - How to customize: `:root` CSS variables for colors, font link for typography, section order
   - Responsive: works on mobile, tablet, desktop
   - Performance: lazy loading, content-visibility, reduced-motion support

---

## Phase 5: Share & Export (Optional)

After delivery, **ask the user:** _"Would you like to share this page? I can deploy it to a live URL (works on any device including phones) or export it as a PDF."_

Options:

- **Deploy to URL** — Shareable link that works on any device
- **Export to PDF** — Static snapshot for email, Slack, print
- **Both**
- **No thanks**

If the user declines, stop here. If they choose one or both, proceed below.

### 5A: Deploy to a Live URL (Vercel)

This deploys the page to Vercel. The link works on any device and stays live until taken down.

**If the user has never deployed before, guide them step by step:**

1. **Check if Vercel CLI is installed** — Run `npx vercel --version`. If not found, install Node.js first.
2. **Check if user is logged in** — Run `npx vercel whoami`. If not, walk them through signup at https://vercel.com/signup and `vercel login`.
3. **Deploy** — Run `bash scripts/deploy.sh <path-to-page>`. Accepts a folder (with index.html) or a single HTML file.
4. **Share the URL** — Tell the user the live URL, that it works on any device, and how to take it down later.

**Deployment gotchas:**

- Local images/videos must travel with the HTML. The deploy script auto-detects `src="..."` references.
- Prefer folder deployments when the page has many assets.
- Redeploying updates the same URL.

### 5B: Export to PDF

Captures each full-viewport section as a screenshot and combines them into a PDF.

1. **Run** `bash scripts/export-pdf.sh <path-to-html> [output.pdf]`
2. **Note:** Animations and scroll interactions are not preserved — the PDF is a static snapshot. Mention this to the user.
3. **If Playwright fails:** Run `npx playwright install chromium`
4. **Deliver the PDF** — auto-opens. Tell the user the file location, size, and that animations are replaced by their final visual state.

---

## Supporting Files

| File | Purpose | When to Read |
| --- | --- | --- |
| [STYLE_PRESETS.md](STYLE_PRESETS.md) | 12 curated visual presets for landing pages | Phase 2 (style selection) |
| [viewport-base.css](viewport-base.css) | Mandatory responsive CSS — copy into every page | Phase 3 (generation) |
| [html-template.md](html-template.md) | HTML structure, JS features, section type patterns | Phase 3 (generation) |
| [scrollytelling-patterns.md](scrollytelling-patterns.md) | Scroll-driven animation snippets and patterns | Phase 3 (generation) |
| [scripts/deploy.sh](scripts/deploy.sh) | Deploy page to Vercel for instant sharing | Phase 5 (sharing) |
| [scripts/export-pdf.sh](scripts/export-pdf.sh) | Export page sections as PDF | Phase 5 (sharing) |
