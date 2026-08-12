# AGENTS.md - AI Coding Agent Guidelines

> Guidelines and interaction protocols for AI coding agents working in the Nostria codebase.

## Developer Persona & Collaboration Rules

- **Target Audience:** The sole developer on this project is an **extremely skilled principal/staff-level engineer**.
- **Communication Style:** High signal, high density, no filler. Skip basic explanations, patronizing introductory fluff, line-by-line tutorials, or hand-holding unless explicitly asked.
- **Execution Strategy:** Move fast with precision. Prioritize production quality, high performance, and innovative solutions, while respecting YAGNI (You Aren't Gonna Need It).
- **Proactive Engineering:** Propose bold, modern architectural or UI improvements if they meaningfully benefit the application. Be direct about technical trade-offs.

## Core Preferences & Guidelines

- **Simplicity:** Keep implementations clean and focused. Avoid over-engineering.
- **Typesafety:** Leverage TypeScript's strict type system fully. Avoid `any`; use `unknown` for dynamic or uncertain payloads.
- **Destructive Actions:** Require explicit user authorization before performing destructive file or git operations.
- **Code Comments:** Write concise, high-value comments that explain *why* complex decisions were made (e.g., Nostr protocol edge cases, performance optimizations). Do not comment self-explanatory code. Keep comments up to date.
- **Surgical Edits:** Provide exact, complete, and production-ready code changes. Avoid partial file snippets with vague placeholders (`// ... existing code ...`).

---

## Tech Stack & Architecture

- **Framework:** Angular 22+ (Standalone components, Signals, Native Control Flow)
- **Language:** TypeScript (Strict mode enabled)
- **UI & Styling:** Angular Material 3, SCSS, Material 3 Design Tokens (`--mat-sys-*`)
- **Protocol:** Nostr (using `nostr-tools` library)
- **Testing:** Playwright (E2E), Karma / Jasmine (Unit)
- **App URL:** `https://nostria.app`
- **Architecture Spec:** See [ARCHITECTURE.md](./ARCHITECTURE.md)

---

## Build / Lint / Test Commands

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

---

## Code Style & Formatting

- **Formatting:** 2-space indentation, single quotes, CRLF line endings, trim trailing whitespace.
- **Naming Conventions:**
  - Components: `app-` prefix in kebab-case (e.g., `app-user-profile`).
  - Directives: `app` prefix in camelCase (e.g., `appAutofocus`).
  - File Names: `name.component.ts`, `name.service.ts`, `name.pipe.ts`, `name.spec.ts`.

---

## TypeScript Guidelines

- **Strict Mode:** Strict type checking is strictly enforced (`noImplicitAny`, `strictNullChecks`).
- **Type Inference:** Prefer explicit types for public APIs, function signatures, and complex state; rely on inference when obvious.
- **Modern Idioms:** Write clean, modern TypeScript. Avoid redundant one-line casting wrappers or verbose imperative boilerplate.

---

## Angular Best Practices (Angular 22+)

### Component Pattern

```typescript
@Component({
  selector: 'app-example',
  imports: [CommonModule, MatButtonModule],
  templateUrl: './example.component.html',
  styleUrl: './example.component.scss',
  changeDetection: ChangeDetectionStrategy.OnPush, // ALWAYS OnPush
  host: {
    '[class.active]': 'isActive()',
    '(click)': 'handleClick()',
  },
})
export class ExampleComponent {
  // Inputs & Outputs
  data = input.required<DataType>();
  selected = output<Item>();

  // Injections & State
  private service = inject(MyService);
  items = signal<Item[]>([]);
  filtered = computed(() => this.items().filter((i) => i.active));
}
```

### Component Rules

| DO | DO NOT |
| --- | --- |
| Use Standalone components exclusively | Do NOT use NgModules |
| Omit `standalone: true` (it is default) | Do NOT write `standalone: true` in `@Component` |
| Use `input()`, `output()`, `model()` functions | Do NOT use `@Input()` or `@Output()` decorators |
| Use `inject()` function | Do NOT use constructor dependency injection |
| Use `host: {}` in `@Component` decorator | Do NOT use `@HostBinding` or `@HostListener` |
| Use `class` and `style` bindings | Do NOT use `ngClass` or `ngStyle` |
| Use `NgOptimizedImage` for static images | Do NOT use `NgOptimizedImage` for inline base64 |
| Set `changeDetection: ChangeDetectionStrategy.OnPush` | Do NOT omit `ChangeDetectionStrategy.OnPush` |

---

## Templates & Control Flow

Use Angular's native control flow syntax exclusively (`@if`, `@for`, `@switch`, `@let`):

```html
@let currentItem = selectedItem();

@if (currentItem) {
  <div class="item-detail">{{ currentItem.name }}</div>
} @else {
  <div class="empty-state">No item selected</div>
}

@for (item of items(); track item.id) {
  <app-item-card [data]="item" (selected)="onSelect($event)" />
} @empty {
  <p>No items found.</p>
}
```

> **DO NOT USE:** `*ngIf`, `*ngFor`, or `*ngSwitch`.

---

## State Management & RxJS Interop

- **Component & Service State:** Use Angular Signals (`signal`, `computed`, `linkedSignal`, `resource`).
- **Signal Updates:** Use `update()` or `set()`. **NEVER** use `mutate()`.
- **RxJS Interop:** Nostr event streams often use RxJS Observables or WebSockets. Convert between Observables and Signals cleanly using `toSignal()` and `toObservable()` from `@angular/core/rxjs-interop`.
- **Subscription Lifecycle:** Always manage cleanup using `takeUntilDestroyed()` or `DestroyRef`.

---

## Services & HTTP

- **Service Scope:** Singleton services must use `{ providedIn: 'root' }`.
- **Dependency Injection:** Inject dependencies via `inject(ServiceName)`.
- **HTTP Requests:** **ALWAYS use native `fetch` API** instead of Angular's `HttpClient`:

```typescript
@Injectable({ providedIn: 'root' })
export class DataService {
  async fetchData<T>(url: string): Promise<T> {
    const response = await fetch(url);
    if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
    return (await response.json()) as T;
  }
}
```

---

## Nostr Protocol Specifics

- **Timestamps:** Timestamps in Nostr events/protocol are in **SECONDS**, not milliseconds.
  - `Math.floor(Date.now() / 1000)` (CORRECT)
  - `Date.now()` (INCORRECT - breaks relays & event validation)
- **Library:** Use `nostr-tools` for cryptographic operations, event signing, and Bech32 entity encoding/decoding (`npub`, `nsec`, `note`, `nevent`, `nprofile`).
- **Relay Connections:** Ensure proper connection pooling and subscription cleanup to prevent memory leaks and WebSocket exhaustion.

---

## Styling & Material 3

- **CSS Variables:** Use Angular Material 3 design tokens (`--mat-sys-*`). Never hardcode hex/rgb color values.
- **Dark Mode:** Support light/dark mode via encapsulation context:
  ```scss
  :host-context(.dark) .my-element {
    background-color: var(--mat-sys-surface-container);
    color: var(--mat-sys-on-surface);
  }
  ```
- **Typography:** **NEVER set `font-weight` in CSS.** Headline fonts do not support variable weights.
- **Buttons:**
  - Primary actions: Use `mat-flat-button` (do **NOT** use `color="primary"` on buttons).
  - Icon buttons: When resizing `mat-icon-button` smaller than default, **never** use `line-height` for centering. Use:
    ```scss
    padding: 0 !important;
    display: flex !important;
    align-items: center;
    justify-content: center;
    ```
- **Dynamic Textareas:** Always use `field-sizing: content` for auto-growing textareas.

---

## Dialogs & Confirmations

- **Dialog Component:** Use `CustomDialogComponent` for all dialog overlays. Do **NOT** use Angular Material `MatDialog`.
- **User Confirmations:** **NEVER use native browser `confirm()` or `alert()`**. Use application dialogs or snackbars for confirmation flows.

---

## SSR & Browser API Safety

Always wrap browser-only API accesses (`window`, `document`, `localStorage`, `WebSocket`) with platform checks:

```typescript
private platformId = inject(PLATFORM_ID);
private isBrowser = isPlatformBrowser(this.platformId);

ngOnInit(): void {
  if (this.isBrowser) {
    const width = window.innerWidth;
  }
}
```

---

## Command Palette Integration

When adding new application features, pages, or major routes, always register corresponding commands in `src/app/components/command-palette-dialog/`. This ensures keyboard shortcut accessibility (`Ctrl+K`).

---

## Documentation & Architecture Updates

- Update [ARCHITECTURE.md](./ARCHITECTURE.md) whenever making non-trivial architectural or structural changes.
- Do not create documentation for minor bug fixes or small refactors. When generating markdown documentation, place files in the `docs/` folder.

---

## E2E Testing Guidelines (Playwright)

### Quick Execution Commands

```bash
npx playwright test --grep @smoke     # Quick smoke suite
npx playwright test --grep @auth      # Authenticated suite
npm run test:e2e:full                 # Full suite execution
npm run test:e2e:report:full          # Generate comprehensive report
```

### Social Preview Tag Verification

When modifying SSR, routing, or metadata resolvers, verify social preview tags with bot user-agents:

```bash
curl -A "Twitterbot/1.0" "https://nostria.app/e/nevent1qvzqqqqqqypzq9lz3z0m5qgzr5zg5ylapwss3tf3cwpjv225vrppu6wy8750heg4qqsqqqpsj6e662lsgy26a5g9nvav4z807m08ryhnx7ljs5dnuhpfl0cs642uw" | grep -E "og:title|og:description|twitter:title|twitter:description|og:image|twitter:image"
```

Verify that page-specific metadata is returned rather than generic homepage fallbacks.

### Test Authoring Conventions

1. Import fixtures from `e2e/fixtures`, not `@playwright/test`.
2. Tag `test.describe()` blocks appropriately (`@public`, `@auth`, `@smoke`, `@metrics`, `@network`, `@security`, `@a11y`, `@visual`).
3. Call `saveConsoleLogs()` at the end of each test block.
4. Use test constants from `e2e/fixtures/test-data.ts`.
5. Call `waitForAppReady()` prior to assertions.

