# GEO: Rich Metadata

AI models use metadata for context understanding and query matching. Rich, structured metadata → improved discoverability.

## Objectives

Comprehensive tags (dietary, cuisine, meal, occasion, skill, equipment) | Detailed timing (prep/cook/rest/total) | Serving/yield + scaling | Ingredient alternatives | Equipment lists | Seasonal/occasion | Storage/reheating | Batch size/scaling

## Search

### Tag Completeness
**Grep**: `tags|categories|keywords`
**Check categories**:
- Dietary: gluten-free, vegan, vegetarian, dairy-free, nut-free, low-carb, keto, paleo
- Cuisine: Italian, Mexican, Thai, Japanese, Mediterranean, Southern
- Meal: breakfast, lunch, dinner, snack, dessert, appetizer
- Occasion: weeknight, weekend, holiday, party, potluck, meal-prep
- Skill: beginner, intermediate, advanced
- Equipment: one-pot, slow-cooker, instant-pot, air-fryer, no-bake
- Time: quick (<30min), moderate (30-60min), project (60+min)
- Texture: crispy, creamy, crunchy, smooth
- Flavor: spicy, sweet, savory, tangy, umami

**Issue**: 1-3 generic tags ("dinner", "pasta")
**Impact**: High - AI can't match specific needs

### Timing Granularity
**Grep**: `prepTime|cookTime|totalTime|restTime|waitTime`
**Check**: Separate prep/cook | Passive time (marinating, rising, cooling) | Active vs hands-off distinction | Per-step timing (optional)

```typescript
timing: {
  prep: number,      // Active prep
  cook: number,      // Active cooking
  passive: number,   // Marinating, rising, cooling
  total: number,     // Sum
}
```

### Yield & Scaling
**Grep**: `servings|yield|serves|makes`
**Check**: Servings count | Serving size ("12 cookies", "8 slices") | Scaling notes ("doubles easily", "halve w/ caution")

```typescript
yield: {
  servings: 4,
  servingSize: "1 generous bowl",
  scalingNotes: "Doubles easily. Halving: use 1 egg yolk only.",
}
```

### Equipment Lists
**Grep**: `equipment|tools|requires|you.*need`
**Check**: Structured list (not buried) | Special equipment noted | Alternatives ("or use fork")
**Impact**: Med-High - users search "recipes without [equipment]"

```typescript
equipment: [
  { name: "Stand mixer", required: false, alternative: "Hand mixer or whisk" },
  { name: "9-inch springform pan", required: true },
]
```

### Ingredient Alternatives
**Grep**: `substitut|alternative|instead|swap|replace`
**Check**: Structured alternatives per ingredient | Dietary swaps | Availability alternatives

```typescript
ingredients: [
  {
    item: "Buttermilk",
    quantity: "1 cup",
    alternatives: [
      "1 cup milk + 1 tbsp lemon juice (sit 5min)",
      "1 cup plain yogurt thinned w/ milk",
    ]
  }
]
```

### Storage
**Grep**: `storage|store|keep|leftover|refrigerat|freeze`
**Check**: Fridge life (days) | Freezer suitability + duration | Reheating | Make-ahead

```typescript
storage: {
  refrigerator: { duration: "3-4 days", notes: "Airtight container" },
  freezer: { duration: "3 months", notes: "Wrap tightly: plastic → foil" },
  reheating: "Microwave 2-3min or oven 350°F 10min",
}
```

### Seasonal/Occasion
**Grep**: `season|occasion|holiday|when.*serve`
**Check tags**: Seasons (spring, summer, fall, winter) | Holidays (thanksgiving, christmas) | Events (game-day, picnic, bbq, brunch)

### Allergen Info
**Grep**: `allergen|contains|warning`
**Check**: Common allergens flagged (nuts, dairy, eggs, soy, shellfish, wheat) | Cross-contamination warnings | Structured data
**Impact**: High - safety-critical, AI filtering

## Common Issues

### Sparse Tags
**Before**: `["dinner", "pasta"]`
**After**: `["quick-dinner", "weeknight", "one-pot", "italian", "vegetarian-adaptable", "kid-friendly", "<30min"]`
**Impact**: High

### Missing Timing Breakdown
Only total time → can't distinguish active vs passive
**Fix**: Split prep/cook/passive

### No Scaling Guidance
Servings noted but no scaling tips
**Fix**: Add scaling notes

### Buried Equipment
Special equipment mid-recipe, not upfront
**Fix**: Structured list at top

### Text-Only Substitutions
Alternatives in paragraphs → AI can't parse reliably
**Fix**: Structured per-ingredient

### No Storage Info
**Fix**: Add storage metadata

## Fixes

### Enhanced Tags
**DB**:
```sql
CREATE TABLE recipe_tags (
  recipe_id UUID,
  tag VARCHAR,
  category VARCHAR, -- 'dietary'|'cuisine'|'meal'|'equipment'|'skill'|'occasion'
  PRIMARY KEY (recipe_id, tag)
);
```

**Auto-suggest**:
```typescript
function suggestTags(recipe: Recipe): string[] {
  const suggestions = []
  if (recipe.totalTime <= 30) suggestions.push("quick", "<30min")
  if (recipe.equipment?.length === 1) suggestions.push("one-pot")
  if (!recipe.ingredients.some(i => i.item.match(/meat|chicken|beef/i)))
    suggestions.push("vegetarian")
  return suggestions
}
```

### Timing Structure
```typescript
interface RecipeTiming {
  prep: number;
  cook: number;
  passive?: number;
  total: number;
  activeTime?: number; // prep + active cook (excl passive)
}
```

### Equipment Schema
```typescript
interface Equipment {
  name: string;
  required: boolean;
  alternative?: string;
  notes?: string;
}
```

### Storage Metadata
```typescript
interface StorageInfo {
  refrigerator?: { duration: string; notes?: string };
  freezer?: { suitable: boolean; duration?: string; notes?: string };
  roomTemp?: { duration: string; notes?: string };
  reheating?: string;
  makeAhead?: string;
}
```

## Schema.org Integration
```typescript
{
  "@type": "Recipe",
  "suitableForDiet": recipe.dietaryTags.map(t => `https://schema.org/${t}Diet`),
  "recipeCategory": recipe.categories,
  "recipeCuisine": recipe.cuisine,
  "keywords": recipe.tags.join(", "),
  "tool": recipe.equipment.map(e => e.name),
  "totalTime": `PT${recipe.timing.total}M`,
  "prepTime": `PT${recipe.timing.prep}M`,
  "cookTime": `PT${recipe.timing.cook}M`,
  "recipeYield": `${recipe.yield.servings} servings`,
}
```

## Validation

Tag coverage: 3+ tags from different categories per recipe
AI test: "Quick vegetarian dinner", "Make-ahead gluten-free desserts", "Instant Pot meal prep"

## Best Practices

Multi-category tags | Timing breakdown | Scaling guidance | Equipment upfront | Structured alternatives | Storage info | Allergen flags | Occasion context

## Refs

Schema.org Recipe: https://schema.org/Recipe | USDA allergens
