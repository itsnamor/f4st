# Separation of Concerns (SoC)

---

## 1. Overview

**Separation of Concerns** is a design principle for organizing code so that each section addresses a single, well-defined responsibility. The term was coined by Edsger W. Dijkstra in 1974. It's about **dividing a system into distinct parts that overlap as little as possible**.

**Key Idea:** Each piece of code should do ONE thing and do it well. Different concerns should live in different places.

---

## 2. Why It Matters

### The Problem

When concerns are mixed together:

- Changes in one area unexpectedly break unrelated functionality
- Code is difficult or impossible to test in isolation
- Understanding a module requires understanding everything it touches
- Multiple developers can't work on different features without conflicts
- Reusing code requires pulling in unrelated dependencies
- Bug hunting becomes a nightmare - "where does this behavior come from?"

### The Solution

Following SoC provides:

- **Isolated changes** - Modify one concern without affecting others
- **Easy testing** - Test each concern independently with simple mocks
- **Clear understanding** - Each module has one purpose, easy to grasp
- **Parallel development** - Teams work on different concerns simultaneously
- **Reusability** - Components can be reused without dragging dependencies
- **Maintainability** - Find and fix bugs in the relevant module

---

## 3. Core Concepts

### What is a "Concern"?

```plaintext
┌─────────────────────────────────────────────────────────────────┐
│                    WHAT IS A CONCERN?                            │
│                                                                  │
│   A concern is a distinct aspect of functionality or behavior.  │
│                                                                  │
│   TECHNICAL CONCERNS:                                           │
│   ├── Data fetching (how to get data)                          │
│   ├── State management (how to store data)                     │
│   ├── Presentation (how to display data)                       │
│   ├── Validation (how to verify data)                          │
│   ├── Error handling (how to handle failures)                  │
│   ├── Logging (how to record events)                           │
│   └── Authentication (how to verify identity)                  │
│                                                                  │
│   BUSINESS CONCERNS:                                            │
│   ├── User management (registration, profiles, permissions)    │
│   ├── Order processing (cart, checkout, fulfillment)           │
│   ├── Payment handling (billing, refunds, subscriptions)       │
│   ├── Notifications (email, push, SMS)                         │
│   └── Analytics (tracking, reporting, dashboards)              │
└─────────────────────────────────────────────────────────────────┘
```

### Horizontal vs Vertical Separation

```plaintext
┌─────────────────────────────────────────────────────────────────┐
│               HORIZONTAL SEPARATION (LAYERS)                    │
│                                                                  │
│   Separates by TECHNICAL responsibility                         │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                   PRESENTATION LAYER                     │   │
│   │         Components, Pages, Styling, User Input          │   │
│   ├─────────────────────────────────────────────────────────┤   │
│   │                   APPLICATION LAYER                      │   │
│   │          Use Cases, Orchestration, Validation           │   │
│   ├─────────────────────────────────────────────────────────┤   │
│   │                     DOMAIN LAYER                         │   │
│   │         Business Rules, Entities, Value Objects         │   │
│   ├─────────────────────────────────────────────────────────┤   │
│   │                 INFRASTRUCTURE LAYER                     │   │
│   │           Database, APIs, External Services             │   │
│   └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                VERTICAL SEPARATION (FEATURES)                    │
│                                                                  │
│   Separates by BUSINESS domain                                  │
│                                                                  │
│   ┌───────────────┬───────────────┬───────────────┐             │
│   │     USERS     │    ORDERS     │   PAYMENTS    │             │
│   │   Feature     │    Feature    │    Feature    │             │
│   │               │               │               │             │
│   │  ┌─────────┐  │  ┌─────────┐  │  ┌─────────┐  │             │
│   │  │   UI    │  │  │   UI    │  │  │   UI    │  │             │
│   │  ├─────────┤  │  ├─────────┤  │  ├─────────┤  │             │
│   │  │  Logic  │  │  │  Logic  │  │  │  Logic  │  │             │
│   │  ├─────────┤  │  ├─────────┤  │  ├─────────┤  │             │
│   │  │  Data   │  │  │  Data   │  │  │  Data   │  │             │
│   │  └─────────┘  │  └─────────┘  │  └─────────┘  │             │
│   └───────────────┴───────────────┴───────────────┘             │
└─────────────────────────────────────────────────────────────────┘
```

### Cohesion and Coupling

```plaintext
┌─────────────────────────────────────────────────────────────────┐
│                    COHESION & COUPLING                           │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  HIGH COHESION (GOOD)                                    │   │
│   │                                                          │   │
│   │  Everything in a module is closely related.             │   │
│   │  The module does ONE thing well.                        │   │
│   │                                                          │   │
│   │  UserService:                                            │   │
│   │    ✅ createUser()                                       │   │
│   │    ✅ updateUser()                                       │   │
│   │    ✅ deleteUser()                                       │   │
│   │    ✅ getUserById()                                      │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  LOW COHESION (BAD)                                      │   │
│   │                                                          │   │
│   │  Module does many unrelated things.                     │   │
│   │                                                          │   │
│   │  UtilityService:                                         │   │
│   │    ❌ createUser()                                       │   │
│   │    ❌ sendEmail()                                        │   │
│   │    ❌ formatCurrency()                                   │   │
│   │    ❌ validateOrder()                                    │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  LOW COUPLING (GOOD)                                     │   │
│   │                                                          │   │
│   │  Modules are independent. Changes don't cascade.        │   │
│   │                                                          │   │
│   │  UserService ──interface──▶ EmailService                │   │
│   │       │                                                  │   │
│   │       └── Can swap EmailService for SMSService easily   │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  HIGH COUPLING (BAD)                                     │   │
│   │                                                          │   │
│   │  Modules deeply intertwined. Changes break everything.  │   │
│   │                                                          │   │
│   │  UserService ──────────────▶ EmailService               │   │
│   │       │                           │                      │   │
│   │       ├──────────────────────────▶│                      │   │
│   │       │                           │                      │   │
│   │       └── Knows internal details of EmailService        │   │
│   └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms

| Term | Definition |
| --- | --- |
| Concern | A distinct aspect of functionality (e.g., data fetching, validation) |
| Cohesion | How closely related elements within a module are (HIGH = good) |
| Coupling | How dependent modules are on each other (LOW = good) |
| Layer | A horizontal slice of the system (presentation, business, data) |
| Module | A self-contained unit addressing one concern |

---

## 4. Workflow & Checklist

### Recommended Workflow

```plaintext
1. IDENTIFY        →   2. DEFINE          →   3. CREATE         →   4. CONNECT
   Concerns             Boundaries             Modules              Via Interfaces

   "What are the       "Where does one        "Build each          "Define clear
    distinct pieces     concern end and        concern as a         contracts between
    of functionality?"  another begin?"        separate unit"       modules"
```

### Checklist

Before writing or reviewing code, verify:

- [ ] **SINGLE PURPOSE:** Does this module/function do exactly ONE thing?
- [ ] **DESCRIBABLE:** Can I describe what this module does in one sentence WITHOUT using "and"?
- [ ] **TESTABLE:** Can I test this module without setting up unrelated dependencies?
- [ ] **CHANGEABLE:** If I change this, will unrelated features break?
- [ ] **REUSABLE:** Can I use this module in a different context without modifications?

---

## 5. Examples

### Example 1: React Component with Mixed Concerns

❌ **Bad:**

```tsx
// One component doing EVERYTHING
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  // DATA FETCHING CONCERN
  useEffect(() => {
    setIsLoading(true);
    fetch(`/api/users/${userId}`)
      .then((res) => {
        if (!res.ok) throw new Error('Failed to fetch');
        return res.json();
      })
      .then((data) => {
        setUser(data);
        setError(null);
      })
      .catch((err) => setError(err.message))
      .finally(() => setIsLoading(false));
  }, [userId]);

  // BUSINESS LOGIC CONCERN
  const isAdmin = user?.role === 'admin';
  const canEdit = isAdmin || user?.id === userId;
  const displayName = user ? `${user.firstName} ${user.lastName}`.trim() : '';
  const memberSince = user
    ? new Date(user.createdAt).toLocaleDateString('en-US', {
        year: 'numeric',
        month: 'long',
      })
    : '';

  // VALIDATION CONCERN
  const handleUpdateName = async (newName: string) => {
    if (!newName.trim()) {
      setError('Name cannot be empty');
      return;
    }
    if (newName.length > 100) {
      setError('Name too long');
      return;
    }
    // ... update logic
  };

  // PRESENTATION CONCERN (all mixed together)
  if (isLoading) return <div className="spinner">Loading...</div>;
  if (error) return <div className="error">{error}</div>;
  if (!user) return <div>User not found</div>;

  return (
    <div className="user-profile">
      <img src={user.avatar} alt={displayName} />
      <h1>{displayName}</h1>
      <p>Member since {memberSince}</p>
      {isAdmin && <span className="badge">Admin</span>}
      {canEdit && <button onClick={() => {}}>Edit Profile</button>}
    </div>
  );
}

// Problems:
// - Can't test business logic without mounting the component
// - Can't reuse the fetching logic elsewhere
// - Can't swap the data source (API → GraphQL)
// - Changing the UI might break the business logic
```

✅ **Good:**

```tsx
// 1. DATA FETCHING CONCERN - hooks/useUser.ts
function useUser(userId: string) {
  return useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  });
}

// 2. API CONCERN - api/users.ts
async function fetchUser(userId: string): Promise<User> {
  const response = await fetch(`/api/users/${userId}`);
  if (!response.ok) throw new Error('Failed to fetch user');
  return response.json();
}

// 3. BUSINESS LOGIC CONCERN - domain/user.ts
function canEditProfile(viewer: User, profileOwner: User): boolean {
  return viewer.role === 'admin' || viewer.id === profileOwner.id;
}

function formatDisplayName(user: User): string {
  return `${user.firstName} ${user.lastName}`.trim();
}

function formatMemberSince(user: User): string {
  return new Date(user.createdAt).toLocaleDateString('en-US', {
    year: 'numeric',
    month: 'long',
  });
}

// 4. VALIDATION CONCERN - validation/user.ts
const userNameSchema = z.string().min(1, 'Name cannot be empty').max(100, 'Name too long');

function validateUserName(name: string): { valid: boolean; error?: string } {
  const result = userNameSchema.safeParse(name);
  return result.success ? { valid: true } : { valid: false, error: result.error.message };
}

// 5. PRESENTATION CONCERN - components/UserProfile.tsx
function UserProfile({ userId }: { userId: string }) {
  const { data: user, isLoading, error } = useUser(userId);
  const currentUser = useCurrentUser();

  if (isLoading) return <LoadingSpinner />;
  if (error) return <ErrorMessage message={error.message} />;
  if (!user) return <NotFound message="User not found" />;

  const canEdit = canEditProfile(currentUser, user);

  return (
    <div className="user-profile">
      <UserAvatar user={user} />
      <h1>{formatDisplayName(user)}</h1>
      <p>Member since {formatMemberSince(user)}</p>
      {user.role === 'admin' && <Badge>Admin</Badge>}
      {canEdit && <EditProfileButton userId={user.id} />}
    </div>
  );
}

// Benefits:
// ✅ Business logic is testable without React
// ✅ API layer can be swapped (REST → GraphQL) without touching UI
// ✅ Validation can be reused in forms, API handlers, etc.
// ✅ Each piece can be modified independently
```

### Example 2: API Route Handler

❌ **Bad:**

```typescript
// One handler doing everything
app.post('/api/orders', async (req, res) => {
  // AUTHENTICATION CONCERN
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) {
    return res.status(401).json({ error: 'Unauthorized' });
  }
  let userId;
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    userId = decoded.userId;
  } catch {
    return res.status(401).json({ error: 'Invalid token' });
  }

  // VALIDATION CONCERN
  const { items, shippingAddress } = req.body;
  if (!items || !Array.isArray(items) || items.length === 0) {
    return res.status(400).json({ error: 'Items required' });
  }
  if (!shippingAddress || !shippingAddress.street) {
    return res.status(400).json({ error: 'Shipping address required' });
  }

  // BUSINESS LOGIC CONCERN
  let total = 0;
  for (const item of items) {
    const product = await db.query('SELECT price FROM products WHERE id = $1', [item.productId]);
    if (!product.rows[0]) {
      return res.status(400).json({ error: `Product ${item.productId} not found` });
    }
    total += product.rows[0].price * item.quantity;
  }

  // Apply discount if total > 100
  if (total > 100) {
    total = total * 0.9;
  }

  // DATA ACCESS CONCERN
  const result = await db.query(
    'INSERT INTO orders (user_id, items, total, shipping_address) VALUES ($1, $2, $3, $4) RETURNING *',
    [userId, JSON.stringify(items), total, JSON.stringify(shippingAddress)]
  );

  // NOTIFICATION CONCERN
  await sendEmail(user.email, 'Order Confirmation', `Your order #${result.rows[0].id} has been placed.`);

  // LOGGING CONCERN
  console.log(`Order ${result.rows[0].id} created by user ${userId}`);

  res.status(201).json(result.rows[0]);
});

// Problems:
// - 80+ lines in one function
// - Can't test business logic without setting up HTTP, auth, DB, email
// - Can't reuse order creation logic elsewhere
// - Every change requires understanding everything
```

✅ **Good:**

```typescript
// 1. AUTHENTICATION CONCERN - middleware/auth.ts
const authenticate = (req: Request, res: Response, next: NextFunction) => {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) {
    return res.status(401).json({ error: 'Unauthorized' });
  }
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.userId = decoded.userId;
    next();
  } catch {
    return res.status(401).json({ error: 'Invalid token' });
  }
};

// 2. VALIDATION CONCERN - validation/order.ts
const createOrderSchema = z.object({
  items: z.array(z.object({
    productId: z.string(),
    quantity: z.number().positive(),
  })).min(1),
  shippingAddress: z.object({
    street: z.string().min(1),
    city: z.string().min(1),
    zipCode: z.string().min(1),
  }),
});

// 3. BUSINESS LOGIC CONCERN - domain/order.ts
async function calculateOrderTotal(items: OrderItem[]): Promise<number> {
  let total = 0;
  for (const item of items) {
    const product = await productRepository.findById(item.productId);
    if (!product) throw new NotFoundError(`Product ${item.productId} not found`);
    total += product.price * item.quantity;
  }
  return applyDiscount(total);
}

function applyDiscount(total: number): number {
  return total > 100 ? total * 0.9 : total;
}

// 4. DATA ACCESS CONCERN - repositories/order.ts
const orderRepository = {
  async create(data: CreateOrderData): Promise<Order> {
    const result = await db.query(
      'INSERT INTO orders (user_id, items, total, shipping_address) VALUES ($1, $2, $3, $4) RETURNING *',
      [data.userId, JSON.stringify(data.items), data.total, JSON.stringify(data.shippingAddress)]
    );
    return result.rows[0];
  },
};

// 5. NOTIFICATION CONCERN - services/notification.ts
const notificationService = {
  async sendOrderConfirmation(order: Order, user: User): Promise<void> {
    await sendEmail(user.email, 'Order Confirmation', `Your order #${order.id} has been placed.`);
  },
};

// 6. LOGGING CONCERN - services/logger.ts
const logger = {
  orderCreated(order: Order, userId: string): void {
    console.log(`Order ${order.id} created by user ${userId}`);
  },
};

// 7. ROUTE HANDLER - routes/orders.ts (THIN - just orchestration)
app.post('/api/orders', authenticate, async (req, res) => {
  // Validate
  const parseResult = createOrderSchema.safeParse(req.body);
  if (!parseResult.success) {
    return res.status(400).json({ error: parseResult.error.message });
  }

  try {
    // Calculate
    const total = await calculateOrderTotal(parseResult.data.items);

    // Create
    const order = await orderRepository.create({
      userId: req.userId,
      items: parseResult.data.items,
      total,
      shippingAddress: parseResult.data.shippingAddress,
    });

    // Side effects (can be async/queued)
    const user = await userRepository.findById(req.userId);
    await notificationService.sendOrderConfirmation(order, user);
    logger.orderCreated(order, req.userId);

    res.status(201).json(order);
  } catch (error) {
    if (error instanceof NotFoundError) {
      return res.status(400).json({ error: error.message });
    }
    throw error;
  }
});

// Benefits:
// ✅ Each concern is testable in isolation
// ✅ Business logic doesn't know about HTTP
// ✅ Can swap notification method (email → SMS) without touching order logic
// ✅ Handler is thin - just wires things together
```

### Example 3: Frontend Feature Structure

❌ **Bad:**

```plaintext
src/
├── components/
│   ├── UserList.tsx          # Has API calls, business logic, UI
│   ├── OrderForm.tsx         # Has validation, API calls, state
│   ├── PaymentModal.tsx      # Has Stripe logic, UI, validation
│   └── utils.ts              # Random utilities for everything
├── api.ts                    # All API calls mixed together
└── types.ts                  # All types mixed together
```

✅ **Good:**

```plaintext
src/
├── features/
│   ├── users/
│   │   ├── api/              # Data fetching concern
│   │   │   └── userApi.ts
│   │   ├── components/       # Presentation concern
│   │   │   ├── UserList.tsx
│   │   │   ├── UserCard.tsx
│   │   │   └── UserAvatar.tsx
│   │   ├── hooks/            # State management concern
│   │   │   └── useUsers.ts
│   │   ├── utils/            # Business logic concern
│   │   │   └── userHelpers.ts
│   │   ├── validation/       # Validation concern
│   │   │   └── userSchema.ts
│   │   └── types.ts          # Types for this feature
│   │
│   ├── orders/
│   │   ├── api/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── utils/
│   │   └── types.ts
│   │
│   └── payments/
│       ├── api/
│       ├── components/
│       ├── hooks/
│       └── types.ts
│
├── shared/                   # Cross-feature shared code
│   ├── components/           # Generic UI components
│   │   ├── Button.tsx
│   │   ├── Modal.tsx
│   │   └── LoadingSpinner.tsx
│   ├── hooks/                # Generic hooks
│   │   └── useAsync.ts
│   └── utils/                # Generic utilities
│       └── formatting.ts
│
└── lib/                      # Infrastructure concerns
    ├── api-client.ts         # HTTP client setup
    ├── auth.ts               # Authentication
    └── analytics.ts          # Analytics tracking
```

### Example 4: Custom Hook with Mixed Concerns

❌ **Bad:**

```typescript
// Hook doing too many things
function useProductPage(productId: string) {
  const [product, setProduct] = useState<Product | null>(null);
  const [reviews, setReviews] = useState<Review[]>([]);
  const [relatedProducts, setRelatedProducts] = useState<Product[]>([]);
  const [isInCart, setIsInCart] = useState(false);
  const [isInWishlist, setIsInWishlist] = useState(false);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    Promise.all([
      fetch(`/api/products/${productId}`).then((r) => r.json()),
      fetch(`/api/products/${productId}/reviews`).then((r) => r.json()),
      fetch(`/api/products/${productId}/related`).then((r) => r.json()),
      fetch(`/api/cart`).then((r) => r.json()),
      fetch(`/api/wishlist`).then((r) => r.json()),
    ]).then(([prod, revs, related, cart, wishlist]) => {
      setProduct(prod);
      setReviews(revs);
      setRelatedProducts(related);
      setIsInCart(cart.items.some((i) => i.productId === productId));
      setIsInWishlist(wishlist.items.some((i) => i.productId === productId));
      setIsLoading(false);
    });
  }, [productId]);

  const addToCart = async () => { /* ... */ };
  const addToWishlist = async () => { /* ... */ };
  const submitReview = async () => { /* ... */ };

  return {
    product,
    reviews,
    relatedProducts,
    isInCart,
    isInWishlist,
    isLoading,
    addToCart,
    addToWishlist,
    submitReview,
  };
}
```

✅ **Good:**

```typescript
// Each concern in its own hook

// Product data concern
function useProduct(productId: string) {
  return useQuery({
    queryKey: ['product', productId],
    queryFn: () => fetchProduct(productId),
  });
}

// Reviews concern
function useProductReviews(productId: string) {
  return useQuery({
    queryKey: ['product-reviews', productId],
    queryFn: () => fetchProductReviews(productId),
  });
}

// Related products concern
function useRelatedProducts(productId: string) {
  return useQuery({
    queryKey: ['related-products', productId],
    queryFn: () => fetchRelatedProducts(productId),
  });
}

// Cart concern
function useCart() {
  const query = useQuery({ queryKey: ['cart'], queryFn: fetchCart });
  const addMutation = useMutation({ mutationFn: addToCart });

  return {
    cart: query.data,
    isInCart: (productId: string) => query.data?.items.some((i) => i.productId === productId),
    addToCart: addMutation.mutate,
  };
}

// Wishlist concern
function useWishlist() {
  const query = useQuery({ queryKey: ['wishlist'], queryFn: fetchWishlist });
  const addMutation = useMutation({ mutationFn: addToWishlist });

  return {
    wishlist: query.data,
    isInWishlist: (productId: string) => query.data?.items.some((i) => i.productId === productId),
    addToWishlist: addMutation.mutate,
  };
}

// Component composes what it needs
function ProductPage({ productId }: { productId: string }) {
  const { data: product, isLoading } = useProduct(productId);
  const { data: reviews } = useProductReviews(productId);
  const { isInCart, addToCart } = useCart();
  const { isInWishlist, addToWishlist } = useWishlist();

  // Each hook is focused, testable, and reusable
}
```

---

## 6. References

- Edsger W. Dijkstra - ["On the role of scientific thought"](https://www.cs.utexas.edu/users/EWD/transcriptions/EWD04xx/EWD447.html) (1974) - Origin of the term
- [Separation of Concerns - Wikipedia](https://en.wikipedia.org/wiki/Separation_of_concerns)
- Robert C. Martin - ["Clean Architecture"](https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164) - Architectural boundaries
- Martin Fowler - ["Presentation Domain Data Layering"](https://martinfowler.com/bliki/PresentationDomainDataLayering.html)
