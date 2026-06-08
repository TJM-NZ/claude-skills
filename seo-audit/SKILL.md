---
name: seo-audit
description: Comprehensive SEO audit framework - analyzes technical SEO, content optimization, performance, structured data, mobile responsiveness, and site architecture. Supports both codebase analysis and live site validation.
when_to_use: User mentions "SEO audit", "SEO check", "search optimization", "SEO review", "check SEO", "improve SEO", "SEO analysis", "search engine optimization"
allowed-tools: Read Grep Glob Bash(git rev-parse *) Bash(git status *) Bash(git diff *) WebFetch
---

Professional SEO specialist — technical SEO, Core Web Vitals, structured data, search best practices.

## Approach

Evidence-based (file:line refs) | Impact-focused (Critical/High/Med/Low/Info) | Actionable fixes | Framework-aware | Dual-mode (codebase + optional live validation)

## Step 1: Detect Environment

**Glob patterns**:
- Framework: `package.json`, `next.config.*`, `nuxt.config.*`, `gatsby-config.*`, `vite.config.*`
- SEO files: `sitemap.{xml,ts,js}`, `robots.{txt,ts,js}`, `app/layout.*`, `pages/_app.*`

**Read**: Identify framework (Next.js/Nuxt/Gatsby/Astro) → note SSR/SSG capability

## Step 2: Discover Scans

Glob `seo-*.md` in `/home/teo-mcarthur/.claude/skills/seo-audit/references` → build scan list

## Step 3: Determine Scope

If user specified → proceed. Else offer:
1. **Full Audit** - all scans
2. **Quick Scan** - technical + schema + performance
3. **Custom** - user picks

## Step 4: Optional Live Site

If URL provided, offer to fetch/validate: homepage, sample pages, sitemap.xml, robots.txt, rendered metadata

## Step 5: Load References

**Full**: Glob + Read all `seo-*.md`
**Quick**: Read `seo-technical.md`, `seo-schema.md`, `seo-performance.md`
**Custom**: Map user request → filenames → Read

## Step 6: Execute Reviews

Per reference file:
1. Read completely
2. Search codebase (Grep/Glob/Read per guidance)
3. Identify issues
4. Document findings
5. Optional: WebFetch live validation

**Context notes**:
- Next.js: `app/` metadata, `generateMetadata()`, `robots.ts`, `sitemap.ts`
- Nuxt: `useHead()`, `useSeoMeta()`, `nuxt.config.ts`
- Gatsby: `gatsby-plugin-react-helmet`, `gatsby-config.js`
- SSR/SSG → meta in initial HTML | CSR → must SSR for crawlers
- Dev `noindex` okay | focus production patterns

## Step 7: Report

### Executive Summary
```
SEO Audit Report | Generated: [date] | Framework: [detected]
Scans: [list] | Findings: Critical X, High Y, Med Z, Low W, Info V
Assessment: [1-2 sentence summary]
```

### Findings Format
```
**[IMPACT] Title**
- Category: [technical/schema/perf/content/arch/mobile/a11y/crawl]
- Location: `file:line` or [URL]
- Issue: [specific problem]
- Impact: [ranking/crawl/UX effect]
- Evidence: ```code```
- Fix: [steps] ```corrected```
- Ref: [Google/Schema.org/Web.dev link]
```

Group by impact (Critical first) → category

### Quick Wins
3-5 easy/high-impact fixes

### Roadmap
Priority 1 (Critical+High) | Priority 2 (Med) | Priority 3 (Low+Info)
Format: [Issue] - Effort: [S/M/L]

### Recommendations
Content strategy | Technical improvements | Performance wins | Monitoring setup

## Step 8: Offer Assistance

Fix Critical/High? | Add Schema.org markup? | Create SEO utilities? | Setup monitoring?

## Impact Ratings

- **Critical**: Blocks crawlers, missing core meta, broken sitemap, severe perf
- **High**: Suboptimal meta, missing schema, poor headings, slow loads
- **Medium**: Missing alt text, weak linking, minor perf, mobile gaps
- **Low**: Minor optimizations, redundant tags
- **Info**: Best practices, enhancements

## Quality Standards

- File:line or URL for all findings
- Critical/High explain ranking impact
- Concrete fixes with code examples
- Group duplicates
- Framework-specific advice

## Skip

Keyword stuffing | Black-hat | Admin/dashboard issues | Dev-only configs clearly marked
