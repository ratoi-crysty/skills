---
name: react-components
description: Conventions for authoring React components — memoization, widgets pattern, UI library usage, single-responsibility sizing. Use when creating or refactoring React components, pages, hooks, or when the user mentions React, JSX/TSX, memo, useCallback, useMemo, widgets, or component structure.
---

# React Components

## Core rules

1. **Memoize every component** with `memo` + named function expression.
2. **Memoize objects and callbacks** passed as props (`useMemo` / `useCallback`).
3. **Widgets pattern** — page/feature components own business logic; split internals into small private subcomponents. Expose minimal public interface (deep module).
4. **Use the project UI library** for generic reusable pieces. Do not import a widget's internal subcomponent from another widget — promote to UI library first.
5. **Single responsibility** — keep components small. Split when a component juggles unrelated concerns.
6. **Minimize custom CSS** — prefer UI library primitives/props/tokens over bespoke styles.

## Component template

```tsx
import { memo, useCallback, useMemo } from 'react';

export interface ComponentNameProps {
  // ...
};

export const ComponentName = memo(function ComponentName({}: ComponentNameProps) {
  return null;
});
```

## Memoizing props

```tsx
const config = useMemo(() => ({ limit: 10, sort: 'asc' }), []);
const handleClick = useCallback((id: string) => doThing(id), [doThing]);

return <Child config={config} onClick={handleClick} />;
```

Rules:
- Inline object/array literals as props → wrap in `useMemo`.
- Inline arrow functions as props → wrap in `useCallback`.
- Primitives and stable references — leave alone.

## Widgets pattern

```
features/user-profile/
├── UserProfile.tsx              # public widget — orchestrator
├── components/                  # private subcomponents (one folder each)
│   ├── UserProfileHeader/
│   │   ├── UserProfileHeader.tsx
│   └── UserProfileStats/
│       ├── UserProfileStats.tsx
├── hooks/                       # widget-scoped hooks
│   └── useUserProfile.ts
├── services/                    # widget-scoped API/data access
│   └── userProfile.service.ts
├── types/                       # widget-scoped types
│   └── userProfile.types.ts
├── utils/                       # widget-scoped helpers (if any)
└── index.ts                     # exports UserProfile only
```

- Widget = orchestrator. Holds queries, mutations, derived state.
- Subcomponents = presentational, narrow props, no cross-widget reuse.
- One folder per child component inside `components/`. Same rule for `hooks/`, `services/`, `types/`, `utils/` — group by kind, then by name.
- Reused outside the widget → promote to UI library (or shared lib for hooks/services/types).

## UI library usage

- Generic reusable pieces (Button, Modal, Input, Table) live in the shared UI library.
- Never reach into `features/*` internals from another feature.
- Need a variant? Extend the UI library component, not copy-paste.

## Styling

- Use UI library components, props, and design tokens.
- Custom CSS only when library cannot express it. Document why in a one-line comment.

## Checklist before committing

- [ ] Component wrapped in `memo(function Name(){})`
- [ ] Object/array/function props memoized
- [ ] No business logic inside leaf subcomponents
- [ ] No cross-widget imports of private subcomponents
- [ ] No new custom CSS where UI library suffices
- [ ] Each child component in its own folder under `components/`; hooks in `hooks/`, services in `services/`, types in `types/`, etc.
