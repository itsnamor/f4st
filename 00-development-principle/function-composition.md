# Function Composition

---

## 1. Overview

**Function Composition** is the process of combining two or more functions to produce a new function where the output of one function becomes the input of the next. It's a fundamental concept in functional programming that enables building complex transformations from simple, reusable building blocks.

**Key Idea:** Small, focused functions combined together create powerful, readable data transformation pipelines.

---

## 2. Why It Matters

### The Problem

When code isn't composed from small functions:

- Large monolithic functions are hard to test and understand
- Deeply nested function calls become unreadable
- Logic cannot be reused in different contexts
- Intermediate variables clutter the code
- Difficult to modify, insert, or remove individual steps
- Can't easily see the data transformation flow

### The Solution

Following Function Composition provides:

- **Testability** - Each small function is easy to test in isolation
- **Readability** - Data flows clearly from left to right (or top to bottom)
- **Reusability** - Functions can be recombined in different ways
- **Maintainability** - Easy to insert, remove, or modify steps
- **Clarity** - The transformation pipeline is self-documenting

---

## 3. Core Concepts

### The Essence of Composition

```plaintext
┌─────────────────────────────────────────────────────────────────┐
│                    FUNCTION COMPOSITION                         │
│                                                                 │
│   Composition: Combine functions where output flows to input    │
│                                                                 │
│   Given: f(x) and g(x)                                          │
│   Compose: (f ∘ g)(x) = f(g(x))                                 │
│                                                                 │
│   ┌───────┐      ┌───────┐      ┌───────┐                       │
│   │ Input │ ───▶ │  f()  │ ───▶ │  g()  │ ───▶ Output           │
│   └───────┘      └───────┘      └───────┘                       │
│                                                                 │
│   Example:                                                      │
│   const addOne = x => x + 1                                     │
│   const double = x => x * 2                                     │
│   const addOneThenDouble = compose(double, addOne)              │
│                                                                 │
│   addOneThenDouble(5) = double(addOne(5)) = double(6) = 12      │
└─────────────────────────────────────────────────────────────────┘
```

### Pipe vs Compose

```plaintext
┌─────────────────────────────────────────────────────────────────┐
│                    PIPE vs COMPOSE                              │
│                                                                 │
│   PIPE: Left-to-right (more intuitive for most developers)      │
│   ────────────────────────────────────────────────────────      │
│   pipe(f, g, h)(x) = h(g(f(x)))                                 │
│                                                                 │
│   Read as: "Take x, apply f, then g, then h"                    │
│                                                                 │
│   x ──▶ f() ──▶ g() ──▶ h() ──▶ result                          │
│                                                                 │
│                                                                 │
│   COMPOSE: Right-to-left (mathematical convention)              │
│   ────────────────────────────────────────────────────────      │
│   compose(f, g, h)(x) = f(g(h(x)))                              │
│                                                                 │
│   Read as: "f of g of h of x"                                   │
│                                                                 │
│   result ◀── f() ◀── g() ◀── h() ◀── x                          │
│                                                                 │
│                                                                 │
│   💡 TIP: Use PIPE for readability - it matches how we read     │
│   code (left to right, top to bottom).                          │
└─────────────────────────────────────────────────────────────────┘
```

### Building Blocks for Composition

```plaintext
┌─────────────────────────────────────────────────────────────────┐
│              REQUIREMENTS FOR COMPOSABLE FUNCTIONS              │
│                                                                 │
│   1. PURE FUNCTIONS                                             │
│      No side effects, same input → same output                  │
│      (Impure functions break the pipeline)                      │
│                                                                 │
│   2. SINGLE RESPONSIBILITY                                      │
│      Each function does ONE thing well                          │
│      (Complex functions are hard to compose)                    │
│                                                                 │
│   3. TYPE ALIGNMENT                                             │
│      Output type of f must match input type of g                │
│      f: A → B, g: B → C, then pipe(f, g): A → C                 │
│                                                                 │
│   4. UNARY PREFERENCE                                           │
│      Functions taking one argument compose most easily          │
│      (Use currying for multi-argument functions)                │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms

| Term | Definition |
| --- | --- |
| Composition | Combining functions where output of one feeds input of next |
| Pipe | Composition from left to right (data flows forward) |
| Compose | Composition from right to left (mathematical style) |
| Currying | Converting multi-argument function to sequence of single-argument functions |
| Point-Free | Function definition without mentioning arguments |

---

## 4. Workflow & Checklist

### Recommended Workflow

```plaintext
1. IDENTIFY        →   2. EXTRACT         →   3. COMPOSE        →   4. TEST
   Steps               Small Functions        Pipeline              Each Part

   "What are the       "Create pure,          "Chain them          "Verify each
    transformation      single-purpose         together with         function works
    steps?"             functions"             pipe/compose"         independently"
```

### Checklist

Before composing functions, verify:

- [ ] **SINGLE PURPOSE:** Is each function doing exactly ONE thing?
- [ ] **PURE:** Are all functions pure (no side effects)?
- [ ] **TYPES MATCH:** Does output type of each function match input of next?
- [ ] **TESTABLE:** Can each function be tested independently?
- [ ] **READABLE:** Does the pipeline clearly express the transformation?

---

## 5. Examples

### Example 1: Data Transformation Pipeline

❌ **Bad:**

```typescript
// Deeply nested, hard to read and modify
function processUser(user: User): ProcessedUser {
  return formatOutput(
    addMetadata(
      sortByDate(
        filterActive(
          normalizeNames(
            validateUsers([user])
          )
        )
      )
    )
  );
}

// What order do these run in? Hard to tell!
// What if I need to add a step? Nested nightmare!
```

✅ **Good:**

```typescript
// Helper: Simple pipe function
const pipe =
  <T>(...fns: Array<(arg: T) => T>) =>
  (value: T): T =>
    fns.reduce((acc, fn) => fn(acc), value);

// Each step is a clear, testable function
const processUsers = pipe(
  validateUsers,
  normalizeNames,
  filterActive,
  sortByDate,
  addMetadata,
  formatOutput
);

// Easy to read: validate → normalize → filter → sort → add metadata → format
// Easy to modify: just add/remove/reorder functions in the pipe

const result = processUsers([user]);

// Can also use method chaining for arrays:
const processedUsers = users
  .filter(isValid)
  .map(normalizeName)
  .filter(isActive)
  .sort(byDate)
  .map(addMetadata);
```

### Example 2: String Processing (Slugify)

❌ **Bad:**

```typescript
// Everything in one function - hard to test individual steps
function createSlug(title: string): string {
  let result = title;

  // Trim whitespace
  result = result.trim();

  // Convert to lowercase
  result = result.toLowerCase();

  // Replace spaces with dashes
  result = result.replace(/\s+/g, '-');

  // Remove special characters
  result = result.replace(/[^a-z0-9-]/g, '');

  // Remove consecutive dashes
  result = result.replace(/-+/g, '-');

  // Remove leading/trailing dashes
  result = result.replace(/^-|-$/g, '');

  return result;
}

// Problems:
// - Can't test individual transformations
// - Can't reuse "spacesToDashes" elsewhere
// - Mutable variable makes it error-prone
```

✅ **Good:**

```typescript
// Small, reusable, testable functions
const trim = (s: string): string => s.trim();

const lowercase = (s: string): string => s.toLowerCase();

const spacesToDashes = (s: string): string => s.replace(/\s+/g, '-');

const removeSpecialChars = (s: string): string => s.replace(/[^a-z0-9-]/g, '');

const removeConsecutiveDashes = (s: string): string => s.replace(/-+/g, '-');

const trimDashes = (s: string): string => s.replace(/^-|-$/g, '');

// Compose into a pipeline
const slugify = pipe(
  trim,
  lowercase,
  spacesToDashes,
  removeSpecialChars,
  removeConsecutiveDashes,
  trimDashes
);

// Usage
slugify('  Hello World! '); // 'hello-world'
slugify('---Multiple---Dashes---'); // 'multiple-dashes'

// Each function is testable:
expect(spacesToDashes('hello world')).toBe('hello-world');
expect(removeSpecialChars('hello@world!')).toBe('helloworld');

// Reusable! Create a variant:
const slugifyKeepUppercase = pipe(
  trim,
  // Skip lowercase!
  spacesToDashes,
  removeSpecialChars,
  removeConsecutiveDashes,
  trimDashes
);
```

### Example 3: Price Calculation Pipeline

❌ **Bad:**

```typescript
// All logic crammed into one function
function calculateFinalPrice(
  basePrice: number,
  quantity: number,
  discountCode: string | null,
  isMember: boolean,
  taxRate: number
): number {
  let price = basePrice * quantity;

  // Apply discount code
  if (discountCode === 'SAVE10') {
    price = price * 0.9;
  } else if (discountCode === 'SAVE20') {
    price = price * 0.8;
  }

  // Apply member discount (additional 5%)
  if (isMember) {
    price = price * 0.95;
  }

  // Apply bulk discount
  if (quantity >= 10) {
    price = price * 0.9;
  }

  // Add tax
  price = price * (1 + taxRate);

  // Round to 2 decimal places
  return Math.round(price * 100) / 100;
}
```

✅ **Good:**

```typescript
// Types for clarity
type PriceContext = {
  subtotal: number;
  quantity: number;
  discountCode: string | null;
  isMember: boolean;
  taxRate: number;
};

// Small, focused functions
const calculateSubtotal = (ctx: PriceContext): PriceContext => ({
  ...ctx,
  subtotal: ctx.subtotal * ctx.quantity,
});

const applyDiscountCode = (ctx: PriceContext): PriceContext => {
  const discounts: Record<string, number> = {
    SAVE10: 0.9,
    SAVE20: 0.8,
  };
  const multiplier = discounts[ctx.discountCode ?? ''] ?? 1;
  return { ...ctx, subtotal: ctx.subtotal * multiplier };
};

const applyMemberDiscount = (ctx: PriceContext): PriceContext => {
  if (!ctx.isMember) return ctx;
  return { ...ctx, subtotal: ctx.subtotal * 0.95 };
};

const applyBulkDiscount = (ctx: PriceContext): PriceContext => {
  if (ctx.quantity < 10) return ctx;
  return { ...ctx, subtotal: ctx.subtotal * 0.9 };
};

const applyTax = (ctx: PriceContext): PriceContext => ({
  ...ctx,
  subtotal: ctx.subtotal * (1 + ctx.taxRate),
});

const roundPrice = (ctx: PriceContext): number =>
  Math.round(ctx.subtotal * 100) / 100;

// Compose the pipeline
const calculateFinalPrice = (
  basePrice: number,
  quantity: number,
  discountCode: string | null,
  isMember: boolean,
  taxRate: number
): number => {
  const pipeline = pipe(
    calculateSubtotal,
    applyDiscountCode,
    applyMemberDiscount,
    applyBulkDiscount,
    applyTax
  );

  const result = pipeline({
    subtotal: basePrice,
    quantity,
    discountCode,
    isMember,
    taxRate,
  });

  return roundPrice(result);
};

// Benefits:
// ✅ Each step is independently testable
// ✅ Easy to add/remove/reorder discount rules
// ✅ Clear what each step does
// ✅ Business logic is explicit
```

### Example 4: React Data Flow with Composition

```tsx
// Composable data transformations for React

// Pure transformation functions
const filterByStatus = (status: string) => (items: Item[]) =>
  items.filter((item) => item.status === status);

const sortByField = <T,>(field: keyof T) => (items: T[]) =>
  [...items].sort((a, b) => (a[field] > b[field] ? 1 : -1));

const take = (n: number) => <T,>(items: T[]) => items.slice(0, n);

const toViewModel = (item: Item): ItemViewModel => ({
  id: item.id,
  title: item.title,
  displayDate: formatDate(item.createdAt),
  statusBadge: getStatusBadge(item.status),
});

// Compose for specific use cases
const getRecentActiveItems = pipe(
  filterByStatus('active'),
  sortByField('createdAt'),
  take(5),
  (items) => items.map(toViewModel)
);

const getTopCompletedItems = pipe(
  filterByStatus('completed'),
  sortByField('completedAt'),
  take(10),
  (items) => items.map(toViewModel)
);

// Use in React
function Dashboard({ items }: { items: Item[] }) {
  const recentActive = getRecentActiveItems(items);
  const topCompleted = getTopCompletedItems(items);

  return (
    <div>
      <Section title="Recent Active">
        <ItemList items={recentActive} />
      </Section>
      <Section title="Recently Completed">
        <ItemList items={topCompleted} />
      </Section>
    </div>
  );
}

// Easy to create new variations:
const getArchivedByTitle = pipe(
  filterByStatus('archived'),
  sortByField('title'),
  (items) => items.map(toViewModel)
);
```

### Example 5: Validation Pipeline

```typescript
// Composable validation

type ValidationResult<T> = { success: true; data: T } | { success: false; error: string };

// Higher-order function for creating validators
const createValidator =
  <T>(predicate: (value: T) => boolean, errorMessage: string) =>
  (result: ValidationResult<T>): ValidationResult<T> => {
    if (!result.success) return result; // Short-circuit on previous failure
    if (!predicate(result.data)) return { success: false, error: errorMessage };
    return result;
  };

// Individual validators
const isNotEmpty = createValidator<string>(
  (s) => s.trim().length > 0,
  'Value cannot be empty'
);

const isEmail = createValidator<string>(
  (s) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(s),
  'Must be a valid email address'
);

const minLength = (min: number) =>
  createValidator<string>((s) => s.length >= min, `Must be at least ${min} characters`);

const maxLength = (max: number) =>
  createValidator<string>((s) => s.length <= max, `Must be at most ${max} characters`);

const matches = (pattern: RegExp, message: string) =>
  createValidator<string>((s) => pattern.test(s), message);

// Compose validators
const validateEmail = pipe(isNotEmpty, isEmail, maxLength(254));

const validatePassword = pipe(
  isNotEmpty,
  minLength(8),
  maxLength(128),
  matches(/[A-Z]/, 'Must contain uppercase letter'),
  matches(/[a-z]/, 'Must contain lowercase letter'),
  matches(/[0-9]/, 'Must contain a number')
);

const validateUsername = pipe(
  isNotEmpty,
  minLength(3),
  maxLength(20),
  matches(/^[a-zA-Z0-9_]+$/, 'Only letters, numbers, and underscores allowed')
);

// Usage
const emailResult = validateEmail({ success: true, data: 'test@example.com' });
const passwordResult = validatePassword({ success: true, data: 'MyPass123' });

// Easy to create form-specific validation
const validateLoginForm = (email: string, password: string) => ({
  email: validateEmail({ success: true, data: email }),
  password: validatePassword({ success: true, data: password }),
});
```

---

## 6. References

- [Function Composition - Wikipedia](https://en.wikipedia.org/wiki/Function_composition_(computer_science))
- ["Professor Frisby's Mostly Adequate Guide to Functional Programming"](https://mostly-adequate.gitbook.io/mostly-adequate-guide/)
- [Ramda.js](https://ramdajs.com/) - Functional programming library with pipe/compose
- Eric Elliott - ["Composing Software"](https://medium.com/javascript-scene/composing-software-the-book-f31c77fc3ddc)
