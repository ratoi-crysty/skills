---
name: angular-components
description: Conventions for authoring Angular components — OnPush change detection, signals-first state, widgets pattern, UI library usage, single-responsibility sizing. Use when creating or refactoring Angular components, directives, pages, or when the user mentions Angular, OnPush, signals, computed, input(), output(), observables, RxJS, or component structure.
---

# Angular Components

## Core rules

1. **Always `ChangeDetectionStrategy.OnPush`** on every component.
2. **Never plain props for state** — use signals (`input()`, `model()`, `signal()`, `computed()`) or observables. No raw class fields driving the template.
3. **Prefer signals over observables.** Reach for RxJS only when you need stream operators (debounce, merge, switchMap, etc.); convert with `toSignal` at the boundary.
4. **Widgets pattern** — page/feature components own business logic; split internals into small private subcomponents. Expose minimal public interface (deep module).
5. **Use the project UI library** for generic reusable pieces. Do not import a widget's internal subcomponent from another widget — promote to UI library first.
6. **Single responsibility** — keep components small. Split when juggling unrelated concerns.
7. **Minimize custom CSS** — prefer UI library primitives, props, design tokens over bespoke styles.
8. **External template + styles** — use `templateUrl` and `styleUrl`/`styleUrls` pointing to sibling `.html` / `.scss` files. No inline `template:` or `styles:` in component decorators.

## Component template

```ts
import { ChangeDetectionStrategy, Component, computed, input, output } from '@angular/core';

@Component({
  selector: 'app-component-name',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  templateUrl: './component-name.component.html',
  styleUrl: './component-name.component.scss',
})
export class ComponentNameComponent {
  readonly value = input.required<string>();
  readonly disabled = input(false);
  readonly changed = output<string>();

  readonly label = computed(() => `Value: ${this.value()}`);
}
```

## State rules

- **Inputs** → `input()` / `input.required()` / `model()` (two-way).
- **Outputs** → `output<T>()`.
- **Local state** → `signal<T>()`.
- **Derived state** → `computed()`. Never a getter that reads signals.
- **Async streams** → keep as `Observable` while operators needed, then `toSignal(stream$, { initialValue })` for the template.
- **Side effects** → `effect()` sparingly; prefer `computed` for derivations.

## Widgets pattern

```
features/user-profile/
├── user-profile.component.ts          # public widget — orchestrator
├── user-profile.component.html
├── user-profile.component.scss
├── components/                        # private subcomponents (one folder each)
│   ├── user-profile-header/
│   │   ├── user-profile-header.component.ts
│   │   ├── user-profile-header.component.html
│   │   └── user-profile-header.component.scss
│   └── user-profile-stats/
│       ├── user-profile-stats.component.ts
│       ├── user-profile-stats.component.html
│       └── user-profile-stats.component.scss
├── services/                          # widget-scoped services
│   └── user-profile.service.ts
├── models/                            # widget-scoped types/interfaces
│   └── user-profile.model.ts
├── pipes/                             # widget-scoped pipes (if any)
├── directives/                        # widget-scoped directives (if any)
└── index.ts                           # exports UserProfileComponent only
```

- Widget = orchestrator. Injects services, holds signals, calls APIs.
- Subcomponents = presentational. Inputs/outputs only, no service injection beyond UI concerns.
- One folder per child component inside `components/`. Same rule for `services/`, `models/`, `pipes/`, `directives/` — group by kind, then by name.
- Reused outside the widget → promote to UI library (or shared lib for services/models).

## UI library usage

- Generic reusable pieces (Button, Dialog, Input, Table) live in the shared UI library.
- Never reach into `features/*` internals from another feature.
- Need a variant? Extend the UI library component, not copy-paste.

## Styling

- Use UI library components, props, and design tokens.
- Custom CSS only when library cannot express it. One-line comment explaining why.

## Checklist before committing

- [ ] `changeDetection: ChangeDetectionStrategy.OnPush` set
- [ ] No plain class fields driving template — all state via signals/observables
- [ ] Signals used; observables only where operators required
- [ ] Derived values via `computed`, not getters
- [ ] No business logic in leaf subcomponents
- [ ] No cross-widget imports of private subcomponents
- [ ] No new custom CSS where UI library suffices
- [ ] Template in external `.html`, styles in external `.scss` — no inline `template:`/`styles:`
- [ ] Each child component in its own folder under `components/`; services in `services/`, models in `models/`, etc.
