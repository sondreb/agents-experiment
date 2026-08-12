# AGENTS.md

Agent brief for the Nostria codebase. Optimize for correctness, leverage, and speed.

## Audience

- Sole developer on this project. Expert-level. Do not tutor, pad, or over-explain.
- Assume fluency with modern Angular, TypeScript, Nostr, SSR, and browser platform APIs.
- Prefer dense technical communication. Skip ceremony, disclaimers, and "as an AI" framing.
- Do not ask questions you can resolve from the repo, docs, or reasonable defaults. Decide and move.

## Operating mode

1. **Ship complete solutions.** Prefer a correct end-to-end change over a partial sketch. If the first approach is weak, iterate until it is solid.
2. **Bias to action.** Resolve ambiguity with the highest-leverage default that fits existing architecture. State assumptions briefly only when they matter.
3. **Match the house style before inventing.** Read nearby code and follow local patterns. Parallel abstractions, "cleanup" refactors, and drive-by renames are noise unless they unlock the task.
4. **YAGNI with teeth.** Simple beats clever. Clever is allowed when it removes real complexity or unlocks meaningful product value.
5. **Innovate when it pays.** Bold ideas are welcome when they improve UX, performance, architecture clarity, or developer velocity. Do not innovate for novelty.
6. **Surgical + complete.** Touch only what the task needs, but finish the task: command palette entries, architecture notes, tests, and edge cases that belong to the change.
7. **No destructive freelancing.** Do not force-push, reset, rewrite history, delete data, or change secrets unless explicitly asked.
8. **Verify what you change.** Run the smallest relevant lint/build/test. Do not claim done on vibes.
9. **Docs only when durable.** No markdown writeups for routine work. Update `ARCHITECTURE.md` for non-trivial design changes. Put generated notes in `docs/` only when documentation is actually warranted.
10. **Speed is a feature.** Minimize exploratory thrash. Search with intent, edit with confidence, validate narrowly, then stop.

## Product context

- **App:** [https://nostria.app](https://nostria.app)
- **Stack:** Angular 22+, TypeScript (strict), Angular Material 3, SCSS, signals, `nostr-tools`, Playwright E2E, Karma/Jasmine unit
- **Domain:** Nostr client. Follow NIPs. Protocol timestamps are **seconds**, never ms.
- **UX language:** code may say `event`; user-facing copy says **post/posts**.
- **Platforms:** web, SSR, Tauri desktop, PWA/mobile concerns exist — keep browser-only APIs gated.

Canonical deep docs:

- [`ARCHITECTURE.md`](./ARCHITECTURE.md) — source of truth for design decisions
- [`TESTING.md`](./TESTING.md) — E2E system
- [`.github/copilot-instructions.md`](./.github/copilot-instructions.md) — expanded style/rules (some overlap; this file wins on tone/workflow)

## Hard constraints

These are non-negotiable. Violating them creates real bugs or inconsistent UX.

| Area | Rule |
| --- | --- |
| HTTP | `fetch` only. Never `HttpClient`. |
| Dialogs | `CustomDialogComponent` only. Never Material dialogs. Never native `confirm()`. |
| Confirmations | App dialogs/snackbars. |
| Components | Standalone only. Do **not** set `standalone: true`. |
| Change detection | `ChangeDetectionStrategy.OnPush` always. |
| Inputs/outputs | `input()` / `output()` — not decorator forms. |
| DI | `inject()` — not constructor injection theater. |
| Host bindings | `host: {}` on the decorator — not `@HostBinding` / `@HostListener`. |
| Templates | Native control flow (`@if` / `@for` / `@switch`). No `*ngIf` / `*ngFor` / `*ngSwitch`. |
| Class/style | Class and style bindings. No `ngClass` / `ngStyle`. |
| Signals | `set` / `update` only. Never `mutate`. |
| Images | `NgOptimizedImage` for static assets (not inline base64). |
| Forms | Reactive forms preferred. |
| Routes | Lazy-load feature routes. |
| Generated code | `src/app/api/` is generated — do not hand-edit. |
| Theming | CSS variables only (`--mat-sys-*`). Never hardcode theme colors. Values will change. |
| Fonts | Never set `font-weight`. Headline font lacks weight variants. |
| Buttons | No `color="primary"`. Primary actions use `mat-flat-button`. |
| Textareas | `field-sizing: content` for auto-grow. |
| Icon buttons | When shrinking `mat-icon-button`, center with flex + zero padding — never `line-height` hacks. |
| Dark mode | Support light + dark. Prefer `:host-context(.dark)` when encapsulation requires it. |
| SSR | Gate `window` / `document` / `localStorage` / etc. behind `isPlatformBrowser`. |
| Command palette | New features/routes get commands under `src/app/components/command-palette-dialog/`. |
| Nostr time | `Math.floor(Date.now() / 1000)` for event created_at and protocol times. |

### Minimal patterns

```ts
@Component({
  selector: 'app-example',
  imports: [CommonModule],
  templateUrl: './example.component.html',
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class ExampleComponent {
  data = input<DataType>();
  selected = output<Item>();
  private readonly service = inject(MyService);
  readonly items = signal<Item[]>([]);
  readonly filtered = computed(() => this.items().filter(i => i.active));
}
```

```html
@if (condition()) {
  <div>Content</div>
}
@for (item of items(); track item.id) {
  <app-item [data]="item" />
}
```

```ts
@Injectable({ providedIn: 'root' })
export class ExampleService {
  private readonly other = inject(OtherService);
  readonly items = signal<Item[]>([]);
}
```

```ts
private readonly isBrowser = isPlatformBrowser(inject(PLATFORM_ID));
if (this.isBrowser) {
  // browser-only
}
```

```scss
background: var(--mat-sys-surface);
color: var(--mat-sys-on-surface);
```

## TypeScript bar

- Strict mode. No implicit `any`. Prefer `unknown` over `any`.
- Infer when obvious; annotate at boundaries and public APIs.
- Write idiomatic TS — discriminated unions, narrow early, avoid cast-wrapper one-liners.
- If the TS looks like Python with types bolted on, rewrite it.
- Aim for code Matt Pocock would not wince at: precise types, minimal assertions, zero type lies.

## Code organization

```
src/app/
├── api/          # generated — do not edit
├── components/   # reusable UI
├── pages/        # route-level pages
├── services/     # business logic
├── interfaces/   # shared types
└── utils/        # pure helpers
```

- Keep components small and single-purpose.
- Prefer inline templates only when genuinely small.
- Comments explain non-obvious intent, invariants, or protocol quirks — not what the next line obviously does.
- Keep comments truthful when behavior changes.
- Formatting: 2-space indent, single quotes, CRLF, trim trailing whitespace.
- Naming: components `app-foo-bar`, directives `appFooBar`.
- Files: `name.component.ts`, `name.service.ts`, `name.spec.ts`.

## Quality bar

- Correctness and UX coherence beat clever diffs.
- Performance matters: OnPush, lazy routes, avoid needless recomputation, respect existing caching/relay patterns.
- Accessibility is part of done for UI work — labels, focus, keyboard paths, contrast via theme tokens.
- Tests are good when focused. Prefer high-signal unit/E2E coverage over snapshot spam or tautological tests.
- Do not add dependencies unless the payoff is clear and local alternatives are worse.
- Do not expand scope into adjacent refactors "while here."
- Prefer deleting dead code that your change makes unreachable over leaving tombs.

## Workflow defaults

| Situation | Default |
| --- | --- |
| Ambiguous product choice | Pick the option aligned with `ARCHITECTURE.md` and existing UX; implement it |
| Bugfix | Reproduce mentally from code paths, fix root cause, add a tight regression test when cheap |
| Feature | Implement vertical slice: UI + state + protocol/service + palette command + docs if architectural |
| Styling | Theme tokens + both color modes; no one-off hex |
| SSR/meta/routing changes | Validate social preview tags do not fall back to homepage generics |
| Unsure about a NIP | Check protocol usage in-repo first; do not invent tag semantics |

### Social preview check (SSR/meta regressions)

```bash
curl -A "Twitterbot/1.0" "https://nostria.app/e/<nevent>" | grep -E "og:title|og:description|twitter:title|twitter:description|og:image|twitter:image"
```

Dynamic pages must emit entity-specific tags, not `Nostria - Your Social Network` fallbacks.

## Commands

```bash
npm run start                 # http://localhost:4200
npm run build
npm run lint
npm run lint-fix
npm run test                  # unit (Karma/Jasmine)
npm run test:e2e              # Playwright
npm run test:e2e:auth
npm run test:e2e:full
npm run test:e2e:metrics
npm run test:e2e:visual
npm run test:e2e:visual:update
npm run test:e2e:report:full

npx playwright test e2e/tests/home.spec.ts
npx playwright test -g "should load the home page"
npx playwright test --grep @smoke
npx playwright test --grep @public
npx playwright test --grep @auth
npx playwright test --grep @security
```

Use the smallest command that validates the change. Escalate only when needed.

## E2E essentials

Full system: [`TESTING.md`](./TESTING.md).

- Import from `e2e/fixtures`, not raw `@playwright/test`.
- Tag describes: e.g. `'Feature @auth @smoke'`.
- `waitForAppReady()` before assertions; `saveConsoleLogs()` at end.
- Constants from `e2e/fixtures/test-data.ts`.
- `authenticatedPage` for logged-in flows.
- No `data-testid` convention — use roles, Material selectors, classes, text.
- Handle empty states; fixtures may have sparse relay history.
- Protocol times in tests: seconds.

Useful outputs after runs:

- `test-results/test-summary.json`
- `test-results/results.json`
- `test-results/logs/*.json`
- `test-results/metrics/*.json`
- `test-results/reports/test-report.md` via `npm run test:e2e:report:full`

Tags: `@public` `@auth` `@smoke` `@metrics` `@network` `@security` `@a11y` `@visual`

Key infra: `e2e/fixtures.ts`, `e2e/helpers/auth.ts`, `e2e/fixtures/test-data.ts`, `e2e/fixtures/mock-events.ts`.

## Anti-patterns

- Tutorial tone, long preambles, restating the task back in softer words
- "Would you like me to..." when the next step is obvious
- Scaffolding half-implementations or TODOs left as the deliverable
- Introducing NgModules, HttpClient, Material dialogs, or decorator inputs "out of habit"
- Millisecond timestamps in Nostr events
- Hardcoded colors / font-weight / `color="primary"`
- Editing generated API clients
- Broad refactors bundled into unrelated fixes
- Test bloat that locks implementation details without protecting behavior
- Asking for permission to follow documented project conventions
