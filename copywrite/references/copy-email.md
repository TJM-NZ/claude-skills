---
name: copy-email
description: Generate email copy - subject lines, transactional emails, notification digests, and drip campaigns
disable-model-invocation: true
---

Generate email copy for transactional, notification, or marketing emails. Do NOT make any code changes - present copy options for the user to review, choose, and refine.

## Before you start

1. **Identify the email type** - ask the user which email to write, or infer from context. Read any existing email templates or sending logic in the codebase to understand the current approach.
2. **Understand the product** - read `PRODUCT.md`, `CLAUDE.md`, and any terminology/conventions docs in the project to absorb the product's purpose, audience, and voice.
3. **Load the tone reference** - look for `.claude/docs/tone.md` (or equivalent tone/voice doc). If it exists, follow it strictly for voice attributes, vocabulary, grammar rules, and "this vs not this" examples. If it doesn't exist, suggest the user runs `/copy-tone` first to establish the brand voice.
4. **Check existing emails** - search the codebase for email templates, sending functions, or email-related components. New emails should match the existing tone and format.

## Email types and guidelines

### Transactional emails (verification, password reset, account changes)

These are functional. The user is expecting them. Get to the point.

For each email, provide **2 options** with:
1. **Subject line** (3-8 words) - clear, scannable, no clickbait
2. **Preheader** (5-12 words) - the preview text shown in inbox list view. Adds context the subject didn't cover.
3. **Body copy** - structured as:
   - **Opening** (1 sentence) - what this email is about
   - **Action** - the button or link text and surrounding context
   - **Fallback** (1 sentence) - what to do if they didn't request this
4. **Button text** (2-4 words)

Rules for transactional:
- No marketing, no upsells, no "while you're here..."
- No greetings like "Hi [Name]" unless the product's existing emails use them
- Subject line should work without the product name (inbox space is limited)
- "If you didn't request this, ignore this email" is mandatory for auth-related emails

### Notification emails (activity, social, updates)

These bring the user back. They need to be worth the interruption.

For each email, provide **2 options** with:
1. **Subject line** (4-10 words) - specific enough to decide whether to open. Include the trigger ("Sarah forked your recipe" not "New activity")
2. **Preheader** (5-12 words) - preview context
3. **Body copy** - structured as:
   - **What happened** (1-2 sentences) - the event, with enough context to be useful without clicking through
   - **CTA** - button or link to see more
4. **Button text** (2-4 words)

Rules for notifications:
- The subject line should contain the useful information, not tease it
- One notification per email, or a digest with a clear list
- Always include an unsubscribe line

### Drip / onboarding emails

These teach or nudge. They should feel helpful, not pushy.

For each email in the sequence, provide **2 options** with:
1. **Subject line** (4-8 words)
2. **Preheader** (5-12 words)
3. **Body copy** (40-80 words) - one idea per email, one action per email
4. **CTA button text** (2-4 words)
5. **Send timing** - when this email fires relative to signup or the previous email

Rules for drip:
- Each email should stand alone. Don't reference previous emails in the sequence.
- No more than one CTA per email
- No "just checking in" or "we noticed you haven't..." guilt-tripping
- If there's nothing useful to say, don't send the email. Recommend removing it from the sequence.

### Marketing / announcement emails

For each email, provide **2 options** with:
1. **Subject line** (4-8 words) - specific, no clickbait, no ALL CAPS
2. **Preheader** (5-12 words)
3. **Body copy** (60-120 words) - what's new, why it matters to the reader, one clear CTA
4. **CTA button text** (2-4 words)

## Copywriting rules

- **Subject lines are everything** - 60% of the email's job is done by the subject line
- **No "Hey [Name]!" energy** - casual greetings are fine if the brand uses them, but forced friendliness is worse than none
- **Preheaders are not afterthoughts** - if you leave them blank, the inbox shows the first line of body text (usually garbage)
- **One purpose per email** - if it does two things, split it into two emails
- **Short paragraphs** - 1-2 sentences max. Email is scanned, not read.
- **Button text matches the promise** - "View recipe" not "Click here". "Reset password" not "Continue".
- **No filler words** - cut "just", "simply", "easily", "really", "very"

## Write like a human, not an AI

- **No emdashes** - use commas, full stops, or rewrite the sentence instead
- **No "unlock", "unleash", "elevate", "empower", "leverage", "streamline"**
- **No "we're thrilled", "we're excited to announce"** - the reader doesn't care about your emotions
- **No "Dear valued user"** or any variation of it
- **No exclamation marks in subject lines** - they trigger spam filters and feel desperate
- **Contractions are fine** - "you'll" not "you will", "here's" not "here is"
- **Prefer plain, concrete words** - "new" not "brand-new", "changed" not "updated"

## After presenting options

1. Present options grouped by email type.
2. Ask the user which options they prefer.
3. Offer to refine - adjust tone, length, or structure based on feedback.
4. Only apply changes to code when the user explicitly approves final copy.
