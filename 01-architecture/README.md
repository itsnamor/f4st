# Application Architecture

## Goals

1. **Fast onboarding** - New developers understand the codebase quickly
2. **Long-term maintainability** - Structure scales without accumulating debt

---

## Overview

The application is organized into **three layers** with a strict dependency flow, plus a **routes** folder as the composition root.

```plaintext
src/
├── main.ts              ← Entry point
├── routes/              ← Composition root (wires everything together)
├── modules/             ← Feature code (business logic)
├── core/                ← Foundation & configuration
└── shared/              ← Pure utilities (zero dependencies)
```

**Dependency Rules:**

```plaintext
┌─────────────────────────────────────────┐
│              routes/                    │  → imports from everywhere
├─────────────────────────────────────────┤
│              modules/                   │  → core/, shared/, other modules
├─────────────────────────────────────────┤
│               core/                     │  → shared/ only
├─────────────────────────────────────────┤
│              shared/                    │  → nothing (completely portable)
└─────────────────────────────────────────┘
```

---

## Quick Reference

| Layer | Purpose | Can Import From |
| --- | --- | --- |
| `routes/` | Route definitions, guards, page mapping | modules/, core/, shared/ |
| `modules/` | Business features | core/, shared/, other modules (via public API) |
| `core/` | Foundation, external integrations | shared/ only |
| `shared/` | Pure utilities | nothing |

**Learn more:**

- [routes/](./layers/routes.md) - Composition root, wires modules into router
- [modules/](./layers/modules.md) - Feature code, business logic
- [core/](./layers/core.md) - Foundation, external integrations
- [shared/](./layers/shared.md) - Pure utilities, zero dependencies

**Guides:**

- [Module Dependencies](./guides/module-dependencies.md) - Public API, dependency rules, sharing code
- [Splitting Modules](./guides/splitting-modules.md) - When and how to split large modules
