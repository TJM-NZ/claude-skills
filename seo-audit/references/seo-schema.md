# Schema.org & Structured Data

JSON-LD markup for rich results (recipe cards, ratings, breadcrumbs, FAQs, etc.).

## Search Patterns

```bash
grep -rn 'type="application/ld\+json"' --include="*.tsx"
grep -rn '"@type".*"Recipe\|Article\|BreadcrumbList"' --include="*.tsx"
```

## Schema Types & Required Props

| Type | Required | Recommended | Issues → Impact |
|------|----------|-------------|-----------------|
| **Recipe** | @type, name, image[], author, datePublished, description, recipeIngredient[], recipeInstructions[] | prepTime, cookTime, totalTime, recipeYield, aggregateRating, nutrition | Missing schema (Crit), Invalid durations PT format (Med), Images not array (High), Missing rating when reviews exist (Med) |
| **Article** | @type, headline, image[], datePublished, author | dateModified, publisher w/ logo, description | Missing on blog/guides (High), headline >110 chars (Med) |
| **BreadcrumbList** | @type, itemListElement[] w/ position, name, item | - | Missing position (High), Not sequential (Med), Last item has URL (Low) |
| **Organization** | @type, name, url, logo | sameAs[], contactPoint | Missing on homepage (Med) |
| **FAQPage** | @type, mainEntity[] w/ Question+Answer | - | Missing on FAQ pages (Med) |
| **WebSite** | @type, name, url | potentialAction (SearchAction) | Missing search action (Low) |

## Examples (Compact)

**Recipe**:
```json
{
  "@context": "https://schema.org",
  "@type": "Recipe",
  "name": "Title",
  "image": ["https://absolute-url.com/1x1.jpg", "https://absolute-url.com/4x3.jpg"],
  "author": { "@type": "Person", "name": "Author" },
  "datePublished": "2024-01-15",
  "description": "Description",
  "prepTime": "PT15M",
  "cookTime": "PT12M",
  "totalTime": "PT27M",
  "recipeYield": "24 cookies",
  "recipeIngredient": ["2 cups flour", "1 cup sugar"],
  "recipeInstructions": [
    { "@type": "HowToStep", "text": "Step 1" }
  ],
  "aggregateRating": { "@type": "AggregateRating", "ratingValue": "4.8", "reviewCount": "142" }
}
```

**Breadcrumb**:
```json
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    { "@type": "ListItem", "position": 1, "name": "Home", "item": "https://site.com" },
    { "@type": "ListItem", "position": 2, "name": "Recipes", "item": "https://site.com/recipes" },
    { "@type": "ListItem", "position": 3, "name": "Current Page" }
  ]
}
```

## Duration Format

Use ISO 8601: `PT1H30M` (1.5 hours), `PT15M` (15 min), not "1.5 hours" or "15 minutes"

## Multiple Schemas

```tsx
// Option 1: Multiple scripts
<script type="application/ld+json">{JSON.stringify(recipeSchema)}</script>
<script type="application/ld+json">{JSON.stringify(breadcrumbSchema)}</script>

// Option 2: @graph
{ "@context": "https://schema.org", "@graph": [recipeSchema, breadcrumbSchema] }
```

## Validation

Missing required props (Crit) | Wrong data types (High) | Invalid URLs (not absolute) (High) | Invalid date/duration format (Med)

Tools: https://search.google.com/test/rich-results | https://validator.schema.org/

## Framework Patterns

**Next.js App Router**:
```tsx
export async function generateMetadata({ params }): Promise<Metadata> {
  return {
    other: { 'script:ld+json': JSON.stringify(schema) }  // Next 14+
  }
}
// Or render in component: <script type="application/ld+json" dangerouslySetInnerHTML={...} />
```

**Nuxt**:
```vue
<script setup>
useHead({ script: [{ type: 'application/ld+json', children: JSON.stringify(schema) }] })
</script>
```
