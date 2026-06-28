# Content Optimization

Headings, alt text, internal linking, content depth, keyword usage, readability.

## Checks

| Area | Search | Issues → Impact |
|------|--------|-----------------|
| **H1** | `grep -rn "<h1" --include="*.tsx"` | Multiple h1s (High), No h1 (Crit), Doesn't match title (Med), Generic "Welcome" (Med) |
| **H2-H6** | `grep -rn "<h[2-6]" --include="*.tsx"` | Skipping levels (h2→h4) (Med), Empty headings (High), For styling not structure (Med) |
| **Alt text** | `grep -rn '<img' --include="*.tsx" \| grep -v 'alt='` | Missing alt (High), alt=filename (High), alt="image" (High), Keyword stuffing (Med), >125 chars (Low) |
| **Internal links** | `grep -rn "<Link\|<a href" --include="*.tsx"` | "Click here" text (Med), No related content links (Med), Orphan pages (High) |
| **Content length** | Analyze page components | Thin content <200 words on important pages (High), Placeholder/lorem ipsum (Crit) |
| **Keyword usage** | Check H1, URL, meta, first 100 words | Keyword stuffing (High), No keywords in prominent places (Med), URL slug ≠ H1 (Med) |
| **Readability** | Check paragraph length | Long paragraphs >5 sentences (Low), No lists for steps (Med), Walls of text (Med) |
| **Dates** | `grep -rn "publishedAt\|updatedAt\|<time>" --include="*.tsx"` | Missing dates (Med), Dates in schema ≠ visible (High) |
| **UGC** | `grep -rn "Review\|Comment" --include="*.tsx"` | Reviews client-side only (High), No aggregateRating (Med) |

## Patterns

**Headings**:
```tsx
<h1>Page Title</h1>              // One per page, primary keyword
<h2>Section with Keywords</h2>   // Logical order
  <h3>Subsection</h3>
<h2>Another Section</h2>
```

**Alt text**:
```tsx
// ❌ <img alt="IMG_1234.jpg" /> or alt="image"
// ✅ <img alt="Stack of chocolate chip cookies on white plate" />
// ✅ Decorative: <img alt="" role="presentation" />
```

**Internal linking**:
```tsx
// ❌ <Link href="/recipe">Click here</Link>
// ✅ View the <Link href="/recipe">chocolate chip cookie recipe</Link>
// ✅ <aside><h3>Related</h3><Link>Similar Recipe</Link></aside>
```

**Readability**:
```tsx
// ❌ Long paragraph with 10+ sentences...
// ✅ Short paragraphs (3-4 sentences)
// ✅ <ul><li>Bullet points for scannability</li></ul>
// ✅ <ol><li>Numbered lists for steps</li></ol>
```

**Dates**:
```tsx
<time dateTime={recipe.publishedAt}>Published {formatDate(recipe.publishedAt)}</time>
{recipe.updatedAt !== recipe.publishedAt && <time>Updated {formatDate(recipe.updatedAt)}</time>}
```

