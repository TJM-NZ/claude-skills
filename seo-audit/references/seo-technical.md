# Technical SEO

Meta tags, canonical URLs, heading hierarchy, HTML semantics, language, robots, viewport.

## Checks

| Area | Search Pattern | Issues → Impact |
|------|----------------|-----------------|
| **Title tags** | `grep -r "export.*metadata\|<title>" --include="*.tsx"` | Missing (Crit), Duplicate (High), >70 chars (Med), Generic "React App" (High), No dynamic gen for content (High) |
| **Meta desc** | `grep -r "description:" --include="*.tsx"` | Missing (High), Duplicate (Med), >170 chars (Low), Generic (Med) |
| **Open Graph** | `grep -r "og:title\|openGraph" --include="*.tsx"` | Missing on shareable (High), Relative img URLs (High), No images (Med), No Twitter cards (Med) |
| **Canonical** | `grep -r "canonical\|alternates.*canonical" --include="*.tsx"` | Missing (High), Relative URLs (Med), Wrong env URL (Crit) |
| **H1** | `grep -rn "<h1" --include="*.tsx"` | Multiple h1s (High), No h1 (Crit), Doesn't match title (Med) |
| **H2-H6** | `grep -rn "<h[2-6]" --include="*.tsx"` | Skipping levels (Med), Used for styling not structure (Med) |
| **Semantic HTML** | `grep -rn "<article>\|<section>\|<nav>" --include="*.tsx"` | Div soup (Med), Missing `<main>` (Med), Multiple `<main>` (Med) |
| **Lang attr** | `grep -rn '<html.*lang' --include="*.tsx"` | Missing lang (High), Wrong code (Med), No hreflang for i18n (High) |
| **Robots meta** | `grep -rn 'meta.*name="robots"' --include="*.tsx"` | Public marked noindex (Crit), Draft not noindex (High) |
| **Viewport** | `grep -rn 'name="viewport"' --include="*.tsx"` | Missing (Crit), Fixed width (High), user-scalable=no (Med) |

## Quick Patterns

**Next.js App Router metadata**:
```tsx
// ✅ Good
export const metadata: Metadata = {
  title: 'Recipe Title - Author | Site',
  description: 'Description 150-160 chars...',
  openGraph: {
    title: 'Recipe Title',
    images: [{ url: 'https://absolute-url.com/img.jpg', width: 1200, height: 630 }]
  },
  alternates: { canonical: 'https://absolute-url.com/path' },
  robots: { index: true, follow: true }
}
```

**Heading hierarchy**:
```tsx
<h1>Page Title</h1>         // One per page
<h2>Section</h2>            // Logical order, no skipping
  <h3>Subsection</h3>
<h2>Another Section</h2>
```

**Semantic HTML**:
```tsx
<header><nav aria-label="Main">...</nav></header>
<main>
  <article><h1>...</h1><p>...</p></article>
  <aside>Related content</aside>
</main>
<footer><nav aria-label="Footer">...</nav></footer>
```

