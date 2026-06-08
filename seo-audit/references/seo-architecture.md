# Site Architecture & URL Structure

URL patterns, internal linking, navigation, pagination, sitemap, robots.txt, duplicate content.

## Checks

| Area | Search | Issues → Impact |
|------|--------|-----------------|
| **URL structure** | `find app/ -name "page.tsx"` | IDs not slugs (High), Underscores not hyphens (Med), Mixed case (Med), >100 chars (Low), Non-descriptive /page1 (High) |
| **URL depth** | `find app/ -name "page.tsx" \| awk -F/ '{print NF-1}'` | Critical content >4 levels deep (High), No categorization (Med) |
| **Navigation** | `grep -rn "nav\|navbar\|breadcrumb" --include="*.tsx" -i` | Important pages not in nav (Med), Missing breadcrumbs on deep pages (Med) |
| **Related links** | `grep -rn "related\|similar" --include="*.tsx" -i` | No related content links (High), Orphan pages (Crit) |
| **Pagination** | `grep -rn "pagination\|page.*next\|page.*prev" --include="*.tsx" -i` | Missing rel=next/prev (Med), All pages same title (High), No canonical (High) |
| **Canonical** | `grep -r "canonical" --include="*.tsx"` | Filter/sort creates dupes (High), Missing on parameterized (High) |
| **Sitemap** | `find . -name "sitemap.{xml,ts,js}"` | No sitemap (Crit), Static not updated (High), Includes drafts (High), Missing lastmod (Low) |
| **Robots.txt** | `find . -name "robots.{txt,ts,js}"` | No robots.txt (Med), Blocks important pages (Crit), Missing sitemap ref (Med) |

## Patterns

**Good URLs**:
```
✅ /recipes/chocolate-chip-cookies
✅ /collections/summer-desserts
✅ /chefs/jane-smith
❌ /recipes?id=12345
❌ /Recipe_Detail/Chocolate_Chip_Cookies
```

**Breadcrumbs**:
```tsx
<nav aria-label="Breadcrumb">
  <ol>
    <li><Link href="/">Home</Link></li>
    <li><Link href="/recipes">Recipes</Link></li>
    <li aria-current="page">Current</li>
  </ol>
</nav>
```

**Pagination**:
```tsx
// Metadata
export const metadata = {
  title: `Recipes - Page ${page}`,
  alternates: { canonical: `https://site.com/recipes${page > 1 ? `?page=${page}` : ''}` }
}
// Links
{page > 1 && <link rel="prev" href={`?page=${page-1}`} />}
{hasNext && <link rel="next" href={`?page=${page+1}`} />}
```

**Sitemap (Next.js)**:
```ts
// app/sitemap.ts
export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const recipes = await getPublishedRecipes()
  return [
    { url: 'https://site.com', lastModified: new Date(), priority: 1 },
    ...recipes.map(r => ({ url: `https://site.com/recipes/${r.slug}`, lastModified: r.updatedAt, priority: 0.7 }))
  ]
}
```

**Robots.txt (Next.js)**:
```ts
// app/robots.ts
export default function robots(): MetadataRoute.Robots {
  return {
    rules: { userAgent: '*', allow: '/', disallow: ['/api/', '/admin/', '/settings/'] },
    sitemap: 'https://site.com/sitemap.xml'
  }
}
```

**Canonical for filters**:
```tsx
// Always canonical to base, ignore query params
export const metadata = { alternates: { canonical: 'https://site.com/recipes' } }
```

## Navigation Structure

Primary nav → Footer links → Breadcrumbs → Related content → No orphan pages → SSR'd (not JS-only)
