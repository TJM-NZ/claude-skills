# GEO Audit Skill

GEO (Generative Engine Optimization) audit for AI-powered search (ChatGPT, Perplexity, Gemini, Claude).

## GEO vs SEO

| Aspect | SEO | GEO |
|--------|-----|-----|
| Goal | Search ranking | AI citation/understanding |
| Schema | Minimal | Rich, complete |
| Content | Keywords | Conversational |
| Authority | Backlinks | Citations/credentials |
| Format | Page depth | Answer-ready |
| Access | Sitemap | API/embed |

## Scan Modules

8 categories:

1. **Structured Data** - Complete Schema.org Recipe, rich author, multiple image ratios, nutrition, ratings
2. **Content Format** - Heading hierarchy, concise paragraphs, numbered steps, semantic HTML, alt text
3. **Conversational** - Natural queries, intent tags, FAQ, problem-solving
4. **Citations** - Author attribution, source credits, technique refs, supported claims
5. **Metadata** - Multi-category tags, timing breakdown, scaling, equipment, alternatives, storage
6. **Answer-Ready** - Key info upfront, quick stats, scannable lists, FAQ, jump links, print-friendly
7. **Authority** - Credentials, reviews, ratings, testing, engagement, version history
8. **Embeddability** - JSON APIs, RSS, oEmbed, sitemap, AI-friendly robots.txt, CORS

## Usage

`/geo-audit` or "run GEO audit" or "optimize for AI search"

## Modes

- **Full** - All 8 categories
- **Quick** - Structured data + content + conversational
- **Custom** - User picks

## Report

**Executive Summary**: Framework, findings count, AI score (1-10), assessment

**Findings**: Impact (Critical→Info) | Category | `file:line` | Issue | AI effect | Evidence + Fix | Refs

**Quick Wins**: 3-5 easy/high-impact

**Roadmap**: P1 (Critical+High) | P2 (Med) | P3 (Low+Info) - Effort: S/M/L

## Impact Ratings

**Critical**: Missing Schema.org, broken JSON-LD, no author
**High**: Incomplete schema, weak conversational, missing metadata
**Medium**: Missing alternatives, weak citations
**Low**: Minor enhancements
**Info**: Best practices

## Live Validation

Optional URL → WebFetch pages, validate JSON-LD, check metadata, test APIs, verify sitemap/robots

## Best Practices

Rich Schema.org | Author credentials | Conversational content | Answer-ready format | Citations | Trust signals | API access | AI crawler-friendly

## Refs

Schema.org Recipe | Google AI content guidelines | OpenAI/Perplexity best practices | E-A-T framework
