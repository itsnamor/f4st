# Web Application Architecture

## Overview

This document proposes an architectural methodology for modern web applications. The approach is framework-agnostic, though examples use React for illustration.

### Goals

1. **Fast onboarding** - New team members can understand and contribute quickly with minimal learning curve. The structure should be intuitive enough that developers can locate code and understand boundaries without extensive documentation.

2. **Long-term maintainability** - Structure supports stable growth and scaling without accumulating technical debt. Changes in one area should have minimal impact on other areas. The architecture should remain consistent as the application grows from small to large scale.

### Non-goals

- Strict naming conventions or code style enforcement
- Framework-specific patterns
- Tooling configuration (covered separately)

---

## Architecture Layers

The application consists of three layers with a unidirectional dependency flow.

```
┌─────────────────────────────────────────┐
│              modules/                   │  Layer 2 - Project-specific
├─────────────────────────────────────────┤
│              shared/                    │  Layer 1 - Reusable utilities
├─────────────────────────────────────────┤
│           infrastructure/               │  Layer 0 - Foundation
└─────────────────────────────────────────┘

Dependency flow: modules → shared → infrastructure
```

The dependency rule is strict: upper layers can import from lower layers, but lower layers must never import from upper layers. This ensures that foundational code remains stable and changes flow predictably through the system.

---

### Layer 0: Infrastructure

Foundation layer containing all essential setup and configuration required to run the application. This layer is established early in the project lifecycle and changes infrequently after initial setup.

#### Characteristics

- **Stability**: Code in this layer should be highly stable. Changes here potentially affect the entire application.
- **Singleton nature**: Many infrastructure components are singletons (one instance per application).
- **Framework integration**: This is where framework-specific configuration lives.

#### Common Components

The following are common examples, but this list is not exhaustive. Any foundational setup that the application depends on belongs here:

| Category | Examples |
|----------|----------|
| Routing | Router configuration, route definitions, navigation utilities |
| State Management | Store setup, state persistence, hydration logic |
| API Layer | HTTP client configuration, request/response interceptors, API base setup |
| Real-time Communication | WebSocket client instance, connection management, reconnection logic |
| Styling | Theme configuration, global styles, CSS framework setup |
| UI Library | Third-party component library configuration and customization |
| Environment | Environment variables, feature flags, runtime configuration |
| Error Handling | Global error boundaries, error reporting setup |
| Logging | Logger instance, log level configuration |
| Authentication | Auth provider setup, token management infrastructure |
| Internationalization | i18n setup, locale configuration |

#### Guidelines

- Keep infrastructure components loosely coupled from each other when possible.
- Provide clear initialization order if components have dependencies.
- Document any global side effects (e.g., axios interceptors that modify all requests).

---

### Layer 1: Shared

Reusable, project-agnostic code that can be copied to another project with minimal or no modifications. This layer contains utilities, helpers, and common abstractions that are not tied to specific business logic.

#### Characteristics

- **Portability**: Code should work in any project with similar technical stack.
- **No business logic**: Should not contain domain-specific rules or data structures.
- **Pure when possible**: Prefer pure functions without side effects for utilities.
- **Well-tested**: Shared code is used widely, so reliability is critical.

#### Categories

| Category | Description | Examples |
|----------|-------------|----------|
| **Utils** | Pure utility functions for common operations | `format-currency.ts`, `format-date.ts`, `slugify.ts`, `deep-clone.ts` |
| **Hooks** | Reusable React hooks with generic functionality | `use-debounce.ts`, `use-local-storage.ts`, `use-media-query.ts`, `use-click-outside.ts` |
| **Components** | Generic UI components not tied to business logic | Layout primitives, loading spinners, empty states, generic modals |
| **Types** | Shared type definitions used across the application | `api-response.type.ts`, `pagination.type.ts`, `nullable.type.ts` |
| **Constants** | Application-wide constants | `http-status.ts`, `keyboard-keys.ts`, `regex-patterns.ts` |
| **Validators** | Generic validation functions | `is-email.ts`, `is-url.ts`, `is-phone-number.ts` |

#### Example Structure

```
shared/
├── utils/
│   ├── format-currency.ts
│   ├── format-date.ts
│   ├── deep-clone.ts
│   └── index.ts
├── hooks/
│   ├── use-debounce.ts
│   ├── use-local-storage.ts
│   ├── use-media-query.ts
│   └── index.ts
├── components/
│   ├── loading-spinner/
│   ├── empty-state/
│   └── index.ts
├── types/
│   ├── api-response.type.ts
│   ├── pagination.type.ts
│   └── index.ts
└── constants/
    ├── http-status.ts
    └── index.ts
```

#### Guidelines

- If a utility requires project-specific configuration, it likely belongs in a module instead.
- Avoid creating "junk drawer" files like `misc.ts` or `helpers.ts`. Be specific with naming.
- Each utility should do one thing well.

---

### Layer 2: Modules

Project-specific code where most development happens. Each module is a self-contained unit that encapsulates related functionality, components, hooks, and types.

#### What is a Module?

A module can be:

- **A feature**: A distinct functional area of the application (`auth`, `dashboard`, `settings`, `user-profile`, `checkout`)
- **A cross-cutting concern**: Functionality used across features but still project-specific (`form-validation`, `permissions`, `analytics`, `notifications`)

The distinction between feature and cross-cutting concern is not enforced structurally. All modules live flat within the `modules/` directory.

#### Characteristics

- **Self-contained**: A module should contain everything it needs to function.
- **Explicit interface**: External code interacts with modules only through their public API.
- **Business logic lives here**: Domain rules, business validation, and feature-specific behavior.

---

## Module Structure

### Public Interface Pattern

Each module exposes its public API through an `index.ts` file (barrel export). This file acts as the contract between the module and the rest of the application.

```
modules/
└── auth/
    ├── index.ts              # Public interface - only exported items are public
    ├── components/
    │   ├── login-form.tsx
    │   └── user-avatar.tsx
    ├── hooks/
    │   ├── use-auth.ts       # Exported
    │   └── use-session.ts    # Internal only
    ├── utils/
    │   └── token.ts          # Internal only
    └── types/
        └── user.type.ts      # Exported types
```

```ts
// modules/auth/index.ts

// Components
export { LoginForm } from './components/login-form'
export { UserAvatar } from './components/user-avatar'

// Hooks
export { useAuth } from './hooks/use-auth'

// Types
export type { User, AuthState, LoginCredentials } from './types/user.type'

// Note: use-session.ts and token.ts are NOT exported
// They are internal implementation details
```

### Why Barrel Exports Matter

1. **Clear boundaries**: Looking at `index.ts` immediately shows what the module offers.
2. **Refactor safety**: Internal file structure can change without affecting consumers.
3. **Intentional API design**: Forces developers to think about what should be public.
4. **Easier imports**: Consumers use clean imports like `@/modules/auth` instead of deep paths.

### Import Rules

| Rule | Correct | Incorrect |
|------|---------|-----------|
| Import from module index | `import { useAuth } from '@/modules/auth'` | |
| No deep imports | | `import { useSession } from '@/modules/auth/hooks/use-session'` |
| Use absolute imports | `@/modules/auth` | `../../modules/auth` |
| Import types explicitly | `import type { User } from '@/modules/auth'` | |

### Cross-module Dependencies

When module A needs functionality from module B, follow this priority:

#### Option 1: Extract to Shared (Preferred)

If the functionality is generic enough, move it to `shared/`. This is the cleanest solution as it eliminates the cross-module dependency entirely.

**Before:**
```
modules/auth/utils/validate-email.ts  ← used by modules/checkout
```

**After:**
```
shared/validators/is-email.ts  ← used by both modules
```

#### Option 2: Import via Public Interface

If the functionality is genuinely business-specific and belongs in a module, the consuming module imports it through the public interface.

```ts
// modules/checkout/components/checkout-form.tsx
import { useAuth } from '@/modules/auth'
import type { User } from '@/modules/auth'
```

**Important considerations:**

- This creates a dependency between modules. Document significant dependencies.
- Avoid circular dependencies. If A depends on B, B must not depend on A.
- If many modules depend on one module, consider if that module should be split or if some parts should move to `shared/`.

---

## Decision Guidelines

This section provides detailed guidance for common architectural decisions.

### When to Create a Module vs Shared

Use this decision tree to determine where code belongs:

```
START
  │
  ├─ Is it already in shared/ or infrastructure/?
  │   └─ YES → Use the existing code. Do not duplicate.
  │
  ├─ Is it tied to a specific business domain or feature?
  │   └─ YES → Create a MODULE
  │       Examples:
  │       - "Order validation" → modules/order-validation
  │       - "User permissions" → modules/permissions
  │       - "Dashboard widgets" → modules/dashboard
  │
  ├─ Does it manage its own state or have side effects specific to this project?
  │   └─ YES → Create a MODULE
  │       Examples:
  │       - "Shopping cart state" → modules/cart
  │       - "Notification queue" → modules/notifications
  │
  ├─ Does it depend on other modules?
  │   └─ YES → Create a MODULE
  │       If code needs to import from modules/, it cannot be in shared/
  │
  ├─ Can it be copied to a completely different project and work immediately?
  │   └─ YES → Add to SHARED
  │       Examples:
  │       - "Format phone number" → shared/utils/format-phone.ts
  │       - "useDebounce hook" → shared/hooks/use-debounce.ts
  │       - "Email regex pattern" → shared/constants/regex-patterns.ts
  │
  └─ Still unsure?
      └─ Default to MODULE. It's easier to move from module to shared
         than to untangle shared code that accumulated dependencies.
```

### Detailed Examples

| Code | Decision | Reasoning |
|------|----------|-----------|
| `formatCurrency(amount, locale)` | **Shared** | Pure function, no business logic, works in any project |
| `formatOrderTotal(order)` | **Module** | Depends on Order type, contains business rules about what's included in total |
| `useDebounce(value, delay)` | **Shared** | Generic hook, no project-specific logic |
| `useAuth()` | **Module** | Tied to project's user model and auth system |
| `validateEmail(email)` | **Shared** | Generic validation, standard email rules |
| `validateCheckoutForm(form)` | **Module** | Business-specific validation rules for checkout |
| `Button`, `Modal`, `Tooltip` | **Infrastructure** (UI Library) | Part of design system setup |
| `OrderCard`, `UserProfileBadge` | **Module** | Business-specific components |
| `Pagination` type | **Shared** | Generic pattern used everywhere |
| `Order` type | **Module** | Business entity specific to this project |

### When to Split a Module

Modules should remain cohesive. Watch for these signals that indicate a module has grown too large:

#### Signals to Split

| Signal | Description | Example |
|--------|-------------|---------|
| **Unrelated features coexist** | Module contains functionality that serves different purposes | `user` module handles profile, settings, billing, notifications, and activity log |
| **Large public interface** | `index.ts` exports more than 15-20 items | Indicates the module is doing too many things |
| **Generic naming** | Module name is vague like `utils`, `common`, `misc`, `helpers` | These names suggest unclear responsibility |
| **Circular dependencies** | Module A imports from B, and B imports from A | Architecture violation that causes maintenance issues |
| **Frequent merge conflicts** | Multiple developers regularly conflict on the same module | Module is a bottleneck |
| **Difficulty explaining** | Hard to describe what the module does in one sentence | Lack of cohesion |

#### How to Split

1. Identify the distinct responsibilities within the module.
2. Create new modules for each responsibility.
3. Move related code to the new modules.
4. Update the original module's `index.ts` to re-export from new modules (temporary, for backward compatibility).
5. Gradually update consumers to import from new modules.
6. Remove the re-exports once migration is complete.

**Example split:**

```
BEFORE:
modules/user/
├── components/profile-card.tsx
├── components/settings-form.tsx
├── components/billing-info.tsx
├── hooks/use-user.ts
├── hooks/use-settings.ts
├── hooks/use-billing.ts
└── index.ts  (exports everything)

AFTER:
modules/user/           → Core user functionality
modules/user-settings/  → Settings feature
modules/billing/        → Billing feature
```

### When a Module Size is Acceptable

A module is healthy when:

- All code serves a single, clear purpose.
- The module name accurately describes its contents.
- New team members can understand the module's scope quickly.
- Most dependencies flow inward (module is consumed, not consuming many others).
- Changes are localized (modifying the module rarely requires changes elsewhere).

---

## Related Documents

- [Development Principles](./01-development-principles.md) - Mindset and principles for maintaining architecture integrity

---

## Summary

| Layer | Purpose | Changes Frequency | Dependency Rule |
|-------|---------|-------------------|-----------------|
| Infrastructure | Foundation setup | Rarely | Cannot import from shared or modules |
| Shared | Reusable utilities | Occasionally | Can import from infrastructure |
| Modules | Business features | Frequently | Can import from shared and infrastructure |

Key principles:
- Unidirectional dependency flow
- Explicit public interfaces via barrel exports
- Extract shared logic rather than creating cross-module dependencies
- Interface-first thinking
- Prefer extending over modifying