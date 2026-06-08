# GEO: Citations & Attribution

AI models favor clear attribution, sources, citations. Proper credit → authority → trustworthiness → higher citation rate.

## Objectives

Author attribution (all recipes) | Recipe inspiration/source credits | Ingredient sources (optional) | Technique references | External citations for claims | Licensing/attribution for adapted recipes

## Search

### Author Attribution
**Grep**: `author|creator|by|posted.*by`
**Check**: Name displayed | Link to profile | Bio/credentials visible | Photo | Consistency across recipes
**Impact**: Critical - AI heavily weights author identity

### Recipe Source/Inspiration
**Grep**: `inspiration|adapted.*from|based.*on|source|original`
**Check**: "Inspired by..." or "Adapted from..." | Links to originals | Credit to cookbooks/chefs/restaurants | Cultural/regional origin

```tsx
<aside className="inspiration">
  <p>Inspired by <a href="...">Julia Child's technique</a>, adapted for modern kitchens.</p>
  <p>Original: <cite>Mastering French Cooking</cite></p>
</aside>
```

### Ingredient Sources
**Check**: Specialty ingredient links (where to buy) | Brand recommendations (when relevant) | Local/sustainable callouts
**Why**: Adds context, helps AI answer "where do I find this?"

### Technique References
**Grep**: `technique|method.*learn|tutorial`
**Check**: Links to technique guides | Video references | Cross-links to related recipes

### Claims w/ Citations
**Check**: Nutritional data sources (USDA, tools) | Health claims backed by sources | Historical/cultural claims referenced
**Issue**: Unsupported claims ("healthiest version") → reduced trust
**Fix**: Remove or cite ("According to [source], this preserves nutrients...")

### License/Permissions
**Grep**: `license|copyright|rights|permission`
**Check**: Permission statements | CC licensing | "Used with permission"

## Common Issues

### Anonymous Content
No author attribution
**Impact**: Critical - AI deprioritizes anonymous
**Fix**: Add author component to all recipes

### Uncredited Adaptations
Clearly adapted but no credit
**Impact**: High - appears to claim others' work
**Fix**: Add inspiration field to schema

### Broken Links
**WebFetch**: Check author profiles → 404s
**Impact**: Medium - reduces trust

### Missing `<cite>`
**Before**: `<p>From "Joy of Cooking" by Irma Rombauer</p>`
**After**: `<p>From <cite>Joy of Cooking</cite> by <span class="author">Irma Rombauer</span></p>`

### Unsupported Health Claims
"Proven to boost immunity" w/o sources
**Impact**: High - AI flags unsupported claims
**Fix**: Remove or add citation

## Fixes

### Add Inspiration Field
**Schema**:
```sql
ALTER TABLE recipes
ADD inspiration_text TEXT,
ADD inspiration_url TEXT,
ADD inspiration_type TEXT; -- 'cookbook'|'chef'|'restaurant'|'family'|'cultural'
```

**Display**:
```tsx
{recipe.inspiration && (
  <aside>
    <h3>Inspiration</h3>
    <p>{recipe.inspiration.text}
      {recipe.inspiration.url && <a href={url}>View original</a>}
    </p>
  </aside>
)}
```

### Enhanced Author Schema
**JSON-LD**:
```typescript
author: {
  "@type": "Person",
  name: author.name,
  url: `${base}/chefs/${author.id}`,
  image: author.avatar,
  description: author.bio,
  sameAs: author.socialLinks, // Social = authority
  jobTitle: author.credentials,
  knowsAbout: author.specialties,
}
```

### Citation Footer
```tsx
{recipe.sources?.length > 0 && (
  <footer className="citations">
    <h3>Sources</h3>
    <ol>
      {recipe.sources.map((s, i) => (
        <li key={i}>
          <cite>{s.title}</cite>{s.author && ` by ${s.author}`}
          {s.url && <a href={s.url} target="_blank">View</a>}
        </li>
      ))}
    </ol>
  </footer>
)}
```

### Technique Links
Auto-link techniques:
```typescript
function enrichTechniqueLinks(text: string): string {
  const techniques = ["kneading", "folding", "tempering"]
  return techniques.reduce((acc, tech) =>
    acc.replace(new RegExp(`\\b${tech}\\b`, "gi"),
      `<a href="/resources/techniques/${tech}">${tech}</a>`), text)
}
```

## Schema.org
```typescript
{
  "@type": "Recipe",
  "isBasedOn": recipe.inspiration?.url,
  "author": { /* rich object */ },
  "citation": recipe.sources?.map(s => ({
    "@type": "CreativeWork",
    name: s.title,
    author: s.author,
    url: s.url,
  })),
}
```

## Validation

AI test: "Who created [recipe] on [site]?" → should return author accurately
Link health: Check external links, verify non-404

## Best Practices

Always credit | Link to profiles | Semantic markup (`<cite>`, `<author>`) | Cite claims | Credit techniques | Acknowledge culture | Permission for adaptations

## Refs

Schema.org CreativeWork: https://schema.org/CreativeWork | citation: https://schema.org/citation | Creative Commons
