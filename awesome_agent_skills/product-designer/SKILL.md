---
name: product-designer
description: >
  Expert Product Designer: UI/UX code execution + user flow strategy. Use this skill for: building or designing any interface (landing page, dashboard, SaaS app, mobile, component); designing user flows, navigation architecture, or cross-screen journeys; reviewing and critiquing existing UI/UX for quality, consistency, or conversion issues; making design system decisions (colors, tokens, typography, spacing); working with React/Tailwind/shadcn/ui; vibecoding in Lovable; writing Lovable prompts for UI screens. ALWAYS trigger for: "design the flow", "map the user journey", "what screens do I need", "how does the user get from X to Y", "review my UI", "improve UX", "make this look better", "this feels off", "what's wrong with this design", "add dark mode", any mention of Pradio screens or features. Also trigger for: wireframe → code, component design, design system audit, accessibility review. Every UI decision has product impact — trigger broadly.
---

# Product Designer Skill

Full-spectrum product design: from strategic design decisions to pixel-perfect, production-grade UI code. Covers discovery, information architecture, visual design system, component implementation, and quality review.

**Primary stack**: React + Tailwind CSS + shadcn/ui (Pradio / Lovable projects)  
**Secondary stacks**: Next.js, Vue, HTML/CSS, React Native, Flutter

---

## 0. Design Process: When to Do What

Before touching code, choose the right mode:

| Situation | Mode | First Step |
|-----------|------|-----------|
| New product / new screen | **Discovery → Flow → Build** | Section 1 → 1.3 → 3 |
| "What screens do I need?" | **Flow Mapping** | Section 1.3 |
| User journey / navigation | **Flow Design** | Section 1.3 |
| Existing UI to improve | **Review → Decide → Fix** | Section 4 → 3 |
| Specific component request | **Component Design** | Section 3 |
| "This looks off" feedback | **Diagnosis** | Section 4 |
| Choosing style / color / font | **Design System** | Section 2 |

---

## 1. Discovery & Design Decisions (Strategic Layer)

Do this before building anything non-trivial.

### 1.1 — Understand the Context

Answer these before designing:
- **Who is the user?** (persona, tech level, emotional state at arrival)
- **What is the job to be done?** (primary action this screen/flow enables)
- **What does success look like?** (conversion, comprehension, task completion, trust)
- **What device context?** (mobile-first? dashboard on desktop? both?)
- **What brand signal does this need to emit?** (trust, innovation, warmth, authority)

### 1.2 — Information Architecture

Before layout, define:
- **Content inventory**: What must exist on this screen?
- **Priority ranking**: What must the user see/do first?
- **Navigation model**: Where does this screen fit in the flow? What comes before/after?
- **Empty/loading/error states**: Define all 5 states per component: default, loading, empty, error, success

### 1.3 — User Flow & Product Navigation

Use this when: designing a new product, planning screens for a feature, or auditing navigation coherence.

#### Flow Mapping Process

1. **Identify entry points** — where does the user enter? (direct URL, email CTA, onboarding, nav)
2. **Map the critical path** — the shortest path from entry to primary value delivery
3. **Identify decision points** — where does the user choose? (branch points in the flow)
4. **Define exits** — success exit, abandonment exit, error exit, and where they go
5. **Mark friction points** — where do users get confused, blocked, or lost?

#### Flow Documentation Format

```
FLOW: [Feature/Journey name]
Entry: [How user arrives]
Goal: [What user needs to accomplish]

Step 1: [Screen name]
  → Primary action: [CTA label]
  → Secondary action: [Optional path]
  → Exit/abandon: [What if user leaves?]

Step 2: [Screen name]
  → ...

Success: [Where user lands after completing goal]
Error path: [What happens on failure]
```

#### Screen Inventory for Common Pradio Flows

| Flow | Minimum Screens |
|------|----------------|
| Onboarding | Landing → Register → Email confirm → Profile setup → First bien |
| Ajouter un bien | Form multi-step: Adresse → Caractéristiques → Acquisition → Simulations |
| Simulation fiscale | Bien selection → Paramètres → Résultats → Export/Save |
| Dashboard principal | Vue globale → Détail bien → Simulation → Actions |

#### Navigation Architecture Principles

- **Max 3 levels deep** — if a user needs 4+ taps to reach a feature, restructure
- **One primary nav** — sidebar (desktop) or bottom nav (mobile), not both simultaneously
- **Persistent home** — always one tap back to dashboard from anywhere
- **Breadcrumbs** for 3+ level hierarchies on desktop
- **No dead ends** — every empty state has a CTA to the next action
- **Back = previous screen** always; never unexpected resets

#### Flow Review Checklist

- [ ] Every screen has one clear primary action
- [ ] No screen requires more than 3 taps to reach from home
- [ ] All error states have recovery paths (not just "go back")
- [ ] Onboarding flow ends at first value moment (not at account creation)
- [ ] Navigation is consistent: same menu in same position on every screen
- [ ] Confirmation dialogs used for destructive actions only (not for navigation)
- [ ] Deep links work for all primary screens (shareable URLs)

### 1.4 — Design Decisions (Log them)

For any non-trivial screen, document:
```
DESIGN DECISION LOG
Screen: [Name]
Primary job: [User's goal]
Comes from: [Previous screen/entry point]
Goes to: [Next screen on success]
Layout choice: [Why this structure]
Visual tone: [Style rationale]
Key trade-offs: [What was sacrificed for simplicity/performance/clarity]
```

---

## 2. Design System (Visual Foundation)

### 2.1 — Style Selection

Match style to product type. For **Pradio** (proptech/fintech SaaS): use **clean professional** — restrained color palette, clear hierarchy, trust-building whitespace, data-forward.

| Product Type | Recommended Style | Avoid |
|---|---|---|
| Fintech / PropTech SaaS | Clean minimal, data-forward | Glassmorphism overuse, dark brutalism |
| Consumer app | Warm, rounded, accessible | Cold corporate, dense text |
| Dashboard / analytics | Structured, contrast-led | Decorative animations, emoji icons |
| Landing page / marketing | Bold, asymmetric, expressive | Generic gradient purple, Inter everywhere |

**Core rule**: Commit to ONE style and execute it perfectly across every screen.

### 2.2 — Color System

Always use semantic tokens, never raw hex in components:

```css
/* Design tokens (Tailwind config or CSS vars) */
--color-brand:       hsl(220 70% 50%);   /* primary action */
--color-brand-hover: hsl(220 70% 45%);
--color-surface:     hsl(0 0% 100%);     /* card/panel bg */
--color-surface-muted: hsl(220 10% 97%); /* subtle bg */
--color-border:      hsl(220 10% 90%);   /* dividers */
--color-text:        hsl(220 15% 15%);   /* body */
--color-text-muted:  hsl(220 10% 50%);   /* secondary */
--color-error:       hsl(0 72% 51%);
--color-success:     hsl(145 60% 36%);
```

**Rules:**
- Minimum 4.5:1 contrast for body text (WCAG AA)
- Dark mode: use tonal variants, never invert
- Never use color as the only meaning indicator (add icon/text)
- For data visualization: use accessible palettes (avoid red/green-only pairs)

### 2.3 — Typography System

```
Scale: 12 / 14 / 16 / 18 / 20 / 24 / 32 / 40 / 48
Body: 16px minimum, line-height 1.5–1.75
Line length: 60–72 chars for prose, unconstrained for data
Weight: 400 body / 500 labels / 600–700 headings
```

**Font pairing by product type:**
- Pradio/SaaS: `Inter` (body) + `Cal Sans` or `DM Sans` (headings) — or Geist
- Marketing landing: Pick one distinctive display font (never Inter for headings on landing pages)
- Data dashboard: Mono for numbers (`tabular-nums`), clean sans for labels

### 2.4 — Spacing System

Use 4px base grid (Tailwind: `p-1` = 4px):

| Token | Value | Use |
|---|---|---|
| `space-1` | 4px | Icon inner padding |
| `space-2` | 8px | Tight component spacing |
| `space-3` | 12px | Element gaps |
| `space-4` | 16px | Component padding |
| `space-6` | 24px | Section spacing |
| `space-8` | 32px | Section separation |
| `space-12` | 48px | Major layout sections |

**Touch targets**: min 44×44px (iOS) / 48×48px (Android)

---

## 3. Component Implementation

### 3.1 — shadcn/ui (Primary Component Layer)

For Pradio and Lovable projects, always prefer shadcn/ui primitives. Never reimplement what shadcn provides.

```tsx
// ✅ Correct: use shadcn primitive + extend with Tailwind
import { Button } from "@/components/ui/button"

<Button variant="default" className="gap-2 font-medium">
  <Plus className="h-4 w-4" />
  Ajouter un bien
</Button>

// ❌ Wrong: custom button from scratch when shadcn covers it
<button className="bg-blue-600 text-white px-4 py-2 rounded">
```

**Extend shadcn components via `className` and `cva` variants — never fork the source.**

### 3.2 — Component API Design

Every component should have:
- `variant` prop (visual style: default, destructive, ghost, outline)
- `size` prop (sm, md, lg)
- `isLoading` state (disable + spinner)
- `disabled` state (visual + no interaction)
- Forwarded `ref` for form libraries
- TypeScript props interface

### 3.3 — Tailwind Patterns

```tsx
// Class composition pattern (use cn utility)
import { cn } from "@/lib/utils"

const cardClass = cn(
  "rounded-xl border border-border bg-surface p-6",
  "transition-shadow hover:shadow-md",
  isSelected && "border-brand ring-2 ring-brand/20",
  className
)
```

**Tailwind rules:**
- Use `max-w-7xl mx-auto px-4 sm:px-6 lg:px-8` for page containers
- Use `grid` over `flex` for 2D layouts
- Always include `dark:` variants for new color assignments
- Use `group-hover:` for parent-triggered child animations

### 3.4 — State Hierarchy

Every interactive component needs all states:
```
default → hover → focus → active → loading → disabled → error → success
```

Don't ship a component without defining all 8 states visually.

### 3.5 — React Performance Rules

- `useMemo` for expensive computations, never for everything
- `React.memo` for list items rendered 10+ times
- Lazy load heavy components: `const Chart = lazy(() => import('./Chart'))`
- Virtualize lists over 50 items (tanstack/virtual)
- No inline functions in JSX for performance-critical paths

---

## 4. UI Review & Critique Framework

Two modes: **Quick Audit** (spot issues fast) or **Full Critique** (structured diagnosis).

### 4.0 — Quick Audit (< 5 min)

Look at the screen and answer:
1. What's the ONE thing the user is supposed to do here? Is it obvious in < 3 seconds?
2. What's visually dominant? Should it be?
3. What would a first-time user misunderstand?
4. What's the most friction-heavy step in this flow?

If any answer is unclear or negative → go to Full Critique.

### 4.1 — Full Critique: Priority Checklist

#### Priority 1 — Critical (block delivery)
- [ ] Color contrast ≥ 4.5:1 for body text
- [ ] Keyboard navigability (Tab order logical, focus rings visible)
- [ ] Touch targets ≥ 44px on mobile
- [ ] No horizontal scroll on mobile
- [ ] All interactive elements have loading + error states
- [ ] Primary action is visually unambiguous (1 clear CTA per screen)

#### Priority 2 — High (fix before launch)
- [ ] Consistent spacing (no arbitrary px values outside scale)
- [ ] All states defined: empty, loading, error, success
- [ ] No raw hex colors in components (use tokens)
- [ ] Typography scale consistent (no rogue font sizes)
- [ ] Icons from one family, consistent stroke width
- [ ] Navigation placement identical across all screens
- [ ] Flow: no dead ends — every empty state has a next action

#### Priority 3 — Medium (next iteration)
- [ ] Animation durations 150–300ms, transform/opacity only
- [ ] Dark mode tokens mapped (not inverted)
- [ ] Line length controlled (≤ 72 chars prose)
- [ ] Responsive tested at 375 / 768 / 1280 / 1440px
- [ ] Tabular numbers for data/prices
- [ ] Back navigation restores scroll position + filter state

#### Priority 4 — Polish
- [ ] Spring/easing (not linear) for animations
- [ ] Skeleton screens for data fetches > 300ms
- [ ] Micro-interactions on key actions (save, delete, confirm)
- [ ] Empty states have illustration + clear CTA

### 4.2 — Critique Output Format

When asked to review/critique UI, always output:

```
## UI Critique: [Screen/Component name]

### 🔴 Critical Issues (fix now)
- [Issue]: [Why it matters] → [Fix]

### 🟠 High Priority (fix before launch)  
- [Issue]: [Why it matters] → [Fix]

### 🟡 Medium (next sprint)
- [Issue] → [Fix]

### ✅ What's working well
- [Specific callout]

### Flow Assessment
- Entry: [Where user comes from] — clear? Y/N
- Primary action: [What it is] — obvious? Y/N
- Exit: [Where user goes next] — defined? Y/N
```

---

## 5. Lovable-Specific Guidance

When building with Lovable (vibecoding), apply these rules:

### 5.1 — Prompt Engineering for Lovable

Structure prompts as:
```
[Component/Screen name]
Stack: React + Tailwind + shadcn/ui
Variant: [style choice]
Data: [describe props/data shape]
States: [list all states needed]
Do NOT: [common AI mistakes to avoid]
```

### 5.2 — Common Lovable Pitfalls to Avoid

- Lovable often uses `useState` where `useQuery` (tanstack) would be correct — always specify data fetching approach
- It generates inline styles instead of Tailwind classes — specify "Tailwind only, no inline styles"
- It defaults to custom components instead of shadcn — specify "use shadcn/ui primitives"
- It ignores loading/error states — always mention them explicitly
- It uses emoji as icons — specify "use Lucide icons only"

### 5.3 — Pradio Design Tokens

For Pradio specifically, always reference the established design system:
- Brand color: professional trust blue
- Surface: clean white / very light gray
- Data emphasis: use tabular numbers, clear hierarchy
- French locale: use `Intl.NumberFormat('fr-FR')` for currency/numbers

---

## 6. Quick Decision Reference

### When to use which layout pattern

| Need | Pattern | Tailwind |
|------|---------|---------|
| Page layout | Sidebar + main | `grid grid-cols-[240px_1fr]` |
| Card grid | Auto-fill | `grid grid-cols-[repeat(auto-fill,minmax(280px,1fr))]` |
| Form layout | Stack | `flex flex-col gap-4` |
| Action bar | Space between | `flex items-center justify-between` |
| Centered content | Container | `max-w-2xl mx-auto` |
| Dashboard stats | 4-col | `grid grid-cols-2 md:grid-cols-4 gap-4` |

### When to use which component

| Use case | shadcn Component |
|----------|-----------------|
| User input | `Input`, `Textarea`, `Select` |
| Confirmation | `AlertDialog` (not `Dialog`) |
| Non-blocking info | `Toast` (Sonner) |
| Settings / filters | `Sheet` (slide-over) |
| Complex data | `DataTable` (tanstack) |
| Navigation | `NavigationMenu` or `Tabs` |
| Status | `Badge` with semantic variant |

---

## 7. Accessibility Baseline

Non-negotiable. Ship nothing without:

```tsx
// Images
<img src={...} alt="Description de l'image" />
// Icônes seules
<Button aria-label="Supprimer la propriété">
  <Trash2 className="h-4 w-4" aria-hidden="true" />
</Button>
// États de formulaire
<Input
  id="price"
  aria-describedby="price-error"
  aria-invalid={!!errors.price}
/>
<p id="price-error" role="alert">{errors.price}</p>
```

- Skip links for keyboard users (`sr-only` until focused)
- `prefers-reduced-motion` respected for all animations
- `prefers-color-scheme` for dark mode
- Form labels always visible (never placeholder-only)

---

## References

- `references/ux-rules.md` — Full 99-rule UX guidelines (accessibility, animation, forms, charts)  
- `references/color-palettes.md` — 161 color palettes by product type  
- `references/font-pairings.md` — 57 curated font pairings  
- `references/chart-patterns.md` — 25 chart types with usage rules  

> Load these files when deep-diving a specific domain. The SKILL.md above covers 80% of cases.
