# GEO: Content Format

AI models prefer clear, scannable, well-structured content. Formatting → parsing quality → citability.

## Objectives

Heading hierarchy (H1→H2→H3, no skips) | Concise paragraphs (2-4 sentences) | Lists (numbered/bulleted) | Key info upfront | Semantic HTML | Alt text | Readable prose

## Search

### Heading Hierarchy
**Grep**: `<h[1-6]|className.*text-[1-6]xl`
**Check**: Single H1 | H2 for major sections (Ingredients, Instructions) | H3 for subsections | No skipped levels
**Why**: AI uses headings for structure, section extraction

### Paragraph Length
**Grep**: `<p>|<Text`
**Issue**: Dense blocks (5+ sentences) → harder parsing
**Ideal**: 2-4 sentences/paragraph, one idea each

### Lists for Structure
**Check**: Ingredients as `<ul>`/`<ol>` (not text) | Steps as `<ol>` (not paragraphs) | Equipment as list
**Grep**: `ingredients.*map|steps.*map` → verify list rendering

### Inverted Pyramid
**Check**: Key info first (what/why) → Details (ingredients/method) → Supporting (history/variations)
**Grep**: Read recipe page → component order
**Issue**: Backstory before recipe → AI may miss data

### Semantic HTML
**Grep**: `<div|<span` vs `<article|<section|<aside|<nav` ratio
**Check**: Recipe in `<article>` | Ingredients/instructions in `<section>` | Notes in `<aside>` | Timestamps in `<time datetime>`

### Alt Text
**Grep**: `<img|<Image|next/image` → check alt attribute
**Check**: Descriptive alt (not "image"/filename) | Empty `alt=""` only for decorative
**Impact**: High - AI uses alt for visual context

### Tables for Data
**Grep**: `<table|<Table`
**Better**: Tables for nutrition, timing, alternatives (vs paragraphs)

## Common Issues

### Dense Blocks
**Before**: 5+ sentence paragraph
**After**: Split into 2-4 sentence chunks

### Skipped Headings
**Before**: `<h1>` → `<h3>` (skip H2)
**After**: `<h1>` → `<h2>` → `<h3>`

### Ingredients as Prose
**Before**: `<p>You'll need 500g flour, 350ml water...</p>`
**After**: `<ul><li>500g flour</li><li>350ml water</li></ul>`

### Missing Alt
**Before**: `<Image src="/recipe.jpg" />`
**After**: `<Image src="/recipe.jpg" alt="Golden-brown sourdough loaf cooling on wire rack" />`

### Non-Semantic
**Before**: `<div class="recipe"><div class="ingredients">...`
**After**: `<article><section aria-labelledby="ingredients">...`

### Backstory Upfront
**Before**: Long story → Recipe
**After**: Recipe header → Quick stats → Ingredients → Steps → `<details>` story `</details>`

## Fixes

### Split Paragraphs
```tsx
// Before
<p>Long paragraph with multiple ideas and sentences...</p>

// After
<p>First idea, concise.</p>
<p>Second idea, clear.</p>
```

### Semantic Structure
```tsx
<article className="recipe">
  <header>
    <h1>{recipe.title}</h1>
    <p>{recipe.description}</p>
  </header>

  <section aria-labelledby="ingredients-heading">
    <h2 id="ingredients-heading">Ingredients</h2>
    <ul>{recipe.ingredients.map(i => <li>{i.text}</li>)}</ul>
  </section>

  <section aria-labelledby="instructions-heading">
    <h2 id="instructions-heading">Instructions</h2>
    <ol>{recipe.steps.map((s, i) => <li key={i}>{s.text}</li>)}</ol>
  </section>

  <aside>{/* Notes/tips */}</aside>
</article>
```

### Alt Text
```tsx
<Image
  src={recipe.image}
  alt="[Dish appearance + key visual details]"
  width={1200}
  height={800}
/>
```

## Validation

AI parsing test: "summarize recipe at [URL]" (ChatGPT) → check accuracy
axe DevTools: semantic HTML, headings, alt text
WAVE: visual structure feedback

## Best Practices

One idea/paragraph | Lists > prose | Headings for structure | Key info first | Semantic HTML | Descriptive alt | Tables for data | Short sentences

## Refs

WCAG: https://www.w3.org/WAI/WCAG21/quickref/ | MDN Semantic HTML
