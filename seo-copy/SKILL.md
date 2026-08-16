---
name: seo-copy
description: Write SEO-optimized page copy — headings, body opening, meta tags, OG, and anchor text in one pass. Use when writing or rewriting copy for pages that need to rank.
when_to_use: User mentions "SEO copy", "copy that ranks", "write copy for SEO", "landing page SEO", "optimize copy for search", "page copy and SEO", "full page SEO"
disable-model-invocation: true
allowed-tools: Read Grep Glob
---

SEO copywriter — writes full-page copy that's search-ready from the start: headlines, subheadlines, body opening, meta tags, OG, and anchor text in one pass. No code changes until approved.

## Step 1: Read Context

- Product/brand: `PRODUCT.md`, `CLAUDE.md`, `README.md`
- Tone: `.claude/docs/tone.md` — if missing, suggest `/copy-tone` first and stop
- Existing pages: read `layout.tsx`, `page.tsx`, index files — note current H1, meta, body copy
- Routes: identify all public-facing pages that need coverage

## Step 2: Scan Existing SEO State

For each in-scope page, check:

| Element | What to look for |
|---------|-----------------|
| H1 | Present? Includes primary keyword? Matches meta title roughly? |
| H2s | Logical hierarchy? Secondary keywords distributed? |
| Meta title | Length (50-60 chars)? Front-loaded keyword? Unique? |
| Meta description | Length (120-155 chars)? CTA or value prop? Unique per page? |
| Body opening | Primary keyword in first 100 words? |
| Anchor text | Internal links TO this page — generic ("click here") or keyword-rich? |

Flag each gap: missing / weak / duplicate / generic.

## Step 3: Determine Scope

If user specified pages → use those.
Else ask: all public pages / specific page / pages with flagged gaps only?

## Step 4: Generate SEO Copy Package

For each page produce all of:

### H1 — Page headline

2 options. Rules:
- Primary keyword embedded naturally (not bolted on)
- Matches or echoes the meta title
- 5–10 words for landing pages; use content itself for dynamic pages
- One keyword only — no stuffing

### H2s — Section subheadlines

2–3 per major section. Rules:
- Distribute secondary keywords across H2s (not all in one)
- Each H2 works as a standalone scan hook
- No repeated words from H1
- Action-oriented or value-focused phrasing

### Body copy opening

1 paragraph (3–4 sentences). Rules:
- Primary keyword in the first sentence, naturally
- States the page's value prop
- Matches search intent — what is someone looking for who lands here?
- No filler openers: cut "We are proud to", "Welcome to", "In today's world"

### Meta title

2 options:
- 50–60 characters max
- Format: `Primary Keyword | Brand` or `Primary Keyword - Brand`
- Front-load the keyword (not `Brand | Keyword`)
- Homepage exception: can lead with brand + tagline
- Dynamic pages: use a template (`{recipe.title} | Brand`)

### Meta description

2 options:
- 120–155 characters (truncated at ~120 on mobile, ~155 on desktop)
- Include a CTA or concrete value prop: "Browse 500+ recipes" not "This is a recipe site"
- Match the page content — mismatches spike bounce rate and tank rankings
- No quotes, no special characters (break in SERPs)
- Unique per page — duplicate descriptions hurt rankings

### Open Graph

- `og:title`: same as or shorter than meta title
- `og:description`: 1–2 sentences, can be more casual than meta description
- `og:type`: website / article / profile / product
- Image art direction note: what the OG image should show (not the copy, just guidance)

### Anchor text (for internal links pointing TO this page)

3–4 options. Rules:
- Includes the primary keyword naturally within a sentence fragment
- No "click here", "read more", "learn more"
- Each reads cleanly as text within a sentence

## Step 5: Present Options

Group all options by page. For each element, show 2 options with a one-line rationale:

```
**H1 — Option A**: Track Every Recipe Change You Make
Why: Primary keyword "recipe" up front, benefit-focused, reads like a feature not a tag line

**H1 — Option B**: Your Recipe History, Automatically Saved
Why: Leads with the user outcome, "recipe" keyword embedded, short and scannable
```

## Step 6: Refine and Apply

1. Ask which options they prefer per element
2. Offer to adjust: keyword emphasis, length, tone, angle
3. Once approved, offer to apply changes to code (metadata exports, layout files, page components)
4. Never edit code before explicit approval

## Copy rules

- **Humans first, search engines second** — Google rewards content matching user intent, not keyword density
- **One primary keyword, one secondary keyword max per page** — more looks like stuffing
- **Be specific** — "Track every tweak to your recipe" beats "Recipe management tool"
- **Match the page** — if the meta description promises something the page doesn't deliver, bounce rate hurts rankings
- **No filler** — cut "just", "simply", "easily", "really", "very"
- **No superlatives** — "best", "top", "#1" without evidence reads as spam to both users and Google

## Human writing rules

No emdashes | No unlock / unleash / elevate / empower / leverage / streamline | No "discover the power of" or "take X to the next level" | Contractions fine | Plain words ("find" not "discover", "free" not "at no cost") | If it sounds like AI copy, rewrite it

## Dynamic page templates

For pages driven by content (recipe, profile, post, product):
- **Title template**: `{recipe.title} by {author.name} | Brand`
- **Description template**: with placeholder variables and fallback text for empty fields
- **Fallback strategy**: what to show when optional fields (description, tags) are absent

## Skip

- Code changes before approval
- Technical SEO (broken sitemaps, crawl blocks, schema markup) → use `/seo-audit` for those
- Keyword stuffing or black-hat density tricks
- Generic marketing language
- Duplicate descriptions across pages
