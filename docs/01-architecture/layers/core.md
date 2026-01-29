# Core Layer

Foundation and configuration layer. Sets up external integrations and provides infrastructure for modules.

## Rules

- **Can only import from** `shared/`
- **Cannot import from** `modules/` or `routes/`
- **Abstraction layer** - Wraps third-party libraries, modules don't import directly from external packages
- **Changes rarely** - Core is stable foundation, changes here affect the entire app

## Examples

```plaintext
core/
├── api/
│   ├── client.ts
│   ├── interceptors.ts
│   └── generated/
├── ui/
│   ├── components/
│   └── theme/
├── stores/
├── config/
├── i18n/
├── router/
├── error/
└── types/
```

| Category | Examples |
| -------- | -------- |
| Routing | Router configuration, navigation utilities |
| API Layer | HTTP client, interceptors, generated queries/mutations/types/hooks |
| UI | Component library setup, theme, extended components |
| Styling | Theme tokens, global styles, CSS framework setup |
| State Management | Store setup, state persistence, hydration logic |
| Environment | Environment variables, feature flags, runtime config |
| Error Handling | Global error boundaries, error reporting (Sentry, etc.) |
| i18n | Internationalization setup, locale configuration |
| Analytics | Google Analytics, Mixpanel, Segment setup |
| Real-time | WebSocket client, connection management, reconnection logic |
| Performance | Web Vitals, monitoring setup |
| Logging | Logger instance, log level configuration |

## When to Add Code Here

✅ **Add to core/ when:**

- It's infrastructure or configuration code
- It wraps a third-party library for app-wide use
- It's shared across multiple modules and not business logic

❌ **Don't add to core/ when:**

- It contains business logic (belongs in modules/)
- It's a pure utility with no dependencies (belongs in shared/)
- It's specific to one feature (keep it in that module)
