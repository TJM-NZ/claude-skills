---
name: copy-sales
description: Generate benefit-driven sales copy for feature sections, "why us" blocks, and landing page body content
disable-model-invocation: true
---

Generate sales and feature copy for landing pages, product pages, or marketing sections. Do NOT make any code changes - present copy options for the user to review, choose, and refine.

## Before you start

Read `PRODUCT.md`, `CLAUDE.md`, terminology docs. Load `.claude/docs/tone.md` — if missing, suggest `/copy-tone` first. Read the current section and the hero copy (sales must feel like a continuation, not a separate voice).

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

No emdashes | no unlock/unleash/elevate/streamline/empower | no delve/tapestry/paradigm/synergy | no ! in headlines | no "Introducing.../Meet..." | no alliteration for its own sake. Contractions fine. Plain concrete words. If it reads like a LinkedIn post, rewrite it.

## After presenting options

1. Present both options side by side so the user can compare.
2. Ask the user which option (or combination) they prefer.
3. Offer to refine - adjust tone, length, or angle based on feedback.
4. Only apply changes to code when the user explicitly approves final copy.
