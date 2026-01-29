# Modules Layer

Feature layer. Contains business logic and cross-cutting functionality organized by domain.

## Rules

- **Can import from** `core/`, `shared/`, and other modules (via public API)
- **Cannot import from** `routes/`
- **Public API** - Each module exports through `index.ts`, other modules cannot deep import
- **Flat structure** - All modules are logically equal, group folders are for organization only

## Examples

```plaintext
modules/
├── auth/                    ← Group folder
│   ├── login/              ← Module
│   ├── register/           ← Module
│   └── forgot-password/    ← Module
├── dashboard/              ← Module (no group needed)
├── settings/               ← Module
└── notifications/          ← Cross-cutting module
```

| Type | Purpose | Examples |
| ---- | ------- | -------- |
| **Business** | Core application features | `login`, `checkout`, `dashboard` |
| **Cross-cutting** | Functionality used across features | `notifications`, `permissions`, `layouts` |

Each module can organize internally as needed:

```plaintext
modules/auth/login/
├── index.ts                ← Public API (barrel export)
├── components/
│   └── login-form.tsx
├── hooks/
│   └── use-login.ts
├── api.ts
├── types.ts
└── utils.ts
```

## When to Add Code Here

✅ **Add to modules/ when:**

- It's a business feature or domain logic
- It's cross-cutting functionality (notifications, permissions, layouts)
- It has UI, state, or API calls related to a feature

❌ **Don't add to modules/ when:**

- It's a pure utility (belongs in shared/)
- It's infrastructure or third-party wrapper (belongs in core/)
- It's route definitions (belongs in routes/)
