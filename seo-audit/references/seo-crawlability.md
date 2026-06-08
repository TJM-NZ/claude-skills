# Crawlability & Indexability

Robots.txt, sitemap, meta robots, SSR, internal links, redirects — can search engines discover and index?

## Checks

| Area | Search | Issues → Impact |
|------|--------|-----------------|
| **Robots.txt** | `find . -name "robots.{txt,ts,js}"` | Missing (Med), Blocks important pages (Crit), Missing sitemap ref (Med), Dev robots.txt in prod (Crit) |
| **Sitemap** | `find . -name "sitemap.{xml,ts,js}"` | Missing (Crit), Static not updated (High), Includes drafts/noindex (High), Missing lastmod (Low) |
| **Meta robots** | `grep -rn 'meta.*name="robots"\|robots.*:' --include="*.tsx"` | Public marked noindex (Crit), Draft not noindex (High), Dev site indexed (High) |
| **Internal links** | `grep -rn "<Link\|<a href" --include="*.tsx"` | Orphan pages (Crit), JS-only links (High), onClick w/o href (High) |
| **SSR** | `grep -rn '"use client"\|useEffect' --include="*.tsx"` | SEO content client-only (Crit), Meta in useEffect (Crit), Schema via JS (High) |
| **Canonicalization** | `grep -rn "canonical\|searchParams" --include="*.tsx"` | Filter URLs dupe content (High), No canonical on params (High) |
| **Redirects** | `grep -rn "redirect\|permanentRedirect" --include="*.tsx"` | Redirect chains (Med), 302 for permanent (Med), Loops (Crit) |

## Patterns

**Robots.txt (Next.js)**:
```ts
// app/robots.ts
export default function robots() {
  return {
    rules: { userAgent: '*', allow: '/', disallow: ['/api/', '/admin/', '/settings/'] },
    sitemap: 'https://site.com/sitemap.xml'
  }
}
```

**Sitemap (Next.js)**:
```ts
// app/sitemap.ts
export default async function sitemap() {
  const recipes = await getPublishedRecipes()  // Only published, no drafts
  return [
    { url: 'https://site.com', lastModified: new Date(), priority: 1 },
    ...recipes.map(r => ({ url: `https://site.com/recipes/${r.slug}`, lastModified: r.updatedAt, priority: 0.7 }))
  ]
}
```

**Meta robots**:
```tsx
// Published
export const metadata = { robots: { index: true, follow: true } }
// Draft/admin
export const metadata = { robots: { index: false, follow: false } }
// Dev env
export const metadata = { robots: { index: process.env.NODE_ENV === 'production' } }
```

**Internal links**:
```tsx
// ❌ <div onClick={() => router.push('/recipes')}>
// ✅ <Link href="/recipes">Recipes</Link>
// ✅ Infinite scroll fallback: <noscript><nav><Link href="?page=2">Next</Link></nav></noscript>
```

**SSR content**:
```tsx
// ❌ Client-side
'use client'
useEffect(() => { fetch('/api/recipe').then(...) }, [])

// ✅ Server-side
export async function generateMetadata({ params }) {
  const recipe = await getRecipe(params.slug)
  return { title: recipe.title, ... }
}
export default async function Page({ params }) {
  const recipe = await getRecipe(params.slug)
  return <article><script type="application/ld+json">...</script>...</article>
}
```

**Canonical for params**:
```tsx
// Filter/sort → always canonical to base
export const metadata = { alternates: { canonical: 'https://site.com/recipes' } }  // ignore searchParams
```

**Redirects (Next.js)**:
```ts
// next.config.js
async redirects() {
  return [
    { source: '/old/:slug', destination: '/new/:slug', permanent: true }  // 301
  ]
}
// Or in component: permanentRedirect(`/new/${slug}`)  (301) | redirect(...)  (307)
```

## Critical Rules

All public pages in sitemap (no drafts/noindex) | Sitemap auto-updates | Public pages indexable | Draft/admin noindex | SSR for SEO content | Links use `<a href>` | No orphans | No redirect chains
