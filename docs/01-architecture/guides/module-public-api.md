# Module Public API

Modules communicate through public APIs defined in `index.ts`. This prevents tight coupling and makes refactoring easier.

## The Rule

Each module must export through `index.ts`. Other modules cannot deep import.

```ts
// Good - import from public API
import { LoginForm, useLogin } from '@/modules/auth/login'

// Bad - deep import into module internals
import { LoginForm } from '@/modules/auth/login/components/login-form'
```

## What to Export

Only export what other modules need:

```ts
// modules/auth/login/index.ts
export { LoginForm } from './components/login-form'
export { useLogin } from './hooks/use-login'
export type { LoginCredentials } from './types'

// Don't export internal utilities, helpers, or implementation details
```

## Why This Matters

- **Refactoring freedom** - Internal structure can change without breaking other modules
- **Clear boundaries** - Easy to see what a module provides
- **Reduced coupling** - Modules depend on contracts, not implementations
