# AGENTS.md

Guidance for AI coding agents working in Nostria.

## Working Agreement

The maintainer is an expert developer. Communicate peer-to-peer:

- Be concise, technical, and direct. Skip tutorials, generic advice, and definitions of standard concepts.
- Lead with decisions, material tradeoffs, and evidence. Do not narrate routine tool use.
- Make reasonable, reversible assumptions and proceed. Ask only when a decision is genuinely blocking, destructive, or changes product behavior in a way that cannot be inferred from the code.
- Challenge a requested approach when repository evidence shows a safer or substantially better option. Explain the tradeoff briefly, then implement the best solution consistent with the intent.
- Surface uncertainty precisely. Never invent repository facts, APIs, test results, or protocol behavior.

## Execution Priorities

Optimize for this order:

1. Correctness, security, and data integrity
2. A complete solution across all affected surfaces
3. Simplicity and maintainability
4. Speed of implementation and validation
5. Novelty

Innovation is welcome when it produces a measurable improvement in UX, performance, reliability, or developer experience. Prefer the smallest coherent design that leaves room to evolve. Avoid speculative abstractions, compatibility layers with no current consumer, and unrelated cleanup.

## Default Workflow

1. Read the relevant implementation, tests, configuration, and nearby conventions before editing.
2. Search for existing helpers and patterns before introducing new ones.
3. Trace the behavior end-to-end across callers, state, UI, persistence, SSR, and protocol boundaries as applicable.
4. Make a surgical but complete change. Preserve unrelated work in a dirty worktree.
5. Add or update focused tests when behavior changes or a regression is plausible.
6. Validate progressively: the narrowest relevant test first, then type-check/lint/build only as needed for confidence.
7. Review the final diff for accidental scope, stale comments, generated files, debug artifacts, and missing integration points.
8. Report what changed, validation performed, and only actionable residual risks.

Do not stop at a plausible patch. Verify the requested outcome directly. Never claim a command passed unless it was run successfully.

## Engineering Standards

- Prefer root-cause fixes over symptom patches.
- Keep changes cohesive; do not refactor unrelated code.
- Preserve existing behavior unless the task explicitly changes it.
- Handle errors explicitly and consistently with nearby code. Do not swallow failures or return success-shaped fallbacks.
- Keep strict type safety. Avoid `any`, double assertions, and casting wrappers; validate uncertain data and narrow `unknown`.
- Use comments only for non-obvious intent, invariants, or protocol constraints. Keep them synchronized with behavior.
- Treat generated code in `src/app/api/` as read-only; regenerate it through the documented generator when necessary.
- Do not add dependencies when a platform API or existing dependency solves the problem adequately.
- Do not introduce secrets, credentials, personal data, or sensitive values into code, logs, fixtures, or documentation.

## Project Facts

- **Stack:** Angular 22+, TypeScript strict mode, Angular Material 3, SCSS
- **Tests:** Karma/Jasmine unit tests and Playwright E2E tests
- **App:** https://nostria.app
- **Protocol:** Nostr using `nostr-tools`
- **Formatting:** 2-space indentation, single quotes, CRLF, no trailing whitespace
- **Naming:** `app-` kebab-case component selectors; `app` camelCase directive selectors
- **Files:** `name.component.ts`, `name.service.ts`, `name.spec.ts`

Consult [ARCHITECTURE.md](./ARCHITECTURE.md) before non-trivial architecture changes and update it when an architectural decision changes. See [TESTING.md](./TESTING.md) for detailed testing guidance and [.github/copilot-instructions.md](.github/copilot-instructions.md) for additional repository instructions.

## TypeScript

- Prefer inference when the type is obvious and explicit types at public or ambiguous boundaries.
- Model valid states rather than relying on boolean combinations or unchecked optional values.
- Prefer discriminated unions, type guards, and exhaustive switches for variant data.
- Avoid one-line helpers whose only purpose is a type assertion.
- Preserve immutability where it improves state predictability.

## Angular

- Use standalone components; do not set `standalone: true`.
- Set `changeDetection: ChangeDetectionStrategy.OnPush` on components.
- Use `input()` and `output()` instead of `@Input` and `@Output`.
- Use `inject()` instead of constructor injection.
- Use signals for local and service state, `computed()` for derived state, and `set()` or `update()` rather than `mutate()`.
- Use native template control flow (`@if`, `@for`, `@switch`), not structural directives.
- Do not use `ngClass` or `ngStyle`; use class and style bindings.
- Put host bindings and listeners in the decorator's `host` object, not `@HostBinding` or `@HostListener`.
- Keep templates declarative and move complex transformations into typed component code.
- Lazy-load feature routes.
- Use `NgOptimizedImage` for static non-base64 images.
- Use reactive forms for non-trivial forms.
- Ensure interactive UI is keyboard accessible, has an accessible name, and exposes state correctly.

### Browser and SSR Boundaries

Guard browser-only APIs with `isPlatformBrowser` or an existing abstraction. Do not access `window`, `document`, storage, media APIs, or observers during SSR.

When changing SSR, routing, `DataResolver`, or metadata generation, run the relevant social-preview check:

```bash
npm run check:social-preview:beta
npm run check:social-preview:prod
```

Verify dynamic pages return entity-specific Open Graph and Twitter metadata rather than homepage fallback values.

## Services and HTTP

- Design services around one responsibility and use `providedIn: 'root'` for application singletons.
- Use `fetch`, not Angular `HttpClient`.
- Check `response.ok` and surface meaningful failures before parsing a response.
- Preserve cancellation where relevant by accepting or creating an `AbortSignal`.
- Validate external JSON at trust boundaries instead of asserting its shape.

## Nostr Protocol

- Follow the applicable NIP and `nostr-tools` behavior; do not infer wire formats from UI needs.
- Nostr timestamps are Unix seconds, never milliseconds:

```typescript
const createdAt = Math.floor(Date.now() / 1000);
```

- Keep event kinds, tags, relay filters, signing, and identifier encoding protocol-correct.
- Do not mutate signed events.
- Treat relay and event data as untrusted input.
- When changing protocol behavior, test representative valid, missing, malformed, and duplicate data where relevant.

## UI, Material, and Styling

- Prefer Angular Material 3 components and existing app components.
- Use `CustomDialogComponent`, not Angular Material dialogs.
- Never use native `alert()`, `confirm()`, or `prompt()` for application flows; use app dialogs and snackbars.
- Do not use `color="primary"` on Material buttons. Use `mat-flat-button` for primary actions.
- Never set `font-weight`; the headline font does not support multiple weights.
- Use Material 3 CSS variables from the theme instead of hardcoded colors, including for dark mode.
- Use `field-sizing: content` for textareas that grow with content.
- When shrinking `mat-icon-button`, center it with padding and flex alignment, never `line-height`.
- Check responsive behavior and both light and dark themes for UI changes.

For new features or routes, add the corresponding command to `src/app/components/command-palette-dialog/`.

## Validation Commands

Choose the smallest command that covers the change. Do not run the full E2E suite by default for a local unit-level change.

```bash
npm run build
npm run lint
npm run test

npx playwright test e2e/tests/home.spec.ts
npx playwright test -g "test name"
npx playwright test --grep @smoke
npx playwright test --grep @auth
npx playwright test --grep @security
npm run test:e2e:full
```

For E2E tests:

- Import the project fixtures rather than `@playwright/test` directly.
- Tag `test.describe()` titles with the relevant category.
- Use `waitForAppReady()` before assertions and `authenticatedPage` for authenticated scenarios.
- Reuse data and timeouts from `e2e/fixtures/test-data.ts`.
- Call `saveConsoleLogs()` at the end of each test.
- Prefer user-visible roles, labels, and text selectors. Use stable existing CSS selectors only when semantic selectors are unavailable.
- Handle legitimate empty states; test accounts may have no relay history.
- Inspect `test-results/test-summary.json` and detailed artifacts after broad E2E runs.

Available tags include `@public`, `@auth`, `@smoke`, `@metrics`, `@network`, `@security`, `@a11y`, and `@visual`.

## Completion Standard

A change is complete only when:

- The requested behavior works across the relevant paths.
- Tests cover important new or corrected behavior where practical.
- Relevant validation passes, or pre-existing/unrelated failures are identified explicitly.
- No debug code, temporary files, stale comments, or accidental generated output remains.
- The final response is concise and includes changed behavior, validation evidence, and genuine remaining risks only.
