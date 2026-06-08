# Performance & Core Web Vitals

Page speed, image optimization, bundle size, fonts, layout shift — ranking factors.

## Core Web Vitals

| Metric | Good | Poor | Issues → Impact |
|--------|------|------|-----------------|
| **LCP** (Largest Contentful Paint) | ≤2.5s | >4s | LCP image lazy-loaded (Crit), Not prioritized (High), Oversized (High) |
| **INP** (Interaction to Next Paint) | ≤200ms | >500ms | Heavy JS bundles (High), No code splitting (Med), Blocking scripts (High) |
| **CLS** (Cumulative Layout Shift) | ≤0.1 | >0.25 | Images w/o dimensions (High), No placeholders (High), Font shifts (Med) |

## Checks

| Area | Search | Issues → Impact |
|------|--------|-----------------|
| **Images** | `grep -rn "<img\|<Image\|next/image" --include="*.tsx"` | Using `<img>` not framework Image (High), No width/height (High), No lazy loading (Med), No alt (High), Large PNG not WebP (Med) |
| **JS bundles** | `grep -rn "import.*from ['\"]moment\|import.*from ['\"]lodash" --include="*.tsx"` | Importing full libraries (High), No code splitting (Med), No dynamic imports (Med) |
| **Scripts** | `grep -rn "<script\|next/script" --include="*.tsx"` | Sync third-party scripts (Crit), Missing defer/async (High) |
| **CSS** | `grep -rn "content:" tailwind.config.*` | Tailwind not purging (High), Entire CSS loaded per page (Med) |
| **Fonts** | `grep -rn "next/font\|@font-face\|googleapis.com/css" --include="*.tsx"` | CDN fonts no preconnect (Med), Missing font-display:swap (High), Not using next/font (Med) |
| **Lazy loading** | `grep -rn 'loading=\|priority' --include="*.tsx"` | All images eager (Med), Hero lazy (Crit), Below-fold eager (Med) |
| **Responsive images** | `grep -rn "sizes=\|srcset=" --include="*.tsx"` | No srcset/sizes (Med), Desktop images on mobile (High) |

## Quick Fixes

**Images**:
```tsx
// ❌ <img src="/img.jpg" />
// ✅ Next.js Image
<Image src="/img.jpg" alt="Descriptive" width={800} height={600} quality={85} loading="lazy" sizes="(max-width: 768px) 100vw, 50vw" />
// Hero: priority, others: loading="lazy"
```

**Code splitting**:
```tsx
// ❌ import _ from 'lodash'
// ✅ import { debounce } from 'lodash-es'
// ✅ const Editor = dynamic(() => import('react-quill'), { ssr: false })
```

**Scripts**:
```tsx
// ❌ <script src="analytics.js"></script>
// ✅ <Script src="analytics.js" strategy="afterInteractive" />
```

**Fonts**:
```tsx
// ❌ <link href="https://fonts.googleapis.com/css2?family=Inter" />
// ✅ import { Inter } from 'next/font/google'
const inter = Inter({ subsets: ['latin'], display: 'swap' })
```

**Layout shift prevention**:
```tsx
// ❌ <img src="/banner.jpg" /> (no dimensions)
// ✅ <div className="aspect-[16/9] relative"><Image src="/banner.jpg" fill /></div>
```

**Preloading**:
```tsx
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preload" as="image" href="/hero.jpg" />  // LCP image only
```

## Framework Patterns

**Next.js SSG/ISR** (better than SSR for perf):
```tsx
export const revalidate = 3600  // ISR
export async function generateStaticParams() { /* SSG */ }
```

**Bundle analysis**: `@next/bundle-analyzer`, check `webpack-bundle-analyzer`
