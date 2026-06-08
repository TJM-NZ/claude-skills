---
name: design-interactions
description: Design distinctive microinteractions and motion - avoids generic hover/fade/scale defaults
disable-model-invocation: true
---

Design microinteractions and motion for specific components or pages. Present interaction concepts for the user to approve before writing any code.

## Before you start

1. **Understand the product** - read `PRODUCT.md`, `CLAUDE.md`, and any tone/design docs. The motion language should match the brand's personality. A playful brand moves differently from a serious one.
2. **Load the design system** - look for `.claude/docs/design-system.md`. If it exists, work within its interactive state definitions. If it doesn't exist, suggest the user runs `/design-system` first.
3. **Load the tone reference** - look for `.claude/docs/tone.md`. Motion has tone. Snappy motion feels confident. Slow easing feels calm. Match the brand voice.
4. **Audit existing motion** - search the codebase for existing transitions, animations, and keyframes. Catalogue what's already there. New interactions must feel like they belong to the same system.
5. **Read the target component** - understand what the user interacts with, what state changes occur, and what feedback the user needs.

## Design process

### Step 1 - Interaction audit

Before proposing anything new, report:
- What interactions currently exist in the target area
- Which feel generic or default
- Which are missing (state changes with no visual feedback)
- Which are excessive (animation that slows the user down)

### Step 2 - Interaction concepts

For each interaction point, propose **2 options**. Each must include:

1. **What triggers it** - hover, click, focus, scroll, state change, data load
2. **What moves and how** - be specific about which property changes, the easing curve, and the duration in ms
3. **Why this motion** - what does it communicate? Acknowledgement, progress, success, connection between elements?
4. **What it replaces** - the generic version it avoids

### Step 3 - Implementation

Write the code only after the user approves. Use CSS transitions/animations or the project's existing animation approach (Framer Motion, CSS keyframes, Tailwind animate, etc.).

## Interaction types

### Hover and focus
- **Purpose**: acknowledge the user's attention, signal interactivity
- Not every element needs a hover state. Only interactive elements.
- Hover should be instant or near-instant (50-150ms). Anything slower feels laggy.
- Focus states are for accessibility, not decoration. They must be visible.

### Click and press
- **Purpose**: confirm the action was registered
- The response should be immediate. No delays.
- Physical metaphor: buttons should feel like they depress. Links should feel like they activate.
- Consider what happens during async actions (loading states).

### State transitions
- **Purpose**: help the user understand what changed
- Content appearing: where did it come from? Animate from the source direction.
- Content disappearing: where is it going? Animate toward the destination.
- Content changing: morph between states rather than cutting.

### Scroll-triggered
- **Purpose**: create rhythm as the user moves through content
- Use sparingly. Most content should be visible immediately.
- Never block content behind a scroll animation. The content is more important than the entrance.
- Stagger entrance of list items only if the stagger is fast (30-60ms between items). Slow staggers waste the user's time.

### Loading and progress
- **Purpose**: communicate that something is happening
- Skeleton screens over spinners where possible.
- Progress indicators should feel honest. Don't fake progress.
- The transition from loading to loaded matters as much as the loading state itself.

### Feedback and validation
- **Purpose**: tell the user if what they did worked
- Success: brief, confident, doesn't demand attention. A subtle colour shift or checkmark.
- Error: must be noticeable but not alarming. Draw the eye without shouting.
- Inline validation should feel like a conversation, not a red alarm.

## Motion rules

### Timing
- **Hover/focus**: 50-150ms. Anything the user triggers directly should feel instant.
- **Small transitions** (colour, border, shadow): 100-200ms
- **Layout shifts** (expanding, collapsing, sliding): 200-350ms
- **Entrances** (elements appearing): 150-300ms
- **Exits** (elements disappearing): 100-200ms. Exits should be faster than entrances.
- **Page transitions**: 200-400ms
- If in doubt, go faster. Almost nobody complains that an animation was too quick.

### Easing
- **Never use `linear`** for UI motion. Linear motion looks mechanical and wrong.
- **`ease-out`** for entrances - elements arriving should decelerate into place
- **`ease-in`** for exits - elements leaving should accelerate away
- **`ease-in-out`** for state changes where the element stays on screen
- **Custom cubic-bezier** when the defaults feel flat. Slight overshoot (`cubic-bezier(0.34, 1.56, 0.64, 1)`) adds physicality to bouncy brands. Tight curves (`cubic-bezier(0.4, 0, 0.2, 1)`) feel precise for serious brands.

### Properties
- **Animate specific properties, not `all`**. `transition-all` is lazy and causes unexpected side effects.
- **Prefer `transform` and `opacity`** for performance. These run on the compositor and won't cause layout reflow.
- **Avoid animating `width`, `height`, `top`, `left`** directly. Use `transform: scale()` or `transform: translate()` instead.
- **`will-change` only when needed** and only on elements that are about to animate, not permanently.

## Banned patterns - the generic motion kit

These are the default AI/template interactions. Avoid them unless you can justify why they're the right choice:

- **`hover:scale-105`** - the universal "I added an interaction" move. Scale rarely makes sense for buttons or cards. If something grows on hover, what physical metaphor is that?
- **`transition-all duration-300`** - 300ms is too slow for most hover states and `transition-all` is imprecise
- **Fade in from opacity-0 for everything** - not every element needs a dramatic entrance. Most content should just be there.
- **`hover:shadow-lg` / `hover:shadow-xl`** - adding a bigger shadow on hover is the card equivalent of `scale-105`. Generic.
- **Infinite pulse/ping animations on badges** - draws attention forever. If it's always pulsing, it's never important.
- **Slide up + fade in on scroll for every section** - the "AOS library default". Makes pages feel slow and theatrical.
- **Bounce animation on CTAs** - "look at me" energy. The copy should do the selling, not the animation.
- **Rotating loading spinners** - skeleton screens are almost always better. If you must spin, make it subtle.
- **`hover:-translate-y-1`** - cards lifting on hover is the most overused card interaction

## Draw from the physical world

Good motion references:
- **Cooking**: the confident slide of a plate across a pass, the snap of a timer, the flip of a recipe card, a pen checking off a list
- **Paper and print**: page turns, card flips, stamps pressing down, tabs sliding
- **Tools**: switches clicking, dials turning, drawers sliding open, latches engaging
- **Natural motion**: gravity (things fall faster than they rise), inertia (things ease to a stop), weight (heavy things move slower)

The best microinteractions feel like the interface has physical properties - weight, friction, elasticity - rather than mathematical ones.

## After presenting concepts

1. Present the interaction audit first so the user sees what exists.
2. Show the proposed interactions with specific timing, easing, and property values.
3. Ask the user which options they prefer.
4. Offer to refine - adjust timing, easing, or approach based on feedback.
5. Only write code after the user approves.
