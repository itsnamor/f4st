# Module Dependencies

Rules for how modules communicate and depend on each other.

## Public API

Each module exports through `index.ts`. Other modules cannot deep import.

```ts
// ✅ Good - import from public API
import { LoginForm, useLogin } from '@/modules/auth/login'

// ❌ Bad - deep import into module internals
import { LoginForm } from '@/modules/auth/login/components/login-form'
```

Only export what other modules need:

```ts
// modules/auth/login/index.ts
export { LoginForm } from './components/login-form'
export { useLogin } from './hooks/use-login'
export type { LoginCredentials } from './types'

// Don't export internal utilities, helpers, or implementation details
```

**Why this matters:**

- Refactoring freedom - internal structure can change without breaking other modules
- Clear boundaries - easy to see what a module provides
- Reduced coupling - modules depend on contracts, not implementations

## Dependency Chain

Modules can import from other modules without depth limits:

```plaintext
modules/checkout/ → modules/cart/ → modules/products/ → modules/inventory/
```

This is fine as long as there are no circular dependencies.

## Circular Dependencies

**Circular dependencies between modules are not allowed.**

```ts
// ❌ Not allowed
// modules/orders/index.ts imports from modules/users/
// modules/users/index.ts imports from modules/orders/
```

If two modules need to share something, extract it:

- **Shared types** → `core/types/`
- **Shared logic** → Create a third module or move to `core/`

```plaintext
# Before: circular dependency
modules/orders/ ←→ modules/users/

# After: extract shared types
core/types/user.ts        ← shared User type
modules/orders/           → imports from core/types/
modules/users/            → imports from core/types/
```

## Sharing Code Between Modules

When two modules need the same code, follow the **Rule of Three**:

1. **First usage**: Code lives in the module that owns it
2. **Second usage**: Duplicate the code (copy to the second module)
3. **Third usage**: Extract to a shared location

**Why duplicate first?**

- Premature abstraction creates wrong abstractions
- Requirements often diverge between modules
- Copying is cheap; wrong abstractions are expensive to fix

**When to extract:**

- A third module needs the same code
- The duplicated code has been stable (no diverging changes)
- The abstraction is clear and well-understood

**Extract to:**

- `core/types/` - Shared type definitions
- `core/` - Shared infrastructure with no business logic
- New module - Shared business logic
