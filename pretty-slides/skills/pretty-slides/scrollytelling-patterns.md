# Scrollytelling Patterns Reference

Use this reference when generating landing pages. Match scroll-driven patterns to the intended feeling.

## Effect-to-Feeling Guide

| Feeling | Scroll Patterns | Visual Cues |
| --- | --- | --- |
| **Cinematic / Premium** | Slow parallax layers, sticky product reveals, scroll-linked video playback | Dark backgrounds, spotlight effects, gradient text, full-bleed media |
| **Technical / Futuristic** | Grid reveals, data scroll animations, blueprint-style line draws | Neon accents, grid patterns, monospace highlights, particle effects |
| **Playful / Friendly** | Bouncy scroll snaps, floating elements, elastic parallax | Rounded corners, pastel colors, organic shapes, playful transitions |
| **Professional / Corporate** | Clean fade-ins, subtle parallax, structured grid reveals | Navy/slate tones, precise spacing, clean data visualizations |
| **Editorial / Magazine** | Horizontal scroll galleries, text-image interplay, pull quote reveals | Strong type hierarchy, serif headlines, layered layouts |
| **Dramatic / Storytelling** | Full-viewport transitions, scene changes on scroll, cinematic wipes | High contrast, dramatic lighting effects, immersive backgrounds |

## Scroll-Triggered Entrance Animations

```css
/* Fade + Slide Up (most versatile — use as default) */
.reveal {
    opacity: 0;
    transform: translateY(40px);
    transition: opacity 0.8s var(--ease-out-expo),
                transform 0.8s var(--ease-out-expo);
}
.visible .reveal {
    opacity: 1;
    transform: translateY(0);
}

/* Scale In (good for cards and images) */
.reveal-scale {
    opacity: 0;
    transform: scale(0.9);
    transition: opacity 0.8s, transform 0.8s var(--ease-out-expo);
}
.visible .reveal-scale {
    opacity: 1;
    transform: scale(1);
}

/* Slide from Left */
.reveal-left {
    opacity: 0;
    transform: translateX(-60px);
    transition: opacity 0.8s, transform 0.8s var(--ease-out-expo);
}
.visible .reveal-left {
    opacity: 1;
    transform: translateX(0);
}

/* Slide from Right */
.reveal-right {
    opacity: 0;
    transform: translateX(60px);
    transition: opacity 0.8s, transform 0.8s var(--ease-out-expo);
}
.visible .reveal-right {
    opacity: 1;
    transform: translateX(0);
}

/* Blur In (premium, cinematic) */
.reveal-blur {
    opacity: 0;
    filter: blur(10px);
    transition: opacity 1s var(--ease-out-expo), filter 1s var(--ease-out-expo);
}
.visible .reveal-blur {
    opacity: 1;
    filter: blur(0);
}

/* Clip Reveal — top to bottom wipe */
.reveal-clip {
    clip-path: inset(0 0 100% 0);
    transition: clip-path 1s var(--ease-out-expo);
}
.visible .reveal-clip {
    clip-path: inset(0 0 0 0);
}

/* Stagger children for sequential reveal */
.reveal:nth-child(1) { transition-delay: 0.1s; }
.reveal:nth-child(2) { transition-delay: 0.2s; }
.reveal:nth-child(3) { transition-delay: 0.3s; }
.reveal:nth-child(4) { transition-delay: 0.4s; }
.reveal:nth-child(5) { transition-delay: 0.5s; }
.reveal:nth-child(6) { transition-delay: 0.6s; }
```

## Scrollytelling Patterns

### Pattern 1: Sticky Media with Text Panels (DJI-style)

The signature scrollytelling pattern: a media element sticks in place while text panels scroll past, each triggering a visual change.

```css
/* Scroll track — defines total scroll distance */
.scroll-track {
    position: relative;
    /* height set inline: calc(300vh) for 3 panels, calc(400vh) for 4, etc. */
}

/* Sticky media container — stays in viewport */
.sticky-media {
    position: sticky;
    top: 0;
    height: 100vh;
    height: 100dvh;
    display: flex;
    align-items: center;
    justify-content: center;
    overflow: hidden;
}

/* Media that transforms with scroll progress */
.scroll-media {
    max-width: 80%;
    max-height: 80%;
    object-fit: contain;
    transition: transform 0.1s linear, opacity 0.3s ease;
    /* CSS transforms driven by --scroll-progress custom property */
    transform: scale(calc(0.8 + var(--scroll-progress, 0) * 0.2))
               rotate(calc(var(--scroll-progress, 0) * 5deg));
}

/* Text panels — scroll over the sticky media */
.scroll-panels {
    position: relative;
    z-index: 2;
    pointer-events: none;
}

.scroll-panel {
    min-height: 80vh;
    display: flex;
    align-items: center;
    padding: clamp(2rem, 5vw, 4rem);
    opacity: 0;
    transform: translateY(20px);
    transition: opacity 0.5s ease, transform 0.5s ease;
    pointer-events: auto;
}

.scroll-panel.active {
    opacity: 1;
    transform: translateY(0);
}

.panel-content {
    max-width: 400px;
    padding: clamp(1.5rem, 3vw, 2.5rem);
    background: rgba(0, 0, 0, 0.7);
    backdrop-filter: blur(8px);
    border-radius: clamp(8px, 1vw, 16px);
}
```

```javascript
class ScrollytellingController {
    constructor(section) {
        this.section = section;
        this.track = section.querySelector('.scroll-track');
        this.panels = section.querySelectorAll('.scroll-panel');
        this.update = this.update.bind(this);
        this.observe();
    }

    observe() {
        const observer = new IntersectionObserver((entries) => {
            entries.forEach(entry => {
                if (entry.isIntersecting) requestAnimationFrame(this.update);
            });
        }, { threshold: 0 });
        observer.observe(this.track);
    }

    update() {
        const rect = this.track.getBoundingClientRect();
        const trackHeight = rect.height - window.innerHeight;
        const progress = Math.max(0, Math.min(1, -rect.top / trackHeight));
        this.section.style.setProperty('--scroll-progress', progress);

        this.panels.forEach(panel => {
            const threshold = parseFloat(panel.dataset.progress);
            const isActive = Math.abs(progress - threshold) < 0.15;
            panel.classList.toggle('active', isActive);
        });

        if (rect.top < window.innerHeight && rect.bottom > 0) {
            requestAnimationFrame(this.update);
        }
    }
}
```

### Pattern 2: Parallax Layers

Multiple elements moving at different speeds for depth perception.

```html
<div class="parallax-section">
    <div class="parallax-layer" data-parallax="-0.3">Background element</div>
    <div class="parallax-layer" data-parallax="-0.1">Midground element</div>
    <div class="parallax-layer" data-parallax="0">Foreground (static)</div>
</div>
```

```javascript
class ParallaxController {
    constructor() {
        this.layers = document.querySelectorAll('[data-parallax]');
        let ticking = false;
        window.addEventListener('scroll', () => {
            if (!ticking) {
                requestAnimationFrame(() => {
                    this.onScroll();
                    ticking = false;
                });
                ticking = true;
            }
        });
    }

    onScroll() {
        const scrollY = window.pageYOffset;
        this.layers.forEach(layer => {
            const speed = parseFloat(layer.dataset.parallax);
            layer.style.transform = `translateY(${scrollY * speed}px)`;
        });
    }
}
```

### Pattern 3: Scroll-Linked Video Playback

Map scroll position to video `currentTime` for frame-by-frame product reveals.

```javascript
class ScrollVideo {
    constructor(videoEl, scrollTrack) {
        this.video = videoEl;
        this.track = scrollTrack;
        this.video.pause();

        let ticking = false;
        window.addEventListener('scroll', () => {
            if (!ticking) {
                requestAnimationFrame(() => {
                    this.update();
                    ticking = false;
                });
                ticking = true;
            }
        });
    }

    update() {
        const rect = this.track.getBoundingClientRect();
        const progress = Math.max(0, Math.min(1,
            -rect.top / (rect.height - window.innerHeight)
        ));
        if (this.video.duration) {
            this.video.currentTime = progress * this.video.duration;
        }
    }
}
```

### Pattern 4: Counter/Number Animation

Animate statistics counting up when the section enters viewport.

```javascript
class CounterAnimation {
    constructor(element) {
        this.element = element;
        this.target = parseInt(element.dataset.target);
        this.duration = 2000;
        this.started = false;
    }

    start() {
        if (this.started) return;
        this.started = true;
        const startTime = performance.now();

        const animate = (now) => {
            const progress = Math.min((now - startTime) / this.duration, 1);
            const eased = 1 - Math.pow(1 - progress, 3); // ease-out cubic
            this.element.textContent = Math.floor(this.target * eased).toLocaleString();
            if (progress < 1) requestAnimationFrame(animate);
        };
        requestAnimationFrame(animate);
    }
}
```

## Background Effects

```css
/* Gradient Mesh — premium product page atmosphere */
.gradient-mesh {
    background:
        radial-gradient(ellipse at 20% 50%, rgba(246, 129, 40, 0.15) 0%, transparent 50%),
        radial-gradient(ellipse at 80% 20%, rgba(255, 174, 102, 0.1) 0%, transparent 50%),
        radial-gradient(ellipse at 50% 80%, rgba(67, 97, 238, 0.08) 0%, transparent 50%),
        var(--bg-primary);
}

/* Noise/Grain Texture — cinematic film look */
.noise-overlay::after {
    content: '';
    position: absolute;
    inset: 0;
    opacity: 0.03;
    pointer-events: none;
    background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E");
}

/* Grid Pattern — technical/blueprint feel */
.grid-bg {
    background-image:
        linear-gradient(rgba(255,255,255,0.03) 1px, transparent 1px),
        linear-gradient(90deg, rgba(255,255,255,0.03) 1px, transparent 1px);
    background-size: 60px 60px;
}

/* Animated Gradient Shift — hero backgrounds */
@keyframes gradient-shift {
    0%, 100% { background-position: 0% 50%; }
    50% { background-position: 100% 50%; }
}
.animated-gradient {
    background: linear-gradient(-45deg, #0a0a0a, #1a0533, #0a0f1c, #0d1117);
    background-size: 400% 400%;
    animation: gradient-shift 15s ease infinite;
}

/* Spotlight Effect — draws eye to center */
.spotlight {
    background: radial-gradient(
        ellipse at 50% 40%,
        rgba(255, 255, 255, 0.05) 0%,
        transparent 70%
    );
}
```

## Interactive Effects

```javascript
/* Magnetic Button — button follows cursor on hover */
class MagneticButton {
    constructor(button) {
        this.button = button;
        button.addEventListener('mousemove', (e) => {
            const rect = button.getBoundingClientRect();
            const x = e.clientX - rect.left - rect.width / 2;
            const y = e.clientY - rect.top - rect.height / 2;
            button.style.transform = `translate(${x * 0.3}px, ${y * 0.3}px)`;
        });
        button.addEventListener('mouseleave', () => {
            button.style.transform = 'translate(0, 0)';
            button.style.transition = 'transform 0.3s var(--ease-out-expo)';
        });
        button.addEventListener('mouseenter', () => {
            button.style.transition = 'none';
        });
    }
}

/* 3D Tilt Card — depth effect on hover */
class TiltCard {
    constructor(element) {
        this.el = element;
        this.el.style.transformStyle = 'preserve-3d';
        this.el.addEventListener('mousemove', (e) => {
            const rect = this.el.getBoundingClientRect();
            const x = (e.clientX - rect.left) / rect.width - 0.5;
            const y = (e.clientY - rect.top) / rect.height - 0.5;
            this.el.style.transform = `perspective(800px) rotateY(${x * 8}deg) rotateX(${-y * 8}deg)`;
        });
        this.el.addEventListener('mouseleave', () => {
            this.el.style.transform = 'perspective(800px) rotateY(0) rotateX(0)';
            this.el.style.transition = 'transform 0.5s var(--ease-out-expo)';
        });
        this.el.addEventListener('mouseenter', () => {
            this.el.style.transition = 'none';
        });
    }
}
```

## Navigation Effects

```css
/* Sticky Nav — transparent to solid on scroll */
.site-nav {
    position: fixed;
    top: 0; left: 0; right: 0;
    z-index: 1000;
    padding: clamp(0.75rem, 2vw, 1.25rem) clamp(1.5rem, 4vw, 4rem);
    display: flex;
    align-items: center;
    justify-content: space-between;
    background: transparent;
    transition: background 0.3s ease, backdrop-filter 0.3s ease;
}

.site-nav.scrolled {
    background: rgba(0, 0, 0, 0.85);
    backdrop-filter: blur(12px);
    -webkit-backdrop-filter: blur(12px);
}

/* Scroll Progress Bar */
.scroll-progress {
    position: fixed;
    top: 0; left: 0;
    height: 3px;
    background: var(--accent);
    z-index: 1001;
    width: 0%;
    will-change: width;
}
```

## Performance Patterns

```css
/* Content Visibility for off-screen sections */
.section:not(.hero-section) {
    content-visibility: auto;
    contain-intrinsic-size: 1px 800px;
}
```

```javascript
/* Throttled scroll handler pattern */
let ticking = false;
window.addEventListener('scroll', () => {
    if (!ticking) {
        requestAnimationFrame(() => {
            updateScrollEffects();
            ticking = false;
        });
        ticking = true;
    }
});

/* Lazy loading via Intersection Observer */
const lazyObserver = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
        if (entry.isIntersecting) {
            const el = entry.target;
            if (el.dataset.src) {
                el.src = el.dataset.src;
                el.removeAttribute('data-src');
            }
            if (el.dataset.bgSrc) {
                el.style.backgroundImage = `url(${el.dataset.bgSrc})`;
                el.removeAttribute('data-bg-src');
            }
            lazyObserver.unobserve(el);
        }
    });
}, { rootMargin: '200px' });

document.querySelectorAll('[data-src], [data-bg-src]').forEach(el => {
    lazyObserver.observe(el);
});
```

## Mobile Considerations

```css
/* Disable heavy effects on mobile */
@media (max-width: 768px) {
    .parallax-layer {
        transform: none !important;
    }

    .scroll-panel {
        min-height: 60vh;
    }

    .panel-content {
        max-width: 100%;
    }
}
```

```javascript
/* Detect mobile and skip heavy effects */
const isMobile = window.matchMedia('(max-width: 768px)').matches;
if (!isMobile) {
    new ParallaxController();
}
```

## Troubleshooting

| Problem | Fix |
| --- | --- |
| Sticky nav not appearing | Check `z-index`, verify scroll listener adds `.scrolled` class |
| Scrollytelling jank | Use `requestAnimationFrame`, avoid layout thrashing, use `transform` only |
| Counter not animating | Verify `data-target` attribute, check Intersection Observer threshold |
| Parallax feels wrong | Adjust speed values (0.1 = subtle, 0.5 = dramatic), disable on mobile |
| Video not playing | Ensure `muted`, `playsinline`, `autoplay` attributes; check mobile restrictions |
| Lazy images flashing | Add placeholder CSS, set `contain-intrinsic-size`, use `rootMargin: 200px` |
| Mobile performance | Disable parallax below 768px, reduce animation complexity, skip scroll-video |
| Scroll progress wrong | Ensure `docHeight = scrollHeight - innerHeight`, not just `scrollHeight` |
| Panels not activating | Check `data-progress` values, verify threshold range (default ±0.15) |
| Content-visibility CLS | Set `contain-intrinsic-size` to approximate section height |
