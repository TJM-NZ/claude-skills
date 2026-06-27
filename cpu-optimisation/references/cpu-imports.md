# Import & Initialisation Cost

Find heavy module loading that inflates cold start time and wastes CPU on initialisation.

## Full Library Imports (Tree-shaking Missed)

Grep: `import \* as|from 'lodash'|from 'moment'|from 'date-fns'|from 'ramda'`

- Pattern: importing entire library when only one or two functions are needed
- Fix: named imports (`import { debounce } from 'lodash'`) or subpath imports (`import debounce from 'lodash/debounce'`)
- UX impact: none to positive (smaller bundle / faster cold start)

## Heavy Dependencies Loaded at Module Level

Read: `package.json` — flag: `aws-sdk`, `puppeteer`, `sharp`, `pdf-lib`, `ffmpeg-static`, `playwright`

- Pattern: these are imported at top of file, so they load on every cold start even if rarely used
- Fix: dynamic import inside the function that needs them: `const sharp = await import('sharp')`
- UX impact: minimal — first call to that function slightly slower; all other requests unaffected

## Barrel File Over-Import

Grep: `from '../components'|from '../lib'|from '../utils'|from '@/components'`

- Pattern: barrel `index.ts` re-exports everything — importing one thing loads the entire barrel
- Fix: import directly from the source file: `from '../components/Button'`
- UX impact: none (same runtime behaviour, smaller parse cost)

## Expensive Module-Level Initialisation

Grep: top of files for `new|connect\(|createClient\(|mongoose\.connect|knex\(` outside functions

- Pattern: DB connections, SDK clients, or heavy object construction at module scope — runs on every cold start
- Fix: lazy-initialise behind a function with a module-scope cache variable (`let client; function getClient() { ... }`)
- UX impact: none (connection established before first use, same as before)

## Unused Imports

Grep: cross-reference `import` statements against actual usage in the file

- Pattern: imported symbols never referenced in the file body
- Fix: remove them
- UX impact: none

## Missing Code Splitting on Heavy Routes

Read: router/page config — look for routes that load heavy components eagerly

- Pattern: admin panels, chart libraries, rich text editors bundled into the main entry point
- Fix: dynamic import / `React.lazy` with `Suspense`
- UX impact: minimal — lazy component shows a suspense fallback on first visit; subsequent visits cached
