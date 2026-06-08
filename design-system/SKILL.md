---
name: design-system
description: Establish and document the foundational design tokens and component patterns for a project
disable-model-invocation: true
---

Establish the foundational design system for a project. Produces a design reference document that the `/design` skill and all future component work should follow.

This skill is meant to be run **once per project** (or when the design system needs recalibrating). It creates or updates a design reference file in the project's docs.

## Before you start

1. **Audit the current state** - read the Tailwind config (or CSS variables), global stylesheet, and 5-10 key components. Catalogue what already exists: colours, font sizes, spacing values, border styles, shadows, radii.
2. **Read the product docs** - read `PRODUCT.md`, `CLAUDE.md`, and any existing design/convention docs. Understand the product domain and audience.
3. **Check for a tone doc** - if `.claude/docs/tone.md` (or equivalent) exists, read it. The design system should feel like the same brand as the voice.
4. **Identify inconsistencies** - find places where the codebase uses different values for the same concept (e.g. three different greys for muted text, two different border radii for cards). These are the problems the design system solves.

## What to produce

Present a **draft design system** to the user for approval. Cover these areas:

### 1. Colour palette

Define the full palette with clear roles:

| Role | Token name | Value | Usage |
|------|-----------|-------|-------|
| Background | `bg-primary` | ... | Main page background |
| Surface | `bg-surface` | ... | Cards, panels, elevated content |
| Text primary | `text-primary` | ... | Body text, headings |
| Text secondary | `text-muted` | ... | Supporting text, labels |
| Accent | `accent` | ... | CTAs, links, active states |
| Accent hover | `accent-hover` | ... | Hover state for accent |
| Border | `border-default` | ... | Dividers, card borders |
| Success | `success` | ... | Confirmations, positive states |
| Error | `error` | ... | Errors, destructive actions |
| Warning | `warning` | ... | Caution states |

Provide both light and dark mode values. Explain the reasoning behind the palette - what mood does it create? What domain reference does it draw from?

**Palette rules:**
- No more than 2 accent colours. Restraint creates identity.
- Every colour needs enough contrast against its background (WCAG AA minimum).
- Name tokens by role, not by colour. `accent` not `blue-500`. Roles survive palette changes.

### 2. Typography scale

Define a type scale with clear hierarchy:

| Level | Size | Weight | Line height | Usage |
|-------|------|--------|-------------|-------|
| Display | ... | ... | ... | Hero headlines only |
| H1 | ... | ... | ... | Page titles |
| H2 | ... | ... | ... | Section headings |
| H3 | ... | ... | ... | Card titles, sub-sections |
| Body | ... | ... | ... | Default text |
| Small | ... | ... | ... | Captions, helper text |
| Micro | ... | ... | ... | Labels, badges, timestamps |

**Typography rules:**
- Max 2 font families (1 is often enough). Justify every additional font.
- The scale should have clear jumps between levels. If two sizes look the same at a glance, one of them is unnecessary.
- Define max line width for body text (usually 60-70 characters).
- Specify letter-spacing adjustments for uppercase text and large display text.

### 3. Spacing system

Define the spacing scale and where each value is used:

| Token | Value | Usage |
|-------|-------|-------|
| `space-xs` | ... | Inside badges, between icon and label |
| `space-sm` | ... | Between related elements, form field padding |
| `space-md` | ... | Between cards, section padding (mobile) |
| `space-lg` | ... | Between sections (desktop) |
| `space-xl` | ... | Major section breaks, hero padding |

**Spacing rules:**
- Use a consistent scale (4px base, 8px base, or whatever the project already uses).
- Define component-level spacing (padding inside cards, gap between form fields) separately from page-level spacing (section margins).

### 4. Borders, shadows, and radii

| Token | Value | Usage |
|-------|-------|-------|
| Border default | ... | Card borders, dividers |
| Border strong | ... | Active/focused elements |
| Border subtle | ... | Faint separators |
| Radius small | ... | Buttons, badges, inputs |
| Radius medium | ... | Cards, dialogs |
| Radius large | ... | Only if needed (modals, large panels) |
| Shadow small | ... | Dropdowns, tooltips |
| Shadow medium | ... | Cards, modals |

**Rules:**
- Pick one border style and commit. If the brand uses solid borders, don't also use shadows for elevation. If it uses shadows, keep borders minimal.
- Fewer radius values is better. 2-3 covers most projects.
- Shadows should be subtle. If you can obviously see the shadow, it's too much.

### 5. Component patterns

Define the visual pattern for recurring component types. Not full component APIs, just the visual rules:

**Buttons:**
- Primary: background, text colour, padding, radius, hover state
- Secondary: how it differs from primary (outline? muted background?)
- Destructive: how it signals danger
- Sizing: how many sizes, and what changes between them (padding? font size? both?)

**Cards:**
- Border or shadow? Both?
- Padding values
- How cards visually group vs separate from each other

**Form inputs:**
- Border style at rest, focus, error
- Label position (above, inline, floating)
- Error message style and position

**Interactive states:**
- Hover: what changes? Colour, background, border, scale?
- Focus: visible focus ring style and colour
- Disabled: how to signal disabled without looking broken
- Active/pressed: does anything happen?

### 6. Banned defaults

List the generic/template patterns this design system deliberately avoids (informed by the product's identity):

- What specific Tailwind defaults to override or never use
- Which common UI patterns don't fit this brand
- Shadow/radius/colour combinations that look too "starter template"

## Process

1. **Present the draft** - show the full design system reference to the user.
2. **Discuss and refine** - the user may want to adjust colours, spacing, or component patterns. Show before/after comparisons where helpful.
3. **Save the reference** - once approved, write the design system doc to the project's docs directory (e.g. `.claude/docs/design-system.md`).
4. **Update Tailwind/CSS** - apply the agreed tokens to the project's Tailwind config or CSS variables. Only after user approval.
5. **Update CLAUDE.md** - add a reference line pointing to the design system doc.

## Rules for writing the design system doc

- **Be specific** - exact values, not "a nice blue". `#2563EB` or `hsl(217, 91%, 60%)`.
- **Show, don't theorise** - describe what each token looks like in practice, not why design systems matter.
- **Keep it under 150 lines** - a design system doc nobody reads is worse than no doc at all.
- **Match reality** - if the project already has an established look, the design system should codify it (with improvements), not replace it with something unrecognisable.
- **Tailwind-native where possible** - if the project uses Tailwind, express tokens as Tailwind config extensions, not a parallel system.
