---
name: copy-hero
description: Generate compelling hero section copy — headlines, subheadlines, and CTAs
disable-model-invocation: true
---

Generate hero section copy for a landing page or feature page. Do NOT make any code changes — present copy options for the user to review, choose, and refine.

## Before you start

1. **Identify the hero component** — ask the user which page/component to write for, or infer from context. Read the file to understand the current copy and structure.
2. **Understand the product** — read `PRODUCT.md`, `CLAUDE.md`, and any terminology/conventions docs in the project to absorb the product's purpose, audience, and voice.
3. **Load the tone reference** — look for `.claude/docs/tone.md` (or equivalent tone/voice doc). If it exists, follow it strictly for voice attributes, vocabulary, grammar rules, and "this vs not this" examples. If it doesn't exist, suggest the user runs `/copy-tone` first to establish the brand voice.
4. **Ask: SEO-optimised or conversion-only?** — before generating, ask the user:
   > "Do you want SEO-optimised hero copy (keyword-aware H1 + subheadline for search rankings) or conversion-only (best hook regardless of keywords)?"

   Default to conversion-only if they don't respond or say they don't care.

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

Only apply when the user selects SEO-optimised in step 4.

**Before generating**, ask for or infer:
- **Primary keyword** — one phrase the page should rank for (e.g. "sourdough bread recipes")
- **Secondary keyword** — one supporting phrase (e.g. "bread baking community")

If the user doesn't know their keywords, suggest 2–3 based on the product and audience, and ask them to pick.

**SEO rules for hero copy:**

| Element | Rule |
|---------|------|
| Headline (H1) | Primary keyword embedded naturally — not forced onto the end. Must still read like a real headline, not a keyword tag. |
| Subheadline | Secondary keyword in the first half if possible. Should expand on the headline without repeating the primary keyword. |
| CTAs | No keyword requirement — keep conversion-focused. |

**What changes in output format:**

Each option gains an extra line:
- **Keyword placement note** — where the primary/secondary keyword lands and why it works there

**What doesn't change:**
- Still 3 options (benefit-led, problem-led, identity-led)
- Same length rules (5–10 word headline, 15–25 word subheadline)
- Same human writing rules — keywords must fit naturally; if they don't, rewrite the sentence until they do

**Red lines:**
- Never stuff the keyword in if it sounds forced — a bad H1 hurts rankings more than a missing keyword
- Never repeat the keyword in both headline and subheadline
- Never sacrifice the hook for the keyword — conversion still matters

## Write like a human, not an AI

- **No emdashes** - use commas, full stops, or rewrite the sentence instead
- **No "unlock", "unleash", "elevate", "empower", "leverage", "streamline"** - these are dead giveaways
- **No "delve", "tapestry", "landscape", "paradigm", "holistic", "synergy"** - corporate AI slop
- **No exclamation marks** in headlines or subheadlines - confidence doesn't shout
- **No starting with "Introducing..." or "Meet..."** - overdone and lazy
- **No alliteration for its own sake** - it reads as try-hard
- **No rhetorical questions in headlines** - they waste the most valuable real estate on the page
- **Contractions are fine** - "you'll" not "you will", "it's" not "it is"
- **Prefer plain, concrete words** - "change" not "transform", "use" not "utilize", "help" not "facilitate"

## After presenting options

1. Ask the user which option (or combination) they prefer.
2. Offer to refine - adjust tone, length, or angle based on feedback.
3. Only apply changes to code when the user explicitly approves final copy.
