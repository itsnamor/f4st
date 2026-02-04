# Routes

Composition root for navigation. Wires modules into the router.

## Rules

- **Can import from** everywhere (`modules/`, `core/`, `shared/`)
- **No one imports from routes/** - This is the final assembly point
- **Route definitions** - Maps paths to page components
- **Guards** - Authentication, authorization, redirects

## Examples

```plaintext
routes/
├── index.ts
├── auth.ts
├── dashboard.ts
└── guards/
    ├── auth.guard.ts
    └── guest.guard.ts
```

Or file-based routing structure depending on your router choice.

## When to Add Code Here

✅ **Add to routes/ when:**

- It's a route definition or path mapping
- It's a navigation guard or middleware

❌ **Don't add to routes/ when:**

- It's a page component (belongs in modules/)
- It's router configuration/setup (belongs in core/router/)
- It's a layout component (belongs in core/ui/ or modules/)
