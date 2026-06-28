# GEO: Embeddability & API Access

### JSON API Endpoints
**Grep**: `app/api|pages/api|route.ts`
**Check**: `/api/recipes` (list), `/api/recipes/[id]` (single), `/api/recipes/search` | Query params (filters, pagination)

**Verify**: Clean JSON (not HTML) | Full recipe data (ingredients, steps, metadata) | Pagination | CORS headers | Rate limit headers

**Response**:
```json
{
  "recipe": {
    "id": "uuid",
    "title": "...",
    "author": { /* object */ },
    "ingredients": [ /* array */ ],
    "instructions": [ /* array */ ],
    "metadata": { "prepTime": 15, "cookTime": 30, "tags": [] },
    "images": [ /* URLs */ ],
    "ratings": { "average": 4.7, "count": 234 }
  }
}
```

### Embeddable Widgets
**Grep**: `embed|widget|iframe`
**Check**: Embed endpoint (`/embed/recipes/[id]`) | Minimal layout (no chrome) | Responsive | Copy embed code

```html
<iframe src="https://remixipe.com/embed/recipes/abc123" width="100%" height="600"></iframe>
```
**Why**: AI tools may embed recipes

### RSS/Atom Feeds
**Grep**: `rss|feed|atom|xml`
**Check**: `/feed` or `/rss` | Recent recipes | Full content or summaries | Proper XML | Images, authors, dates
**Why**: AI aggregators consume RSS

### oEmbed Support
**Grep**: `oembed`
**Check discovery**:
```html
<link rel="alternate" type="application/json+oembed" href="https://remixipe.com/oembed?url=..." title="Recipe" />
```
**Why**: Rich previews in AI-generated content

### Sitemap
**Grep**: `sitemap|sitemap.xml|sitemap.ts`
**Check**: All recipes | Regular updates | Last modified dates | Priority/frequency | Images

**Next.js**:
```typescript
// app/sitemap.ts
export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const recipes = await getAllRecipes()
  return recipes.map(r => ({
    url: `https://remixipe.com/recipes/${r.slug}`,
    lastModified: r.updatedAt,
    changeFrequency: 'weekly',
    priority: 0.8,
  }))
}
```

### Robots.txt
**Grep**: `robots.txt|robots.ts`
**Check**: Allows AI crawlers | Sitemap reference | Appropriate disallow rules (admin, api internals)

```
User-agent: *
Allow: /

User-agent: GPTBot
Allow: /

User-agent: ChatGPT-User
Allow: /

User-agent: PerplexityBot
Allow: /

User-agent: Claude-Web
Allow: /

Disallow: /api/
Disallow: /admin/
Disallow: /auth/

Sitemap: https://remixipe.com/sitemap.xml
```
**Why**: Explicit AI bot allowance

### CORS Config
**Grep**: `cors|Access-Control-Allow-Origin`
**Check**: API allows cross-origin (for embedding) | Proper headers | Not fully open (security)

**Next.js API**:
```typescript
export async function GET(request: Request) {
  const data = await getRecipeData()
  return NextResponse.json(data, {
    headers: {
      'Access-Control-Allow-Origin': '*',
      'Access-Control-Allow-Methods': 'GET',
      'Access-Control-Allow-Headers': 'Content-Type',
    },
  })
}
```

### Rate Limiting
**Grep**: `rate.*limit|ratelimit|throttle`
**Check**: Reasonable limits | Clear headers | 429 on exceeded
**Good**: Allow AI crawlers higher limits

## Common Issues

### No Public API
Recipe data only via HTML scraping
**Impact**: High - AI prefers JSON
**Fix**: Create public API endpoints

### Blocked AI Crawlers
robots.txt blocks GPTBot, PerplexityBot
**Impact**: Critical - invisible to AI
**Fix**: Explicitly allow AI crawlers

### Missing Sitemap
No sitemap or static/outdated
**Impact**: High - AI can't discover all content
**Fix**: Dynamic sitemap generation

### No Embed Support
Can't embed on other sites
**Impact**: Medium - missed integrations
**Fix**: Minimal embed endpoint

### Restrictive CORS
API blocks all cross-origin
**Impact**: Medium - prevents integrations
**Fix**: Allow CORS for GET on public data

### Aggressive Rate Limiting
Limits block crawlers
**Impact**: High - AI can't index
**Fix**: Higher limits for AI bots (User-Agent)

### No RSS Feed
**Impact**: Low-Med - some aggregators use RSS
**Fix**: Generate RSS/Atom

## Fixes

### Create Recipe API
```typescript
// app/api/recipes/[id]/route.ts
import { NextResponse } from "next/server"
import { getRecipe } from "@/lib/recipes/queries"

export async function GET(request: Request, { params }: { params: { id: string } }) {
  try {
    const recipe = await getRecipe(params.id)
    if (!recipe || recipe.status !== "published") {
      return NextResponse.json({ error: "Not found" }, { status: 404 })
    }

    const apiData = {
      recipe: {
        id: recipe.id,
        title: recipe.title,
        slug: recipe.slug,
        description: recipe.description,
        author: { id: recipe.author.id, name: recipe.author.name, avatar: recipe.author.avatar, profile_url: `https://remixipe.com/chefs/${recipe.author.id}` },
        ingredients: recipe.ingredients.map(i => ({ item: i.item, quantity: i.quantity, unit: i.unit, notes: i.notes })),
        instructions: recipe.steps.map((s, i) => ({ step: i + 1, text: s.text })),
        timing: { prep_minutes: recipe.prepTime, cook_minutes: recipe.cookTime, total_minutes: recipe.prepTime + recipe.cookTime },
        servings: recipe.servings,
        images: recipe.images?.map(img => img.url) || [],
        tags: recipe.tags,
        ratings: { average: recipe.averageRating, count: recipe.reviewCount },
        published_at: recipe.createdAt,
        updated_at: recipe.updatedAt,
        url: `https://remixipe.com/recipes/${recipe.slug}`,
      }
    }

    return NextResponse.json(apiData, {
      headers: {
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Methods': 'GET',
        'Cache-Control': 'public, s-maxage=3600, stale-while-revalidate=86400',
      },
    })
  } catch (error) {
    return NextResponse.json({ error: "Internal error" }, { status: 500 })
  }
}
```

### Embed Endpoint
```typescript
// app/embed/recipes/[id]/page.tsx
import { getRecipe } from "@/lib/recipes/queries"
import { RecipeCardMinimal } from "@/components/recipe/recipe-card-minimal"

export default async function RecipeEmbedPage({ params }: { params: { id: string } }) {
  const recipe = await getRecipe(params.id)
  if (!recipe) return <div>Not found</div>

  return (
    <html>
      <head><meta name="robots" content="noindex" /></head>
      <body className="embed"><RecipeCardMinimal recipe={recipe} /></body>
    </html>
  )
}
```

### RSS Feed
```typescript
// app/feed/route.ts
import { getAllRecipes } from "@/lib/recipes/queries"
import RSS from "rss"

export async function GET() {
  const recipes = await getAllRecipes({ limit: 50 })

  const feed = new RSS({
    title: "Remixipe - Latest Recipes",
    description: "GitHub for recipes",
    feed_url: "https://remixipe.com/feed",
    site_url: "https://remixipe.com",
    language: "en",
    ttl: 60,
  })

  recipes.forEach(r => {
    feed.item({
      title: r.title,
      description: r.description,
      url: `https://remixipe.com/recipes/${r.slug}`,
      author: r.author.name,
      date: r.createdAt,
      enclosure: r.coverImage ? { url: r.coverImage, type: "image/jpeg" } : undefined,
      categories: r.tags,
    })
  })

  return new Response(feed.xml(), {
    headers: { "Content-Type": "application/xml; charset=utf-8", "Cache-Control": "public, s-maxage=3600" },
  })
}
```

### robots.txt
```typescript
// app/robots.ts
export default function robots(): MetadataRoute.Robots {
  return {
    rules: [
      { userAgent: "*", allow: "/", disallow: ["/api/", "/admin/", "/auth/"] },
      { userAgent: ["GPTBot", "ChatGPT-User", "PerplexityBot", "Claude-Web"], allow: "/", disallow: ["/admin/", "/auth/"] },
    ],
    sitemap: "https://remixipe.com/sitemap.xml",
  }
}
```

### oEmbed
```typescript
// app/oembed/route.ts
export async function GET(request: Request) {
  const { searchParams } = new URL(request.url)
  const url = searchParams.get("url")
  if (!url) return NextResponse.json({ error: "URL required" }, { status: 400 })

  const recipe = await getRecipeByUrl(url)
  if (!recipe) return NextResponse.json({ error: "Not found" }, { status: 404 })

  return NextResponse.json({
    version: "1.0",
    type: "rich",
    provider_name: "Remixipe",
    provider_url: "https://remixipe.com",
    title: recipe.title,
    author_name: recipe.author.name,
    author_url: `https://remixipe.com/chefs/${recipe.author.id}`,
    html: `<iframe src="https://remixipe.com/embed/recipes/${recipe.id}" width="600" height="400"></iframe>`,
    width: 600,
    height: 400,
    thumbnail_url: recipe.coverImage,
  })
}
```

**Discovery**:
```tsx
// generateMetadata
other: { "oembed-url": `https://remixipe.com/oembed?url=${encodeURIComponent(recipeUrl)}` }
```

## Validation

**API**: `curl https://remixipe.com/api/recipes/[id]` → check JSON, CORS
**robots.txt**: Google validator
**Sitemap**: Submit to Search Console
**Embed**: Test iframe on external site

