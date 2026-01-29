# Splitting Modules

Modules grow over time. Knowing when and how to split them keeps the codebase maintainable.

## Warning Signs

A module may need splitting when:

- **Too many files** - Hard to navigate, takes time to find things
- **Multiple domains** - The module handles unrelated concerns
- **Shared state confusion** - Unclear what state belongs to what feature
- **Long import paths** - Deep nesting within the module
- **Circular dependencies** - Components within the module depend on each other in complex ways
- **Hard to name** - If you struggle to describe what the module does in one sentence

## How to Split

### 1. Identify Boundaries

Look for natural seams:

```plaintext
# Before: one large module
modules/user-management/
├── components/
│   ├── user-list.tsx
│   ├── user-profile.tsx
│   ├── user-settings.tsx
│   ├── role-list.tsx
│   ├── role-editor.tsx
│   └── permission-matrix.tsx
├── hooks/
└── api.ts
```

### 2. Extract into Separate Modules

```plaintext
# After: focused modules
modules/
├── user-management/
│   ├── users/
│   │   ├── user-list/
│   │   └── user-profile/
│   └── roles/
│       ├── role-list/
│       ├── role-editor/
│       └── permissions/
```

Or if truly independent:

```plaintext
modules/
├── users/
│   ├── user-list/
│   └── user-profile/
├── roles/
│   ├── role-list/
│   └── role-editor/
└── permissions/
```

### 3. Define Public APIs

Each new module exports only what others need:

```ts
// modules/users/user-profile/index.ts
export { UserProfile } from './components/user-profile'
export { useUserProfile } from './hooks/use-user-profile'
export type { User } from './types'
```

## When NOT to Split

- **Premature optimization** - Don't split just because a module might grow
- **Artificial boundaries** - If split modules would constantly import from each other
- **Single responsibility** - If the module does one thing well, size is less important
