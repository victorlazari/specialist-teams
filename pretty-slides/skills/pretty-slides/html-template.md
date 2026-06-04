# HTML Landing Page Template

Reference architecture for generating scrollytelling landing pages. Every page follows this structure.

## Base HTML Structure

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Page Title</title>

    <!-- Fonts: use Fontshare or Google Fonts — never system fonts -->
    <link rel="stylesheet" href="https://api.fontshare.com/v2/css?f[]=..." />

    <style>
      /* ===========================================
           CSS CUSTOM PROPERTIES (THEME)
           Change these to change the whole look
           =========================================== */
      :root {
        /* Colors — from chosen style preset */
        --bg-primary: #000000;
        --bg-secondary: #0a0a0a;
        --bg-card: #111111;
        --text-primary: #ffffff;
        --text-secondary: #9ca3af;
        --accent: #f68128;
        --accent-secondary: #ffae66;
        --cta-bg: #f68128;
        --cta-text: #000000;

        /* Typography — MUST use clamp() */
        --font-display: "Clash Display", sans-serif;
        --font-body: "General Sans", sans-serif;
        --hero-title-size: clamp(2.5rem, 8vw, 6rem);
        --hero-subtitle-size: clamp(1rem, 2.5vw, 1.5rem);
        --section-title-size: clamp(1.75rem, 5vw, 3.5rem);
        --h3-size: clamp(1.25rem, 3vw, 2rem);
        --body-size: clamp(0.875rem, 1.5vw, 1.125rem);
        --small-size: clamp(0.75rem, 1vw, 0.875rem);

        /* Spacing — MUST use clamp() */
        --section-padding: clamp(3rem, 8vh, 8rem) clamp(1.5rem, 5vw, 6rem);
        --content-max-width: min(90vw, 1200px);
        --content-gap: clamp(1.5rem, 3vw, 3rem);
        --card-gap: clamp(1rem, 2vw, 2rem);

        /* Animation */
        --ease-out-expo: cubic-bezier(0.16, 1, 0.3, 1);
        --ease-out-back: cubic-bezier(0.34, 1.56, 0.64, 1);
        --duration-normal: 0.6s;
        --duration-slow: 1s;
      }

      /* ===========================================
           BASE STYLES
           =========================================== */
      * {
        margin: 0;
        padding: 0;
        box-sizing: border-box;
      }

      /* --- PASTE viewport-base.css CONTENTS HERE --- */

      /* ===========================================
           REVEAL ANIMATIONS
           Trigger via .visible class (added by JS on scroll)
           =========================================== */
      .reveal {
        opacity: 0;
        transform: translateY(40px);
        transition:
          opacity var(--duration-normal) var(--ease-out-expo),
          transform var(--duration-normal) var(--ease-out-expo);
      }

      .section.visible .reveal {
        opacity: 1;
        transform: translateY(0);
      }

      /* Stagger children for sequential reveal */
      .reveal:nth-child(1) { transition-delay: 0.1s; }
      .reveal:nth-child(2) { transition-delay: 0.2s; }
      .reveal:nth-child(3) { transition-delay: 0.3s; }
      .reveal:nth-child(4) { transition-delay: 0.4s; }
      .reveal:nth-child(5) { transition-delay: 0.5s; }
      .reveal:nth-child(6) { transition-delay: 0.6s; }

      /* ... preset-specific styles ... */
    </style>
  </head>
  <body>
    <!-- === SCROLL PROGRESS BAR === -->
    <div class="scroll-progress"></div>

    <!-- === STICKY NAVIGATION === -->
    <nav class="site-nav">
      <div class="nav-brand">Brand Name</div>
      <div class="nav-links">
        <a href="#hero">Home</a>
        <a href="#features">Features</a>
        <a href="#specs">Specs</a>
        <a href="#pricing">Pricing</a>
      </div>
      <a href="#cta" class="nav-cta cta-button">Get Started</a>
    </nav>

    <main>
      <!-- === HERO SECTION === -->
      <section class="section hero-section" id="hero">
        <div class="hero-bg">
          <!-- Gradient mesh, video, or image background -->
        </div>
        <div class="hero-content">
          <h1 class="reveal">Hero Headline</h1>
          <p class="hero-subtitle reveal">
            Compelling subheadline that supports the main message
          </p>
          <a href="#cta" class="reveal cta-button">Primary Call to Action</a>
        </div>
      </section>

      <!-- === FEATURE SECTION === -->
      <section class="section feature-section" id="features">
        <div class="section-content">
          <h2 class="reveal">Key Features</h2>
          <p class="section-subtitle reveal">Supporting description</p>
          <div class="feature-grid">
            <div class="feature-card reveal">
              <div class="feature-icon"><!-- SVG or emoji --></div>
              <h3>Feature Title</h3>
              <p>Short description of what this feature does.</p>
            </div>
            <!-- More feature cards... -->
          </div>
        </div>
      </section>

      <!-- === SCROLLYTELLING SECTION === -->
      <section class="section scrollytelling-section" id="story">
        <div class="scroll-track" style="height: calc(300vh)">
          <div class="sticky-media">
            <img data-src="product.jpg" alt="Product" class="scroll-media" />
          </div>
          <div class="scroll-panels">
            <div class="scroll-panel" data-progress="0">
              <div class="panel-content">
                <h3>First reveal</h3>
                <p>Text that appears at scroll start</p>
              </div>
            </div>
            <div class="scroll-panel" data-progress="0.33">
              <div class="panel-content">
                <h3>Second reveal</h3>
                <p>Text that appears at 33% scroll</p>
              </div>
            </div>
            <div class="scroll-panel" data-progress="0.66">
              <div class="panel-content">
                <h3>Third reveal</h3>
                <p>Text that appears at 66% scroll</p>
              </div>
            </div>
          </div>
        </div>
      </section>

      <!-- === SPEC GRID SECTION === -->
      <section class="section spec-section" id="specs">
        <div class="section-content">
          <h2 class="reveal">Specifications</h2>
          <div class="spec-grid">
            <div class="spec-item reveal">
              <span class="spec-value">4K</span>
              <span class="spec-label">Resolution</span>
            </div>
            <!-- More spec items... -->
          </div>
        </div>
      </section>

      <!-- === STATS SECTION === -->
      <section class="section stats-section" id="stats">
        <div class="section-content">
          <h2 class="reveal">By the Numbers</h2>
          <div class="stats-grid">
            <div class="stat-item reveal">
              <span class="stat-number" data-target="10000">0</span>
              <span class="stat-suffix">+</span>
              <span class="stat-label">Active Users</span>
            </div>
            <div class="stat-item reveal">
              <span class="stat-number" data-target="99">0</span>
              <span class="stat-suffix">%</span>
              <span class="stat-label">Uptime</span>
            </div>
            <div class="stat-item reveal">
              <span class="stat-number" data-target="50">0</span>
              <span class="stat-suffix">ms</span>
              <span class="stat-label">Response Time</span>
            </div>
          </div>
        </div>
      </section>

      <!-- === TESTIMONIAL SECTION === -->
      <section class="section testimonial-section" id="testimonials">
        <div class="section-content">
          <h2 class="reveal">What People Say</h2>
          <div class="testimonial-grid">
            <blockquote class="testimonial-card reveal">
              <p>"Quote text here."</p>
              <cite>
                <strong>Name</strong>
                <span>Title, Company</span>
              </cite>
            </blockquote>
            <!-- More testimonials... -->
          </div>
        </div>
      </section>

      <!-- === IMAGE GALLERY SECTION === -->
      <section class="section gallery-section" id="gallery">
        <div class="section-content">
          <h2 class="reveal">Gallery</h2>
          <div class="gallery-grid">
            <div class="gallery-item reveal">
              <img data-src="image1.jpg" alt="Description" />
            </div>
            <!-- More gallery items... -->
          </div>
        </div>
      </section>

      <!-- === COMPARISON TABLE SECTION === -->
      <section class="section comparison-section" id="compare">
        <div class="section-content">
          <h2 class="reveal">Compare</h2>
          <div class="comparison-table reveal">
            <table>
              <thead>
                <tr>
                  <th>Feature</th>
                  <th>Basic</th>
                  <th class="highlighted">Pro</th>
                  <th>Enterprise</th>
                </tr>
              </thead>
              <tbody>
                <tr>
                  <td>Feature name</td>
                  <td>Value</td>
                  <td class="highlighted">Value</td>
                  <td>Value</td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </section>

      <!-- === CTA SECTION === -->
      <section class="section cta-section" id="cta">
        <div class="section-content">
          <h2 class="reveal">Ready to Get Started?</h2>
          <p class="reveal">Supporting text that creates urgency or reassurance</p>
          <div class="cta-buttons reveal">
            <a href="#" class="cta-button primary">Primary Action</a>
            <a href="#" class="cta-button secondary">Secondary Action</a>
          </div>
        </div>
      </section>
    </main>

    <!-- === FOOTER === -->
    <footer class="site-footer">
      <div class="footer-content">
        <div class="footer-brand">Brand Name</div>
        <div class="footer-links">
          <div class="footer-column">
            <h4>Product</h4>
            <a href="#">Features</a>
            <a href="#">Pricing</a>
          </div>
          <div class="footer-column">
            <h4>Company</h4>
            <a href="#">About</a>
            <a href="#">Contact</a>
          </div>
        </div>
        <div class="footer-bottom">
          <p>&copy; 2024 Brand. All rights reserved.</p>
        </div>
      </div>
    </footer>

    <script>
      /* ===========================================
           LANDING PAGE CONTROLLER
           =========================================== */
      class LandingPage {
        constructor() {
          this.sections = document.querySelectorAll(".section");
          this.navLinks = document.querySelectorAll(".nav-links a");
          this.setupIntersectionObserver();
          this.setupScrollProgress();
          this.setupStickyNav();
          this.setupScrollytelling();
          this.setupCounterAnimations();
          this.setupLazyLoading();
          this.setupSmoothScroll();
        }

        /* --- Intersection Observer: .visible on scroll --- */
        setupIntersectionObserver() {
          const observer = new IntersectionObserver(
            (entries) => {
              entries.forEach((entry) => {
                if (entry.isIntersecting) {
                  entry.target.classList.add("visible");
                }
              });
            },
            { threshold: 0.15, rootMargin: "0px 0px -50px 0px" }
          );
          this.sections.forEach((section) => observer.observe(section));
        }

        /* --- Scroll Progress Bar --- */
        setupScrollProgress() {
          const progressBar = document.querySelector(".scroll-progress");
          if (!progressBar) return;

          let ticking = false;
          window.addEventListener("scroll", () => {
            if (!ticking) {
              requestAnimationFrame(() => {
                const scrollTop = window.pageYOffset;
                const docHeight =
                  document.documentElement.scrollHeight - window.innerHeight;
                const progress = (scrollTop / docHeight) * 100;
                progressBar.style.width = progress + "%";
                ticking = false;
              });
              ticking = true;
            }
          });
        }

        /* --- Sticky Nav: transparent → solid on scroll --- */
        setupStickyNav() {
          const nav = document.querySelector(".site-nav");
          if (!nav) return;

          let ticking = false;
          window.addEventListener("scroll", () => {
            if (!ticking) {
              requestAnimationFrame(() => {
                nav.classList.toggle("scrolled", window.pageYOffset > 100);
                this.updateActiveNavLink();
                ticking = false;
              });
              ticking = true;
            }
          });
        }

        /* --- Update active nav link based on scroll position --- */
        updateActiveNavLink() {
          let current = "";
          this.sections.forEach((section) => {
            const rect = section.getBoundingClientRect();
            if (rect.top <= 150 && rect.bottom > 150) {
              current = section.id;
            }
          });
          this.navLinks.forEach((link) => {
            link.classList.toggle(
              "active",
              link.getAttribute("href") === "#" + current
            );
          });
        }

        /* --- Scrollytelling: scroll-mapped animations --- */
        setupScrollytelling() {
          document
            .querySelectorAll(".scrollytelling-section")
            .forEach((section) => {
              const track = section.querySelector(".scroll-track");
              const panels = section.querySelectorAll(".scroll-panel");
              if (!track || !panels.length) return;

              const update = () => {
                const rect = track.getBoundingClientRect();
                const trackHeight = rect.height - window.innerHeight;
                const progress = Math.max(
                  0,
                  Math.min(1, -rect.top / trackHeight)
                );
                section.style.setProperty("--scroll-progress", progress);

                panels.forEach((panel) => {
                  const threshold = parseFloat(panel.dataset.progress);
                  const isActive = Math.abs(progress - threshold) < 0.15;
                  panel.classList.toggle("active", isActive);
                });

                if (rect.top < window.innerHeight && rect.bottom > 0) {
                  requestAnimationFrame(update);
                }
              };

              const observer = new IntersectionObserver(
                (entries) => {
                  entries.forEach((entry) => {
                    if (entry.isIntersecting)
                      requestAnimationFrame(update);
                  });
                },
                { threshold: 0 }
              );
              observer.observe(track);
            });
        }

        /* --- Counter Animations --- */
        setupCounterAnimations() {
          const counters = document.querySelectorAll("[data-target]");
          if (!counters.length) return;

          const observer = new IntersectionObserver(
            (entries) => {
              entries.forEach((entry) => {
                if (entry.isIntersecting) {
                  this.animateCounter(entry.target);
                  observer.unobserve(entry.target);
                }
              });
            },
            { threshold: 0.5 }
          );
          counters.forEach((counter) => observer.observe(counter));
        }

        animateCounter(element) {
          const target = parseInt(element.dataset.target);
          const duration = 2000;
          const startTime = performance.now();

          const animate = (now) => {
            const progress = Math.min((now - startTime) / duration, 1);
            const eased = 1 - Math.pow(1 - progress, 3);
            element.textContent = Math.floor(
              target * eased
            ).toLocaleString();
            if (progress < 1) requestAnimationFrame(animate);
          };
          requestAnimationFrame(animate);
        }

        /* --- Lazy Loading --- */
        setupLazyLoading() {
          const lazyElements = document.querySelectorAll("[data-src]");
          if (!lazyElements.length) return;

          const observer = new IntersectionObserver(
            (entries) => {
              entries.forEach((entry) => {
                if (entry.isIntersecting) {
                  const el = entry.target;
                  el.src = el.dataset.src;
                  el.removeAttribute("data-src");
                  observer.unobserve(el);
                }
              });
            },
            { rootMargin: "200px" }
          );
          lazyElements.forEach((el) => observer.observe(el));
        }

        /* --- Smooth Scroll for Anchor Links --- */
        setupSmoothScroll() {
          document.querySelectorAll('a[href^="#"]').forEach((anchor) => {
            anchor.addEventListener("click", (e) => {
              e.preventDefault();
              const target = document.querySelector(
                anchor.getAttribute("href")
              );
              if (target) {
                target.scrollIntoView({ behavior: "smooth", block: "start" });
              }
            });
          });
        }
      }

      new LandingPage();
    </script>
  </body>
</html>
```

## Required JavaScript Features

Every landing page must include:

1. **LandingPage Class** — Main controller with:
   - Intersection Observer for scroll-triggered `.visible` class
   - Scroll progress bar (width maps to scroll position)
   - Sticky nav (transparent → solid after scrolling past hero)
   - Active nav link highlighting based on current section
   - Smooth scroll for all anchor links
   - Counter/number animations (counting up when visible)
   - Lazy loading via `data-src` attribute swapping
   - Scrollytelling controller (maps scroll position within sticky sections)

2. **Scrollytelling Controller** — For scroll-driven sequences:
   - Sticky media container with scroll track
   - Progress calculation: `progress = -rect.top / (rect.height - window.innerHeight)`
   - CSS custom property `--scroll-progress` updated on each frame
   - Panel activation based on `data-progress` threshold proximity
   - Only runs `requestAnimationFrame` loop while section is in viewport

3. **Performance Optimizations** (mandatory):
   - `content-visibility: auto` on all non-hero sections
   - `contain-intrinsic-size` for layout stability
   - Lazy loading via Intersection Observer (rootMargin: 200px)
   - All scroll handlers throttled via `requestAnimationFrame`
   - `will-change` only on actively animating elements (never as a permanent prop)
   - Videos: `playsinline`, `muted`, `loop`, `autoplay` for ambient backgrounds

## Section Type CSS Patterns

### Hero Section

```css
.hero-section {
  min-height: 100vh;
  min-height: 100dvh;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  position: relative;
  overflow: hidden;
}

.hero-bg {
  position: absolute;
  inset: 0;
  z-index: 0;
}

.hero-content {
  position: relative;
  z-index: 1;
  max-width: min(90vw, 800px);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: var(--content-gap);
}

.hero-subtitle {
  font-size: var(--hero-subtitle-size);
  color: var(--text-secondary);
  max-width: 600px;
}
```

### Feature Grid

```css
.feature-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 300px), 1fr));
  gap: var(--card-gap);
  margin-top: var(--content-gap);
}

.feature-card {
  padding: clamp(1.5rem, 3vw, 2.5rem);
  border-radius: clamp(8px, 1vw, 16px);
  background: var(--bg-card);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.feature-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.2);
}

.feature-icon {
  font-size: clamp(1.5rem, 3vw, 2.5rem);
  margin-bottom: var(--element-gap);
}
```

### Spec Grid

```css
.spec-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 200px), 1fr));
  gap: var(--card-gap);
  margin-top: var(--content-gap);
}

.spec-item {
  text-align: center;
  padding: clamp(1rem, 2vw, 2rem);
}

.spec-value {
  font-family: var(--font-display);
  font-size: clamp(2rem, 4vw, 3rem);
  font-weight: 700;
  color: var(--accent);
  display: block;
}

.spec-label {
  font-size: var(--small-size);
  color: var(--text-secondary);
  text-transform: uppercase;
  letter-spacing: 0.1em;
}
```

### Testimonial Grid

```css
.testimonial-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 320px), 1fr));
  gap: var(--card-gap);
  margin-top: var(--content-gap);
}

.testimonial-card {
  padding: clamp(1.5rem, 3vw, 2.5rem);
  border-radius: clamp(8px, 1vw, 16px);
  background: var(--bg-card);
  border-left: 3px solid var(--accent);
}

.testimonial-card p {
  font-size: var(--body-size);
  font-style: italic;
  line-height: 1.7;
  margin-bottom: var(--element-gap);
}

.testimonial-card cite {
  display: flex;
  flex-direction: column;
  font-style: normal;
}

.testimonial-card cite strong {
  color: var(--text-primary);
}

.testimonial-card cite span {
  font-size: var(--small-size);
  color: var(--text-secondary);
}
```

### Gallery Grid

```css
.gallery-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 280px), 1fr));
  gap: var(--card-gap);
  margin-top: var(--content-gap);
}

.gallery-item {
  border-radius: clamp(8px, 1vw, 16px);
  overflow: hidden;
  aspect-ratio: 16 / 10;
}

.gallery-item img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.5s var(--ease-out-expo);
}

.gallery-item:hover img {
  transform: scale(1.05);
}
```

### Comparison Table

```css
.comparison-table {
  overflow-x: auto;
  margin-top: var(--content-gap);
}

.comparison-table table {
  width: 100%;
  border-collapse: collapse;
  font-size: var(--body-size);
}

.comparison-table th,
.comparison-table td {
  padding: clamp(0.75rem, 1.5vw, 1.25rem);
  text-align: center;
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
}

.comparison-table th {
  font-family: var(--font-display);
  font-weight: 600;
}

.comparison-table .highlighted {
  background: rgba(var(--accent-rgb, 246, 129, 40), 0.08);
}
```

### CTA Section

```css
.cta-section {
  text-align: center;
  background: var(--bg-secondary);
  padding: clamp(4rem, 10vh, 10rem) var(--section-padding);
}

.cta-buttons {
  display: flex;
  gap: var(--element-gap);
  justify-content: center;
  flex-wrap: wrap;
  margin-top: var(--content-gap);
}

.cta-button.secondary {
  background: transparent;
  border: 2px solid var(--text-secondary);
  color: var(--text-primary);
}
```

### Footer

```css
.site-footer {
  padding: clamp(3rem, 6vh, 6rem) clamp(1.5rem, 5vw, 6rem);
  background: var(--bg-secondary);
  border-top: 1px solid rgba(255, 255, 255, 0.05);
}

.footer-content {
  max-width: var(--content-max-width);
  margin: 0 auto;
}

.footer-links {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: var(--card-gap);
  margin: var(--content-gap) 0;
}

.footer-column a {
  display: block;
  color: var(--text-secondary);
  text-decoration: none;
  font-size: var(--small-size);
  padding: 0.25rem 0;
}

.footer-bottom {
  border-top: 1px solid rgba(255, 255, 255, 0.05);
  padding-top: var(--content-gap);
  font-size: var(--small-size);
  color: var(--text-secondary);
}
```

## Image Pipeline (Skip If No Media)

If user chose "No media" in Phase 1, skip this entirely.

**Dependency:** `pip install Pillow`

### Image Processing

```python
from PIL import Image, ImageDraw

# Resize for web (keep under 1MB per image)
def resize_max(input_path, output_path, max_dim=1600):
    img = Image.open(input_path)
    img.thumbnail((max_dim, max_dim), Image.LANCZOS)
    img.save(output_path, quality=85)

# Circular crop for avatars/logos
def crop_circle(input_path, output_path):
    img = Image.open(input_path).convert('RGBA')
    w, h = img.size
    size = min(w, h)
    left, top = (w - size) // 2, (h - size) // 2
    img = img.crop((left, top, left + size, top + size))
    mask = Image.new('L', (size, size), 0)
    ImageDraw.Draw(mask).ellipse([0, 0, size, size], fill=255)
    img.putalpha(mask)
    img.save(output_path, 'PNG')
```

### Image Placement

**Use lazy-loaded file paths** — images load on scroll:

```html
<img data-src="assets/product.jpg" alt="Product" class="section-image" />
```

```css
.section-image {
  max-width: 100%;
  max-height: min(60vh, 500px);
  object-fit: contain;
  border-radius: clamp(8px, 1vw, 16px);
}
```

**Adapt styling to match the chosen style's aesthetic.** Never repeat the same image in multiple sections (except logos in nav + footer).

---

## Code Quality

**Comments:** Every section needs clear `/* === SECTION NAME === */` comment blocks.

**Accessibility:**

- Semantic HTML (`<section>`, `<nav>`, `<header>`, `<footer>`, `<main>`)
- Keyboard-accessible navigation
- ARIA labels on interactive elements
- `prefers-reduced-motion` support (included in viewport-base.css)
- Color contrast ratios meet WCAG AA

**Performance:**

- `content-visibility: auto` on non-hero sections
- Lazy loading for all media
- Throttled scroll handlers
- No layout thrashing in animation loops

## File Structure

Single landing pages:

```
landing-page.html    # Self-contained, all CSS/JS inline
assets/              # Images/videos only, if any
```

Multiple pages in one project:

```
[name].html
[name]-assets/
```
