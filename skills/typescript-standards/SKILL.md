---
name: typescript-standards
description: TypeScript typing conventions — ban `any`, require explicit function signatures, explicit variable types, and named interfaces over inline object types. Use when writing or refactoring TypeScript (.ts/.tsx) code, reviewing types, or when the user mentions TypeScript, type safety, `any`, `unknown`, interfaces, or type annotations.
---

# TypeScript Standards

## Core rules

1. **Never use `any`** — use `unknown` and narrow.
2. **Explicit function signatures** — annotate every parameter and return type.
3. **Explicit variable declarations** — annotate variables receiving values from functions, props, or destructured payloads.
4. **Named types** — declare an `interface` / `type` instead of inline object types.

## 1. Ban `any`, use `unknown`

```ts
// ❌ Bad
function parseData(data: any): void {}

// ✅ Good
function parseData(data: unknown): void {
  if (typeof data === 'string') {
    // narrowed to string
  }
}
```

- `unknown` forces narrowing before use — safer than `any`.
- Narrow with `typeof`, `instanceof`, discriminated unions, or type guards.
- No `as any` escape hatches. If a cast is unavoidable, cast to the specific type and leave a one-line comment explaining why.

## 2. Explicit function signatures

```ts
// ❌ Bad
function getMediaUrl(id) {
  return `/api/media/${id}`;
}

// ✅ Good
function getMediaUrl(id: string): string {
  return `/api/media/${id}`;
}
```

Applies to:
- Function declarations, function expressions, arrow functions.
- Class methods, including constructors' parameters.
- Callbacks passed inline when the signature is non-trivial.

Return type included even when inferable — explicit prevents accidental drift.

## 3. Explicit variable types

```ts
// ❌ Bad
const url = getMediaUrl('123');
const { data } = response;

// ✅ Good
const url: string = getMediaUrl('123');
const { data }: MediaResponse = response;
```

Annotate when:
- Value comes from a function return, prop, API payload, or destructure.
- Reader cannot see the type at a glance from the right-hand side.

Skip annotation only for trivial literals (`const count = 0`).

## 4. Named types over inline objects

```ts
// ❌ Bad
function getMedia(id: string): { title: string } {
  return { title: 'My Video' };
}

// ✅ Good
interface MediaResponse {
  title: string;
}

function getMedia(id: string): MediaResponse {
  return { title: 'My Video' };
}
```

- `interface` for object shapes; `type` for unions, intersections, mapped types.
- Co-locate type with its primary consumer; promote to a shared `types/` folder when reused.
- Inline object types allowed only for one-off tuple-like internal helpers.

## Checklist before committing

- [ ] No `any` (including `as any`, `Array<any>`, `Record<string, any>`)
- [ ] Every function/method has typed parameters and return type
- [ ] Variables from function returns / props / destructures are annotated
- [ ] No inline `{ ... }` object types in signatures — extracted to named `interface` / `type`
- [ ] `unknown` values are narrowed before use
