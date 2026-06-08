---
name: design
description: Design unique, distinctive UI sections and components - avoids generic template patterns
disable-model-invocation: true
---

Design distinctive UI for a specific page, section, or component. Present a visual direction and detailed implementation plan for the user to approve before writing any code.

## Before you start

1. **Understand the product** - read `PRODUCT.md`, `CLAUDE.md`, and any style/convention docs. Understand what the product does, who uses it, and what feeling it should evoke.
2. **Load the design system** - look for `.claude/docs/design-system.md` (or equivalent design tokens doc). If it exists, follow it strictly for colours, typography, spacing, borders, and component patterns. If it doesn't exist, suggest the user runs `/design-system` first to establish the foundation.
3. **Audit the current design** - read the existing CSS/Tailwind config, global styles, and key components. Understand the current colour palette, typography, spacing system, and any existing design tokens.
4. **Read the target section** - read the file(s) to be designed. Understand the content structure, what data is displayed, and how users interact with it.
5. **Identify the product's domain** - every product lives in a world with its own visual language. A recipe app has kitchen/craft metaphors. A finance app has precision/trust. A music app has rhythm/movement. Find the domain and draw from it.

## Design process

### Step 1 - Visual concept (present before any code)

Propose **2 distinct visual directions**. Each direction must include:

1. **Concept name** (2-3 words) - a shorthand for the direction
2. **Mood** (1 sentence) - what it should feel like to look at
3. **Domain reference** (1 sentence) - what real-world object, space, or experience this draws from. Not other websites. Think physical spaces, printed matter, tools, materials.
4. **Layout approach** - describe the spatial arrangement. How does content flow? What breaks the grid? What has visual weight?
5. **Key visual moves** (3-5 specific details) - the concrete choices that make this direction distinctive. Be precise: "8px solid left border on cards in the accent colour" not "add some visual interest"
6. **What it avoids** - name the generic pattern this direction deliberately sidesteps

The two directions should feel meaningfully different from each other, not variations on the same theme.

### Step 2 - Refinement

After the user picks a direction, flesh out:

1. **Detailed layout** - exact structure, responsive behaviour, spacing
2. **Typography choices** - sizes, weights, cases. What text is large? What's small? What's uppercase? What's muted?
3. **Colour usage** - how the existing palette is applied in this section. Where does colour appear? Where is it absent?
4. **Interactive states** - hover, focus, active. What moves? What changes colour? What doesn't change at all?
5. **Responsive strategy** - what changes on mobile vs desktop. Not just "stack vertically" but what the mobile version actually looks and feels like.

### Step 3 - Implementation

Write the code only after the user approves the refined design. Use the project's existing tech (Tailwind, CSS modules, whatever's already in use).

## Design rules - what makes it distinctive

### Layout
- **Break symmetry on purpose** - not everything needs to be centered. Off-center headlines, asymmetric grids, and varied column widths create visual interest.
- **Use whitespace as a design element** - generous, intentional spacing says more than decoration. But know where to cluster things tight for contrast.
- **Vary the rhythm** - if every section is the same height with the same padding, the page feels like a spreadsheet. Alternate dense and spacious sections.
- **Let content dictate layout** - a list of 6 features doesn't have to be a 3x2 grid. Maybe it's a staggered list. Maybe it's a scrolling strip. Let the content's nature suggest the form.

### Typography
- **Type hierarchy does the heavy lifting** - size, weight, and spacing contrasts create structure without needing boxes or borders.
- **Consider text case deliberately** - a small uppercase label above a large lowercase heading is a specific choice, not a default.
- **Line length matters** - don't let text run wider than ~65 characters. Narrow columns of text feel intentional.

### Colour and texture
- **Restraint over rainbow** - one or two accent colours used sparingly hits harder than colour everywhere.
- **Backgrounds aren't always flat** - subtle grain, noise textures, or slight gradients add warmth without being loud.
- **Dark/light mode isn't just inverted colours** - the design should feel intentional in both modes, not auto-generated.

### Details
- **Borders and dividers have character** - a 2px solid border feels different from a 1px dashed line feels different from a subtle shadow. Choose based on the mood.
- **Icons are optional** - not every feature card needs an icon. Sometimes a bold number, a coloured dot, or nothing at all works better.
- **Animation should have purpose** - don't animate for the sake of it. If something moves, it should communicate state change or draw attention to something important.

## Banned patterns - the generic template kit

These are not inherently bad, but they're so overused that they signal "template". Avoid them unless there's a strong reason:

- **Centred hero with two buttons and nothing else** - add visual interest, texture, or asymmetry
- **3-column feature grid with icon + title + description** - find a different way to present features
- **Gradient text on headings** - it screams 2023 SaaS template
- **Floating cards with large border-radius and drop shadows** - the shadcn/ui default look
- **Bento grid layouts** - every startup page looks like this now
- **Purple-to-blue gradients** - the most generic colour choice possible
- **"Trusted by" logo bars** - fine if you actually have logos, but don't use placeholder logos
- **Oversized hero sections with 50% of viewport as whitespace** - the content should earn the space
- **Identical card components repeated in a flat grid** - vary the presentation even slightly
- **Glassmorphism / frosted glass effects** - dated and overused

## Draw from the physical world

The strongest visual identities reference something real. Some starting points by domain:

- **Food/cooking** - recipe cards, handwritten notes, stained cookbook pages, kitchen chalkboards, market signage, food packaging, ceramic textures
- **Developer tools** - terminal interfaces, monospace type, dense information displays, technical drawings, engineering notebooks
- **Creative tools** - studio walls, sketchbook pages, material swatches, gallery layouts, printed zines
- **Finance** - newspaper financial pages, ledger books, ticker tape, embossed stationery
- **Music** - vinyl sleeves, gig posters, equaliser displays, cassette labels
- **Education** - textbook layouts, index cards, library catalogues, blackboard chalk

Don't copy these literally. Use them as a starting point for texture, layout rhythm, and typographic choices.

## After presenting directions

1. Present the 2 visual directions with enough detail that the user can picture them.
2. Ask the user which direction (or elements from both) they prefer.
3. Refine the chosen direction with full implementation detail.
4. Only write code after the user approves the refined design.
