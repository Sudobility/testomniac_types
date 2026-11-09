# Testomniac Types

> **Git policy — never auto-commit or auto-push.** Leave your work in the working tree.
> Run `git commit`, `git push`, `gh pr create`, or `scripts/push_all.sh` **only when the user
> explicitly asks in that turn**. Approval for an earlier change does not carry forward, and
> finishing a task is not permission to commit it.

Shared TypeScript type definitions for the Testomniac testing-platform ecosystem. Foundation
layer — every other Testomniac project depends on this package.

**npm**: `@sudobility/testomniac_types` — published to **GitHub Packages**, not npmjs.

## Tech Stack

- TypeScript (strict), ESM-only (`"type": "module"`), target ES2020, `moduleResolution: bundler`
- Runtime / package manager: **Bun** (do not use npm/yarn/pnpm to install)
- Build: `tsc`. Test: Vitest. Lint: ESLint (flat config). Format: Prettier.
- Peer dependency: `@sudobility/types` (not bundled; consumers install it).

## Commands

```bash
bun run build          # tsc -p tsconfig.esm.json → dist/ (emits .d.ts + maps)
bun run dev            # tsc --watch
bun run typecheck      # tsc --noEmit
bun run lint           # eslint src (lint:fix for --fix)
bun run format         # prettier --write
bun run test:unit      # vitest run (134 cases)  ← NOT `bun test`
bun run test:watch     # vitest watch
bun run verify         # typecheck + lint + test:unit + build (run before publishing)
```

- **Use `bun run test:unit`, never `bun test`.** There is no `test` script; `bun test` runs
  Bun's own runner (not Vitest) and will not run these suites.
- Two tsconfigs: `tsconfig.json` sets `noEmit: true` (typecheck/editor only); the build
  config is `tsconfig.esm.json` (adds `outDir`, `declaration`, maps, `noEmit: false`).
- `prepublishOnly` runs a sibling `../workflows/scripts/check-publish-safety.cjs` guard (if
  present) then `clean` + `verify`. CI publishes via a reusable `johnqh/workflows` workflow.

## Architecture / layout

All public API is the single barrel `src/index.ts` (~3100 lines), which also re-exports
`src/replay-selector.ts`. Source files:

- `src/index.ts` — all types, enums, and the runtime helpers
- `src/replay-selector.ts` — replay-selector grammar + encode/parse codec
- `src/*.test.ts` — `index`, `replay-selector`, `scan-next-response` Vitest suites

Content groups: enums/constants, in-memory domain objects (TestInteraction, TestSurface,
ElementIdentity, Expectation, scan/graph payloads), and ~150 API contract types.

## Conventions

- **Enum pattern**: `export const Name = { Key: 'value' } as const;` plus a same-named value
  alias `export type Name = (typeof Name)[keyof typeof Name];` (PascalCase keys). No TS `enum`.
- **API contract naming**: `Create<Entity>Request`, `Update<Entity>Request`, `<Entity>Response`.
  (Note: it's `Create<Entity>Request`, not `<Entity>CreateRequest`.)
- Use `interface` for object shapes, `type` for unions/aliases.
- Deprecated items are **kept** and marked `/** @deprecated Use X */` — don't delete without
  checking consumers; many are type aliases for renamed entities (e.g. `ProjectResponse` →
  `ProductResponse`, `AppResponse` → `RunnerResponse`).
- Sections delimited by `// ===` comment banners.

## Gotchas

- **Not a pure types package.** It ships real runtime code: `successResponse` / `errorResponse`,
  `isSuccessResponse` / `isErrorResponse`, `resolvePlaywrightRole`, `getExpertiseRuleDefinition`
  / `getExpertiseIdForRuleId`, the generated `ExpertiseRuleCatalog`, the replay-selector codec,
  and the constants `DESKTOP_SCREENS` / `MOBILE_SCREENS` / `UI_COLOR_VALUES`. Keep this runtime
  surface tiny and add no runtime dependencies.
- Keep `replay-selector.ts` DOM/Node-free (ES2020 lib only): it is encoded by hand rather than
  with `URLSearchParams`, yet its output must stay parseable by `new URLSearchParams(...)`.
- `BaseResponse<T>` is the standard API envelope; all responses conform to it.
- Changes here affect ALL consumer projects — weigh downstream impact; run `bun run verify`.

## Git Workflow

- Do not use feature branches for code changes. Always stay on the current branch.
