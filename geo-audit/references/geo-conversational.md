# GEO: Conversational Alignment

AI users phrase queries naturally ("quick weeknight dinner"). Content matching conversational patterns → higher citation rate.

## Objectives

Match natural queries | Answer common questions explicitly | Conversational phrases | Context for AI recommendations | Question-based discovery (Who/What/When/Where/Why/How)

## Search

### Recipe Titles
**Grep**: `title.*recipe|recipes.*title`
**Check**: Conversational patterns
- ✅ "Quick 30-Minute Chicken Stir-Fry", "Easy One-Pot Pasta", "Crispy Oven Potatoes (No Deep Frying)"
- ❌ "Chicken Stir-Fry Recipe", "Pasta", "Roasted Potatoes"
**Why**: Match how users ask ("find me quick chicken dinner")

### Descriptions
**Grep**: `description|summary|excerpt`
**Check conversational hooks**: Time ("ready in 30min"), Difficulty ("beginner-friendly"), Use case ("weeknight dinners"), Problem-solving ("pantry staples"), Audience ("kid-friendly")

**Bad**: "A delicious pasta dish."
**Good**: "Quick 20-min pasta for weeknights. Pantry staples, one pot, minimal cleanup. Kid-friendly, adaptable for picky eaters."

### Keywords/Tags
**Grep**: `tags|keywords|categories`
**Check**: Intent-based ("quick dinner", "meal prep") | Occasion ("weeknight", "party") | Dietary | Equipment ("one-pot", "air fryer") | Skill ("beginner")
**Avoid**: Generic singles ("pasta", "chicken") w/o context

### Question-Answering
**Check**: What (in description), Why (benefits), When (occasions), Who (audience), How (time), Where (origin)

### Common Query Patterns
**Time**: "quick <30min", "make-ahead", "what can I cook tonight"
**Ingredient**: "recipes using chicken", "leftover rice ideas"
**Dietary**: "gluten-free dessert", "vegan dinner"
**Skill**: "easy for beginners", "impress guests"
**Equipment**: "one-pot", "instant pot", "no-oven"

### Notes/Tips
**Grep**: `notes|tips|variations`
**Check**: Substitutions ("No buttermilk? Use milk+lemon"), Issues ("dough sticky → add flour 1 tbsp"), Storage ("3 days fridge"), Scaling ("doubles easily"), Make-ahead ("prep night before")

### Search Metadata
**Grep**: `generateMetadata`
**Verify**: Title has qualifiers (quick/easy/healthy) | Description answers what+why | Keywords cover natural queries

## Common Issues

### Generic Titles
**Before**: "Chocolate Chip Cookies"
**After**: "Chewy Chocolate Chip Cookies (Brown Butter, No Mixer)"
**Impact**: High - AI can't match specific needs

### Feature-Focused Descriptions
**Before**: "Uses unique technique for flavor."
**After**: "Beginner-friendly, no special equipment. Ready in 45min w/ hands-off baking."

### Missing Context
Recipe doesn't answer "when would I make this?"
**Fix**: Add occasion tags, serving suggestions, use case

### Technical-Only Tags
**Before**: `["baking", "dessert", "cookies"]`
**After**: `["easy baking", "kid-friendly dessert", "chewy cookies", "beginner-friendly", "no mixer"]`

## Fixes

### Enrich Titles
```typescript
// Add qualifiers
title: `${recipe.title}${recipe.totalTime <= 30 ? " (Quick 30-Min)" : ""}${recipe.tags.includes("beginner") ? " - Easy" : ""}`
```

### Conversational Description
```typescript
function generateDescription(recipe: Recipe): string {
  const parts = [
    recipe.description,
    recipe.totalTime <= 30 && "Ready <30min.",
    recipe.difficulty === "beginner" && "Beginner-friendly.",
    recipe.tags.includes("one-pot") && "One pot, minimal cleanup.",
    recipe.tags.includes("meal-prep") && "Perfect for meal prep.",
    recipe.dietaryTags.length && `${recipe.dietaryTags.join(", ")}.`,
  ].filter(Boolean)
  return parts.join(" ")
}
```

### Intent Tags
```typescript
const intentTags = {
  quick: recipe.totalTime <= 30,
  "make-ahead": recipe.tags.includes("meal-prep"),
  "crowd-pleaser": recipe.servings >= 8,
  "budget-friendly": recipe.tags.includes("pantry"),
  "one-pot": recipe.equipment?.length === 1,
}
```

### FAQ Section
```tsx
<section aria-labelledby="faq">
  <h2 id="faq">Common Questions</h2>
  <dl>
    <dt>Can I make ahead?</dt>
    <dd>Yes! Prep through step 3, refrigerate overnight...</dd>
    <dt>Substitute {ingredient}?</dt>
    <dd>Try {alt1} or {alt2}...</dd>
    <dt>Store leftovers?</dt>
    <dd>Airtight {days} days...</dd>
  </dl>
</section>
```
**Why**: AI extracts FAQ content directly

## Validation

Ask ChatGPT/Perplexity:
- "Quick weeknight dinner recipe"
- "Beginner-friendly dessert"
- "One-pot <30min meals"

Check if recipes appear w/ correct context.

## Best Practices

Titles answer what+why | Descriptions solve problems | Tags match queries ("quick dinner" not "dinner") | FAQ sections | Use cases | Problem-solving notes | Audience clarity

## Refs

Google conversational search | Perplexity content guidelines | ChatGPT plugin best practices
