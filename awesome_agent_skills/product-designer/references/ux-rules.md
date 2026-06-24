# UX Rules Reference

Full 99-rule UX guidelines drawn from WCAG, Apple HIG, and Material Design.

## 1. Accessibility (CRITICAL)

- `color-contrast` — Minimum 4.5:1 for normal text; 3:1 for large text (≥18px bold or ≥24px regular)
- `focus-states` — Visible focus rings on all interactive elements (2–4px outline, never removed)
- `alt-text` — Descriptive alt for meaningful images; `alt=""` for decorative
- `aria-labels` — `aria-label` on icon-only buttons; `aria-describedby` for input hints
- `keyboard-nav` — Tab order matches visual order; Escape closes modals; Enter/Space activates buttons
- `form-labels` — Every input has a visible `<label>` with `for` attribute
- `skip-links` — "Skip to main content" link visible on keyboard focus
- `heading-hierarchy` — Sequential h1→h6, no level skip; one h1 per page
- `color-not-only` — Never convey meaning via color alone; add icon or text
- `dynamic-type` — Support system text scaling; test at 200% zoom without horizontal scroll
- `reduced-motion` — `@media (prefers-reduced-motion: reduce)` disables/reduces all animations
- `voiceover-sr` — Test with VoiceOver (iOS) and NVDA (Windows); logical reading order
- `escape-routes` — All modals and drawers have close affordance; Escape key works
- `keyboard-shortcuts` — Don't conflict with system shortcuts; document custom ones

## 2. Touch & Interaction (CRITICAL)

- `touch-target-size` — Min 44×44pt (iOS) / 48×48dp (Android); extend hit area with padding
- `touch-spacing` — Minimum 8px gap between adjacent touch targets
- `hover-vs-tap` — Primary actions on click/tap, never hover-only
- `loading-buttons` — Disable button + show spinner during async; prevent double-submit
- `error-feedback` — Error message adjacent to the field that caused it; red + icon
- `cursor-pointer` — `cursor: pointer` on all clickable elements (Web)
- `gesture-conflicts` — No horizontal swipe on primary scroll content
- `tap-delay` — `touch-action: manipulation` on interactive elements
- `press-feedback` — Visual state change within 100ms of tap (ripple, opacity, scale)
- `safe-area-awareness` — Content clear of notch, Dynamic Island, home indicator

## 3. Performance (HIGH)

- `image-optimization` — WebP/AVIF, `srcset`, `loading="lazy"` for below-fold
- `image-dimension` — Always declare `width`/`height` or `aspect-ratio` to prevent CLS
- `font-loading` — `font-display: swap`; preload only critical font weights
- `lazy-loading` — Route-level code splitting; `React.lazy` + `Suspense`
- `virtualize-lists` — 50+ items: use tanstack/virtual or react-window
- `progressive-loading` — Skeleton screens for data fetching > 300ms
- `debounce-throttle` — Debounce search inputs (300ms); throttle scroll handlers
- `bundle-splitting` — Separate vendor, UI library, and app code chunks
- `no-layout-thrashing` — Batch DOM reads before writes; avoid repeated reflows

## 4. Style & Visual (HIGH)

- `style-match` — Match style to product type and audience
- `consistency` — Same style system across all screens; no per-page exceptions
- `no-emoji-icons` — Use vector icon library (Lucide, Heroicons); no emoji as structural icons
- `elevation-consistent` — One shadow scale across cards, modals, dropdowns
- `state-clarity` — Hover/pressed/disabled states visually distinct but on-style
- `dark-mode-pairing` — Design light + dark together; test both before shipping
- `icon-style-consistent` — One icon set; consistent stroke width; filled OR outline per hierarchy level
- `primary-action` — One primary CTA per screen; secondary actions visually subordinate
- `blur-purpose` — Background blur = content behind is dismissable; not decorative

## 5. Layout & Responsive (HIGH)

- `viewport-meta` — `width=device-width, initial-scale=1`; never disable zoom
- `mobile-first` — Design at 375px, then scale up
- `breakpoints` — 375 / 768 / 1024 / 1440px
- `readable-font-size` — 16px minimum body on mobile (iOS auto-zoom threshold)
- `line-length-control` — Mobile 35–60 chars; desktop 60–75 chars
- `no-horizontal-scroll` — Zero tolerance on mobile
- `container-width` — `max-w-7xl mx-auto` for wide layouts; `max-w-2xl` for reading content
- `z-index-scale` — Define z-index tokens: base(0) / dropdown(10) / sticky(20) / modal(40) / toast(100)
- `viewport-units` — `min-h-dvh` over `100vh` on mobile (accounts for browser chrome)
- `content-priority` — Mobile: show only essential content first; progressive disclosure

## 6. Typography & Color (MEDIUM)

- `line-height` — Body: 1.5–1.75; headings: 1.1–1.3
- `font-scale` — Use defined scale: 12/14/16/18/20/24/32/40/48px
- `contrast-readability` — Body text minimum 4.5:1; use oklch/p3 for modern color spaces
- `color-semantic` — Semantic tokens only in components; never raw hex
- `color-dark-mode` — Desaturated tonal variants; test contrast independently from light mode
- `truncation-strategy` — Wrap > truncate; when truncating, provide tooltip with full text
- `number-tabular` — `font-variant-numeric: tabular-nums` for all data/prices/timers
- `whitespace-balance` — Whitespace groups related items; separates sections; avoids clutter

## 7. Animation (MEDIUM)

- `duration-timing` — Micro: 150–200ms; component: 200–300ms; page: 300–400ms
- `transform-performance` — Only animate `transform` and `opacity` (GPU-composited)
- `easing` — Enter: `ease-out`; exit: `ease-in`; natural: spring curves
- `motion-meaning` — Every animation encodes meaning; nothing purely decorative
- `exit-faster-than-enter` — Exit ~60–70% duration of enter
- `stagger-sequence` — List items stagger 30–50ms; not all-at-once
- `interruptible` — All animations cancellable by user interaction
- `scale-feedback` — Tappable cards: subtle scale 0.97–1.02 on press

## 8. Forms & Feedback (MEDIUM)

- `visible-labels` — Always visible; placeholder text supplements, never replaces label
- `error-near-field` — Error message directly below the field; `role="alert"`
- `helper-text` — Format hints before error; constraints visible at input time
- `progressive-disclosure` — Show complexity only when needed; don't overwhelm upfront
- `field-validation-timing` — Validate on blur, not on every keystroke (except format masks)
- `success-confirmation` — Explicit success state after form submit; not just "nothing broke"
- `autofill-support` — Correct `autocomplete` attributes on all standard fields
- `input-type` — Use correct `type` attribute (`email`, `tel`, `number`, `date`)

## 9. Navigation (HIGH)

- `predictable-back` — Back always goes to previous screen; no unexpected resets
- `bottom-nav-limit` — Maximum 5 items in bottom nav; top-level only
- `deep-linking` — Every screen has a shareable URL (web) or universal link (mobile)
- `state-preservation` — Back navigation restores scroll position, filters, inputs
- `modal-escape` — Clear close affordance + Escape key; swipe-down on mobile
- `search-accessible` — Search reachable within 1 tap from any screen
- `nav-consistency` — Navigation placement identical across all screens
- `adaptive-navigation` — ≥1024px: sidebar; <1024px: bottom nav or hamburger
- `destructive-nav-separation` — Delete/logout visually and spatially separated from primary nav

## 10. Charts & Data (LOW)

- `chart-type-match` — Trend: line; comparison: bar; proportion (≤5 categories): donut; distribution: histogram
- `accessible-colors` — Accessible palettes; never red/green-only for meaning
- `legend-visible` — Always shown; interactive (click to toggle series)
- `tooltip-on-interact` — Hover (web) / tap (mobile) shows exact values
- `empty-data-state` — Meaningful message + CTA; never blank axes
- `no-pie-overuse` — ≤5 categories for pie/donut; otherwise use bar chart
- `responsive-chart` — Simplify or reflow on mobile; fewer axis labels, horizontal bars
- `screen-reader-summary` — `aria-label` describing the chart's key insight
- `gridline-subtle` — Low contrast gridlines (gray-100/200); data is hero, not grid
