# SEO Audit

Comprehensive SEO audit framework analyzing technical SEO, content optimization, performance, structured data, mobile responsiveness, and site architecture. Supports both codebase analysis and live site validation.

## What It Does

Systematic SEO review across all critical ranking factors:

- **Technical SEO** - Meta tags, canonical URLs, redirects, robots.txt, sitemap.xml
- **Content Optimization** - Headings, keywords, readability, duplicate content
- **Performance** - Core Web Vitals, page speed, image optimization, caching
- **Structured Data** - Schema.org markup, JSON-LD, rich snippets
- **Mobile Responsiveness** - Viewport, touch targets, mobile-first design
- **Crawlability** - Indexing issues, crawl budget, URL structure
- **Accessibility** - Alt text, semantic HTML, ARIA labels
- **Site Architecture** - Internal linking, sitemap structure, navigation

## When to Use

- Launching new website or major redesign
- SEO performance declining
- Low search visibility
- Pre-deployment SEO validation
- Competitive SEO analysis
- Framework migration (CSR → SSR)

## Framework Support

Optimized detection and recommendations for:
- **Next.js** - App Router metadata, generateMetadata(), sitemap.ts, robots.ts
- **Nuxt** - useHead(), useSeoMeta(), nuxt.config.ts
- **Gatsby** - gatsby-plugin-react-helmet, gatsby-config.js
- **Astro** - Built-in SEO support
- SSR/SSG vs CSR detection

## Approach

1. **Detect environment** - Identify framework and rendering strategy
2. **Discover audit modules** - Map relevant SEO checks
3. **Determine scope** - Full audit, quick scan, or custom selection
4. **Optional live validation** - WebFetch for actual rendered output
5. **Execute reviews** - Codebase + optional live site checks
6. **Report findings** - Impact-based (Critical → Info) with fixes

## Output Format

```
**[IMPACT] SEO Issue**
- Category: [technical/schema/perf/content/arch/mobile/a11y/crawl]
- Location: `file:line` or [URL]
- Issue: [specific problem]
- Impact: [ranking/crawl/UX effect]
- Evidence: [code or live example]
- Fix: [implementation steps + code]
- Reference: [Google/Schema.org/Web.dev documentation]
```

## Impact Ratings

- **Critical** - Blocks crawlers, missing core meta, broken sitemap, severe performance
- **High** - Suboptimal meta tags, missing schema, poor headings, slow loads
- **Medium** - Missing alt text, weak internal linking, minor performance issues
- **Low** - Minor optimizations, redundant tags
- **Info** - Best practices, enhancement opportunities

## Example Usage

```bash
/seo-audit
```

Options:
1. Full Audit (all SEO checks)
2. Quick Scan (technical + schema + performance)
3. Custom (select specific audit areas)

Optional: Provide live URL for dual-mode validation

## Key Checks

### Technical SEO
- Title tags (50-60 chars, unique per page)
- Meta descriptions (150-160 chars)
- Canonical URLs (prevent duplicate content)
- Open Graph / Twitter Cards
- robots.txt configuration
- XML sitemap generation

### Structured Data
- JSON-LD Schema.org markup
- Article, Product, Recipe, Event schemas
- Breadcrumb navigation
- Organization/Person schema
- Rich snippet eligibility

### Performance
- Largest Contentful Paint (LCP < 2.5s)
- First Input Delay (FID < 100ms)
- Cumulative Layout Shift (CLS < 0.1)
- Image optimization (WebP, lazy loading)
- Minification, compression

### Content
- H1-H6 heading hierarchy
- Keyword optimization (not stuffing)
- Content length and depth
- Duplicate content detection
- Readability scores

## Deliverables

- Executive summary with SEO score
- Prioritized findings (Critical → Info)
- Quick wins (easy, high-impact fixes)
- Roadmap (Priority 1/2/3 with effort estimates)
- Framework-specific implementation guidance

## For Employers

Demonstrates:
- Full-stack SEO understanding (technical + content)
- Framework-specific optimization knowledge
- Performance optimization expertise (Core Web Vitals)
- Structured data implementation (Schema.org)
- Business impact awareness (SEO → traffic → revenue)
