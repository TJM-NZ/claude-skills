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
