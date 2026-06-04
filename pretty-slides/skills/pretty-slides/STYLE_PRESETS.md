# Style Presets Reference

Curated visual styles for Pretty Slides landing pages. Each preset is inspired by real premium brand pages — no generic "AI slop" aesthetics. **CSS-generated visuals only — no stock imagery.**

**Viewport CSS:** For mandatory base styles, see [viewport-base.css](viewport-base.css). Include in every landing page.

---

## Dark Themes

### 1. Midnight Cinema

**Vibe:** Premium, cinematic, immersive — like a DJI or Apple product page

**Layout:** Full-bleed hero with gradient mesh background, alternating full-width and card-based sections, generous negative space. Product imagery is the star; everything else recedes.

**Typography:**
- Display: `Clash Display` (600/700) — Fontshare
- Body: `General Sans` (400/500) — Fontshare

**Colors:**
```css
:root {
    --bg-primary: #000000;
    --bg-secondary: #0a0a0a;
    --bg-card: #111111;
    --text-primary: #ffffff;
    --text-secondary: #9ca3af;
    --accent: #F68128;
    --accent-secondary: #FFAE66;
    --cta-bg: #F68128;
    --cta-text: #000000;
}
```

**Signature Elements:**
- Gradient text highlights using `background-clip: text` with orange-to-peach gradients
- Full-viewport hero with layered radial gradient mesh
- Scroll-driven product reveal sequences
- Floating spec cards with glassmorphic blur
- Cinematic fade transitions between sections
- Grain texture overlay for film-like quality

---

### 2. Obsidian Product

**Vibe:** Sleek, technical, premium hardware aesthetic

**Layout:** Asymmetric grid layouts, sticky scroll sequences for product deep-dives, floating UI cards with blueprint aesthetics.

**Typography:**
- Display: `Cabinet Grotesk` (700/800) — Fontshare
- Body: `Outfit` (400/500) — Google

**Colors:**
```css
:root {
    --bg-primary: #0a0a0a;
    --bg-secondary: #121220;
    --bg-card: rgba(255, 255, 255, 0.04);
    --text-primary: #f0f0f5;
    --text-secondary: #8888a0;
    --accent: #4361ee;
    --accent-secondary: #7c8aff;
    --cta-bg: #4361ee;
    --cta-text: #ffffff;
}
```

**Signature Elements:**
- Blueprint grid background pattern (subtle white lines on dark)
- Glassmorphic cards with `backdrop-filter: blur()`
- Spec comparison grids with alternating row backgrounds
- Floating UI elements with soft box-shadows
- Horizontal scroll galleries for product angles
- Subtle blue glow on interactive elements

---

### 3. Dark Editorial

**Vibe:** Sophisticated, magazine-quality, storytelling-focused

**Layout:** Editorial grid with mixed column widths, large pull quotes as visual anchors, text-image interplay with generous margins.

**Typography:**
- Display: `Fraunces` (700/900) — Google, distinctive serif
- Body: `Source Serif 4` (400/500) — Google

**Colors:**
```css
:root {
    --bg-primary: #111111;
    --bg-secondary: #1a1a1a;
    --bg-card: #1e1e1e;
    --text-primary: #f5f3ee;
    --text-secondary: #a09a90;
    --accent: #c9a55a;
    --accent-secondary: #c07070;
    --cta-bg: #c9a55a;
    --cta-text: #111111;
}
```

**Signature Elements:**
- Large drop caps on opening paragraphs
- Pull quotes with gold left-border accent
- Horizontal rules as elegant section dividers (`<hr>` styled as thin gold lines)
- Serif headlines creating visual contrast with sans-serif body
- Mixed-width editorial columns
- Warm accent tones (gold, muted rose)

---

### 4. Neon Product

**Vibe:** Futuristic, tech-forward, gaming/crypto aesthetic

**Layout:** Grid-based with neon-bordered sections, floating cards with glow effects, particle-style backgrounds.

**Typography:**
- Display: `Clash Display` (700) — Fontshare
- Mono: `JetBrains Mono` (400) — Google

**Colors:**
```css
:root {
    --bg-primary: #0a0f1c;
    --bg-secondary: #0d1525;
    --bg-card: rgba(0, 255, 204, 0.05);
    --text-primary: #ffffff;
    --text-secondary: #5a6a8a;
    --accent: #00ffcc;
    --accent-secondary: #ff00aa;
    --cta-bg: #00ffcc;
    --cta-text: #0a0f1c;
}
```

**Signature Elements:**
- Neon glow borders (`box-shadow` with accent color at low opacity)
- Grid pattern background (subtle lines)
- Glitch text effect on hover (CSS `clip-path` animation)
- Animated gradient borders on cards
- Monospace accents for data/technical content
- Dual-accent color scheme (cyan + magenta)

---

## Light Themes

### 5. Swiss Landing

**Vibe:** Clean, precise, Bauhaus-inspired, trustworthy

**Layout:** Strong grid system with visible structure, asymmetric but balanced composition, extreme whitespace.

**Typography:**
- Display: `Archivo` (700/800) — Google
- Body: `Nunito` (400/500) — Google

**Colors:**
```css
:root {
    --bg-primary: #ffffff;
    --bg-secondary: #f5f5f5;
    --bg-card: #ffffff;
    --text-primary: #0a0a0a;
    --text-secondary: #555555;
    --accent: #ff3300;
    --accent-secondary: #0a0a0a;
    --cta-bg: #0a0a0a;
    --cta-text: #ffffff;
}
```

**Signature Elements:**
- Visible grid lines as design elements
- Bold section numbers (01. 02. 03.) in large display type
- Red accent used sparingly for maximum impact
- Sharp corners, no border-radius
- Geometric shapes (circles, lines) as decoration
- High-contrast black-on-white with red punctuation

---

### 6. Paper Craft

**Vibe:** Warm, tactile, handcrafted feel

**Layout:** Card-based with soft shadows suggesting layered paper, rounded corners, warm tonal depth.

**Typography:**
- Display: `Bricolage Grotesque` (700/800) — Google
- Body: `DM Sans` (400/500) — Google

**Colors:**
```css
:root {
    --bg-primary: #faf7f2;
    --bg-secondary: #f0ebe3;
    --bg-card: #ffffff;
    --text-primary: #2a2a2a;
    --text-secondary: #6b6560;
    --accent: #c4724e;
    --accent-secondary: #7a9e7e;
    --cta-bg: #c4724e;
    --cta-text: #ffffff;
}
```

**Signature Elements:**
- Paper texture backgrounds (subtle CSS noise)
- Multi-layer card shadows for depth
- Rounded corners on everything (16px+)
- Earth tones with terracotta and sage green accents
- Warm, inviting spacing
- Hand-drawn-feel SVG icons

---

### 7. Soft Studio

**Vibe:** Modern, minimal, tech-startup friendly

**Layout:** Centered content, generous padding, card grids with soft shadows, gradient sections.

**Typography:**
- Display: `Plus Jakarta Sans` (700/800) — Google
- Body: `Plus Jakarta Sans` (400/500) — Google

**Colors:**
```css
:root {
    --bg-primary: #fafafa;
    --bg-secondary: #f0f0ff;
    --bg-card: #ffffff;
    --text-primary: #1e293b;
    --text-secondary: #64748b;
    --accent: #7c3aed;
    --accent-secondary: #38bdf8;
    --cta-bg: #7c3aed;
    --cta-text: #ffffff;
}
```

**Signature Elements:**
- Soft gradient backgrounds (lavender to white)
- Rounded cards with hover elevation transitions
- Pill-shaped CTA buttons
- Icon grids with soft colored backgrounds
- Subtle shadow layers
- Dual-color gradient accent elements

---

### 8. Warm Editorial

**Vibe:** Storytelling, magazine aesthetic, lifestyle brand

**Layout:** Mixed-column editorial grid, large images, text wraps around media with generous gutters.

**Typography:**
- Display: `Cormorant` (600/700) — Google, elegant serif
- Body: `Work Sans` (400/500) — Google

**Colors:**
```css
:root {
    --bg-primary: #f5efe6;
    --bg-secondary: #ebe3d6;
    --bg-card: #ffffff;
    --text-primary: #2c1810;
    --text-secondary: #6b5a4e;
    --accent: #d4763c;
    --accent-secondary: #5a6b4a;
    --cta-bg: #2c1810;
    --cta-text: #f5efe6;
}
```

**Signature Elements:**
- Serif headlines with generous letter-spacing
- Large editorial-style images with subtle borders
- Text-image interplay (offset grids)
- Natural earth tones (warm beige, brown, olive)
- Thin horizontal rules between sections
- Italic subheadings and pull quotes

---

## Specialty Themes

### 9. Split Contrast

**Vibe:** Bold, eye-catching, split-screen storytelling

**Layout:** Vertical split sections with two-color backgrounds, content on one side and media on the other. Alternating split direction per section.

**Typography:**
- Display: `Syne` (700/800) — Google
- Body: `Space Grotesk` (400/500) — Google

**Colors:**
```css
:root {
    --bg-primary: #fff8f0;
    --bg-secondary: #1a1a3e;
    --bg-card: rgba(255, 255, 255, 0.9);
    --text-primary: #1a1a3e;
    --text-secondary: #555;
    --accent: #ff6b6b;
    --accent-secondary: #1a1a3e;
    --cta-bg: #ff6b6b;
    --cta-text: #ffffff;
}
```

**Signature Elements:**
- Two-tone section backgrounds (e.g., coral + cream, navy + mint)
- Bold color blocking with diagonal or straight dividers
- High contrast between adjacent sections
- Playful badge/pill elements
- Split-screen hero (text left, media right)
- Alternating layouts to create visual rhythm

---

### 10. Sunset Gradient

**Vibe:** Warm, vibrant, festival/event energy

**Layout:** Full-bleed gradient sections with floating cards, radial glow effects behind key elements.

**Typography:**
- Display: `Manrope` (700/800) — Google
- Body: `Manrope` (400/500) — Google

**Colors:**
```css
:root {
    --bg-primary: #1a0533;
    --bg-secondary: #250845;
    --bg-card: rgba(255, 255, 255, 0.08);
    --text-primary: #ffffff;
    --text-secondary: #c0a0d0;
    --accent: #ff6b35;
    --accent-secondary: #ffd166;
    --cta-bg: #ff6b35;
    --cta-text: #ffffff;
}
```

**Signature Elements:**
- Multi-stop gradient backgrounds (purple → orange → pink)
- Radial glow effects behind hero elements
- Floating card grids with warm shadows
- Warm-to-cool color progressions through page
- Golden accents for highlights and stats
- Animated gradient shifts on hover

---

### 11. Natural Texture

**Vibe:** Organic, grounded, sustainability/wellness brand

**Layout:** Organic shapes as section dividers, rounded containers, earth-tone palette throughout.

**Typography:**
- Display: `Playfair Display` (700) — Google
- Body: `Lato` (400/500) — Google

**Colors:**
```css
:root {
    --bg-primary: #f5f0e8;
    --bg-secondary: #e8dcc8;
    --bg-card: #ffffff;
    --text-primary: #1a2f1a;
    --text-secondary: #5a6b5a;
    --accent: #8fac7e;
    --accent-secondary: #c07050;
    --cta-bg: #1a2f1a;
    --cta-text: #f5f0e8;
}
```

**Signature Elements:**
- Organic blob shapes via CSS `border-radius` as section dividers
- Earth tones throughout (forest, sand, sage, terracotta)
- Rounded everything (20px+ border-radius)
- Nature-inspired color progressions
- Textured backgrounds (subtle grain or paper)
- Serif display with clean sans body contrast

---

### 12. Electric Showcase

**Vibe:** Bold, modern, SaaS/tech product launch

**Layout:** Hero with animated gradient mesh, feature grid with icon cards, pricing table, alternating light/dark sections.

**Typography:**
- Display: `Satoshi` (700/900) — Fontshare
- Body: `General Sans` (400/500) — Fontshare

**Colors:**
```css
:root {
    --bg-primary: #0f0720;
    --bg-secondary: #150a30;
    --bg-card: rgba(124, 58, 237, 0.08);
    --text-primary: #ffffff;
    --text-secondary: #a0a0c0;
    --accent: #7c3aed;
    --accent-secondary: #06b6d4;
    --cta-bg: linear-gradient(135deg, #7c3aed, #06b6d4);
    --cta-text: #ffffff;
}
```

**Signature Elements:**
- Animated gradient mesh hero background (CSS `@keyframes` shifting gradients)
- Glassmorphic pricing cards with hover effects
- Feature icon grids with colored backgrounds
- Aurora/northern-lights gradient effects
- Gradient CTA buttons
- Dual-accent purple-to-cyan color system

---

## Font Pairing Quick Reference

| Preset | Display Font | Body Font | Source |
| --- | --- | --- | --- |
| Midnight Cinema | Clash Display | General Sans | Fontshare |
| Obsidian Product | Cabinet Grotesk | Outfit | Fontshare / Google |
| Dark Editorial | Fraunces | Source Serif 4 | Google |
| Neon Product | Clash Display | JetBrains Mono | Fontshare / Google |
| Swiss Landing | Archivo | Nunito | Google |
| Paper Craft | Bricolage Grotesque | DM Sans | Google |
| Soft Studio | Plus Jakarta Sans | Plus Jakarta Sans | Google |
| Warm Editorial | Cormorant | Work Sans | Google |
| Split Contrast | Syne | Space Grotesk | Google |
| Sunset Gradient | Manrope | Manrope | Google |
| Natural Texture | Playfair Display | Lato | Google |
| Electric Showcase | Satoshi | General Sans | Fontshare |

---

## DO NOT USE (Generic AI Patterns)

**Fonts:** Inter, Roboto, Arial, system fonts as display

**Colors:** `#6366f1` (generic indigo), purple gradients on white, the default Tailwind palette verbatim

**Layouts:** Everything centered with identical spacing, generic hero + 3-card + CTA cookie cutter, every section same width

**Decorations:** Stock photo backgrounds, gratuitous glassmorphism without purpose, generic gradient blobs, identical card grids without hierarchy

**Interactions:** Animations on everything (pick 2-3 high-impact moments), parallax for parallax's sake, scroll-jacking that fights the user

---

## CSS Gotchas

### Negating CSS Functions

**WRONG — silently ignored by browsers (no console error):**
```css
right: -clamp(28px, 3.5vw, 44px);   /* Browser ignores this */
margin-left: -min(10vw, 100px);      /* Browser ignores this */
```

**CORRECT — wrap in `calc()`:**
```css
right: calc(-1 * clamp(28px, 3.5vw, 44px));  /* Works */
margin-left: calc(-1 * min(10vw, 100px));     /* Works */
```

CSS does not allow a leading `-` before function names. The browser silently discards the entire declaration. **Always use `calc(-1 * ...)` to negate CSS function values.**
