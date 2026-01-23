# Development Principles

This document outlines the mindset and principles required to maintain architecture integrity over time. These principles apply to all developers working within the [architecture](./00-architecture.md).

---

## Overview

The architecture provides structure, but structure alone does not guarantee maintainability. How developers think about and interact with the codebase matters equally. This guide covers:

1. Interface-first thinking
2. Managing public interfaces
3. Handling breaking changes
4. Type definition conventions

---

## Interface-first Thinking

When creating or modifying a module, think about its interface before its implementation.

### 1. Design the Public Interface First

Before writing implementation code, answer these questions:

- What will consumers need from this module?
- What is the minimum API surface required?
- What types need to be exposed?

Write the `index.ts` exports before writing the implementation. This forces clarity about the module's purpose and boundaries.

```ts
// modules/auth/index.ts
// Write this FIRST, before implementing the functions

// Components
export { LoginForm } from './components/login-form'
export { UserAvatar } from './components/user-avatar'

// Hooks
export { useAuth } from './hooks/use-auth'

// Types
export type { User, AuthState, LoginCredentials } from './types/user.type'
```

### 2. Export Less Rather Than More

Every export is a commitment to maintain that API.

| Principle | Explanation |
|-----------|-------------|
| Start minimal | Export only what is immediately needed. |
| Add on demand | Add new exports when there is a concrete use case, not "just in case." |
| Internal by default | Assume code is internal unless there is a reason to expose it. |

More exports mean:
- More surface area to maintain
- More potential breaking changes
- More documentation to write
- More testing to ensure stability

### 3. Treat Exports as Contracts

Once something is exported and used by other code, it becomes a contract.

- Changing it affects all consumers.
- Breaking changes require migration planning.
- Removing it requires deprecation process.

**Rule of thumb:**
- Think twice before exporting.
- Think three times before changing an export.

### 4. Prefer Extending Over Modifying

When requirements change, prefer adding new functionality rather than changing existing functionality.

| Instead of... | Do this... |
|---------------|------------|
| Changing a function signature | Create a new function with the new signature |
| Adding a required parameter | Add an optional parameter with a default value |
| Changing return type | Create a new function that returns the new type |
| Renaming an export | Export under both names, deprecate the old one |

This approach:
- Maintains backward compatibility
- Gives consumers time to migrate
- Reduces the risk of breaking production code

---

## Managing Public Interfaces

### What Belongs in index.ts

The `index.ts` file is the public interface of a module. Include:

| Export Type | When to Include |
|-------------|-----------------|
| Components | When other modules need to render them |
| Hooks | When other modules need the functionality |
| Types | When other modules need to type their code |
| Constants | When values are part of the public API |
| Utility functions | When logic needs to be reused outside the module |

### What Does NOT Belong in index.ts

Do not export internal implementation details:

| Do Not Export | Reason |
|---------------|--------|
| Internal helper functions | Implementation detail, may change |
| Internal components | Not designed for external use |
| Internal types | Only used within the module |
| Internal constants | Configuration specific to module internals |

### Organizing Exports

Group exports by category for readability:

```ts
// modules/checkout/index.ts

// --- Components ---
export { CheckoutForm } from './components/checkout-form'
export { OrderSummary } from './components/order-summary'
export { PaymentMethodSelector } from './components/payment-method-selector'

// --- Hooks ---
export { useCheckout } from './hooks/use-checkout'
export { usePaymentMethods } from './hooks/use-payment-methods'

// --- Types ---
export type { CheckoutState, CheckoutItem } from './types/checkout.type'
export type { PaymentMethod, PaymentResult } from './types/payment.type'

// --- Constants ---
export { CHECKOUT_STEPS } from './constants/steps'
```

---

## Handling Interface Changes

Changes to public interfaces require careful consideration.

### Change Risk Levels

| Change Type | Risk | Impact |
|-------------|------|--------|
| Add new export | Safe | None. Existing code unaffected. |
| Add optional parameter | Safe | Existing calls continue to work. |
| Add required parameter | Breaking | All existing calls will fail. |
| Change parameter type | Breaking | Type errors in all consumers. |
| Change return type | Breaking | Consumers expecting old type will break. |
| Rename export | Breaking | Import statements will fail. |
| Remove export | Breaking | All usages will fail. |

### Safe Changes

These changes can be made freely:

```ts
// Adding a new export - SAFE
export { NewComponent } from './components/new-component'

// Adding optional parameter - SAFE
// Before
export const formatOrder = (order: Order) => { ... }

// After
export const formatOrder = (order: Order, options?: FormatOptions) => { ... }
```

### Breaking Changes

These changes require a deprecation process:

```ts
// Changing parameter type - BREAKING
// Do NOT do this:
export const getUser = (id: number) => { ... }  // was string, now number

// Instead, create a new function:
export const getUser = (id: string) => { ... }      // keep original
export const getUserById = (id: number) => { ... }  // new function
```

---

## Deprecation Process

When an export needs to change or be removed, follow this process.

### Step 1: Mark as Deprecated

Add a JSDoc `@deprecated` annotation with:
- What to use instead
- When it will be removed
- Migration instructions

```ts
/**
 * @deprecated Use `getCurrentUser()` instead. Will be removed in v2.0.
 *
 * Migration:
 * - Before: const user = getUser()
 * - After:  const user = getCurrentUser()
 */
export const getUser = () => getCurrentUser()
```

### Step 2: Provide the Alternative

Create the new function/component/type that replaces the deprecated one:

```ts
/**
 * Returns the currently authenticated user.
 * Returns null if no user is authenticated.
 */
export const getCurrentUser = (): User | null => {
  // Implementation
}
```

### Step 3: Communicate

Inform the team about the deprecation:
- Announce in team channel
- Document in changelog
- Mention in PR description

### Step 4: Set Timeline

Be clear about when the deprecated export will be removed:
- Specific version (e.g., "v2.0")
- Specific date (e.g., "2025-Q2")
- After migration confirmation

### Step 5: Verify and Remove

Before removing:
1. Search codebase for usages
2. Confirm all usages are migrated
3. Remove the deprecated export
4. Update changelog

---

## Type Definition Conventions

### Use Type Aliases

Use `type` instead of `interface` for consistency:

```ts
// Preferred
type User = {
  id: string
  email: string
  displayName: string
}

// Avoid
interface User {
  id: string
  email: string
  displayName: string
}
```

### File Naming

Type definition files use the `.type.ts` suffix:

```
modules/auth/types/
├── user.type.ts
├── session.type.ts
└── credentials.type.ts
```

### Organizing Types

Keep types close to where they are used:

| Type Scope | Location |
|------------|----------|
| Module-specific | `modules/[module]/types/` |
| Shared across modules | `shared/types/` |
| Infrastructure types | `infrastructure/[component]/types/` |

### Type Export Pattern

Export types explicitly using `export type`:

```ts
// modules/auth/index.ts
export type { User, AuthState } from './types/user.type'
export type { Session } from './types/session.type'
```

```ts
// Consumer
import type { User } from '@/modules/auth'
```

### Composing Types

Build complex types from simpler ones:

```ts
// modules/auth/types/user.type.ts

export type UserId = string

export type User = {
  id: UserId
  email: string
  displayName: string
  createdAt: Date
  updatedAt: Date
}

export type UserProfile = User & {
  avatar: string | null
  bio: string | null
  socialLinks: SocialLinks
}

export type UserSummary = Pick<User, 'id' | 'displayName'>
```

---

## Summary

| Principle | Description |
|-----------|-------------|
| Interface-first | Design the public API before implementation |
| Minimal exports | Export only what is needed |
| Contracts | Treat exports as commitments |
| Extend over modify | Add new rather than change existing |
| Deprecation process | Plan and communicate breaking changes |
| Type conventions | Use type aliases, explicit exports, clear organization |

These principles ensure the codebase remains maintainable as it grows and the team scales.
