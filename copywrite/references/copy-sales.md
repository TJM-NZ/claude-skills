---
name: copy-sales
description: Generate benefit-driven sales copy for feature sections, "why us" blocks, and landing page body content
disable-model-invocation: true
---

Generate sales and feature copy for landing pages, product pages, or marketing sections. Do NOT make any code changes - present copy options for the user to review, choose, and refine.

## Before you start

1. **Identify the target section** - ask the user which page/component to write for, or infer from context. Read the file to understand the current copy, structure, and how many feature items exist.
2. **Understand the product** - read `PRODUCT.md`, `CLAUDE.md`, and any terminology/conventions docs in the project to absorb the product's purpose, audience, and voice.
3. **Load the tone reference** - look for `.claude/docs/tone.md` (or equivalent tone/voice doc). If it exists, follow it strictly for voice attributes, vocabulary, grammar rules, and "this vs not this" examples. If it doesn't exist, suggest the user runs `/copy-tone` first to establish the brand voice.
4. **Read the hero copy** - the sales section must feel like a continuation of the hero, not a separate voice. Match the tone and energy.

## What to produce

### For feature grids / cards

For each feature item, generate **2 options**:

**Option A - Outcome-first**
Lead with what the user can do or what happens. The feature name is secondary.

**Option B - Scenario-first**
Lead with a relatable situation, then show how the feature helps.

Each feature item needs:
1. **Title** (2-5 words) - name the capability in the user's language, not technical jargon
2. **Description** (15-30 words) - one concrete benefit or scenario. Must answer "so what?" for the reader.

### For "why us" / comparison sections

Generate **2 options**:

**Option A - Before/after framing**
Show the old way vs the new way. No need to name competitors.

**Option B - Stacking benefits**
List 3-5 specific, concrete things the user gets. No fluff.

### For long-form body sections

Generate **2 options** with:
1. **Section headline** (5-10 words)
2. **Body copy** (40-80 words) - conversational, specific, and grounded in real use cases
3. **Optional CTA** if the section warrants one

## Copywriting rules

- **Benefits over features** - "See exactly what you changed last time" beats "Version history tracking"
- **One idea per feature card** - don't cram two benefits into one description
- **Concrete over abstract** - use specific examples ("swap butter for coconut oil") not vague promises ("customize to your needs")
- **Consistent tone across all items** - the whole section should read like one person wrote it
- **Parallel structure** - if one card starts with a verb, they all should. If one is a statement, they all should.
- **No filler words** - cut "just", "simply", "easily", "really", "very"
- **No jargon the user wouldn't use** - write in the language of the audience, not the developer
- **No superlatives without proof** - avoid "best", "ultimate", "revolutionary" unless backed by data

## Write like a human, not an AI

- **No emdashes** - use commas, full stops, or rewrite the sentence instead
- **No "unlock", "unleash", "elevate", "empower", "leverage", "streamline"** - these are dead giveaways
- **No "delve", "tapestry", "landscape", "paradigm", "holistic", "synergy"** - corporate AI slop
- **No exclamation marks** in headlines or descriptions - confidence doesn't shout
- **No starting with "Introducing..." or "Meet..."** - overdone and lazy
- **No alliteration for its own sake** - it reads as try-hard
- **Contractions are fine** - "you'll" not "you will", "it's" not "it is"
- **Prefer plain, concrete words** - "change" not "transform", "use" not "utilize", "help" not "facilitate"

## After presenting options

1. Present both options side by side so the user can compare.
2. Ask the user which option (or combination) they prefer.
3. Offer to refine - adjust tone, length, or angle based on feedback.
4. Only apply changes to code when the user explicitly approves final copy.
