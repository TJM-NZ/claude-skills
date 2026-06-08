# Accessibility & SEO

Semantic HTML, ARIA, keyboard nav, color contrast — affects both UX and rankings.

## Why A11y → SEO

Semantic HTML helps crawlers understand structure | Alt text provides image context | Heading hierarchy benefits screen readers + bots | Better engagement metrics | Google quality signal

## Checks

| Area | Search | Issues → Impact |
|------|--------|-----------------|
| **Semantic HTML** | `grep -rn "<main\|<nav\|<article>" --include="*.tsx"` | Div soup (High), Missing `<main>` (Med), Multiple `<main>` (Med), Nav not `<nav>` (Med) |
| **ARIA labels** | `grep -rn "aria-label\|aria-labelledby" --include="*.tsx"` | Icon buttons no label (High), Multiple navs no labels (Med), Decorative icons not hidden (Low) |
| **Keyboard nav** | `grep -rn "focus:\|outline-none" --include="*.tsx"` | outline:none no alternative (High), Positive tabindex (High), Interactive divs not buttons (High) |
| **Form labels** | `grep -rn "<label\|<input" --include="*.tsx"` | Inputs w/o labels (Crit), Placeholder-only (High), Errors not associated (High) |
| **Image alt** | `grep -rn '<img' --include="*.tsx" \| grep -v 'alt='` | Missing alt (Crit), alt=filename (High), "image of" redundancy (Low), Decorative w/ alt (Low) |
| **Color contrast** | `grep -rn "text-gray\|bg-gray" --include="*.tsx"` | Low contrast <4.5:1 (High), Color-only indicators (Med) |
| **Headings** | `grep -rn "<h[1-6]" --include="*.tsx"` | Multiple h1 (High), Skipping levels (Med), Empty (High) |
| **Link text** | `grep -rn 'click here\|read more' --include="*.tsx" -i` | Generic text (Med), New window no warning (Med) |

## Patterns

**Semantic HTML**:
```tsx
<header><nav aria-label="Main">...</nav></header>
<main>
  <article><h1>...</h1><p>...</p></article>
  <aside>Related</aside>
</main>
<footer><nav aria-label="Footer">...</nav></footer>
```

**ARIA**:
```tsx
// ❌ <button><X /></button> | <nav>...</nav><nav>...</nav>
// ✅ <button aria-label="Close"><X aria-hidden="true" /></button>
// ✅ <nav aria-label="Main">...</nav> | <nav aria-label="Footer">...</nav>
```

**Keyboard**:
```tsx
// ❌ <div onClick={...}>Click</div> | <button className="outline-none">
// ✅ <button onClick={...}>Click</button> | <button className="outline-none focus:ring-2">
// ✅ Custom: <div role="button" tabIndex={0} onClick={...} onKeyDown={(e) => e.key === 'Enter' && ...}>
```

**Form labels**:
```tsx
// ❌ <input placeholder="Email" />
// ✅ <label htmlFor="email">Email</label>
//    <input id="email" type="email" required aria-invalid={error} aria-describedby={error && "email-error"} />
//    {error && <span id="email-error" role="alert">{error}</span>}
```

**Alt text**:
```tsx
// ❌ alt="IMG_1234.jpg" | alt="image of cookies"
// ✅ alt="Stack of chocolate chip cookies on white plate"
// ✅ Decorative: alt="" role="presentation"
```

**Contrast**:
```tsx
// ❌ text-gray-400 on bg-gray-100 (low contrast)
// ✅ text-gray-900 on bg-white | text-gray-100 on bg-gray-900
// ✅ Error: color + icon + text, not color alone
```

**Link text**:
```tsx
// ❌ <a href="/recipe">Click here</a>
// ✅ View the <Link href="/recipe">chocolate chip cookie recipe</Link>
// ✅ External: <a target="_blank" rel="noopener" aria-label="Link (opens new window)">Text <ExternalIcon aria-hidden /></a>
```

## WCAG Standards

Normal text: 4.5:1 contrast (AA) | Large text 18pt+: 3:1 | Tap targets: min 44x44px | Keyboard accessible | Labels on all inputs
