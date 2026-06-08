---
name: copy-tone
description: Establish and document the brand voice for a project - produces a tone reference doc that all copy skills consume
disable-model-invocation: true
---

Establish the brand voice and tone for a project. Produces a tone reference document that all other copy skills (`copy-hero`, `copy-sales`, `copy-ux`, `copy-email`, `copy-seo`) should reference when writing.

This skill is meant to be run **once per project** (or when the brand voice needs recalibrating). It creates or updates a tone reference file in the project's docs.

## Before you start

1. **Read the product docs** - read `PRODUCT.md`, `CLAUDE.md`, and any existing brand/tone/style docs. Understand the product, who it's for, and what problem it solves.
2. **Sample existing copy** - read 5-10 files that contain user-facing text: landing pages, email templates, UI components with labels/descriptions, error messages, empty states. Get a feel for how the product currently talks.
3. **Identify the audience** - who reads this copy? What's their relationship with the product? Are they browsing, evaluating, actively using, or troubleshooting? Different audiences tolerate different tones.

## What to produce

Present a **draft tone reference** to the user for approval. The reference has these sections:

### 1. Voice attributes (pick 3-4)

Define the brand's voice as a set of paired qualities. Each pair is a spectrum, and you place the brand on it.

Format:
```
[Attribute] but not [excess of that attribute]
```

Examples:
- "Confident but not arrogant"
- "Casual but not sloppy"
- "Helpful but not patronising"
- "Warm but not saccharine"
- "Direct but not blunt"
- "Playful but not silly"
- "Technical but not jargon-heavy"

Pick 3-4 pairs that feel right for this product. Explain each in one sentence.

### 2. Tone spectrum

The voice stays constant but tone shifts by context. Map out how the tone adjusts:

| Context | Tone | Example |
|---------|------|---------|
| Marketing (landing page, hero) | [describe] | [one sentence example] |
| Product UI (labels, buttons) | [describe] | [one sentence example] |
| Success states | [describe] | [one sentence example] |
| Error states | [describe] | [one sentence example] |
| Transactional email | [describe] | [one sentence example] |
| Onboarding | [describe] | [one sentence example] |

### 3. Vocabulary

Two lists:

**Words we use** - 10-15 words/phrases that feel right for this brand. These should come from the product's domain and the audience's natural language.

**Words we don't use** - 10-15 words/phrases to avoid. Include:
- AI slop words (unlock, unleash, elevate, leverage, streamline, delve, etc.)
- Corporate words that don't fit the brand
- Overly casual words if the brand is more serious (or vice versa)
- Competitor terminology the product deliberately avoids

### 4. Grammar and mechanics

Short, definitive rules:

- Contractions: yes or no
- Oxford comma: yes or no
- Sentence case vs title case for headings
- Sentence case vs title case for buttons
- Period at the end of button text: yes or no
- Exclamation marks: when (if ever)
- Ampersands vs "and"
- Numbers: spell out or use digits, and at what threshold

### 5. This vs not this

Provide 5-6 pairs showing a sentence in the brand's voice vs the same idea written wrong. These should cover different contexts (marketing, UI, error, email).

Format:
```
This: "Your recipe has been saved."
Not this: "Awesome! Your recipe has been successfully saved to your account!"

This: "Couldn't save that recipe. Try again."
Not this: "Oops! We encountered an error while attempting to save your recipe."
```

## Process

1. **Present the draft** - show the full tone reference to the user.
2. **Discuss and refine** - the user may want to adjust attributes, add/remove vocabulary, or change examples.
3. **Save the reference** - once approved, write the tone reference to the project's docs directory (e.g. `.claude/docs/tone.md` or `docs/tone.md`, matching the project's existing doc structure).
4. **Update CLAUDE.md** - add a reference line pointing to the tone doc so other skills and prompts can find it.

## Rules for writing the tone doc

- **Be prescriptive, not descriptive** - "We use contractions" not "The brand tends to favour a conversational approach that may include contractions"
- **Use real examples from the product's domain** - not generic placeholder text
- **Keep it under 100 lines** - if it's too long, nobody (human or AI) will read it
- **No meta-commentary** - the doc should be a reference, not an essay about branding theory
