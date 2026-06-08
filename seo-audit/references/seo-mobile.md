# Mobile Responsiveness & Usability

Mobile-first indexing: viewport, responsive design, touch targets, mobile performance, forms.

## Checks

| Area | Search | Issues → Impact |
|------|--------|-----------------|
| **Viewport** | `grep -rn 'name="viewport"' --include="*.tsx"` | Missing (Crit), Fixed width (Crit), user-scalable=no (High), max-scale=1 (Med) |
| **Responsive** | `grep -rn "sm:\|md:\|lg:" --include="*.tsx"` | Fixed px widths (High), No breakpoints (Crit), Text <16px (High), Horizontal scroll (High) |
| **Tap targets** | `grep -rn "<button\|<Link" --include="*.tsx"` | Buttons <44px (High), Links too close (Med), Tiny icons (High) |
| **Touch nav** | `grep -rn "hover:\|onMouseEnter" --include="*.tsx"` | Hover-only dropdowns (High), Menu items close (Med), No mobile nav (High) |
| **Mobile images** | `grep -rn "sizes=\|srcset=" --include="*.tsx"` | Desktop images on mobile (High), No lazy load (Med) |
| **Forms** | `grep -rn "<input\|type=" --include="*.tsx"` | Generic text inputs (Med), Fields <44px (High), Missing autocomplete (Med) |
| **Intrusive** | `grep -rn "modal\|popup" --include="*.tsx" -i` | Full-screen popups on mobile (High - Google penalty) |

## Patterns

**Viewport**:
```tsx
// ❌ width=1024 or user-scalable=no
// ✅ export const metadata = { viewport: { width: 'device-width', initialScale: 1 } }
```

**Responsive**:
```tsx
// ❌ <div style={{ width: '1200px' }}>
// ✅ <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
// ✅ <div className="w-full max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
// ✅ <h1 className="text-2xl sm:text-3xl lg:text-4xl">  (min 16px base)
```

**Tap targets** (min 48x48px):
```tsx
// ❌ <button className="w-6 h-6 p-1">×</button>
// ✅ <button className="min-w-12 min-h-12 p-3"><X className="w-6 h-6" /></button>
// ✅ <Link className="inline-block py-3 px-4 min-h-12">Home</Link>
```

**Touch-friendly nav**:
```tsx
// ❌ hover-only: <div className="group"><div className="hidden group-hover:block">
// ✅ click/tap: <button onClick={() => setOpen(!open)} className="min-h-12">Menu</button>
```

**Mobile images**:
```tsx
<Image sizes="(max-width: 640px) 100vw, (max-width: 1024px) 50vw, 1200px" ... />
// Mobile-specific: const isMobile = useMediaQuery('(max-width: 768px)')
```

**Forms**:
```tsx
// ❌ <input type="text" name="email" />
// ✅ <input type="email" autoComplete="email" className="min-h-12 text-base" />
// ✅ <input type="tel" autoComplete="tel" />
// ✅ <input type="number" inputMode="decimal" />
```

**Non-intrusive**:
```tsx
// ❌ <Modal open={true} className="fixed inset-0">  (full-screen popup)
// ✅ <div className="sticky bottom-0 p-4">  (banner, dismissible)
```

## Mobile-First Indexing

Google uses mobile version for ranking → Content parity mobile/desktop → Structured data on mobile → Navigation works w/o JS where possible
