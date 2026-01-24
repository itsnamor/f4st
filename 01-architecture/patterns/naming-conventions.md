# Naming Conventions

Consistent naming helps developers navigate the codebase quickly and understand code purpose at a glance.

---

## File Naming

Use **kebab-case** for all files and folders:

```plaintext
✅ Correct
user-profile.tsx
use-auth.ts
format-currency.ts
api-response.type.ts

❌ Wrong
UserProfile.tsx
useAuth.ts
formatCurrency.ts
ApiResponse.type.ts
```

**Why kebab-case:**
- Case-insensitive file systems (macOS, Windows) don't distinguish `User.ts` from `user.ts`
- Consistent with URL conventions
- Easy to read and type

---

## File Suffixes

Use suffixes to indicate file purpose:

| Suffix | Purpose | Example |
| ------ | ------- | ------- |
| `.tsx` | React component | `login-form.tsx` |
| `.ts` | TypeScript (non-component) | `format-date.ts` |
| `.type.ts` | Type definitions | `user.type.ts` |
| `.test.ts` | Unit tests | `format-date.test.ts` |
| `.stories.tsx` | Storybook stories | `button.stories.tsx` |
| `.mock.ts` | Mock data/functions | `user.mock.ts` |
| `.constant.ts` | Constants (optional) | `http-status.constant.ts` |

---

## Folder Structure

### Modules

```plaintext
modules/
├── auth/                    # Feature module
├── user-profile/            # Use kebab-case
├── checkout/
└── notifications/
```

### Within a Module

```plaintext
modules/auth/
├── index.ts
├── components/              # Plural
├── hooks/                   # Plural
├── utils/                   # Plural
├── types/                   # Plural
├── api/                     # Singular (it's one API layer)
└── constants/               # Plural
```

---

## Export Naming

### Components

Use **PascalCase**, match file name:

```typescript
// login-form.tsx
export function LoginForm() { ... }

// user-avatar.tsx
export function UserAvatar() { ... }
```

### Hooks

Use **camelCase** with `use` prefix:

```typescript
// use-auth.ts
export function useAuth() { ... }

// use-debounce.ts
export function useDebounce() { ... }
```

### Utilities

Use **camelCase**, be descriptive:

```typescript
// format-currency.ts
export function formatCurrency() { ... }

// deep-clone.ts
export function deepClone() { ... }
```

### Types

Use **PascalCase**:

```typescript
// user.type.ts
export interface User { ... }
export type AuthState = 'idle' | 'loading' | 'authenticated'
```

### Constants

Use **SCREAMING_SNAKE_CASE** for true constants:

```typescript
// http-status.ts
export const HTTP_STATUS = {
  OK: 200,
  NOT_FOUND: 404,
} as const
```

---

## Naming Guidelines

### Be Specific, Not Generic

```plaintext
✅ Specific
format-currency.ts
validate-email.ts
use-click-outside.ts

❌ Generic
utils.ts
helpers.ts
misc.ts
```

### Match File and Export Names

```typescript
// File: login-form.tsx
// ✅ Export matches file
export function LoginForm() { ... }

// ❌ Export doesn't match
export function AuthForm() { ... }
```

### Avoid Redundant Prefixes

```plaintext
✅ Clean
modules/auth/components/login-form.tsx

❌ Redundant
modules/auth/components/auth-login-form.tsx
```

The folder path already provides context.

---

## Module Naming

| Type | Convention | Examples |
| ---- | ---------- | -------- |
| Feature | Noun (what it is) | `auth`, `dashboard`, `checkout` |
| Cross-cutting | Noun or action | `notifications`, `analytics`, `permissions` |

**Avoid:**
- Verbs alone: `authenticate` → use `auth`
- Plural when singular is clearer: `users` → use `user` (for user management)
- Generic names: `common`, `utils`, `misc`

---

## Quick Reference

| What | Convention | Example |
| ---- | ---------- | ------- |
| Files/folders | kebab-case | `user-profile.tsx` |
| Components | PascalCase | `UserProfile` |
| Hooks | camelCase + use | `useAuth` |
| Functions | camelCase | `formatDate` |
| Types/Interfaces | PascalCase | `User`, `AuthState` |
| Constants | SCREAMING_SNAKE | `HTTP_STATUS` |
| Modules | kebab-case | `user-profile/` |

---

## Related

- [Modules Layer](../layers/modules.md) - Module structure
- [Barrel Exports](./barrel-exports.md) - Export organization
