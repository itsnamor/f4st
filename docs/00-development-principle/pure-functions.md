# Pure Functions

---

## 1. Overview

**Pure Functions** are functions that always produce the same output for the same input and cause no observable side effects. They are the building blocks of functional programming and enable code that is predictable, testable, and easy to reason about.

**Key Idea:** A pure function is like a mathematical function: f(x) always equals the same value for any given x, and computing f(x) doesn't change anything else in the world.

---

## 2. Why It Matters

### The Problem

When functions have side effects or hidden dependencies:

- Functions produce unpredictable, inconsistent results
- Testing requires complex setup, mocking, and state management
- Debugging is a nightmare - "why did this value change?"
- Parallel execution causes race conditions and data corruption
- Refactoring is risky - moving code might break hidden dependencies
- Caching is unsafe - you can't memoize unpredictable functions

### The Solution

Following Pure Functions provides:

- **Predictable behavior** - Same input always gives same output
- **Easy testing** - Just assert input → output, no mocking needed
- **Simple debugging** - Function behavior is self-contained
- **Safe parallelization** - No shared mutable state to corrupt
- **Safe refactoring** - Move code freely without breaking hidden connections
- **Free caching** - Memoization is always safe for pure functions

---

## 3. Core Concepts

### The Two Rules of Pure Functions

```plaintext
┌─────────────────────────────────────────────────────────────────┐
│                  RULE 1: DETERMINISTIC                           │
│                                                                  │
│   Same inputs ALWAYS produce the same output.                   │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  add(2, 3) → 5     (first call)                         │   │
│   │  add(2, 3) → 5     (second call)                        │   │
│   │  add(2, 3) → 5     (millionth call)                     │   │
│   │  add(2, 3) → 5     (in a different thread)              │   │
│   │  add(2, 3) → 5     (next year)                          │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│   The function depends ONLY on its explicit parameters.         │
│   No hidden inputs: no globals, no "this", no Date.now().       │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                  RULE 2: NO SIDE EFFECTS                         │
│                                                                  │
│   The function doesn't change anything outside itself.          │
│                                                                  │
│   ❌ NO modifying input arguments                               │
│   ❌ NO modifying global/shared variables                       │
│   ❌ NO writing to console, files, or network                   │
│   ❌ NO reading from external sources (DB, API, DOM)            │
│   ❌ NO calling other impure functions                          │
│   ❌ NO generating random numbers                                │
│   ❌ NO getting current date/time                                │
│                                                                  │
│   The ONLY observable effect is the return value.               │
└─────────────────────────────────────────────────────────────────┘
```

### Pure vs Impure Examples

```plaintext
┌─────────────────────────────────────────────────────────────────┐
│                    PURE FUNCTIONS                                │
│                                                                  │
│   ✅ add(a, b) → a + b                                          │
│   ✅ uppercase(str) → str.toUpperCase()                         │
│   ✅ filter(arr, fn) → arr.filter(fn)                           │
│   ✅ calculateTax(amount, rate) → amount * rate                 │
│   ✅ formatDate(date) → date.toISOString()                      │
│                                                                  │
│   Characteristics:                                              │
│   • Output depends only on inputs                               │
│   • Can be called any number of times safely                    │
│   • Can be cached/memoized                                      │
│   • Can run in parallel                                         │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                   IMPURE FUNCTIONS                               │
│                                                                  │
│   ❌ Math.random() → different each call                        │
│   ❌ Date.now() → different each call                           │
│   ❌ console.log() → writes to external system                  │
│   ❌ fetch() → reads from external system                       │
│   ❌ array.push() → mutates the array                           │
│   ❌ localStorage.setItem() → modifies external state           │
│   ❌ document.getElementById() → reads from DOM                 │
│                                                                  │
│   Characteristics:                                              │
│   • Unpredictable or externally dependent                       │
│   • May produce different results each call                     │
│   • Unsafe to cache                                             │
│   • May cause race conditions in parallel                       │
└─────────────────────────────────────────────────────────────────┘
```

### The Functional Core / Imperative Shell Pattern

```plaintext
┌─────────────────────────────────────────────────────────────────┐
│           FUNCTIONAL CORE / IMPERATIVE SHELL                    │
│                                                                 │
│   Structure your application with:                              │
│   • A PURE CORE of business logic (no side effects)             │
│   • An IMPURE SHELL at the boundaries (handles I/O)             │
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                   IMPURE SHELL                          │   │
│   │     (handles I/O, network, database, user input)        │   │
│   │                                                         │   │
│   │   ┌─────────────────────────────────────────────────┐   │   │
│   │   │                  PURE CORE                      │   │   │
│   │   │                                                 │   │   │
│   │   │    Business logic, calculations, transforms     │   │   │
│   │   │    Validations, formatting, filtering           │   │   │
│   │   │                                                 │   │   │
│   │   │    ✅ Easy to test                              │   │   │
│   │   │    ✅ Easy to reason about                      │   │   │
│   │   │    ✅ Safe to refactor                          │   │   │
│   │   └─────────────────────────────────────────────────┘   │   │
│   │                                                         │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│   The shell calls pure functions, handles their results,        │
│   and performs necessary side effects.                          │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms

| Term | Definition |
| --- | --- |
| Pure Function | A function with no side effects that always returns the same output for the same input |
| Side Effect | Any observable change outside the function's scope |
| Deterministic | Always produces the same output for the same input |
| Referential Transparency | An expression can be replaced with its value without changing behavior |
| Memoization | Caching function results based on inputs (only safe for pure functions) |

---

## 4. Workflow & Checklist

### Recommended Workflow

```plaintext
1. IDENTIFY        →   2. EXTRACT         →   3. INJECT         →   4. PUSH TO
   Pure Logic           Dependencies          As Parameters         Boundaries

   "What is pure       "What external        "Pass external        "Handle I/O at
    calculation?"       data is needed?"      data as args"         app edges"
```

### Checklist

Before writing a function, verify:

- [ ] **INPUTS ONLY:** Does the function depend ONLY on its explicit parameters?
- [ ] **NO MUTATION:** Does it avoid modifying its input arguments?
- [ ] **NO GLOBALS:** Does it avoid reading or writing global/shared state?
- [ ] **NO I/O:** Does it avoid network calls, file operations, or console output?
- [ ] **DETERMINISTIC:** Will it always return the same result for the same inputs?

---

## 5. Examples

### Example 1: Array Operations

❌ **Bad:**

```typescript
// Impure: Mutates the input array
function addItem(items: string[], item: string): string[] {
  items.push(item); // MUTATION!
  return items;
}

const original = ['a', 'b'];
const modified = addItem(original, 'c');

console.log(original); // ['a', 'b', 'c'] - Original was DESTROYED!
console.log(modified); // ['a', 'b', 'c']
console.log(original === modified); // true - Same reference!

// Problems:
// - Caller's data was unexpectedly modified
// - Bugs are hard to track - "who changed my array?"
// - Can't safely pass arrays to functions
```

✅ **Good:**

```typescript
// Pure: Returns a NEW array, input unchanged
function addItem(items: string[], item: string): string[] {
  return [...items, item];
}

const original = ['a', 'b'];
const modified = addItem(original, 'c');

console.log(original); // ['a', 'b'] - Original preserved!
console.log(modified); // ['a', 'b', 'c'] - New array
console.log(original === modified); // false - Different references

// More pure array operations:
const removeItem = (items: string[], index: number): string[] => [
  ...items.slice(0, index),
  ...items.slice(index + 1),
];

const updateItem = (items: string[], index: number, newValue: string): string[] =>
  items.map((item, i) => (i === index ? newValue : item));

// Benefits:
// - Original data is never corrupted
// - Functions are predictable and safe
// - Easy to test: just check input → output
```

### Example 2: Date/Time Handling

❌ **Bad:**

```typescript
// Impure: Depends on current time (hidden input)
function getGreeting(): string {
  const hour = new Date().getHours(); // Hidden dependency!
  if (hour < 12) return 'Good morning';
  if (hour < 17) return 'Good afternoon';
  return 'Good evening';
}

// Problems:
// - Different result depending on when you call it
// - Can't test reliably (need to mock Date)
// - Can't predict the output
```

✅ **Good:**

```typescript
// Pure: All dependencies are explicit parameters
function getGreeting(hour: number): string {
  if (hour < 12) return 'Good morning';
  if (hour < 17) return 'Good afternoon';
  return 'Good evening';
}

// Testing is trivial:
expect(getGreeting(9)).toBe('Good morning');
expect(getGreeting(14)).toBe('Good afternoon');
expect(getGreeting(20)).toBe('Good evening');

// Impure code is pushed to the boundary:
function GreetingComponent() {
  // Impure: reading current time happens at the edge
  const currentHour = new Date().getHours();

  // Pure: business logic is isolated
  const greeting = getGreeting(currentHour);

  return <h1>{greeting}</h1>;
}
```

### Example 3: Object Transformation

❌ **Bad:**

```typescript
// Impure: Mutates the input object
function updateUserAge(user: User, newAge: number): User {
  user.age = newAge; // MUTATION!
  user.updatedAt = new Date(); // Side effect: depends on current time!
  return user;
}

const user = { name: 'Alice', age: 25, updatedAt: null };
const updated = updateUserAge(user, 26);

console.log(user.age); // 26 - Original was modified!
console.log(user === updated); // true - Same object
```

✅ **Good:**

```typescript
// Pure: Returns a new object, time is passed in
function updateUserAge(user: User, newAge: number, now: Date): User {
  return {
    ...user,
    age: newAge,
    updatedAt: now,
  };
}

const user = { name: 'Alice', age: 25, updatedAt: null };
const updated = updateUserAge(user, 26, new Date());

console.log(user.age); // 25 - Original preserved!
console.log(user === updated); // false - New object

// Even better: Separate concerns
function updateUserAge(user: User, newAge: number): User {
  return { ...user, age: newAge };
}

function withTimestamp<T>(obj: T, now: Date): T & { updatedAt: Date } {
  return { ...obj, updatedAt: now };
}

// Compose at the boundary:
const updated = withTimestamp(updateUserAge(user, 26), new Date());
```

### Example 4: Calculations with External Dependencies

❌ **Bad:**

```typescript
// Impure: Reads from global config and external service
const TAX_RATE = 0.08; // Global variable

async function calculateOrderTotal(orderId: string): Promise<number> {
  // Side effect: Network call
  const order = await fetch(`/api/orders/${orderId}`).then((r) => r.json());

  // Side effect: Reading global variable
  const tax = order.subtotal * TAX_RATE;

  // Side effect: Logging
  console.log(`Calculated total for order ${orderId}`);

  return order.subtotal + tax + order.shipping;
}

// Problems:
// - Can't test without network mocking
// - Depends on global TAX_RATE (hidden input)
// - Console.log is a side effect
// - Can't reuse the calculation logic
```

✅ **Good:**

```typescript
// Pure: All dependencies passed as parameters
type Order = {
  subtotal: number;
  shipping: number;
};

type TaxConfig = {
  rate: number;
};

function calculateOrderTotal(order: Order, taxConfig: TaxConfig): number {
  const tax = order.subtotal * taxConfig.rate;
  return order.subtotal + tax + order.shipping;
}

// Testing is trivial:
expect(
  calculateOrderTotal({ subtotal: 100, shipping: 10 }, { rate: 0.08 })
).toBe(118);

// Impure operations happen at the boundary:
async function loadAndCalculateOrderTotal(orderId: string): Promise<number> {
  // Impure: Network call at the boundary
  const order = await fetch(`/api/orders/${orderId}`).then((r) => r.json());
  const taxConfig = await fetch('/api/config/tax').then((r) => r.json());

  // Pure: Calculation is isolated
  const total = calculateOrderTotal(order, taxConfig);

  // Impure: Logging at the boundary
  logger.info(`Calculated total for order ${orderId}: ${total}`);

  return total;
}
```

### Example 5: React Component Logic

❌ **Bad:**

```tsx
// Impure component with side effects mixed into render logic
function ProductCard({ productId }: { productId: string }) {
  const [product, setProduct] = useState<Product | null>(null);

  useEffect(() => {
    // Side effect in component
    fetch(`/api/products/${productId}`)
      .then((r) => r.json())
      .then(setProduct);
  }, [productId]);

  if (!product) return <div>Loading...</div>;

  // Business logic mixed with presentation
  const discountedPrice = product.price * (1 - product.discountPercent / 100);
  const savings = product.price - discountedPrice;
  const isOnSale = product.discountPercent > 0;
  const formattedPrice = `$${discountedPrice.toFixed(2)}`;
  const formattedSavings = `$${savings.toFixed(2)}`;

  return (
    <div>
      <h2>{product.name}</h2>
      <p>{formattedPrice}</p>
      {isOnSale && <span>Save {formattedSavings}!</span>}
    </div>
  );
}

// Problems:
// - Can't test pricing logic without rendering component
// - Can't reuse pricing calculations elsewhere
// - Hard to reason about what affects the display
```

✅ **Good:**

```tsx
// Pure functions for all business logic
function calculateDiscountedPrice(price: number, discountPercent: number): number {
  return price * (1 - discountPercent / 100);
}

function calculateSavings(originalPrice: number, discountedPrice: number): number {
  return originalPrice - discountedPrice;
}

function isOnSale(discountPercent: number): boolean {
  return discountPercent > 0;
}

function formatCurrency(amount: number): string {
  return `$${amount.toFixed(2)}`;
}

// Pure derived data calculation
function getProductDisplayData(product: Product) {
  const discountedPrice = calculateDiscountedPrice(product.price, product.discountPercent);
  const savings = calculateSavings(product.price, discountedPrice);

  return {
    name: product.name,
    formattedPrice: formatCurrency(discountedPrice),
    formattedSavings: formatCurrency(savings),
    showSalesBadge: isOnSale(product.discountPercent),
  };
}

// Component is just presentation
function ProductCard({ productId }: { productId: string }) {
  const { data: product, isLoading } = useProduct(productId); // Hook handles side effect

  if (isLoading || !product) return <div>Loading...</div>;

  // Pure transformation
  const display = getProductDisplayData(product);

  return (
    <div>
      <h2>{display.name}</h2>
      <p>{display.formattedPrice}</p>
      {display.showSalesBadge && <span>Save {display.formattedSavings}!</span>}
    </div>
  );
}

// Now all business logic is easily testable:
test('calculateDiscountedPrice', () => {
  expect(calculateDiscountedPrice(100, 20)).toBe(80);
  expect(calculateDiscountedPrice(50, 10)).toBe(45);
});

test('getProductDisplayData', () => {
  const product = { name: 'Widget', price: 100, discountPercent: 20 };
  const display = getProductDisplayData(product);

  expect(display.formattedPrice).toBe('$80.00');
  expect(display.formattedSavings).toBe('$20.00');
  expect(display.showSalesBadge).toBe(true);
});
```

---

## 6. References

- [Pure Function - Wikipedia](https://en.wikipedia.org/wiki/Pure_function)
- Gary Bernhardt - ["Functional Core, Imperative Shell"](https://www.destroyallsoftware.com/screencasts/catalog/functional-core-imperative-shell)
- ["Professor Frisby's Mostly Adequate Guide to Functional Programming"](https://mostly-adequate.gitbook.io/mostly-adequate-guide/)
- Eric Elliott - ["Master the JavaScript Interview: What is a Pure Function?"](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-pure-function-d1c076bec976)
