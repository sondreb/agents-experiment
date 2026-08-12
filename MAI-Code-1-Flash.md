# AGENTS.md - AI Coding Agent Guidelines

> High-signal instructions for AI coding agents working in the Nostria codebase. Treat the user as an expert engineer, not a novice. Prioritize correctness, maintainability, speed, and thoughtful innovation.

## Operating principles

- Act like a senior peer, not a tutor. Do not over-explain, hand-hold, or restate obvious things.
- Prefer the smallest change that fully solves the problem. Avoid churn, speculative rewrites, and unnecessary abstractions.
- Optimize for quality over verbosity: robust implementation, clear naming, minimal diff, and thoughtful edge cases.
- When multiple valid approaches exist, choose the one that best fits the existing architecture and avoids future maintenance pain.
- If a better approach is obvious and materially improves the app, implement it. Otherwise stay conservative and surgical.

## What good output looks like

- Solve the actual problem, not a nearby one.
- Make changes that are easy to review and reason about.
- Preserve or improve performance, accessibility, SSR safety, and Nostr correctness.
- Keep the codebase consistent with existing patterns before introducing new ones.
- Surface tradeoffs briefly when they matter. Avoid generic commentary.

## Project context

- Tech stack: Angular 22+, TypeScript (strict), Angular Material 3, SCSS, Playwright (E2E), Karma/Jasmine (unit)
- App URL: https://nostria.app
- Protocol: Nostr (nostr-tools library)
- This is a one-developer project with a high-skill user. Do not treat the user like a beginner or default to overly simplified guidance.

## Working style

- Inspect existing patterns before changing code. Reuse existing services, components, utilities, and conventions whenever they already cover the need.
- Be decisive. If a request is ambiguous, make a reasonable assumption and note it in your summary rather than blocking on minor clarifications.
- Avoid unnecessary questions; do the work unless the user explicitly asks for a choice that materially changes scope.
- Do not over-validate with boilerplate. Validate the changed behavior with the smallest relevant test or build step that provides confidence.
- When touching architecture, update [ARCHITECTURE.md](./ARCHITECTURE.md) and keep the documentation aligned with the implementation.
- Do not add new dependencies unless they are clearly justified by the problem and fit the existing stack.

## Code quality expectations

- TypeScript strict mode is required. No implicit any. Avoid `any` unless it is truly unavoidable and clearly justified.
- Prefer simple, explicit code over cleverness. Readable code wins unless it introduces meaningful complexity or performance cost.
- Keep functions focused and composable. Avoid "just in case" abstractions.
- Keep comments sparse and meaningful. Explain non-obvious intent, not obvious code.
- Avoid mutating signals, arrays, or objects in place when a safe immutable update is straightforward.
- If a change affects user-facing behavior, consider accessibility, empty states, and error handling.

## Angular and frontend conventions

- Standalone components only. No NgModules.
- Do not set `standalone: true`; it is the default in Angular 21+.
- Use signals for local component state and computed values where they are natural.
- Prefer `input()` and `output()` over decorators.
- Use `ChangeDetectionStrategy.OnPush` for components.
- Do not use `@HostBinding` or `@HostListener`; use the `host` object in the component decorator.
- Do not use `ngClass` or `ngStyle`; use class and style bindings instead.
- Use `NgOptimizedImage` for static images. Do not use inline base64 images where `NgOptimizedImage` applies.
- Prefer native control flow (`@if`, `@for`, `@switch`) over the old structural directives.
- Follow Angular Material 3 patterns and CSS variables rather than legacy styling approaches.

## Services, HTTP, and state

- Keep services focused on a single responsibility.
- Prefer `providedIn: 'root'` for singleton services.
- Use `inject()` instead of constructor injection.
- Use `fetch` for HTTP requests; do not reach for `HttpClient`.
- Keep data transformations predictable and typed. Favor explicit parsing/normalization over ad-hoc assumptions.

## Nostr, SSR, and platform safety

- Nostr timestamps are in seconds, not milliseconds. Use `Math.floor(Date.now() / 1000)` for timestamps.
- Be SSR-safe. Do not access browser APIs directly without guarding for the browser environment.
- When working with routing, metadata, or dynamic rendering, preserve SSR behavior and social preview correctness.

## Styling

- Use CSS variables from Angular Material 3; avoid hardcoded colors.
- Do not set `font-weight`; the current font does not support it.
- Use `field-sizing: content` for auto-growing textareas.
- Do not use `color="primary"` on buttons. Use `mat-flat-button` for primary actions.
- When resizing `mat-icon-button` smaller than its default size, do not use `line-height` to center the icon. Use `padding: 0 !important; display: flex !important; align-items: center; justify-content: center;` instead.
- Support dark and light mode by using the established theme tokens and CSS variables.

## Dialogs and user flows

- Use `CustomDialogComponent`, not Angular Material dialogs.
- Do not use native `confirm()` dialogs. Use app dialogs and snackbars for confirmation flows.
- Keep confirmation and destructive flows explicit and user-safe.

## Project structure

```text
src/app/
├── api/          # Generated (DO NOT EDIT)
├── components/   # Reusable UI
├── pages/        # Route-level pages
├── services/     # Business logic
├── interfaces/   # TypeScript interfaces
└── utils/        # Utilities
```

## Command palette

- Add commands for new features under `src/app/components/command-palette-dialog/` so they remain available through Ctrl+K.

## Build, lint, and test commands

```bash
npm run start                 # Dev server at http://localhost:4200
npm run build                 # Production build
npm run lint                  # Run ESLint
npm run lint-fix              # Auto-fix ESLint issues
npm run test                  # Run unit tests (Karma/Jasmine)
npm run test:e2e              # Run all E2E tests (Playwright)
npm run test:e2e:ui           # Playwright UI mode
npm run test:e2e:headed       # Run with visible browser
npm run test:e2e:debug        # Debug mode
npm run test:e2e:auth         # Run only @auth tests
npm run test:e2e:full         # Run all tests with full artifacts
npm run test:e2e:metrics      # Run performance/metrics tests
npm run test:e2e:visual       # Run visual regression tests
npm run test:e2e:visual:update # Update visual baselines
npm run test:e2e:report:full  # Generate comprehensive Markdown report

# Run single E2E test file
npx playwright test e2e/tests/home.spec.ts

# Run single test by name
npx playwright test -g "should load the home page"

# Run tests by tag
npx playwright test --grep @public
npx playwright test --grep @auth
npx playwright test --grep @security
```

## E2E testing guidance

### Running tests

Use these commands when E2E validation is relevant:

```bash
# Quick smoke test (public pages only)
npx playwright test --grep @smoke

# Full suite
npm run test:e2e:full

# Specific test category
npx playwright test --grep @auth      # Authenticated tests
npx playwright test --grep @metrics   # Performance tests
npx playwright test --grep @security  # Security tests
npx playwright test --grep @network   # Network tests
```

### Interpreting results

After running tests, inspect these outputs:

1. **Quick summary**: `test-results/test-summary.json` — total/passed/failed counts
2. **Detailed results**: `test-results/results.json` — per-test status and errors
3. **Console logs**: `test-results/logs/*.json` — categorized browser console output
4. **Performance data**: `test-results/metrics/*.json` — Web Vitals, bundle sizes, memory
5. **Network data**: `test-results/network/*.json` — HTTP requests, WebSocket connections
6. **Full report**: Run `npm run test:e2e:report:full` to generate `test-results/reports/test-report.md`

### Social preview regression check

When changing SSR, routing, `DataResolver`, or metadata generation, validate social preview tags before merging.

Use bot user agents and verify dynamic pages do not return homepage fallback tags:

```bash
# Validate OG/Twitter meta for a known event URL
curl -A "Twitterbot/1.0" "https://nostria.app/e/nevent1qvzqqqqqqypzq9lz3z0m5qgzr5zg5ylapwss3tf3cwpjv225vrppu6wy8750heg4qqsqqqpsj6e662lsgy26a5g9nvav4z807m08ryhnx7ljs5dnuhpfl0cs642uw" | grep -E "og:title|og:description|twitter:title|twitter:description|og:image|twitter:image"
```

Expected outcome: event/profile/article-specific tags are present and not generic homepage values like `Nostria - Your Social Network`.

### Test tags

Tests are tagged for filtering. Use `--grep` to select:

| Tag         | Description                    |
| ----------- | ------------------------------ |
| `@public`   | No authentication required     |
| `@auth`     | Requires logged-in account     |
| `@smoke`    | Critical path, fast CI         |
| `@metrics`  | Performance/metrics collection |
| `@network`  | Network/WebSocket monitoring   |
| `@security` | Security validation            |
| `@a11y`     | Accessibility checks           |
| `@visual`   | Visual regression screenshots  |

### Writing E2E tests

When creating new E2E tests, follow these conventions:

1. **Import from `e2e/fixtures`**, not `@playwright/test` directly.
2. **Tag tests** in the `test.describe()` title (for example, `'Feature @auth @smoke'`).
3. **Use `saveConsoleLogs()`** at the end of every test.
4. **Use constants** from `e2e/fixtures/test-data.ts` (profiles, routes, timeouts).
5. **Use `waitForAppReady()`** before making assertions.
6. **Use `authenticatedPage`** fixture for tests needing login.
7. **No `data-testid`** attributes exist — use Angular Material selectors, CSS classes, or text content.
8. **Nostr timestamps** are in seconds: `Math.floor(Date.now() / 1000)`.
9. **Handle empty states** — test accounts may have no relay history.

### Test infrastructure files

| File                               | Purpose                                                                    |
| ---------------------------------- | -------------------------------------------------------------------------- |
| `e2e/fixtures.ts`                  | Extended Playwright fixtures (authenticatedPage, performanceMetrics, etc.) |
| `e2e/helpers/auth.ts`              | Auth injection/cleanup via TestAuthHelper                                  |
| `e2e/helpers/console-analyzer.ts`  | Log categorization and assertions                                          |
| `e2e/helpers/metrics-collector.ts` | Performance data aggregation                                               |
| `e2e/helpers/websocket-monitor.ts` | CDP-based WebSocket inspection                                             |
| `e2e/helpers/report-generator.ts`  | Full report generation (JSON + Markdown)                                   |
| `e2e/fixtures/test-data.ts`        | Centralized constants (profiles, relays, routes, timeouts)                 |
| `e2e/fixtures/mock-events.ts`      | Nostr event factory functions                                              |
| `e2e/fixtures/test-isolation.ts`   | App state reset helpers                                                    |

### Reporting tools

```bash
# Generate comprehensive report after test run
npm run test:e2e:report:full
# Output: test-results/reports/test-report.md + full-report.json

# View HTML report
npm run test:e2e:report
```

The full report includes: test results table, performance metrics with pass/fail indicators, console error summary, network health, memory trends, and actionable improvement recommendations.
