---
name: copy-seo
description: Generate SEO copy - meta titles, meta descriptions, OG tags, and structured data text
disable-model-invocation: true
---

Generate SEO copy for page metadata, social sharing, and search engine visibility. Do NOT make any code changes - present copy options for the user to review, choose, and refine.

## Before you start

Read `PRODUCT.md`, `CLAUDE.md`, terminology docs. Load `.claude/docs/tone.md` — if missing, suggest `/copy-tone` first. Check existing metadata exports (`metadata`, `generateMetadata`, or `<head>`) and the route structure to know which public pages need coverage.

## What to produce

### Meta titles

For each page, provide **2 options**.

Format: `Page Description | Brand Name` or `Page Description - Brand Name`

Rules:
- **50-60 characters max** (Google truncates at ~60)
- **Front-load the important words** - "Sourdough Bread Recipe | Remixipe" not "Remixipe | Sourdough Bread Recipe"
- **No keyword stuffing** - one primary keyword per title
- **Homepage title can break the pattern** - it's the one place you can lead with brand + tagline
- **Dynamic pages need templates** - recipe pages, profile pages, etc. should use the content itself (recipe title, user name)

### Meta descriptions

For each page, provide **2 options**.

Rules:
- **120-155 characters** (Google truncates at ~155 on desktop, ~120 on mobile)
- **Include a call to action or value proposition** - "Browse 500+ community recipes" not "This is a recipe website"
- **Match search intent** - what would someone searching for this page actually want to know?
- **No quotes or special characters** - they can break in SERPs
- **Unique per page** - duplicate descriptions across pages hurt rankings

### Open Graph tags (social sharing)

For key pages (homepage, explore, shared content), provide:
1. **og:title** (same as or shorter than meta title)
2. **og:description** (can be more casual than meta description, 1-2 sentences)
3. **og:type** (website, article, profile, etc.)
4. **Image guidance** - recommend what the OG image should show (not the copy itself, but art direction notes)

### Dynamic page templates

For pages that use dynamic content (recipe pages, user profiles, collections), provide:
1. **Title template** - with placeholder variables (e.g. `{recipe.title} by {author.name} | Brand`)
2. **Description template** - with placeholders and fallback text for missing fields
3. **Guidance on fallbacks** - what to show when optional fields (description, tags) are empty

## Copywriting rules

- **Write for humans first, search engines second** - Google rewards content that matches user intent, not keyword density
- **No keyword stuffing** - one primary keyword, one secondary keyword max per page
- **Be specific** - "Track every change to your recipes" beats "Recipe management tool"
- **Match the page content** - if the meta description promises something the page doesn't deliver, bounce rate goes up and rankings go down
- **No filler words** - cut "just", "simply", "easily", "really", "very"
- **No superlatives** - "best", "top", "#1" without proof looks spammy to both users and Google

## Write like a human, not an AI

No emdashes | no unlock/unleash/elevate/streamline | no "discover the power of.../take X to the next level". Contractions fine. Plain words ("find" not "discover", "free" not "at no cost"). Meta descriptions show in SERPs — if it reads like marketing fluff, people won't click.

## After presenting options

1. Present options grouped by page.
2. Ask the user which options they prefer for each page.
3. Offer to refine - adjust keywords, length, or framing based on feedback.
4. Only apply changes to code when the user explicitly approves final copy.
