---
name: copy-hero
description: Generate compelling hero section copy — headlines, subheadlines, and CTAs
disable-model-invocation: true
---

Generate hero section copy for a landing page or feature page. Do NOT make any code changes — present copy options for the user to review, choose, and refine.

## Before you start

Read `PRODUCT.md`, `CLAUDE.md`, terminology docs. Load `.claude/docs/tone.md` — if missing, suggest `/copy-tone` first. Read the current hero component to understand existing copy and structure.

**Ask before generating:**
> "Do you want SEO-optimised hero copy (keyword-aware H1 + subheadline) or conversion-only (best hook regardless of keywords)?"

Default to conversion-only if the user doesn't care.

## What to produce

For each hero section, generate **3 distinct options** across different copywriting angles:

### Option A — Benefit-led
Lead with what the user gains. Focus on the outcome, not the product.

### Option B — Problem-led
Lead with the pain point or frustration the product solves. Create recognition ("that's me").

### Option C — Identity-led
Lead with who the user becomes. Aspirational, speaks to the user's self-image.

## For each option, provide:

1. **Headline** (5-10 words) - the main hook. Must be instantly scannable.
2. **Subheadline** (15-25 words) - expands on the headline. Adds specificity and credibility without repeating.
3. **Primary CTA** (2-5 words) - the main button text. Action-oriented, low friction.
4. **Secondary CTA** (2-5 words) - the alternative action (e.g. "Explore", "See examples", "Learn more").
5. **Rationale** (1-2 sentences) - why this angle works for this product/audience.

## Copywriting rules

- **No filler words** - cut "just", "simply", "easily", "really", "very"
- **No jargon the user wouldn't use** - write in the language of the audience, not the developer
- **Headline and subheadline must not repeat each other** - the subhead adds new information
- **CTAs must reduce perceived risk** - prefer "Start free", "Try it out", "See how it works" over "Sign up now", "Register"
- **Specificity beats vagueness** - "Track every tweak to your recipes" beats "Manage your recipes better"
- **Short sentences, strong verbs** - active voice, present tense
- **No superlatives without proof** - avoid "best", "ultimate", "revolutionary" unless backed by data

## SEO-optimised mode

Only apply when user selects SEO-optimised.

Ask for (or infer from product docs): **primary keyword** and **secondary keyword**. If they don't know, suggest 2–3 options based on the product and ask them to pick.

| Element | Rule |
|---------|------|
| Headline (H1) | Primary keyword embedded naturally — must still read as a real headline |
| Subheadline | Secondary keyword in the first half if possible — no repeating the primary |
| CTAs | No keyword requirement — keep conversion-focused |

Each option gains a **keyword placement note** explaining where the keyword lands and why it fits.

Red lines: never force a keyword if it sounds stuffed — rewrite the sentence until it fits naturally. Never repeat the same keyword in both headline and subheadline. Never sacrifice the hook for the keyword.

## Write like a human, not an AI

No emdashes | no unlock/unleash/elevate/streamline/empower | no delve/tapestry/paradigm/synergy | no ! in headlines | no "Introducing.../Meet..." | no alliteration for its own sake | no rhetorical question openers. Contractions fine. Plain concrete words. If it reads like a LinkedIn post, rewrite it.

## After presenting options

1. Ask the user which option (or combination) they prefer.
2. Offer to refine - adjust tone, length, or angle based on feedback.
3. Only apply changes to code when the user explicitly approves final copy.
