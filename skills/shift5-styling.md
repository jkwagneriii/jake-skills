---
name: shift5-styling
description: Apply Shift5 design patterns — grid-bordered bento layouts, sticky clip-path scroll reveals, staggered entrance animations, fluid typography, film grain textures, section theming, image reveals, navbar overlays, and button treatments — to any project using CSS custom properties as a fusion layer.
argument-hint: "[optional: specific pattern to apply, e.g. 'bento grid', 'sticky reveal', 'scroll animations']"
---

# Shift5 Styling System

A portable design language for bold, editorial-grade interfaces. Every pattern uses `--s5-*` CSS custom properties so it fuses with any existing design system without overriding it.

## When to Use

Invoke this skill when the user asks to:
- Apply "Shift5 styling", "bento grid", "grid borders", or "editorial layout"
- Add scroll-triggered entrance animations or staggered reveals
- Create sticky scroll sections with clip-path reveals
- Apply film grain, grayscale media, or textural overlays
- Build a full-screen overlay navbar with scroll detection
- Set up fluid typography with clamp()
- Apply section theming (light/dark/accent alternation)

---

## 1 · Fusion Protocol — Mapping `--s5-*` to Any Design System

Before applying any pattern, create a CSS custom property bridge. This is the **only file you need to customize** — all patterns below reference these variables.

```css
/* Add to your global CSS (e.g. index.css, globals.css) */
:root {
  /* ── Surface / Background ─────────────────────────── */
  --s5-surface:        #DBD4C9;   /* Map to your bg color */
  --s5-surface-light:  #D0AC9C;   /* Map to your secondary bg */
  --s5-surface-dark:   #2A2A22;   /* Map to your dark bg */

  /* ── Text ─────────────────────────────────────────── */
  --s5-ink:            #2A2A22;   /* Map to your primary text */
  --s5-ink-muted:      #5C5647;   /* Map to your secondary text */
  --s5-ink-faint:      #8A8374;   /* Map to your tertiary text */

  /* ── Accent ───────────────────────────────────────── */
  --s5-accent:         #455602;   /* Map to your primary brand color */
  --s5-accent-soft:    #B39B75;   /* Map to your secondary accent */

  /* ── Borders ──────────────────────────────────────── */
  --s5-border:         #2A2A22;
  --s5-border-light:   rgba(255,255,255,0.1);

  /* ── Fonts ────────────────────────────────────────── */
  --s5-font-sans:      'Inter', system-ui, sans-serif;
  --s5-font-mono:      'JetBrains Mono', ui-monospace, monospace;
}
```

**How to fuse:** Replace each `--s5-*` value with the project's existing design tokens. For Tailwind projects, also extend `tailwind.config.js` (see Section 9). For vanilla CSS or other frameworks, reference the variables directly.

---

## 2 · Grid-Bordered Bento System

The signature layout — a CSS grid where borders are applied via parent/child selectors so cells share a single 1px border line.

### CSS

```css
/* Light theme (default) */
.grid-bordered {
  border-top: 1px solid var(--s5-border);
  border-left: 1px solid var(--s5-border);
}
.grid-bordered > * {
  border-bottom: 1px solid var(--s5-border);
  border-right: 1px solid var(--s5-border);
}

/* Dark theme variant */
.grid-bordered-dark {
  border-top: 1px solid var(--s5-border-light);
  border-left: 1px solid var(--s5-border-light);
}
.grid-bordered-dark > * {
  border-bottom: 1px solid var(--s5-border-light);
  border-right: 1px solid var(--s5-border-light);
}
```

### Usage — Equal Columns

```html
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 grid-bordered">
  <div class="p-8">Cell 1</div>
  <div class="p-8">Cell 2</div>
  <div class="p-8">Cell 3</div>
  <div class="p-8">Cell 4</div>
  <div class="p-8">Cell 5</div>
  <div class="p-8">Cell 6</div>
</div>
```

### Usage — 12-Column Asymmetric Split

```html
<div class="grid grid-cols-12 grid-bordered">
  <div class="col-span-12 md:col-span-5 p-8">Narrow side</div>
  <div class="col-span-12 md:col-span-7 p-8">Wide side</div>
</div>
```

---

## 3 · Cell Hover Interaction

Cells that transition background + text color on hover. Works standalone or inside `.grid-bordered`.

### CSS

```css
.cell-hover {
  transition: background-color 0.3s cubic-bezier(0.22,1,0.36,1),
              color 0.3s cubic-bezier(0.22,1,0.36,1);
  cursor: pointer;
}
.cell-hover:hover {
  background: var(--s5-accent);
  color: white;
  box-shadow: inset 0 0 0 1px color-mix(in srgb, var(--s5-accent) 30%, transparent);
}
```

### With Tailwind `group`/`group-hover` (for child element transforms)

```jsx
<div className="cell-hover group p-8">
  <div className="group-hover:scale-105 transition-transform duration-500 ease-out">
    <Icon />
  </div>
  <h3 className="text-title">Card Title</h3>
</div>
```

---

## 4 · Scroll Animation System

Two paths: **CSS + IntersectionObserver** (zero dependencies) and **Framer Motion** (richer control).

### Path A — CSS + IntersectionObserver (Zero Dependencies)

#### CSS Classes

```css
/* Base entrance animation */
.animate-on-scroll {
  opacity: 0;
  transform: translateY(30px);
  transition: opacity 0.8s cubic-bezier(0.16,1,0.3,1),
              transform 0.8s cubic-bezier(0.16,1,0.3,1);
}
.animate-on-scroll.is-visible {
  opacity: 1;
  transform: translateY(0);
}

/* Stagger delays for sibling elements */
.stagger-1 { transition-delay: 0.06s; }
.stagger-2 { transition-delay: 0.12s; }
.stagger-3 { transition-delay: 0.18s; }
.stagger-4 { transition-delay: 0.24s; }
.stagger-5 { transition-delay: 0.30s; }
.stagger-6 { transition-delay: 0.36s; }
```

#### React Hook — `useScrollReveal`

```js
import { useEffect, useRef } from 'react'

// Single element reveal
export function useScrollReveal(threshold = 0.1) {
  const ref = useRef(null)

  useEffect(() => {
    const el = ref.current
    if (!el) return

    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          entry.target.classList.add('is-visible')
          observer.unobserve(entry.target)
        }
      },
      { threshold, rootMargin: '0px 0px -40px 0px' }
    )

    observer.observe(el)
    return () => observer.disconnect()
  }, [threshold])

  return ref
}

// Bulk reveal — observes all .animate-on-scroll, .img-reveal-left, .img-reveal-center
export function useScrollRevealAll() {
  useEffect(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            entry.target.classList.add('is-visible')
            observer.unobserve(entry.target)
          }
        })
      },
      { threshold: 0.1, rootMargin: '0px 0px -40px 0px' }
    )

    document
      .querySelectorAll('.animate-on-scroll, .img-reveal-left, .img-reveal-center')
      .forEach((el) => observer.observe(el))
    return () => observer.disconnect()
  }, [])
}
```

#### HTML Usage

```html
<h2 class="animate-on-scroll">Heading</h2>
<p class="animate-on-scroll stagger-1">First paragraph</p>
<p class="animate-on-scroll stagger-2">Second paragraph</p>
```

### Path B — Framer Motion

#### Variants

```js
const containerVariants = {
  hidden: {},
  visible: { transition: { staggerChildren: 0.1 } },
}

const cardVariants = {
  hidden: { opacity: 0, y: 60 },
  visible: {
    opacity: 1,
    y: 0,
    transition: { duration: 0.6, ease: [0.16, 1, 0.3, 1] },
  },
}
```

#### Component Usage

```jsx
import { motion } from 'framer-motion'

<motion.div
  className="grid grid-cols-1 md:grid-cols-3 grid-bordered"
  variants={containerVariants}
  initial="hidden"
  whileInView="visible"
  viewport={{ once: true, margin: '-100px' }}
>
  {items.map((item) => (
    <motion.div key={item.id} variants={cardVariants} className="p-8">
      {item.content}
    </motion.div>
  ))}
</motion.div>
```

---

## 5 · Image Reveal Animations

Clip-path animations that reveal images on scroll.

### CSS

```css
/* Center-expand reveal */
.img-reveal-center {
  clip-path: inset(50% 50% 50% 50%);
  transition: clip-path 0.8s cubic-bezier(0.16, 1, 0.3, 1);
}
.img-reveal-center.is-visible {
  clip-path: inset(0);
}

/* Left-wipe reveal */
.img-reveal-left {
  clip-path: inset(0 100% 0 0);
  transition: clip-path 0.8s cubic-bezier(0.16, 1, 0.3, 1);
}
.img-reveal-left.is-visible {
  clip-path: inset(0);
}
```

### Usage

```html
<img src="hero.jpg" class="img-reveal-center" alt="..." />
<img src="side.jpg" class="img-reveal-left stagger-2" alt="..." />
```

These work automatically with `useScrollRevealAll()` — no extra wiring needed.

---

## 6 · Sticky Scroll Section

A section that sticks to the viewport while a clip-path reveal animates based on scroll position. Uses Framer Motion's `useScroll` + `useTransform`.

### Hook — `useStickyClipReveal`

```js
import { useScroll, useTransform } from 'framer-motion'

export function useStickyClipReveal(ref) {
  const { scrollYProgress } = useScroll({
    target: ref,
    offset: ['start end', 'start start'],
  })

  const clipPath = useTransform(
    scrollYProgress,
    [0, 1],
    ['inset(100% 0 0 0)', 'inset(0% 0 0 0)']
  )

  return { clipPath }
}
```

### Component — `StickySection`

```jsx
import { useRef } from 'react'
import { motion } from 'framer-motion'
import { useStickyClipReveal } from '../hooks/useStickyClipReveal'

export default function StickySection({ children, zIndex = 10, className = '' }) {
  const ref = useRef(null)
  const { clipPath } = useStickyClipReveal(ref)

  return (
    <div ref={ref} className="min-h-[200vh]" style={{ zIndex, position: 'relative' }}>
      <motion.section
        className={`sticky-section sticky top-0 h-screen overflow-hidden ${className}`}
        style={{ clipPath, willChange: 'clip-path' }}
      >
        {children}
      </motion.section>
    </div>
  )
}
```

### Usage — Stacking Sections

```jsx
<StickySection zIndex={10} className="bg-[var(--s5-surface)]">
  <div className="h-screen flex items-center justify-center">
    <h2 className="text-hero">First Section</h2>
  </div>
</StickySection>

<StickySection zIndex={20} className="bg-[var(--s5-surface-dark)] text-white">
  <div className="h-screen flex items-center justify-center">
    <h2 className="text-hero">Second Section (reveals over first)</h2>
  </div>
</StickySection>
```

**Key mechanics:**
- `min-h-[200vh]` creates scroll distance for the animation
- `sticky top-0` pins the inner section
- Increasing `zIndex` ensures later sections stack on top
- `clipPath` animates from `inset(100% 0 0 0)` (hidden) to `inset(0)` (fully visible)

---

## 7 · Fluid Typography Scale

Bundled `clamp()` sizes with lineHeight, letterSpacing, and fontWeight. Add to Tailwind config or use as CSS variables.

### Tailwind Config

```js
// tailwind.config.js → theme.extend.fontSize
fontSize: {
  'mega':      ['clamp(4rem, 12vw, 12rem)',    { lineHeight: '0.95',  letterSpacing: '-0.05em', fontWeight: '800' }],
  'hero':      ['clamp(2.5rem, 6vw, 5.5rem)',  { lineHeight: '0.9',   letterSpacing: '-0.04em', fontWeight: '700' }],
  'display':   ['clamp(2rem, 3.5vw, 3rem)',    { lineHeight: '1.1',   letterSpacing: '-0.025em', fontWeight: '700' }],
  'title':     ['clamp(1.25rem, 2vw, 1.75rem)',{ lineHeight: '1.15',  letterSpacing: '-0.02em', fontWeight: '600' }],
  'body':      ['1rem',                         { lineHeight: '1.6',   fontWeight: '400' }],
  'mono-body': ['0.6875rem',                    { lineHeight: '1.6',   letterSpacing: '0.04em', fontWeight: '500' }],
  'mono-sm':   ['0.625rem',                     { lineHeight: '1.5',   letterSpacing: '0.05em', fontWeight: '500' }],
  'label':     ['0.6875rem',                    { lineHeight: '1.33',  letterSpacing: '0.08em', fontWeight: '600' }],
},
```

### CSS Custom Properties (non-Tailwind)

```css
:root {
  --s5-text-mega:    clamp(4rem, 12vw, 12rem);
  --s5-text-hero:    clamp(2.5rem, 6vw, 5.5rem);
  --s5-text-display: clamp(2rem, 3.5vw, 3rem);
  --s5-text-title:   clamp(1.25rem, 2vw, 1.75rem);
}
```

### Dual-Font Strategy

- **Sans** (`--s5-font-sans`): Headlines, body text — tight tracking, high contrast weights
- **Mono** (`--s5-font-mono`): Labels, buttons, metadata — uppercase, wide tracking

```css
.mono-upper {
  font-family: var(--s5-font-mono);
  font-size: 0.6875rem;
  line-height: 1.6;
  letter-spacing: 0.04em;
  font-weight: 500;
  text-transform: uppercase;
}
```

---

## 8 · Section Theming

Alternate section backgrounds to create visual rhythm. Use border separators between sections.

### Pattern

```html
<!-- Light section (default) -->
<section class="bg-[var(--s5-surface)] text-[var(--s5-ink)]">
  <div class="px-8 md:px-12 lg:px-16 py-[clamp(4rem,8vw,8rem)]">
    ...
  </div>
</section>

<!-- Dark section -->
<section class="bg-[var(--s5-surface-dark)] text-[var(--s5-surface)]">
  <div class="px-8 md:px-12 lg:px-16 py-[clamp(4rem,8vw,8rem)]">
    ...
  </div>
</section>

<!-- Accent section -->
<section class="bg-[var(--s5-accent)] text-white">
  <div class="px-8 md:px-12 lg:px-16 py-[clamp(4rem,8vw,8rem)]">
    ...
  </div>
</section>
```

### Border Separators

```html
<section class="border-t border-[var(--s5-border)]">
  ...
</section>
```

### Section Spacing (Tailwind)

```js
// tailwind.config.js → theme.extend.spacing
spacing: {
  'section-y': 'clamp(4rem, 8vw, 8rem)',
  'section-x': 'clamp(1.25rem, 4vw, 4rem)',
},
```

---

## 9 · Special Effects

### Film Grain Overlay

Adds subtle texture over any element. Non-interactive (pointer-events: none).

```css
.film-grain {
  position: relative;
}
.film-grain::after {
  content: '';
  position: absolute;
  inset: 0;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noise)' opacity='0.4'/%3E%3C/svg%3E");
  opacity: 0.05;
  pointer-events: none;
  z-index: 1;
}
```

### Grayscale Media

```css
.grayscale-img {
  filter: grayscale(100%) contrast(120%) brightness(0.95);
}
```

### Falling Particles (Framer Motion)

A React component for animated particle rain. Configurable colors, speed, density.

```jsx
import { motion } from 'framer-motion'

export function FallingPattern({
  color = 'var(--s5-surface)',
  backgroundColor = 'var(--s5-accent)',
  duration = 150,
  blurIntensity = '1em',
  density = 1,
  className = '',
}) {
  // Uses radial-gradient patterns animated via backgroundPosition
  // The pattern creates rain-like falling dots with a perforated overlay
  const patterns = [
    `radial-gradient(4px 100px at 0px 235px, ${color}, transparent)`,
    `radial-gradient(1.5px 1.5px at 150px 117.5px, ${color} 100%, transparent 150%)`,
    // ... repeat with varying positions for natural randomness
  ]

  return (
    <div className={`relative h-full w-full p-1 ${className}`}>
      <motion.div initial={{ opacity: 0 }} animate={{ opacity: 1 }} transition={{ duration: 0.2 }} className="w-full h-full">
        <motion.div
          className="relative w-full h-full z-0"
          style={{
            backgroundColor,
            backgroundImage: patterns.join(', '),
            backgroundSize: '300px 235px, 300px 235px, ...',
          }}
          animate={{
            backgroundPosition: ['startPositions', 'endPositions'],
            transition: { duration, ease: 'linear', repeat: Infinity },
          }}
        />
      </motion.div>
      {/* Perforated overlay */}
      <div
        className="absolute inset-0 z-[1]"
        style={{
          backdropFilter: `blur(${blurIntensity})`,
          backgroundImage: `radial-gradient(circle at 50% 50%, transparent 0, transparent 2px, ${backgroundColor} 2px)`,
          backgroundSize: `${8 * density}px ${8 * density}px`,
        }}
      />
    </div>
  )
}
```

### Watermark Text

Large, faded text behind content for editorial depth.

```html
<div class="relative">
  <span class="absolute top-0 left-0 text-mega opacity-5 select-none pointer-events-none" aria-hidden="true">
    SHIFT5
  </span>
  <div class="relative z-10">Actual content here</div>
</div>
```

---

## 10 · Button Patterns

### Primary (Solid)

```html
<button class="inline-flex items-center gap-3 px-8 py-4 bg-[var(--s5-accent)] text-white font-mono text-[0.6875rem] uppercase tracking-widest hover:opacity-90 transition-opacity duration-300 cursor-pointer">
  Get Started
</button>
```

### Secondary (Outline)

```html
<button class="inline-flex items-center gap-2 px-5 py-2.5 font-mono text-[0.625rem] uppercase tracking-widest border border-current cursor-pointer hover:opacity-70 transition-opacity duration-300">
  Explore
</button>
```

### Key Characteristics
- `font-mono` + `uppercase` + `tracking-widest` — the Shift5 signature
- `border-radius: 0` (sharp corners, no rounding)
- No box-shadow — clean flat treatment
- Hover via opacity shift, not color change

---

## 11 · Navbar Pattern

Scroll-aware navbar: transparent at top → opaque on scroll. Hamburger → X morph. Full-screen overlay menu.

### Structure

```jsx
import { useState, useEffect } from 'react'

export default function Navbar({ links }) {
  const [open, setOpen] = useState(false)
  const [scrolled, setScrolled] = useState(false)

  useEffect(() => {
    const onScroll = () => setScrolled(window.scrollY > 50)
    window.addEventListener('scroll', onScroll, { passive: true })
    return () => window.removeEventListener('scroll', onScroll)
  }, [])

  return (
    <>
      {/* Fixed nav bar */}
      <nav className={`fixed top-0 left-0 right-0 z-50 transition-all duration-300 ${
        scrolled && !open ? 'bg-[var(--s5-surface-dark)]/85 backdrop-blur-md' : ''
      }`}>
        <div className="flex items-center justify-between px-8 md:px-12 lg:px-16 pt-6 pb-4">
          {/* Logo slot */}
          <a href="#" className="flex items-center gap-2">
            {/* Your logo here */}
          </a>

          {/* Hamburger button — morphs to X */}
          <button onClick={() => setOpen(!open)} className="flex items-center gap-3 cursor-pointer" aria-label="Toggle menu">
            <span className="flex flex-col gap-[5px]">
              <span className={`block w-6 h-[2px] transition-all duration-300 origin-center ${
                scrolled && !open ? 'bg-[var(--s5-surface)]' : 'bg-[var(--s5-ink)]'
              } ${open ? 'rotate-45 translate-y-[7px] !bg-[var(--s5-surface)]' : ''}`} />
              <span className={`block w-6 h-[2px] transition-all duration-300 ${
                scrolled && !open ? 'bg-[var(--s5-surface)]' : 'bg-[var(--s5-ink)]'
              } ${open ? 'opacity-0' : ''}`} />
              <span className={`block w-6 h-[2px] transition-all duration-300 origin-center ${
                scrolled && !open ? 'bg-[var(--s5-surface)]' : 'bg-[var(--s5-ink)]'
              } ${open ? '-rotate-45 -translate-y-[7px] !bg-[var(--s5-surface)]' : ''}`} />
            </span>
            <span className={`font-mono text-[0.625rem] uppercase tracking-widest hidden sm:block transition-colors duration-300 ${
              scrolled && !open ? 'text-[var(--s5-surface)]' : 'text-[var(--s5-ink)]'
            } ${open ? '!text-[var(--s5-surface)]' : ''}`}>Menu</span>
          </button>
        </div>
      </nav>

      {/* Full-screen overlay menu */}
      <div className={`fixed inset-0 z-40 bg-[var(--s5-surface-dark)] transition-all duration-500 flex flex-col justify-center ${
        open ? 'opacity-100 visible' : 'opacity-0 invisible'
      }`}>
        <div className="px-8 md:px-12 lg:px-16">
          <div className="flex flex-col gap-4">
            {links.map((link, i) => (
              <a
                key={link.label}
                href={link.href}
                onClick={() => setOpen(false)}
                className="text-hero text-white hover:text-[var(--s5-accent)] transition-colors duration-300 cursor-pointer tracking-tighter"
                style={{ animationDelay: `${i * 0.05}s` }}
              >
                {link.label}
              </a>
            ))}
          </div>
        </div>
      </div>
    </>
  )
}
```

---

## 12 · Accessibility

Always include reduced-motion handling:

```css
@media (prefers-reduced-motion: reduce) {
  .animate-on-scroll {
    opacity: 1;
    transform: none;
    transition: none;
  }
  * {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
  .sticky-section {
    position: relative !important;
  }
  .sticky-section [style*="clip-path"] {
    clip-path: none !important;
  }
  .img-reveal-center,
  .img-reveal-left {
    clip-path: none !important;
  }
}
```

---

## 13 · Pre-Build Checklist

Before implementing, verify:

- [ ] `--s5-*` custom properties are defined and mapped to the project's design tokens
- [ ] Fonts loaded (Inter + JetBrains Mono, or project equivalents)
- [ ] `prefers-reduced-motion` CSS block is included
- [ ] `overflow-x: clip` on `<body>` to prevent horizontal scroll from animations
- [ ] If using Framer Motion: `framer-motion` is installed
- [ ] If using sticky sections: test scroll behavior on mobile (iOS Safari can be tricky with `position: sticky`)
- [ ] `scroll-behavior: smooth` on `<html>` for anchor links

---

## 14 · Quick Reference

| Pattern | Class / Approach | Section |
|---|---|---|
| Grid borders (light) | `.grid-bordered` | §2 |
| Grid borders (dark) | `.grid-bordered-dark` | §2 |
| Cell hover | `.cell-hover` | §3 |
| Scroll fade-up | `.animate-on-scroll` + `.stagger-N` | §4 |
| Framer stagger | `containerVariants` + `cardVariants` | §4 |
| Image reveal (center) | `.img-reveal-center` | §5 |
| Image reveal (left) | `.img-reveal-left` | §5 |
| Sticky clip reveal | `<StickySection>` | §6 |
| Fluid type | `text-mega`, `text-hero`, `text-display`, `text-title` | §7 |
| Mono label style | `.mono-upper` | §7 |
| Film grain | `.film-grain` | §9 |
| Grayscale media | `.grayscale-img` | §9 |
| Falling particles | `<FallingPattern>` | §9 |
| Button (solid) | Primary pattern | §10 |
| Button (outline) | Secondary pattern | §10 |
| Navbar overlay | `<Navbar>` | §11 |
| Reduced motion | `@media (prefers-reduced-motion)` | §12 |
