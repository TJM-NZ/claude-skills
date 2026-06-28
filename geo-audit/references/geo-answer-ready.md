# GEO: Answer-Ready Format

### Information Hierarchy
**Read**: Recipe page components → order

**Ideal**:
1. Above fold: Title, image, quick stats (time/servings/difficulty)
2. Summary: 1-2 sentences (what it is, why make it)
3. Metadata: Dietary tags, cuisine, equipment
4. Ingredients: Scannable list w/ quantities
5. Instructions: Numbered steps
6. Supporting: Notes, tips, variations, storage
7. Story: Optional backstory (below main content)

**Issue**: Story before recipe → AI extracts wrong info
**Impact**: High

### Quick Stats Box
**Grep**: `prep.*time|cook.*time|servings|difficulty`
**Check**: Prep, cook, total, servings, difficulty, calories (optional) - scannable at glance

```tsx
<div className="quick-stats">
  <dl>
    <dt>Prep</dt><dd>15min</dd>
    <dt>Cook</dt><dd>30min</dd>
    <dt>Total</dt><dd>45min</dd>
    <dt>Servings</dt><dd>4</dd>
    <dt>Difficulty</dt><dd>Easy</dd>
  </dl>
</div>
```

### Ingredient Formatting
**Grep**: `ingredients.*map|recipeIngredient`
**Check**: Structured list (not paragraph) | Consistent quantities | Group headings (Wet/Dry, Sauce/Base) | Checkboxes (optional)

**Bad**: "You'll need 2 cups flour, 1 tsp salt..."
**Good**: List w/ quantities

### Numbered Instructions
**Grep**: `instructions.*map|steps.*map`
**Check**: Numbered list | One action/step | Timing within ("cook 10min") | Visual cues ("until golden")

**Bad**: Compound paragraph w/ multiple actions
**Good**: `1. Cream butter 3min. 2. Add eggs one at a time...`

### FAQ Section
**Grep**: `faq|question|can.*i|how.*do`
**Check**: Dedicated FAQ | Common questions | Structured Q&A

**Common**: Make ahead? | Freeze? | Substitute [ingredient]? | Store leftovers? | Double/halve? | Why did [X] happen?

```tsx
<section aria-labelledby="faq">
  <h2 id="faq">FAQ</h2>
  <dl>
    <dt>Can I make ahead?</dt>
    <dd>Yes! Prep through step 5, refrigerate 24h...</dd>
  </dl>
</section>
```

### Print-Friendly
**Grep**: `print|@media.*print`
**Check**: Print stylesheet | Removes nav/ads | Recipe on single page | QR back to recipe (optional)

### Jump Links
**Grep**: `#ingredients|#instructions|jump.*to`
**Check**: "Jump to Recipe" (skips story) | Section anchors | "Back to Top"

### Summary/TL;DR
Short summary at top

**Good**: "Quick weeknight pasta 20min using pantry staples."
**Bad**: Missing or buried

## Common Issues

### Story-First Layout
Long backstory before recipe
**Impact**: High - AI misses data
**Fix**: Move story to bottom or `<details>`

### Paragraph Instructions
Steps as prose, not numbered
**Impact**: High - AI can't extract reliably
**Fix**: Ordered list

### Missing Quick Stats
Time/servings buried in text
**Impact**: Med-High - AI can't answer "how long?"
**Fix**: Stats box at top

### No FAQ
**Impact**: Medium - AI can't answer "Can I freeze?"
**Fix**: Add FAQ section

### Compound Steps
Multiple actions in one step
**Impact**: Medium - hard to parse

**Before**: "Preheat oven, grease pan, mix dry, cream butter, combine..."
**After**: Separate numbered steps

### No Print Styles
**Impact**: Low-Med - missed clean export

## Fixes

### Reorder Layout
```tsx
<article className="recipe">
  <RecipeHeader title={recipe.title} image={recipe.image} />
  <QuickStats {...recipe} />
  <Summary>{recipe.description}</Summary>
  <JumpLinks />

  <section id="ingredients">
    <h2>Ingredients</h2>
    <IngredientList items={recipe.ingredients} />
  </section>

  <section id="instructions">
    <h2>Instructions</h2>
    <ol>{recipe.steps.map((s, i) => <li key={i}>{s.text}</li>)}</ol>
  </section>

  <section id="notes">
    <h2>Notes</h2>
    <RecipeNotes notes={recipe.notes} />
  </section>

  <section id="faq">
    <h2>FAQ</h2>
    <FAQ items={recipe.faqs} />
  </section>

  {recipe.story && (
    <details>
      <summary>About This Recipe</summary>
      <div dangerouslySetInnerHTML={{ __html: recipe.story }} />
    </details>
  )}
</article>
```

### Quick Stats Component
```typescript
interface QuickStatsProps {
  prepTime: number;
  cookTime: number;
  servings: number;
  difficulty: "easy" | "medium" | "hard";
  calories?: number;
}
```

### FAQ Schema
```sql
CREATE TABLE recipe_faqs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  recipe_id UUID REFERENCES recipes(id) ON DELETE CASCADE,
  question TEXT NOT NULL,
  answer TEXT NOT NULL,
  order_index INT DEFAULT 0
);
```

### Print Styles
```css
@media print {
  nav, footer, .ad, .comments, .share { display: none !important; }
  .recipe { max-width: 100%; font-size: 12pt; }
  .recipe-steps li { page-break-inside: avoid; margin-bottom: 0.5em; }
  .ingredients, .instructions { page-break-inside: avoid; }
}
```

### Jump Links
```tsx
<nav className="jump-links" aria-label="Recipe nav">
  <a href="#ingredients">Ingredients</a>
  <a href="#instructions">Instructions</a>
  <a href="#notes">Notes</a>
</nav>
```

## Validation

AI test: "Extract ingredients/steps from [URL]" → check accuracy
Print preview: Recipe fits 1-2 pages, clean, readable

