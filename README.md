# @sudobility/testomniac_types

Shared TypeScript type definitions for the Testomniac testing-platform ecosystem.

Foundation layer — all other Testomniac projects depend on this package. It is almost
entirely types, plus a small set of pure runtime helpers (response wrappers, the
expertise-rule catalog, and the replay-selector codec).

- **Package**: `@sudobility/testomniac_types` (ESM-only, `"type": "module"`)
- **Registry**: GitHub Packages (`@sudobility` scope → `https://npm.pkg.github.com`)
- **License**: BUSL-1.1

## Installation

This package is published to GitHub Packages, so consumers need an `.npmrc` that routes
the `@sudobility` scope there:

```
@sudobility:registry=https://npm.pkg.github.com
```

Then install (the project uses Bun):

```bash
bun add @sudobility/testomniac_types
bun add @sudobility/types   # required peer dependency
```

`@sudobility/types` is a **peer dependency** (`^1.9.78`) — it is not bundled, so consumers
must install it themselves. Several base types and enums are re-exported from it (see below).

## Usage

```ts
import {
  TestInteraction,
  TestRunResponse,
  TestRunStatus,
  PlaywrightAction,
  successResponse,
  errorResponse,
  isSuccessResponse,
  isErrorResponse,
  resolvePlaywrightRole,
} from '@sudobility/testomniac_types';

// Wrap API responses (both helpers also stamp a `timestamp`)
const ok = successResponse({ items: [] });
// { success: true, data: { items: [] }, timestamp: '2026-...' }
const err = errorResponse('Not found');
// { success: false, error: 'Not found', timestamp: '2026-...' }

// Narrow a BaseResponse with the type guards
if (isSuccessResponse(response)) {
  console.log(response.data);
}

// Map an HTML tag (and optional input type / explicit ARIA role) to a Playwright role
const role = resolvePlaywrightRole('button'); // "button"
```

## What's inside

Everything is exported from the `src/index.ts` barrel (which also re-exports the
replay-selector module). Types fall into a few groups.

### Enums & constants

Enums are declared as `as const` objects paired with a value-type alias of the same name:

```ts
export const PlaywrightAction = { Click: 'click', Fill: 'fill', /* ... */ } as const;
export type PlaywrightAction = (typeof PlaywrightAction)[keyof typeof PlaywrightAction];
```

Notable ones include:

- `PlaywrightAction` — goto, click, dblclick, fill, type, press, selectOption, check, hover, etc.
- `ExpectationType` — ~100 assertion kinds (`page_loaded`, `no_console_errors`, `form_submitted_successfully`, …)
- `ExpectationSeverity` — `must_pass`, `should_pass`, `info`
- `ExpertiseId` / `ExpertiseRuleId` — stable finding ownership areas and rule IDs
- `LocatorStrategy` — `test-id`, `role-name`, `label`, `placeholder`, `text`, `alt-text`, `css`
- `TestType` — render, interaction, form, form_negative, password, navigation, e2e
- `TestRunStatus` — pending, planned, running, completed, failed
- `FindingType` / `FindingPriority`, `RecurrenceType`, `SizeClass`, `AuthProvider`,
  `EnvironmentKind`, `RunnerType`, `UiColor`
- `HtmlComponentType` — page scaffold kinds (topMenu, footer, breadcrumb, leftMenu, …)
- `UiPatternType` — repeated UI patterns (card, table, modal, carousel, accordion, …)

Also exported as runtime values: `DESKTOP_SCREENS` / `MOBILE_SCREENS` (`Screen[]`),
`UI_COLOR_VALUES`, and `ExpertiseRuleCatalog` (a generated `Record<ExpertiseRuleId, …>`).

### Domain objects

In-memory shapes used during test generation and execution — e.g. `ElementIdentity`,
`ElementLocator`, `ActionableItem`, `FormInfo` / `FormField`, `UiPattern`, `Expectation`,
`TestAction`, `TestStep`, `TestInteraction`, `TestSurface`, `PageHashes`, `UserData`,
`Credentials`, plus the scan/graph payloads (`ScanNextRequest`/`ScanNextResponse`,
`ScanPageSignals`, `NavigationPlanRequest`/`NavigationPlanResponse`, `CapturedNetworkRequest`).

### API contract types

~150 request/response types following a consistent naming pattern:

- `Create<Entity>Request`, `Update<Entity>Request` — mutation payloads
- `<Entity>Response` — API response rows

e.g. `CreateProductRequest` / `ProductResponse`, `CreateTestRunRequest` / `TestRunResponse`,
`CreateTestInteractionRunRequest` / `TestInteractionRunResponse`, `EnsureTestRunFindingRequest` /
`TestRunFindingResponse`, and many more. Deprecated entries are kept and marked with
`/** @deprecated Use X */` JSDoc (e.g. `ProjectResponse` → `ProductResponse`).

### Helpers (runtime)

- `successResponse<T>(data)` / `errorResponse(error)` — build a `BaseResponse<T>`
- `isSuccessResponse(response)` / `isErrorResponse(response)` — type guards over `BaseResponse`
- `resolvePlaywrightRole(tagName, inputType?, ariaRole?)` — HTML element → ARIA/Playwright role
- `getExpertiseRuleDefinition(ruleId)` / `getExpertiseIdForRuleId(ruleId)` — look up rule metadata
- Replay-selector codec (`src/replay-selector.ts`): `encodeReplaySelector`, `parseReplaySelector`,
  `isReplaySelector`, `REPLAY_SELECTOR_PREFIX`, `ReplaySelectorMetadata`

### Re-exports from `@sudobility/types`

Types `ApiResponse`, `BaseResponse`, `NetworkClient`, `Optional`; the entity types
(`Entity`, `EntityMember`, `EntityInvitation`, `CreateEntityRequest`, …); and the enums
`EntityType`, `EntityRole`, `InvitationStatus`.

## Project layout

```
src/
├── index.ts                   # Barrel: all types, enums, response helpers (~3100 lines)
├── replay-selector.ts         # Replay-selector grammar + encode/parse (runtime)
├── index.test.ts              # Tests for index.ts
├── replay-selector.test.ts    # Tests for the replay-selector codec
└── scan-next-response.test.ts # Tests for scan/next payload shapes
```

## Development

```bash
bun run build          # Build ESM output: tsc -p tsconfig.esm.json → dist/
bun run dev            # tsc --watch
bun run typecheck      # Type-check only (tsc --noEmit)
bun run lint           # ESLint over src
bun run lint:fix       # ESLint with --fix
bun run format         # Prettier --write
bun run format:check   # Prettier --check
bun run test:unit      # Run the Vitest suite once (134 test cases)
bun run test:watch     # Vitest in watch mode
bun run test:coverage  # Vitest with coverage
bun run clean          # rimraf dist
bun run verify         # typecheck + lint + test:unit + build (run before publishing)
```

> Note: use `bun run test:unit`, **not** `bun test` — the latter invokes Bun's built-in
> test runner, not Vitest, and does not run these suites.

Two tsconfigs: `tsconfig.json` has `noEmit: true` (used for typecheck / editor); the build
uses `tsconfig.esm.json`, which sets `outDir: dist` and emits declarations + source maps.

## Architecture

```
@sudobility/testomniac_types (this package)
    ▲  depended on by
testomniac_api              — backend API server
testomniac_client           — API client SDK
testomniac_lib              — business-logic library
testomniac_app              — React web app
testomniac_app_rn           — React Native mobile app
testomniac_runner_service   — shared execution library
testomniac_runner           — server-side polling worker
testomniac_extension        — Chrome extension
testomniac_api_mcp          — MCP server for API endpoints
testomniac_runner_mcp       — MCP server for browser automation
```

Because every Testomniac project consumes these types, a change here ripples across the
whole ecosystem — always weigh downstream impact and run `bun run verify` before publishing.

## License

BUSL-1.1 — see `package.json`.
