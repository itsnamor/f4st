# YAGNI (You Aren't Gonna Need It)

---

## 1. Overview

**YAGNI** is a principle from Extreme Programming (XP) that states you should not add functionality until it is actually needed. It fights against the natural developer tendency to build for imagined future requirements, which often turn out to be wrong or never materialize.

**Key Idea:** Build only what you need TODAY. Tomorrow's requirements will be clearer tomorrow.

---

## 2. Why It Matters

### The Problem

When developers build for imagined future requirements:

- Wasted development time on features that are never used
- Increased codebase complexity that slows everyone down
- More surface area for bugs in unused code paths
- Wrong abstractions built on incorrect assumptions about the future
- Harder onboarding - new developers must understand unused code
- Delayed delivery of features that ARE needed now

### The Solution

Following YAGNI provides:

- **Focused development** - Energy spent on real, validated needs
- **Leaner codebase** - Less code means fewer bugs and easier maintenance
- **Faster delivery** - Ship what's needed now, iterate based on feedback
- **Correct abstractions** - Built when you have real use cases to guide design
- **Lower cognitive load** - No need to understand "just in case" code

---

## 3. Core Concepts

### The Cost of Unused Code

```plaintext
┌─────────────────────────────────────────────────────────────────┐
│                  UNUSED CODE IS NOT FREE                         │
│                                                                  │
│   Every line of code has ongoing costs:                         │
│                                                                  │
│   📖 READING COST                                               │
│      Developers must read and understand it                     │
│                                                                  │
│   🔧 MAINTENANCE COST                                           │
│      Must be updated when dependencies change                   │
│                                                                  │
│   🧪 TESTING COST                                               │
│      Should be tested, even if unused                           │
│                                                                  │
│   🔄 REFACTORING COST                                           │
│      Must be considered during any refactoring                  │
│                                                                  │
│   🐛 BUG RISK                                                   │
│      More code = more potential bugs                            │
│                                                                  │
│   ⏱️ BUILD TIME                                                 │
│      Adds to compilation and bundle size                        │
│                                                                  │
│   "The best code is no code at all." — Jeff Atwood              │
└─────────────────────────────────────────────────────────────────┘
```

### Speculative Generality

```plaintext
┌─────────────────────────────────────────────────────────────────┐
│                   SPECULATIVE GENERALITY                         │
│                                                                  │
│   Common phrases that signal YAGNI violations:                  │
│                                                                  │
│   ❌ "We might need this later..."                              │
│   ❌ "What if someday we want to..."                            │
│   ❌ "Let's make it configurable just in case..."               │
│   ❌ "It's easy to add now, harder later..."                    │
│   ❌ "Other projects might want to use this..."                 │
│   ❌ "Let's build a plugin system for flexibility..."           │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                    THE REALITY                           │   │
│   │                                                          │   │
│   │   • 80% of "future features" never get built            │   │
│   │   • Requirements change - your guess is likely wrong    │   │
│   │   • When you DO need it, you'll understand it better    │   │
│   │   • Adding later is usually NOT harder than adding now  │   │
│   └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### When YAGNI Applies (and When It Doesn't)

```plaintext
┌─────────────────────────────────────────────────────────────────┐
│                    YAGNI APPLIES TO                              │
│                                                                  │
│   ✅ Features and functionality                                 │
│   ✅ Configuration options                                      │
│   ✅ Abstraction layers                                         │
│   ✅ Plugin/extension systems                                   │
│   ✅ "Flexibility" for unknown future needs                     │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                  YAGNI DOES NOT APPLY TO                         │
│                                                                  │
│   ❌ Security measures (always needed)                          │
│   ❌ Error handling (always needed)                             │
│   ❌ Accessibility (always needed)                              │
│   ❌ Code quality and readability                               │
│   ❌ Known architectural constraints (scaling, etc.)            │
│   ❌ Legal/compliance requirements                              │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms

| Term | Definition |
| --- | --- |
| YAGNI | "You Aren't Gonna Need It" - don't build until needed |
| Speculative Generality | Adding flexibility for imagined future needs |
| Over-engineering | Building more complexity than required |
| Gold Plating | Adding extra features beyond requirements |
| MVP | Minimum Viable Product - build only what's essential |

---

## 4. Workflow & Checklist

### Recommended Workflow

```plaintext
1. REQUIREMENT     →   2. MINIMUM         →   3. DELIVER        →   4. ITERATE
   Analysis             Implementation         & Validate            Based on Feedback

   "What is           "What's the            "Does it solve       "What do users
    actually           smallest thing         the problem?"         ACTUALLY need
    needed NOW?"       that works?"                                  next?"
```

### Checklist

Before writing code, verify:

- [ ] **EXPLICIT:** Is this feature explicitly required in current requirements?
- [ ] **REAL:** Am I solving a REAL problem or an IMAGINED one?
- [ ] **MINIMAL:** Am I building the minimum needed, not the maximum possible?
- [ ] **ASKED:** Did someone ask for this, or am I assuming they'll want it?
- [ ] **USED:** If I delete this code, would any CURRENT functionality break?

---

## 5. Examples

### Example 1: Over-Configured Component

❌ **Bad:**

```typescript
// Building for every possible future scenario
interface ButtonProps {
  children: React.ReactNode;
  onClick?: () => void;

  // Variants for every possible use case
  variant?: 'primary' | 'secondary' | 'tertiary' | 'ghost' | 'link' | 'danger' | 'warning' | 'success' | 'info';
  size?: 'xs' | 'sm' | 'md' | 'lg' | 'xl' | 'xxl';

  // Icon configuration "just in case"
  icon?: React.ComponentType;
  iconPosition?: 'left' | 'right' | 'top' | 'bottom' | 'only';
  iconSize?: number;
  iconColor?: string;

  // Loading states "we might need"
  loading?: boolean;
  loadingText?: string;
  loadingPosition?: 'left' | 'right' | 'center' | 'replace';
  loadingSpinner?: React.ComponentType;

  // Animation options "for flexibility"
  animateOnHover?: boolean;
  animateOnClick?: boolean;
  animationDuration?: number;
  animationType?: 'scale' | 'pulse' | 'shake' | 'bounce';

  // Tooltip "someone might want"
  tooltip?: string;
  tooltipPosition?: 'top' | 'bottom' | 'left' | 'right';
  tooltipDelay?: number;

  // ... and 30 more props
}

// Result: 500+ lines of component code, most of it unused
// Result: Every change requires understanding all this complexity
// Result: Bugs hide in untested code paths
```

✅ **Good:**

```typescript
// Build only what's actually used in the app today
interface ButtonProps {
  children: React.ReactNode;
  onClick?: () => void;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
  className?: string; // Escape hatch for edge cases
}

function Button({
  children,
  onClick,
  variant = 'primary',
  disabled,
  className,
}: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={cn(
        'px-4 py-2 rounded-md font-medium transition-colors',
        variant === 'primary' && 'bg-blue-600 text-white hover:bg-blue-700',
        variant === 'secondary' && 'bg-gray-200 text-gray-800 hover:bg-gray-300',
        disabled && 'opacity-50 cursor-not-allowed',
        className
      )}
    >
      {children}
    </button>
  );
}

// When you ACTUALLY need icons:
// Add iconLeft?: React.ReactNode prop (not a whole icon system)

// When you ACTUALLY need loading:
// Add loading?: boolean prop (with sensible default behavior)

// When you ACTUALLY need more variants:
// Add them one at a time as real requirements emerge
```

### Example 2: Premature Plugin Architecture

❌ **Bad:**

```typescript
// Building a plugin system for ONE notification type
interface NotificationPlugin {
  name: string;
  version: string;
  initialize(): Promise<void>;
  send(message: NotificationMessage): Promise<NotificationResult>;
  dispose(): void;
}

interface NotificationPluginConfig {
  enabled: boolean;
  priority: number;
  retryPolicy: RetryPolicy;
  fallbackPlugin?: string;
}

class NotificationPluginManager {
  private plugins: Map<string, NotificationPlugin> = new Map();
  private configs: Map<string, NotificationPluginConfig> = new Map();
  private hooks: NotificationHooks = {};

  register(plugin: NotificationPlugin, config?: NotificationPluginConfig): void {
    // 50 lines of registration logic
  }

  unregister(pluginName: string): void {
    // 30 lines of cleanup logic
  }

  async send(message: NotificationMessage): Promise<NotificationResult[]> {
    // 100 lines of plugin orchestration, fallbacks, retries
  }

  // ... 500 more lines of plugin infrastructure
}

// REALITY: We only send console.log notifications
// REALITY: We've spent 3 days building unused infrastructure
```

✅ **Good:**

```typescript
// Start with the simplest thing that works
function sendNotification(message: string): void {
  console.log(`[Notification] ${message}`);
}

// That's it. 3 lines.

// LATER, when you ACTUALLY need email notifications:
async function sendNotification(message: string, options?: { email?: string }): Promise<void> {
  console.log(`[Notification] ${message}`);

  if (options?.email) {
    await sendEmail(options.email, message);
  }
}

// LATER, when you ACTUALLY need multiple channels and they have different behaviors:
// THEN consider if a plugin system makes sense
// By then, you'll have 3+ real implementations to guide the abstraction
```

### Example 3: Unnecessary Database Flexibility

❌ **Bad:**

```typescript
// Building a "database-agnostic" repository layer "for flexibility"
interface DatabaseAdapter {
  connect(): Promise<void>;
  disconnect(): Promise<void>;
  query<T>(sql: string, params: unknown[]): Promise<T[]>;
  execute(sql: string, params: unknown[]): Promise<ExecuteResult>;
  transaction<T>(fn: (tx: Transaction) => Promise<T>): Promise<T>;
}

class PostgresAdapter implements DatabaseAdapter { /* 200 lines */ }
class MySQLAdapter implements DatabaseAdapter { /* 200 lines */ }
class SQLiteAdapter implements DatabaseAdapter { /* 200 lines */ }
class MongoAdapter implements DatabaseAdapter { /* 200 lines - doesn't even fit well */ }

interface RepositoryConfig {
  adapter: 'postgres' | 'mysql' | 'sqlite' | 'mongo';
  connectionString: string;
  poolSize: number;
  // ... 20 more config options
}

// REALITY: We use PostgreSQL. We've always used PostgreSQL.
// REALITY: We have zero plans to switch databases.
// REALITY: 800 lines of unused adapter code to maintain.
```

✅ **Good:**

```typescript
// Use PostgreSQL directly - it's what we use
import { Pool } from 'pg';

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
});

export async function query<T>(sql: string, params: unknown[] = []): Promise<T[]> {
  const result = await pool.query(sql, params);
  return result.rows;
}

export async function getUser(id: string): Promise<User | null> {
  const rows = await query<User>('SELECT * FROM users WHERE id = $1', [id]);
  return rows[0] ?? null;
}

// Simple, direct, works.
// IF we ever need to switch databases (unlikely), we'll migrate then.
// The migration effort is the same whether we have abstractions or not.
```

### Example 4: Feature Flags Before Features

❌ **Bad:**

```typescript
// Building feature flag infrastructure before any features
interface FeatureFlag {
  name: string;
  enabled: boolean;
  rolloutPercentage: number;
  targetUsers: string[];
  targetGroups: string[];
  startDate?: Date;
  endDate?: Date;
  variants: FeatureVariant[];
  metadata: Record<string, unknown>;
}

class FeatureFlagService {
  private flags: Map<string, FeatureFlag> = new Map();
  private cache: FeatureFlagCache;
  private analytics: FeatureFlagAnalytics;

  async isEnabled(flagName: string, userId: string): Promise<boolean> {
    // 100 lines of flag evaluation logic
    // Handles rollout percentages, user targeting, date ranges, etc.
  }

  async getVariant(flagName: string, userId: string): Promise<string> {
    // A/B testing logic
  }

  // ... 500 more lines
}

// REALITY: We have zero feature flags defined
// REALITY: We don't know what features will need flags
// REALITY: When we DO need flags, LaunchDarkly exists
```

✅ **Good:**

```typescript
// No feature flags until we need them

// LATER, when we have ONE feature that needs a flag:
const ENABLE_NEW_CHECKOUT = process.env.ENABLE_NEW_CHECKOUT === 'true';

function CheckoutPage() {
  if (ENABLE_NEW_CHECKOUT) {
    return <NewCheckout />;
  }
  return <LegacyCheckout />;
}

// LATER, when we need user-specific flags:
// Consider a service like LaunchDarkly, or build minimal infrastructure
// By then, you'll know exactly what you need
```

### Example 5: Premature Optimization

❌ **Bad:**

```typescript
// Building caching infrastructure "for performance"
interface CacheConfig {
  strategy: 'lru' | 'lfu' | 'fifo' | 'ttl';
  maxSize: number;
  ttlMs: number;
  onEvict?: (key: string, value: unknown) => void;
  serialize?: (value: unknown) => string;
  deserialize?: (data: string) => unknown;
}

class CacheManager {
  private localCache: LocalCache;
  private redisCache: RedisCache;
  private memcachedCache: MemcachedCache;

  async get<T>(key: string, options?: CacheOptions): Promise<T | null> {
    // Try local cache first
    // Then try Redis
    // Then try Memcached
    // Handle cache stampede
    // Handle serialization
    // ... 200 lines
  }

  // ... 500 more lines
}

// REALITY: We have 10 users
// REALITY: Our API responds in 50ms
// REALITY: We've spent a week on caching we don't need
```

✅ **Good:**

```typescript
// No caching initially - just fetch the data
async function getProducts(): Promise<Product[]> {
  return await db.query('SELECT * FROM products WHERE active = true');
}

// LATER, when you MEASURE a performance problem:
// Add the simplest caching that solves it

const productsCache = new Map<string, { data: Product[]; expiry: number }>();

async function getProducts(): Promise<Product[]> {
  const cached = productsCache.get('all');
  if (cached && cached.expiry > Date.now()) {
    return cached.data;
  }

  const products = await db.query('SELECT * FROM products WHERE active = true');
  productsCache.set('all', { data: products, expiry: Date.now() + 60000 });
  return products;
}

// Simple in-memory cache. 10 lines. Solves the actual problem.
// Add Redis only when you ACTUALLY need distributed caching.
```

---

## 6. References

- Kent Beck - ["Extreme Programming Explained"](https://www.amazon.com/Extreme-Programming-Explained-Embrace-Change/dp/0321278658) - Origin of YAGNI
- [YAGNI - Wikipedia](https://en.wikipedia.org/wiki/You_aren%27t_gonna_need_it)
- Martin Fowler - ["Yagni"](https://martinfowler.com/bliki/Yagni.html) - Detailed explanation
- Ron Jeffries - ["You're NOT gonna need it!"](https://ronjeffries.com/xprog/articles/practices/pracnotneed/) - Original XP article
