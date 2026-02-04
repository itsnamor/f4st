# Referential Transparency

---

## 1. Overview

**Referential Transparency** is a property of expressions where an expression can be replaced with its corresponding value without changing the program's behavior. If `f(x)` always returns `5`, then anywhere you see `f(x)` you can substitute `5` and the program will work identically.

**Key Idea:** An expression is referentially transparent if you can replace it with its value everywhere it appears, and nothing changes.

---

## 2. Why It Matters

### The Problem

When expressions are NOT referentially transparent:

- Cannot reason about code by substituting equals for equals
- Order of evaluation matters and can change results
- Caching/memoization is unsafe (might return stale or wrong data)
- Refactoring (extracting/inlining expressions) can break code
- Parallel execution may produce different results
- Testing becomes harder (hidden dependencies)

### The Solution

Following Referential Transparency provides:

- **Equational reasoning** - Substitute and simplify to understand code
- **Order independence** - Evaluation order doesn't affect correctness
- **Safe caching** - Memoize any expression without worry
- **Safe refactoring** - Extract or inline expressions freely
- **Safe parallelization** - Run expressions concurrently without race conditions
- **Simple testing** - No hidden state to set up or mock

---

## 3. Core Concepts

### The Substitution Principle

```plaintext
┌─────────────────────────────────────────────────────────────────┐
│                   SUBSTITUTION PRINCIPLE                         │
│                                                                  │
│   If an expression is REFERENTIALLY TRANSPARENT:                │
│   You can replace it with its value without changing behavior.  │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  Given: add(2, 3) = 5 (always, every time)              │   │
│   │                                                          │   │
│   │  Then these are EQUIVALENT:                              │   │
│   │                                                          │   │
│   │    const x = add(2, 3);           const x = 5;           │   │
│   │    const y = add(2, 3);     =     const y = 5;           │   │
│   │    const z = x + y;               const z = x + y;       │   │
│   │                                                          │   │
│   │  And this optimization is SAFE:                          │   │
│   │                                                          │   │
│   │    const result = add(2, 3) + add(2, 3);                 │   │
│   │    // Can be optimized to:                               │   │
│   │    const temp = add(2, 3);  // compute once              │   │
│   │    const result = temp + temp;                           │   │
│   └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### Transparent vs Opaque

```plaintext
┌─────────────────────────────────────────────────────────────────┐
│            REFERENTIALLY TRANSPARENT (SAFE)                      │
│                                                                  │
│   ✅ Math.max(2, 3)         → always 3                          │
│   ✅ "hello".toUpperCase()  → always "HELLO"                    │
│   ✅ [1, 2, 3].length       → always 3                          │
│   ✅ add(2, 3)              → always 5 (if add is pure)         │
│   ✅ user.name              → same for same user object         │
│                                                                  │
│   These can be replaced with their values anywhere.             │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│            REFERENTIALLY OPAQUE (UNSAFE)                         │
│                                                                  │
│   ❌ Date.now()             → different each millisecond        │
│   ❌ Math.random()          → different each call               │
│   ❌ fetchUser(id)          → depends on server state           │
│   ❌ readFile(path)         → file might change                 │
│   ❌ counter++              → modifies external state           │
│   ❌ console.log(x)         → side effect (writes output)       │
│   ❌ array.pop()            → modifies the array                │
│                                                                  │
│   These CANNOT be replaced with their values safely.            │
│   The result depends on WHEN or HOW MANY TIMES you call them.   │
└─────────────────────────────────────────────────────────────────┘
```

### Why Substitution Fails for Opaque Expressions

```plaintext
┌─────────────────────────────────────────────────────────────────┐
│         WHY OPAQUE EXPRESSIONS BREAK SUBSTITUTION                │
│                                                                  │
│   Example: getNextId() returns incrementing IDs                 │
│                                                                  │
│   let counter = 0;                                              │
│   const getNextId = () => ++counter;                            │
│                                                                  │
│   // This code:                                                 │
│   const a = getNextId();  // a = 1                              │
│   const b = getNextId();  // b = 2                              │
│                                                                  │
│   // Is NOT equivalent to:                                      │
│   const a = 1;            // a = 1                              │
│   const b = 1;            // b = 1  ← WRONG!                    │
│                                                                  │
│   // Nor is it equivalent to:                                   │
│   const temp = getNextId();  // temp = 1                        │
│   const a = temp;            // a = 1                           │
│   const b = temp;            // b = 1  ← WRONG!                 │
│                                                                  │
│   Substitution BREAKS because getNextId() is opaque.            │
│   Each call returns something DIFFERENT.                        │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms

| Term | Definition |
| --- | --- |
| Referentially Transparent | Expression that can be replaced by its value without changing behavior |
| Referentially Opaque | Expression whose value depends on when/how often it's called |
| Equational Reasoning | Proving program properties by substituting equals for equals |
| Memoization | Caching function results (only safe for transparent expressions) |

---

## 4. Workflow & Checklist

### Recommended Workflow

```plaintext
1. IDENTIFY        →   2. MAKE            →   3. PUSH OPAQUE    →   4. VERIFY
   Dependencies         Transparent            To Boundaries          Substitution

   "What external       "Pass all deps        "Keep I/O at          "Can I replace
    state does this      as parameters"        the edges"            this with its
    depend on?"                                                       value?"
```

### Checklist

Before writing an expression, verify:

- [ ] **SAME RESULT:** Will this expression return the same value every time for the same inputs?
- [ ] **NO HIDDEN STATE:** Does it depend only on its explicit inputs, not hidden state?
- [ ] **NO SIDE EFFECTS:** Does it avoid modifying anything outside its scope?
- [ ] **SUBSTITUTABLE:** Could I replace this call with its result everywhere?
- [ ] **MEMOIZABLE:** Would it be safe to cache this expression's result?

---

## 5. Examples

### Example 1: Transparent vs Opaque Functions

❌ **Bad:**

```typescript
// Referentially OPAQUE - different result each call
let counter = 0;

function getNextId(): number {
  counter += 1;  // Side effect: modifies external state
  return counter;
}

// These are NOT equivalent:
const a = getNextId();  // 1
const b = getNextId();  // 2

// vs
const a = 1;
const b = 1;  // Wrong! They should be different

// Problems:
// - Can't substitute getNextId() with its result
// - Can't memoize
// - Can't parallelize
// - Testing requires resetting global counter
```

✅ **Good:**

```typescript
// Referentially TRANSPARENT - same input, same output
function createId(prefix: string, number: number): string {
  return `${prefix}-${number}`;
}

// These ARE equivalent:
const a = createId('user', 42);  // 'user-42'
const b = createId('user', 42);  // 'user-42'

// vs
const a = 'user-42';
const b = 'user-42';  // Correct! Same thing

// Safe to optimize:
const temp = createId('user', 42);
const a = temp;
const b = temp;  // Still correct!

// Benefits:
// - Can substitute freely
// - Can memoize safely
// - Can run in parallel
// - Testing is trivial
```

### Example 2: Making Opaque Code Transparent

❌ **Bad:**

```typescript
// Opaque: depends on current time (hidden dependency)
function formatTimestamp(): string {
  const now = new Date();  // Hidden dependency!
  return now.toISOString();
}

// Problems:
// - formatTimestamp() !== formatTimestamp() (different each call)
// - Can't test reliably without mocking Date
// - Can't substitute with a value

function isExpired(): boolean {
  const now = Date.now();  // Hidden dependency!
  const expiryTime = getExpiryTime();  // Another hidden dependency!
  return now > expiryTime;
}
```

✅ **Good:**

```typescript
// Transparent: all dependencies are explicit parameters
function formatTimestamp(date: Date): string {
  return date.toISOString();
}

// Now it's referentially transparent!
// formatTimestamp(someDate) === formatTimestamp(someDate) (always)

function isExpired(currentTime: number, expiryTime: number): boolean {
  return currentTime > expiryTime;
}

// Testing is trivial:
expect(isExpired(1000, 500)).toBe(true);
expect(isExpired(500, 1000)).toBe(false);

// Push opaque operations to the boundary:
function ExpiredBanner({ expiryTime }: { expiryTime: number }) {
  // Opaque: reading current time
  const currentTime = Date.now();

  // Transparent: pure comparison
  const expired = isExpired(currentTime, expiryTime);

  return expired ? <Banner>Expired!</Banner> : null;
}
```

### Example 3: Making Random Operations Transparent

❌ **Bad:**

```typescript
// Opaque: depends on Math.random() (unpredictable)
function shuffleArray<T>(array: T[]): T[] {
  const result = [...array];
  for (let i = result.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));  // Opaque!
    [result[i], result[j]] = [result[j], result[i]];
  }
  return result;
}

// shuffleArray([1, 2, 3]) !== shuffleArray([1, 2, 3])
// Can't test reliably
// Can't substitute
```

✅ **Good:**

```typescript
// Transparent: randomness source is explicit parameter
type RandomFn = () => number;

function shuffleArray<T>(array: T[], random: RandomFn): T[] {
  const result = [...array];
  for (let i = result.length - 1; i > 0; i--) {
    const j = Math.floor(random() * (i + 1));  // Explicit dependency
    [result[i], result[j]] = [result[j], result[i]];
  }
  return result;
}

// For testing: use a seeded/predictable random
function createSeededRandom(seed: number): RandomFn {
  return () => {
    seed = (seed * 1103515245 + 12345) & 0x7fffffff;
    return seed / 0x7fffffff;
  };
}

// Now testing is deterministic!
const seeded = createSeededRandom(42);
expect(shuffleArray([1, 2, 3, 4], seeded)).toEqual([3, 1, 4, 2]);  // Always!

// For production: pass Math.random
const shuffled = shuffleArray(items, Math.random);
```

### Example 4: Memoization Safety

❌ **Bad:**

```typescript
// UNSAFE to memoize - referentially opaque
function getCurrentUserPermissions(): string[] {
  const user = getCurrentUser();  // Changes based on auth state
  return user.permissions;
}

// Memoizing would return stale data:
const memoized = memoize(getCurrentUserPermissions);
memoized();  // ['read'] (for user A)
// User B logs in...
memoized();  // Still ['read']! Wrong - should be ['read', 'write']
```

✅ **Good:**

```typescript
// SAFE to memoize - referentially transparent
function getUserPermissions(user: User): string[] {
  return user.permissions;
}

// Memoizing by user is safe:
const memoized = memoize(getUserPermissions);
memoized(userA);  // ['read']
memoized(userB);  // ['read', 'write']
memoized(userA);  // ['read'] - cache hit, correct!

// For computed values, memoization is powerful:
const computeExpensiveStats = memoize((data: DataSet): Stats => {
  // Complex calculations...
  return stats;
});

// Same data = same stats (from cache)
computeExpensiveStats(data);  // Computed
computeExpensiveStats(data);  // Cached!
```

### Example 5: React and Referential Transparency

❌ **Bad:**

```tsx
// Component with opaque expressions
function UserGreeting() {
  // Opaque: different each render
  const timestamp = Date.now();
  const randomColor = `hsl(${Math.random() * 360}, 70%, 50%)`;

  // This component will flicker/change unexpectedly
  return (
    <div style={{ color: randomColor }}>
      Rendered at {timestamp}
    </div>
  );
}

// useMemo won't help - dependencies are always "new"
function ExpensiveComponent({ user }: { user: User }) {
  // Opaque function - unsafe to memoize
  const getGreeting = () => {
    if (Date.now() % 2 === 0) return 'Hello';  // Opaque!
    return 'Hi';
  };

  // useMemo can't optimize this
  const greeting = useMemo(() => getGreeting(), []);  // Unstable!
}
```

✅ **Good:**

```tsx
// Component with transparent expressions
function UserGreeting({ user, currentTime }: { user: User; currentTime: Date }) {
  // Transparent: same props = same output
  const greeting = getGreeting(user);  // Pure function
  const formattedTime = formatTime(currentTime);  // Pure function

  return (
    <div>
      {greeting}, {user.name}! It's {formattedTime}
    </div>
  );
}

// Pure utility functions
function getGreeting(user: User): string {
  if (user.prefersFormal) return 'Good day';
  return 'Hey';
}

function formatTime(date: Date): string {
  return date.toLocaleTimeString();
}

// useMemo works correctly with transparent expressions
function ExpensiveComponent({ data }: { data: DataSet }) {
  // Transparent: depends only on data
  const stats = useMemo(() => computeStats(data), [data]);

  // React.memo works correctly
  return <StatsDisplay stats={stats} />;
}

// computeStats is pure - same data = same stats
function computeStats(data: DataSet): Stats {
  return {
    count: data.items.length,
    average: data.items.reduce((a, b) => a + b, 0) / data.items.length,
    // ... more pure calculations
  };
}
```

---

## 6. References

- [Referential Transparency - Wikipedia](https://en.wikipedia.org/wiki/Referential_transparency)
- ["Functional Programming in Scala"](https://www.manning.com/books/functional-programming-in-scala) - Chiusano & Bjarnason
- ["Haskell Programming from First Principles"](https://haskellbook.com/) - Allen & Moronuki
- ["Structure and Interpretation of Computer Programs"](https://mitpress.mit.edu/sites/default/files/sicp/index.html) - Abelson & Sussman
