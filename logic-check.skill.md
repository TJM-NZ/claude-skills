# Logic Check Skill

Systematically verify code logic, flow patterns, and convention adherence to catch common issues before they ship.

## Usage

Invoke with `/logic-check` or when user mentions:
- "check the logic"
- "verify the flow"
- "make sure this works correctly"
- "check for issues"
- "systematic check"
- "audit the code"

## Trigger Conditions

Use this skill when:
- User completes a feature and wants to verify correctness
- Before creating a PR
- User reports a bug and wants to prevent similar issues
- Refactoring auth, i18n, or data flow
- Adding new pages or API routes

## What to Check

### 1. Auth Flow Issues

**Middleware red flags** (`src/middleware.ts`):
- ❌ NEVER call `auth()` in middleware (causes infinite redirects)
- ❌ NEVER check session/user in middleware
- ✅ Middleware should ONLY handle locale routing (next-intl)

**Auth redirect loops**:
- Check for circular redirects between locale URLs (`/` ↔ `/en` ↔ `/mi`)
- Verify authenticated homepage redirects use `preferredLocale` correctly
- Ensure deep links respect URL locale (no redirects)
- Look for redirects in layout.tsx that could conflict with middleware

**Session handling**:
- `auth()` calls should be in page/route handlers, NOT middleware
- Check for proper error handling in auth flows
- Verify redirect URLs after sign-in/sign-out are correct

### 2. i18n Issues

**Missing translation keys**:
- Compare `messages/en.json` vs `messages/mi.json` for parity
- Check both files have same keys at same nesting level
- Report any missing translations

**Hardcoded UI strings**:
- Scan JSX/TSX for hardcoded English text not using `t()` or `getTranslations()`
- Exceptions: user-generated content, recipe data, resource markdown
- Look for patterns like: `<button>Save</button>` instead of `<button>{t('common.save')}</button>`
- Check form labels, placeholders, error messages, validation messages

**Locale URL handling**:
- Verify imports use `@/i18n/navigation` NOT `next/navigation`
- Check `Link`, `useRouter`, `usePathname`, `redirect` come from i18n wrapper
- Look for direct pathname construction like `href="/recipes/${slug}"` (should use Link component)

**Homepage redirect logic**:
- Only homepage (`/`) should check user's `preferredLocale`
- Deep links must respect URL locale (no redirects)
- Cookie fallback should only apply to unauthenticated homepage visits

### 3. Data Flow Patterns

**Server actions**:
- All server actions should return `ActionResult<T>` type
- Must have `"use server"` directive
- Should include try/catch with proper error handling
- Check for `revalidatePath()` or `revalidateTag()` after mutations
- Verify auth checks exist where needed

**API routes**:
- Proper HTTP status codes (200, 400, 401, 403, 404, 500)
- Consistent error response format: `{ error: string }`
- Auth checks at route level if needed
- Input validation using Zod schemas

**Loading states**:
- Forms show loading state during submission
- Buttons disabled during async operations
- Optimistic updates where appropriate

**Error handling**:
- Try/catch blocks in async operations
- User-friendly error messages (translated)
- Proper error boundaries for React components

### 4. Convention Adherence

**File structure** (from `.claude/docs/conventions.md`):
- Server actions in `src/lib/{feature}/actions.ts`
- Queries in `src/lib/{feature}/queries.ts`
- Types in `src/lib/types/{feature}.ts`
- Components in `src/components/{feature}/`
- UI primitives in `src/components/ui/`

**Naming conventions**:
- Server components: `ComponentName` (PascalCase)
- Client components: `"use client"` at top, PascalCase
- Server actions: `actionNameAction()` (camelCase + Action suffix)
- Types: `TypeName` (PascalCase)

**Server vs Client**:
- Default to server components unless interaction/hooks needed
- `"use client"` only when necessary (forms, hooks, events)
- No `auth()` in client components (use server)
- No direct database queries in client components

**Import patterns**:
- Use `@/` path alias for src imports
- Group imports: external, internal, types, styles
- No circular dependencies

## Checklist

For each check, report:
1. **Issue type** (Auth Flow / i18n / Data Flow / Convention)
2. **Severity** (Critical / High / Medium / Low)
3. **Location** (file:line)
4. **Problem** (what's wrong)
5. **Fix** (how to resolve)

## Output Format

```markdown
## Logic Check Results

### Critical Issues
- 🔴 [Auth Flow] Infinite redirect loop in src/middleware.ts:15
  - Problem: Calling auth() in middleware
  - Fix: Remove auth() call; move to page.tsx

### High Priority
- 🟠 [i18n] Missing translation key in messages/mi.json
  - Problem: Key "profile.specialty.style" exists in en.json but not mi.json
  - Fix: Add translation to mi.json

### Medium Priority
- 🟡 [Data Flow] Server action missing error handling
  - Problem: src/lib/recipes/actions.ts:42 no try/catch
  - Fix: Wrap in try/catch, return ActionResult with error

### Low Priority
- 🟢 [Convention] Import not using path alias
  - Problem: src/components/recipe/card.tsx:3 uses relative import
  - Fix: Change to @/components/ui/button

### Summary
- Total issues: 12
- Critical: 1
- High: 3
- Medium: 5
- Low: 3
```

## Anti-patterns to Flag

### Auth
- `auth()` in middleware.ts
- Circular redirects between locales
- Auth checks in root layout.tsx
- Missing auth checks in protected routes

### i18n
- Hardcoded strings: `<h1>Welcome</h1>`
- Wrong import: `import { useRouter } from 'next/navigation'`
- Direct pathname: `href="/recipes"`
- Missing translation keys in either locale file

### Data Flow
- Server actions without ActionResult return type
- No error handling in async functions
- Missing revalidation after mutations
- Unhandled loading states

### Conventions
- Server actions in component files
- Client components without "use client"
- Relative imports instead of @/ alias
- Wrong file locations

## Scope

By default, check:
- `src/app/[locale]/` (all pages)
- `src/middleware.ts` (critical for auth/i18n)
- `src/lib/*/actions.ts` (server actions)
- `src/components/` (UI components)
- `messages/*.json` (translations)

User can specify narrower scope: specific file, directory, or feature area.

## Examples

**User**: "Check the auth flow"
**Action**: Focus on middleware.ts, auth routes, protected pages, session handling

**User**: "Make sure i18n is correct"
**Action**: Compare translation files, scan for hardcoded strings, check locale URL handling

**User**: "Verify the recipe feature"
**Action**: Check src/app/[locale]/recipes, src/lib/recipes, src/components/recipe

**User**: "Logic check before PR"
**Action**: Full systematic check of all modified files (use git diff to find scope)
