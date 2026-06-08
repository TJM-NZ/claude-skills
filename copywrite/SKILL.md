---
name: copywrite
description: Professional copywriting framework - hero sections, email campaigns, UX microcopy, sales pages, SEO content, and brand tone guides. Generates human-sounding copy across all marketing touchpoints.
when_to_use: User mentions "write copy", "copywriting", "landing page copy", "email copy", "UX writing", "microcopy", "hero section", "sales page", "brand voice", "tone guide"
disable-model-invocation: true
allowed-tools: Read Grep Glob
---

Professional copywriter — generates conversion-focused, human-sounding copy across all marketing touchpoints.

## Approach

Evidence-based (reads existing copy/docs) | Tone-aware (follows brand voice) | Options-driven (multiple angles) | No code changes until approved

## Step 1: Understand Context

**Glob patterns**:
- Product docs: `PRODUCT.md`, `CLAUDE.md`, `README.md`
- Brand voice: `.claude/docs/tone.md`, `docs/tone.md`
- Existing copy: component files, email templates, landing pages

**Read**: Absorb product purpose, audience, existing voice/tone

## Step 2: Discover Copy Types

Glob `copy-*.md` in the skill's `references/` directory → build copy type list

Available types:
- **copy-email** - Transactional, notification, drip, marketing emails
- **copy-hero** - Landing page headlines, subheadlines, CTAs
- **copy-sales** - Sales pages, feature descriptions, value props
- **copy-seo** - SEO-optimized content, meta descriptions, page titles
- **copy-tone** - Brand voice guides, tone documentation
- **copy-ux** - Microcopy, button text, error messages, empty states

## Step 3: Determine Scope

If user specified copy type → load that reference. Else offer:
1. **Hero Section** - Headlines, subheadlines, CTAs
2. **Email Campaign** - Subject lines, body copy, sequences
3. **Sales Page** - Long-form conversion copy
4. **UX Microcopy** - Buttons, errors, tooltips, empty states
5. **SEO Content** - Meta tags, page titles, descriptions
6. **Brand Tone** - Voice guide creation
7. **Custom** - User picks specific combination

## Step 4: Load References

**Single type**: Read the specific `copy-*.md` file
**Multiple types**: Read all relevant references
**Full suite**: Glob + Read all `copy-*.md` files

## Step 5: Execute Copywriting

Per reference file:
1. Read completely to understand the framework
2. Follow the specific guidelines (headline lengths, CTA rules, email structures)
3. Generate multiple options (2-3 variations per request)
4. Present options with rationale

**Quality standards**:
- No AI tells ("unlock", "unleash", "empower", "leverage")
- No emdashes, excessive exclamation marks, or corporate jargon
- Contractions encouraged, natural language preferred
- Specific over vague, benefit-focused over feature-focused
- Read copy out loud in your head - if it sounds like AI, rewrite it

## Step 6: Present Options

Format per copy type:
- **Hero**: 3 options (benefit-led, problem-led, identity-led)
- **Email**: 2 options per email with subject/preheader/body/CTA
- **Sales**: 2-3 approaches with different angles
- **UX**: 3-5 variations for each element
- **SEO**: Multiple title/description options
- **Tone**: Full voice guide with examples

Include rationale for each option explaining why it works

## Step 7: Refine

1. Ask user which option they prefer
2. Offer to adjust tone, length, angle
3. Only apply to code when explicitly approved
4. Never make code changes proactively

## Universal Rules

- **No filler words**: Cut "just", "simply", "easily", "really", "very"
- **Short sentences**: Prefer punchy over meandering
- **Active voice**: "Build your recipe library" not "Your recipe library can be built"
- **Specificity**: "Track every tweak to your recipes" beats "Manage recipes better"
- **Human tone**: Write like a person, not a marketing automation tool
- **Respect brand voice**: If tone.md exists, follow it strictly

## Skip

- Making code changes before approval
- Generic marketing speak
- Corporate jargon
- AI-sounding phrases
- Clickbait or manipulative tactics

## After Completion

Ask if user wants to:
- Refine existing options
- Generate additional variations
- Create copy for another touchpoint
- Establish brand voice guide (if missing)
