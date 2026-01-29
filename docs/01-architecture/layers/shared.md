# Shared Layer

Pure utilities with zero dependencies. Code here can be copied to any project and work immediately.

## Rules

- **Zero dependencies** - Cannot import from `core/` or `modules/`
- **Portability** - Code can be copied to any project and work immediately
- **No business logic** - Should not contain domain-specific rules
- **Pure when possible** - Prefer pure functions without side effects
- **Well-tested** - Shared code is used widely, reliability is critical

## Examples

```plaintext
shared/
├── utils/
│   ├── format-date.ts
│   ├── format-currency.ts
│   └── slugify.ts
├── hooks/
│   ├── use-debounce.ts
│   ├── use-local-storage.ts
│   └── use-media-query.ts
├── validators/
│   ├── is-email.ts
│   └── is-phone.ts
└── constants/
    └── regex-patterns.ts
```

## When to Add Code Here

✅ **Add to shared/ when:**

- The code is a general-purpose utility
- It has no dependencies on app-specific code
- It could be useful in a completely different project

❌ **Don't add to shared/ when:**

- It contains business logic or domain rules
- It depends on app configuration, API, or state
- It's only used by one module (keep it in that module)
