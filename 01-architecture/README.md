# Modern web application architecture

## Overview

This document proposes an architectural methodology for modern web applications. The approach is framework-agnostic, though examples use React for illustration.

### Goals

1. **Fast onboarding** - New team members can understand and contribute quickly with minimal learning curve. The structure should be intuitive enough that developers can locate code and understand boundaries without extensive documentation.

2. **Long-term maintainability** - Structure supports stable growth and scaling without accumulating technical debt. Changes in one area should have minimal impact on other areas. The architecture should remain consistent as the application grows from small to large scale.

---

## Architecture Layers

The application consists of three layers with a unidirectional dependency flow.

```plaintext
┌─────────────────────────────────────────┐
│              modules/                   │  Layer 2 - Project-specific
├─────────────────────────────────────────┤
│              shared/                    │  Layer 1 - Reusable utilities
├─────────────────────────────────────────┤
│               core/                     │  Layer 0 - Foundation
└─────────────────────────────────────────┘
```

The dependency rule is strict: upper layers can import from lower layers, but lower layers must never import from upper layers. This ensures that foundational code remains stable and changes flow predictably through the system.

---
