# Declarative Programming

---

## 1. Overview

**Declarative Programming** is a paradigm that expresses the logic of computation without describing its control flow. Instead of specifying step-by-step instructions for HOW to accomplish a task, you describe WHAT result you want and let the underlying system figure out how to achieve it.

**Key Idea:** Describe WHAT you want, not HOW to get it. Focus on the outcome, not the process.

---

## 2. Why It Matters

### The Problem

When code is imperative (step-by-step instructions):

- Code is cluttered with implementation details (loops, indices, temp variables)
- Intent is obscured by low-level mechanics
- More code means more opportunities for bugs
- Manual implementations may miss optimization opportunities
- Difficult to adapt to different platforms or contexts
- Harder to reason about what the code actually achieves

### The Solution

Following Declarative Programming provides:

- **Clear intent** - Code reads like a description of the result
- **Less code** - No boilerplate for iteration, accumulation, etc.
- **Fewer bugs** - Less code = fewer places for bugs to hide
- **Optimization** - Underlying system can optimize execution
- **Portability** - Same declaration can work across contexts
- **Easier reasoning** - Focus on what, not how

---

## 3. Core Concepts

### Imperative vs Declarative

```plaintext
┌─────────────────────────────────────────────────────────────────┐
│                  IMPERATIVE vs DECLARATIVE                       │
│                                                                  │
│   IMPERATIVE: Describes HOW to do something                     │
│   ───────────────────────────────────────────────────────────   │
│   "Take the first item. Check if it's positive. If yes, add it │
│    to result. Move to next item. Repeat until done."            │
│                                                                  │
│   const result = [];                                            │
│   for (let i = 0; i < numbers.length; i++) {                   │
│     if (numbers[i] > 0) {                                       │
│       result.push(numbers[i]);                                  │
│     }                                                           │
│   }                                                             │
│                                                                  │
│   DECLARATIVE: Describes WHAT you want                          │
│   ───────────────────────────────────────────────────────────   │
│   "Give me the positive numbers."                               │
│                                                                  │
│   const result = numbers.filter(n => n > 0);                    │
│                                                                  │
│   Same result, different mindset.                               │
└─────────────────────────────────────────────────────────────────┘
```

### The Spectrum of Declarativeness

```plaintext
┌─────────────────────────────────────────────────────────────────┐
│           SPECTRUM: IMPERATIVE ←───────────→ DECLARATIVE        │
│                                                                  │
│   MORE IMPERATIVE                      MORE DECLARATIVE         │
│   ───────────────────────────────────────────────────────────   │
│   Assembly        →   C                                         │
│   for loops       →   .map(), .filter(), .reduce()              │
│   DOM manipulation→   React/Vue components                      │
│   Manual SQL      →   ORM queries                               │
│   Callbacks       →   async/await                               │
│   CSS imperative  →   CSS declarations                          │
│   XMLHttpRequest  →   fetch + React Query                       │
│                                                                  │
│   🎯 Goal: Move right on this spectrum when possible            │
└─────────────────────────────────────────────────────────────────┘
```

### Examples of Declarative Patterns

```plaintext
┌─────────────────────────────────────────────────────────────────┐
│              DECLARATIVE PATTERNS IN PRACTICE                    │
│                                                                  │
│   SQL:                                                          │
│   SELECT name, email FROM users WHERE active = true             │
│   (Not: "open file, read line by line, parse, check field...")  │
│                                                                  │
│   CSS:                                                          │
│   .box { display: flex; justify-content: center; }              │
│   (Not: "calculate width, divide by 2, set left margin...")     │
│                                                                  │
│   React:                                                        │
│   <UserList users={users} />                                    │
│   (Not: "create ul, for each user create li, append...")        │
│                                                                  │
│   Regex:                                                        │
│   /\d{3}-\d{4}/                                                 │
│   (Not: "check char is digit, count 3, check hyphen...")        │
│                                                                  │
│   HTML:                                                         │
│   <form action="/submit" method="POST">                         │
│   (Not: "on click, serialize fields, create request...")        │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms

| Term | Definition |
| --- | --- |
| Imperative | Specifying step-by-step instructions (HOW) |
| Declarative | Specifying desired outcomes (WHAT) |
| DSL | Domain-Specific Language - declarative syntax for specific domains |
| Abstraction | Hiding implementation details behind simpler interfaces |

---

## 4. Workflow & Checklist

### Recommended Workflow

```plaintext
1. IDENTIFY        →   2. FIND            →   3. EXPRESS        →   4. LET SYSTEM
   Goal                 Declarative API       Intent               Handle Details

   "What result       "Is there map,         "Write what I        "Trust the
    do I want?"        filter, React,         want, not how"       implementation"
                       SQL, etc.?"
```

### Checklist

Before writing code, verify:

- [ ] **WHAT NOT HOW:** Am I describing the outcome, not the steps?
- [ ] **BUILT-IN:** Is there a built-in method or library that does this?
- [ ] **NO LOOPS:** Can `for` loops be replaced with `map`/`filter`/`reduce`?
- [ ] **NO MANUAL STATE:** Am I avoiding manual index tracking and accumulators?
- [ ] **READABLE:** Does the code read as a description of what I want?

---

## 5. Examples

### Example 1: Array Transformation

❌ **Bad:**

```typescript
// Imperative: HOW to transform
function getActiveUserEmails(users: User[]): string[] {
  const result: string[] = [];

  for (let i = 0; i < users.length; i++) {
    const user = users[i];
    if (user.isActive) {
      if (user.email) {
        const upperEmail = user.email.toUpperCase();
        result.push(upperEmail);
      }
    }
  }

  return result;
}

// Problems:
// - Manual iteration (i++, users[i])
// - Manual accumulation (result.push)
// - Multiple concerns mixed together
// - Intent buried in mechanics
```

✅ **Good:**

```typescript
// Declarative: WHAT we want
const getActiveUserEmails = (users: User[]): string[] =>
  users
    .filter((user) => user.isActive)
    .filter((user) => user.email)
    .map((user) => user.email!.toUpperCase());

// Or even more declaratively with predicates:
const isActive = (user: User) => user.isActive;
const hasEmail = (user: User) => Boolean(user.email);
const toUpperEmail = (user: User) => user.email!.toUpperCase();

const getActiveUserEmails = (users: User[]): string[] =>
  users.filter(isActive).filter(hasEmail).map(toUpperEmail);

// Benefits:
// - Reads like English: "filter active, filter has email, map to upper email"
// - No manual iteration
// - No mutable accumulator
// - Clear intent
```

### Example 2: UI Rendering

❌ **Bad:**

```typescript
// Imperative DOM manipulation
function renderUserList(users: User[]): void {
  const container = document.getElementById('users');
  if (!container) return;

  // Clear existing content
  container.innerHTML = '';

  // Create header
  const header = document.createElement('h2');
  header.textContent = 'Users';
  header.className = 'user-list-header';
  container.appendChild(header);

  // Create list
  const ul = document.createElement('ul');
  ul.className = 'user-list';

  // Add each user
  for (const user of users) {
    const li = document.createElement('li');
    li.className = 'user-item';

    const nameSpan = document.createElement('span');
    nameSpan.className = 'user-name';
    nameSpan.textContent = user.name;
    li.appendChild(nameSpan);

    const emailSpan = document.createElement('span');
    emailSpan.className = 'user-email';
    emailSpan.textContent = user.email;
    li.appendChild(emailSpan);

    if (user.isAdmin) {
      const badge = document.createElement('span');
      badge.className = 'admin-badge';
      badge.textContent = 'Admin';
      li.appendChild(badge);
    }

    ul.appendChild(li);
  }

  container.appendChild(ul);
}

// Problems:
// - 40+ lines for a simple list
// - Manual DOM creation and manipulation
// - Easy to make mistakes (appendChild order, etc.)
// - Hard to see the final structure
```

✅ **Good:**

```tsx
// Declarative React component
function UserList({ users }: { users: User[] }) {
  return (
    <div>
      <h2 className="user-list-header">Users</h2>
      <ul className="user-list">
        {users.map((user) => (
          <li key={user.id} className="user-item">
            <span className="user-name">{user.name}</span>
            <span className="user-email">{user.email}</span>
            {user.isAdmin && <span className="admin-badge">Admin</span>}
          </li>
        ))}
      </ul>
    </div>
  );
}

// Benefits:
// - Describes WHAT the UI looks like
// - Structure is visible in the code
// - React handles the HOW (DOM updates)
// - Easy to understand and modify
```

### Example 3: Data Aggregation

❌ **Bad:**

```typescript
// Imperative: Manual grouping and counting
function getOrderStats(orders: Order[]): OrderStats {
  let totalRevenue = 0;
  let completedCount = 0;
  let pendingCount = 0;
  let cancelledCount = 0;
  const revenueByMonth: Record<string, number> = {};
  const ordersByCustomer: Record<string, Order[]> = {};

  for (let i = 0; i < orders.length; i++) {
    const order = orders[i];

    // Total revenue
    totalRevenue += order.total;

    // Count by status
    if (order.status === 'completed') {
      completedCount++;
    } else if (order.status === 'pending') {
      pendingCount++;
    } else if (order.status === 'cancelled') {
      cancelledCount++;
    }

    // Group by month
    const month = order.date.substring(0, 7);
    if (!revenueByMonth[month]) {
      revenueByMonth[month] = 0;
    }
    revenueByMonth[month] += order.total;

    // Group by customer
    if (!ordersByCustomer[order.customerId]) {
      ordersByCustomer[order.customerId] = [];
    }
    ordersByCustomer[order.customerId].push(order);
  }

  return {
    totalRevenue,
    completedCount,
    pendingCount,
    cancelledCount,
    revenueByMonth,
    ordersByCustomer,
  };
}
```

✅ **Good:**

```typescript
// Declarative: Express WHAT each aggregation is

// Helper for grouping (could use lodash.groupBy)
const groupBy = <T, K extends string>(items: T[], keyFn: (item: T) => K): Record<K, T[]> =>
  items.reduce(
    (acc, item) => {
      const key = keyFn(item);
      acc[key] = [...(acc[key] || []), item];
      return acc;
    },
    {} as Record<K, T[]>
  );

// Helper for summing
const sumBy = <T>(items: T[], valueFn: (item: T) => number): number =>
  items.reduce((sum, item) => sum + valueFn(item), 0);

// Helper for counting
const countBy = <T>(items: T[], predicate: (item: T) => boolean): number =>
  items.filter(predicate).length;

// Declarative stats calculation
function getOrderStats(orders: Order[]): OrderStats {
  return {
    // WHAT: total of all order totals
    totalRevenue: sumBy(orders, (o) => o.total),

    // WHAT: count where status is X
    completedCount: countBy(orders, (o) => o.status === 'completed'),
    pendingCount: countBy(orders, (o) => o.status === 'pending'),
    cancelledCount: countBy(orders, (o) => o.status === 'cancelled'),

    // WHAT: orders grouped by month, then sum totals
    revenueByMonth: Object.fromEntries(
      Object.entries(groupBy(orders, (o) => o.date.substring(0, 7))).map(([month, monthOrders]) => [
        month,
        sumBy(monthOrders, (o) => o.total),
      ])
    ),

    // WHAT: orders grouped by customer
    ordersByCustomer: groupBy(orders, (o) => o.customerId),
  };
}

// Each line declares WHAT we want, not HOW to compute it
```

### Example 4: Configuration vs Code

❌ **Bad:**

```typescript
// Imperative: Logic scattered in code
function validateForm(data: FormData): ValidationErrors {
  const errors: ValidationErrors = {};

  if (!data.email) {
    errors.email = 'Email is required';
  } else if (!data.email.includes('@')) {
    errors.email = 'Invalid email format';
  } else if (data.email.length > 254) {
    errors.email = 'Email too long';
  }

  if (!data.password) {
    errors.password = 'Password is required';
  } else if (data.password.length < 8) {
    errors.password = 'Password must be at least 8 characters';
  } else if (!/[A-Z]/.test(data.password)) {
    errors.password = 'Password must contain uppercase';
  }

  if (!data.age) {
    errors.age = 'Age is required';
  } else if (data.age < 18) {
    errors.age = 'Must be 18 or older';
  } else if (data.age > 120) {
    errors.age = 'Invalid age';
  }

  return errors;
}
```

✅ **Good:**

```typescript
// Declarative: Schema-based validation (using Zod)
import { z } from 'zod';

// DECLARE what valid data looks like
const formSchema = z.object({
  email: z
    .string()
    .min(1, 'Email is required')
    .email('Invalid email format')
    .max(254, 'Email too long'),

  password: z
    .string()
    .min(1, 'Password is required')
    .min(8, 'Password must be at least 8 characters')
    .regex(/[A-Z]/, 'Password must contain uppercase'),

  age: z
    .number()
    .min(1, 'Age is required')
    .min(18, 'Must be 18 or older')
    .max(120, 'Invalid age'),
});

// Validation is now a declaration, not a procedure
function validateForm(data: unknown): ValidationResult {
  const result = formSchema.safeParse(data);

  if (result.success) {
    return { valid: true, data: result.data };
  }

  return {
    valid: false,
    errors: result.error.flatten().fieldErrors,
  };
}

// Benefits:
// - Schema is self-documenting
// - Can generate TypeScript types from schema
// - Easy to add/modify rules
// - Validation logic is DECLARED, not coded
```

### Example 5: State Updates

❌ **Bad:**

```typescript
// Imperative state updates
function updateShoppingCart(cart: Cart, action: CartAction): Cart {
  if (action.type === 'ADD_ITEM') {
    const existingIndex = cart.items.findIndex((i) => i.productId === action.productId);

    if (existingIndex >= 0) {
      const newItems = [...cart.items];
      newItems[existingIndex] = {
        ...newItems[existingIndex],
        quantity: newItems[existingIndex].quantity + action.quantity,
      };
      return { ...cart, items: newItems };
    } else {
      return {
        ...cart,
        items: [...cart.items, { productId: action.productId, quantity: action.quantity }],
      };
    }
  }

  // ... more imperative logic
}
```

✅ **Good:**

```typescript
// Declarative with Immer
import { produce } from 'immer';

function updateShoppingCart(cart: Cart, action: CartAction): Cart {
  return produce(cart, (draft) => {
    switch (action.type) {
      case 'ADD_ITEM': {
        const existing = draft.items.find((i) => i.productId === action.productId);
        if (existing) {
          existing.quantity += action.quantity;
        } else {
          draft.items.push({ productId: action.productId, quantity: action.quantity });
        }
        break;
      }

      case 'REMOVE_ITEM':
        draft.items = draft.items.filter((i) => i.productId !== action.productId);
        break;

      case 'CLEAR_CART':
        draft.items = [];
        break;
    }
  });
}

// Immer lets us DESCRIBE changes as if mutating
// But produces immutable updates behind the scenes
// We say WHAT should change, Immer figures out HOW
```

---

## 6. References

- [Declarative Programming - Wikipedia](https://en.wikipedia.org/wiki/Declarative_programming)
- [React Docs: Thinking in React](https://react.dev/learn/thinking-in-react) - Declarative UI
- [Zod](https://zod.dev/) - Declarative schema validation
- ["Structure and Interpretation of Computer Programs"](https://mitpress.mit.edu/sites/default/files/sicp/index.html) - Abelson & Sussman
