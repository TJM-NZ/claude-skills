---
name: copy-ux
description: Generate UX microcopy - buttons, empty states, error messages, tooltips, onboarding text, and in-product guidance
disable-model-invocation: true
---

Generate UX microcopy for in-product text. Do NOT make any code changes - present copy options for the user to review, choose, and refine.

## Before you start

1. **Identify what needs copy** - ask the user which component, page, or interaction to write for, or infer from context. Read the file to understand the current UI structure and any existing copy.
2. **Understand the product** - read `PRODUCT.md`, `CLAUDE.md`, and any terminology/conventions docs in the project to absorb the product's purpose, audience, and voice.
3. **Load the tone reference** - look for `.claude/docs/tone.md` (or equivalent tone/voice doc). If it exists, follow it strictly for voice attributes, vocabulary, grammar rules, and "this vs not this" examples. If it doesn't exist, suggest the user runs `/copy-tone` first to establish the brand voice.
4. **Audit existing microcopy** - search the codebase for similar patterns (other empty states, other error messages, other button labels) to ensure consistency. New copy should match the existing tone.

## Copy types and guidelines

### Buttons and CTAs
- **2-4 words max** - every extra word adds friction
- **Start with a verb** - "Save recipe", "Add ingredient", not "Recipe save" or "New ingredient"
- **Match the action to the outcome** - "Delete" not "Remove" if it's permanent. "Hide" not "Delete" if it's reversible.
- **Destructive actions need clarity** - "Delete this recipe" not "Delete". The user should know what they're losing.
- Provide **2 options** per button.

### Empty states
- **Acknowledge the emptiness without being cute about it** - don't overdo the personality
- **Tell the user what goes here** - "Recipes you create will show up here" beats "Nothing to see here"
- **Include a clear next action** - a button or link to fix the empty state
- **No sad faces, crying icons, or "oops"** - treat the user like an adult
- Provide **2 options** per empty state, each with:
  1. **Headline** (3-8 words)
  2. **Body** (10-20 words)
  3. **Action label** (2-4 words) - the button/link to resolve the empty state

### Error messages
- **Say what happened, then what to do** - "That recipe couldn't be saved. Check your connection and try again."
- **No blame** - never "You entered an invalid..." - instead "That doesn't look like a valid..."
- **No technical jargon** - "Something went wrong" not "500 Internal Server Error"
- **Be specific when you can** - "That email is already registered" beats "Invalid input"
- **No humour in errors** - the user is already frustrated
- Provide **2 options** per error message.

### Tooltips and helper text
- **One sentence max** - if it needs more, the UI design needs rethinking
- **Answer "what is this?" or "why?"** - not "how" (that's documentation)
- **No "Please note that..."** or "This field is for..."** - get to the point
- Provide **2 options** per tooltip.

### Onboarding and guidance text
- **Short, actionable steps** - "Add your first recipe" not "Welcome to the recipe management experience"
- **Progressive disclosure** - don't explain everything upfront
- **Celebrate completion without being patronising** - "Recipe saved" not "Great job, you saved your first recipe"
- Provide **2 options** per onboarding step.

### Confirmation dialogs
- **Title states the action** - "Delete this recipe?" not "Are you sure?"
- **Body states the consequence** - "This will permanently remove the recipe and all its versions."
- **Button labels match the title** - "Delete recipe" / "Keep recipe", not "OK" / "Cancel"
- Provide **2 options** per dialog.

## Copywriting rules

- **Brevity is everything** - microcopy is measured in words, not sentences
- **Consistent vocabulary** - if you call it "recipe" in one place, don't call it "dish" in another. Check the project's terminology docs.
- **Consistent casing** - match the existing pattern in the codebase (sentence case vs title case)
- **No filler words** - cut "just", "simply", "easily", "really", "very", "please"
- **No passive voice** - "We couldn't save that" not "The save could not be completed"

## Write like a human, not an AI

The copy must read like a real person wrote it. Avoid these AI tells:

- **No emdashes** - use commas, full stops, or rewrite the sentence instead
- **No "unlock", "unleash", "elevate", "empower", "leverage", "streamline"**
- **No exclamation marks** - they feel fake in product UI
- **Contractions are fine and encouraged** - "can't" not "cannot", "won't" not "will not"
- **Prefer plain, concrete words** - "save" not "persist", "show" not "display", "name" not "designation"
- If it sounds like a robot wrote it, rewrite it.

## After presenting options

1. Present options grouped by copy type (buttons, empty states, errors, etc.).
2. Ask the user which options they prefer for each item.
3. Offer to refine - adjust tone, length, or phrasing based on feedback.
4. Only apply changes to code when the user explicitly approves final copy.
