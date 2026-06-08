# GEO: Structured Data

AI models rely on Schema.org for content understanding. Rich, complete markup → higher AI citability.

## Objectives

Complete Recipe schema | Rich author data + credentials | Multiple image ratios | Nutrition data | Ratings/reviews | Categories/cuisine | Ingredient alternatives

## Search

### JSON-LD Location
**Grep**: `@type.*Recipe|application/ld\+json` (files_with_matches)
**Check**: `<script type="application/ld+json">` | Server-rendered (not client-only) | Valid JSON

### Required Fields (Critical)
- `@context: "https://schema.org"`
- `@type: "Recipe"`
- `name`, `image` (array, 3+ ratios), `author` (Person w/ details)
- `datePublished`, `description` (160-320 chars)
- `recipeIngredient` (array), `recipeInstructions` (HowToStep array)

### High Priority Fields
**Grep**: `prepTime|cookTime|recipeYield|recipeCategory|recipeCuisine|nutrition|aggregateRating`

- `prepTime`, `cookTime`, `totalTime` (ISO 8601: `PT15M`)
- `recipeYield` (servings)
- `recipeCategory` (meal type), `recipeCuisine`
- `keywords` (tags)
- `nutrition` (NutritionInformation: calories, protein, etc.)
- `aggregateRating` (rating + count)
- `review` (array of Review objects)

### Medium Priority
- `suitableForDiet` (GlutenFreeDiet, VeganDiet, etc.)
- `tool` (equipment)
- `video` (VideoObject)
- `isPartOf` (collection)

### Author Richness
**Grep**: `@type.*Person`
**Check**: `name`, `url`, `image`, `jobTitle`/`description`, `sameAs` (social profiles)

### Organization Data
**Grep**: `@type.*Organization`
**Check**: Brand name, logo, url, `sameAs`

### Images
Multiple ratios (16x9, 4x3, 1x1) | Min 1200px width | Absolute URLs | `ImageObject` w/ width/height

## Common Issues

### Missing Fields
**Impact**: Critical (name/image/ingredients/instructions) | High (timing/nutrition/ratings)
**Grep**: Schema generation functions → compare vs full Recipe spec

### Client-Side Only
**Grep**: `useState.*schema|useEffect.*ld\+json`
**Issue**: AI crawlers may not execute JS
**Fix**: Move to `generateMetadata()` or server component

### Invalid JSON
**WebFetch**: Live page → validate w/ Schema.org validator
**Errors**: Unescaped quotes, missing commas, incorrect date formats (use ISO 8601)

### Sparse Data
Minimal fields only → AI prefers rich, complete data
**Check**: Field count in schema vs full spec

## Fixes

### Enrich Schema
```typescript
// Before: minimal
{ "@type": "Recipe", name, image }

// After: rich
{
  "@context": "https://schema.org",
  "@type": "Recipe",
  name: recipe.title,
  description: recipe.description,
  image: [
    { "@type": "ImageObject", url: img_16x9, width: 1200, height: 675 },
    { "@type": "ImageObject", url: img_4x3, width: 1200, height: 900 },
    { "@type": "ImageObject", url: img_1x1, width: 1200, height: 1200 },
  ],
  author: {
    "@type": "Person",
    name: author.name,
    url: `${base}/chefs/${author.id}`,
    image: author.avatar,
    description: author.bio,
    sameAs: author.socialLinks,
  },
  datePublished: recipe.createdAt,
  dateModified: recipe.updatedAt,
  prepTime: `PT${recipe.prepTime}M`,
  cookTime: `PT${recipe.cookTime}M`,
  totalTime: `PT${recipe.prepTime + recipe.cookTime}M`,
  recipeYield: `${recipe.servings} servings`,
  recipeCategory: recipe.category,
  recipeCuisine: recipe.cuisine,
  keywords: recipe.tags.join(", "),
  recipeIngredient: recipe.ingredients.map(i => i.text),
  recipeInstructions: recipe.steps.map((s, i) => ({
    "@type": "HowToStep",
    position: i + 1,
    text: s.text,
  })),
  nutrition: {
    "@type": "NutritionInformation",
    calories: `${recipe.nutrition.calories} calories`,
    proteinContent: `${recipe.nutrition.protein}g`,
    fatContent: `${recipe.nutrition.fat}g`,
    carbohydrateContent: `${recipe.nutrition.carbs}g`,
  },
  aggregateRating: recipe.reviewCount > 0 ? {
    "@type": "AggregateRating",
    ratingValue: recipe.averageRating,
    reviewCount: recipe.reviewCount,
  } : undefined,
  suitableForDiet: recipe.dietaryTags.map(tag => `https://schema.org/${tag}Diet`),
}
```

### Server-Side Rendering
```typescript
// app/recipes/[slug]/page.tsx
export async function generateMetadata({ params }): Promise<Metadata> {
  const recipe = await getRecipe(params.slug)
  return {
    other: { 'ld+json': JSON.stringify(getRecipeSchema(recipe)) },
  }
}
```

## Validation

Google Rich Results Test | Schema.org Validator | AI test: "summarize recipe at [URL]" (ChatGPT/Perplexity)

## Refs

Schema.org Recipe: https://schema.org/Recipe | Person: https://schema.org/Person | Google Recipe Guidelines
