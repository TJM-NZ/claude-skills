---
name: geo-audit
description: GEO (Generative Engine Optimization) audit - AI discoverability, structured data richness, conversational alignment, citations, LLM-ready formatting for ChatGPT/Perplexity/Gemini/Claude
when_to_use: User mentions "GEO audit", "GEO check", "generative engine optimization", "AI search optimization", "LLM discoverability", "optimize for AI", "ChatGPT optimization", "Perplexity optimization"
allowed-tools: Read Grep Glob Bash(git rev-parse *) Bash(git status *) Bash(git diff *) WebFetch
---

GEO specialist: AI discoverability, structured data, conversational optimization, citation signals

## Approach

Evidence-based (`file:line`) | Impact ratings (Critical→High→Med→Low→Info) | Actionable fixes | Framework-aware | Codebase + optional live validation

## Step 1: Detect Environment

**Glob**: `package.json`, `next.config.*`, `nuxt.config.*`, `gatsby-config.*`, `vite.config.*`, `app/**/*layout.*`, `app/api/**/*`
**Read**: Framework → SSR/SSG capability → structured data approach

## Step 2: Discover Scans

Glob `geo-*.md` in `/home/teo-mcarthur/.claude/skills/geo-audit/references`

## Step 3: Scope

User-specified → proceed | Else offer:
1. **Full** - all scans
2. **Quick** - structured-data + content-format + conversational
3. **Custom** - user picks

## Step 4: Optional Live Site

URL provided → WebFetch: homepage, recipe pages, API responses, JSON-LD, embed formats

## Step 5: Load References

**Full**: Read all `geo-*.md`
**Quick**: Read `geo-structured-data.md`, `geo-content-format.md`, `geo-conversational.md`
**Custom**: User selection → Read matched files

## Step 6: Execute Reviews

Per reference file: Read → Search (Grep/Glob/Read per guidance) → Identify issues → Document → Optional WebFetch validation

**Context**:
- Next.js: `generateMetadata()`, JSON-LD in layouts/pages, API routes
- AI preference: JSON-LD > microdata | complete Schema.org > minimal | clear structure
- Citations: author attribution, source links, inspiration credits
- Conversational: match natural queries ("quick weeknight dinner")
- Answer-ready: key info upfront, numbered steps, timing visible

## Step 7: Report

### Executive Summary
```
GEO Audit | [date] | Framework: [name]
Scans: [list] | Findings: Critical X, High Y, Med Z, Low W, Info V
AI Score: [1-10] | [1-2 sentence assessment]
```

### Findings
```
**[IMPACT] Title**
Category: [category] | Location: `file:line` or URL
Issue: [problem] | Impact: [AI effect]
Evidence: ```code```
Fix: ```corrected``` + [steps]
Ref: [link]
```
Group: Impact desc → Category

### Quick Wins
3-5 easy/high-impact fixes

### Roadmap
P1 (Critical+High) | P2 (Med) | P3 (Low+Info)
Format: [Issue] - Effort: S/M/L

### Recommendations
Schema enrichment | Content format | Citations | API/embed

## Step 8: Offer Assistance

Fix Critical/High? | Add Schema.org? | Optimize format? | Setup citations? | Create APIs?

## Impact Ratings

**Critical**: Missing Schema.org, broken JSON-LD, no author, unparseable
**High**: Incomplete schema, weak conversational, missing metadata, poor structure
**Medium**: Missing alternatives, weak citations, limited context
**Low**: Minor enhancements, formatting tweaks
**Info**: Best practices, advanced optimizations

## Quality Standards

`file:line` or URL | Critical/High → explain AI impact | Concrete fixes + code | Group duplicates | Framework-specific | Reference AI docs (OpenAI, Gemini, Perplexity)

## GEO ≠ SEO

AI understanding/citation vs search ranking:
Schema richness > minimal | Conversational > keywords | Citations > backlinks | Exports > sitemap | Answer-ready > depth

## Skip

Keyword density | Link building | Page rank | Admin/dashboard | Dev-only configs
