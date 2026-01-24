# Core Layer

The foundation layer containing all essential setup and configuration required to run the application.

---

## Characteristics

- **Stability** - Code here changes rarely. Modifications potentially affect the entire application.
- **Singleton nature** - Many components are single instances (one router, one API client, one theme).
- **Framework integration** - This is where framework-specific configuration lives.
- **No business logic** - Pure infrastructure, no domain rules.

---

## What Belongs Here

| Category | Examples |
| -------- | -------- |
| Routing | Router configuration, route definitions, navigation utilities |
| State Management | Store setup, state persistence, hydration logic |
| API Layer | HTTP client configuration, interceptors, base URL setup |
| Real-time | WebSocket client, connection management, reconnection logic |
| Styling | Theme configuration, global styles, CSS framework setup |
| UI Library | Component library configuration and customization |
| Environment | Environment variables, feature flags, runtime config |
| Error Handling | Global error boundaries, error reporting (Sentry, etc.) |
| Logging | Logger instance, log level configuration |
| Auth Infrastructure | Auth provider setup, token management |
| i18n | Internationalization setup, locale configuration |

---

## Example Structure

```plaintext
core/
├── api/
│   ├── client.ts           # Axios/fetch instance with interceptors
│   └── index.ts
├── router/
│   ├── router.tsx          # Router configuration
│   ├── routes.ts           # Route definitions
│   └── index.ts
├── store/
│   ├── store.ts            # Redux/Zustand store setup
│   └── index.ts
├── theme/
│   ├── theme.ts            # Theme tokens, colors, typography
│   ├── global-styles.ts    # Global CSS
│   └── index.ts
├── auth/
│   ├── auth-provider.tsx   # Auth context provider
│   ├── token-manager.ts    # Token storage, refresh logic
│   └── index.ts
└── index.ts                # Re-exports all core modules
```

---

## Guidelines

### Do

- Keep components loosely coupled from each other
- Provide clear initialization order if components have dependencies
- Document global side effects (e.g., axios interceptors)
- Export configuration through barrel files

### Don't

- Import from `modules/` - this breaks the dependency rule
- Put business logic here - that belongs in modules
- Create feature-specific code - keep it generic

---

## Dependency Rule

```plaintext
core/ → can import from shared/ only
core/ → CANNOT import from modules/
```

This allows core to use pure utilities from shared (e.g., custom cache strategies using shared utils).

---

## Checklist: Does This Belong in Core?

```plaintext
□ Is it required for the app to run at all?
□ Is it configured once and used everywhere?
□ Is it independent of specific business features?
□ Would removing it break the entire application?

All YES → core/
Any NO  → Consider shared/ or modules/
```

---

## Common Mistakes

| Mistake | Why It's Wrong | Correct Approach |
| ------- | -------------- | ---------------- |
| Putting `useAuth()` hook in core | Hook contains business logic about user state | Put hook in `modules/auth`, keep only provider in core |
| API endpoints in core | Endpoints are feature-specific | Keep base client in core, endpoints in modules |
| Feature flags logic in core | Business rules about features | Config in core, logic in modules |

---

## Related

- [Shared Layer](./shared.md) - For reusable utilities
- [Modules Layer](./modules.md) - For business features
