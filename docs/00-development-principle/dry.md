# DRY (Don't Repeat Yourself)

---

## 1. Overview

**DRY** is a principle that states every piece of knowledge must have a single, unambiguous, authoritative representation within a system. It was coined by Andy Hunt and Dave Thomas in "The Pragmatic Programmer". DRY is about **knowledge duplication**, not just code duplication.

**Key Idea:** If you need to change something, you should only need to change it in ONE place.

---

## 2. Why It Matters

### The Problem

When developers duplicate knowledge across the codebase:

- Changes require updates in multiple places (easy to miss one)
- Inconsistencies creep in when one copy is updated but others are forgotten
- Bug fixes must be applied repeatedly across all copies
- Code bloat makes the codebase harder to navigate
- Different team members may implement the same logic differently

### The Solution

Following DRY provides:

- **Single Source of Truth** - One authoritative location for each concept
- **Automatic propagation** - Change once, applied everywhere
- **Smaller codebase** - Less code to read, understand, and maintain
- **Consistency** - Same behavior guaranteed across the application
- **Easier refactoring** - Changes are localized

---

## 3. Core Concepts

### Knowledge vs Code Duplication

```plaintext
┌─────────────────────────────────────────────────────────────────┐
│                    CRITICAL DISTINCTION                          │
│                                                                  │
│   DRY is about KNOWLEDGE, not CODE.                             │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  Same Code, Different Knowledge = ✅ ACCEPTABLE          │   │
│   │                                                          │   │
│   │  // These look identical but represent different rules   │   │
│   │  const validateAge = (age) => age >= 18;     // Legal    │   │
│   │  const validateScore = (score) => score >= 18; // Passing │   │
│   │                                                          │   │
│   │  If legal age changes to 21, passing score stays 18.     │   │
│   │  They happen to look the same NOW but are different.     │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  Same Knowledge, Different Places = ❌ VIOLATION         │   │
│   │                                                          │   │
│   │  // user-form.ts                                         │   │
│   │  const isValidEmail = (e) => /^[^\s@]+@[^\s@]+$/.test(e);│   │
│   │                                                          │   │
│   │  // order-form.ts                                        │   │
│   │  const checkEmail = (e) => /^[^\s@]+@[^\s@]+$/.test(e);  │   │
│   │                                                          │   │
│   │  Both represent "what is a valid email" - same knowledge │   │
│   └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### Types of Duplication

```plaintext
┌─────────────────────────────────────────────────────────────────┐
│                     TYPES OF DRY VIOLATIONS                      │
│                                                                  │
│   1. CODE DUPLICATION                                           │
│      Same logic copy-pasted in multiple places                  │
│      → Extract to a shared function                             │
│                                                                  │
│   2. DATA DUPLICATION                                           │
│      Same value hardcoded in multiple places                    │
│      → Extract to a constant or configuration                   │
│                                                                  │
│   3. TYPE DUPLICATION                                           │
│      Same shape defined in multiple places                      │
│      → Extract to a shared type/interface                       │
│                                                                  │
│   4. DOCUMENTATION DUPLICATION                                  │
│      Same information in code comments AND external docs        │
│      → Keep in one place, reference from other                  │
│                                                                  │
│   5. ALGORITHM DUPLICATION                                      │
│      Same business logic implemented differently                │
│      → Unify into a single implementation                       │
└─────────────────────────────────────────────────────────────────┘
```

### The Rule of Three

```plaintext
┌─────────────────────────────────────────────────────────────────┐
│                      RULE OF THREE                               │
│                                                                  │
│   Don't abstract on first occurrence.                           │
│   Note it on second occurrence.                                 │
│   Extract on third occurrence.                                  │
│                                                                  │
│   1st time: Write it                                            │
│   2nd time: Wince, but write it again (note the duplication)    │
│   3rd time: Extract and refactor                                │
│                                                                  │
│   WHY? Premature abstraction is often WRONG abstraction.        │
│   Wait until you see the pattern clearly before abstracting.    │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms

| Term | Definition |
| --- | --- |
| DRY | "Don't Repeat Yourself" - single source of truth for knowledge |
| WET | "Write Everything Twice" or "We Enjoy Typing" - the anti-pattern |
| Single Source of Truth | One authoritative location for each piece of knowledge |
| Rule of Three | Wait for three occurrences before abstracting |
| Abstraction | A generalization that captures common patterns |

---

## 4. Workflow & Checklist

### Recommended Workflow

```plaintext
1. IDENTIFY        →   2. VERIFY          →   3. EXTRACT        →   4. TEST
   Duplication          Same Knowledge        Abstraction           Behavior

   "Have I seen       "Is this the SAME      "Create shared       "Does everything
    this before?"       business rule?"        function/type"       still work?"
```

### Checklist

Before writing or reviewing code, verify:

- [ ] **SEARCH:** Is this logic already implemented elsewhere in the codebase?
- [ ] **KNOWLEDGE:** If I'm duplicating, is it truly different business knowledge?
- [ ] **THRESHOLD:** Have I seen this pattern at least 2-3 times? (Rule of Three)
- [ ] **CLARITY:** Will extracting this make the code MORE readable, not less?
- [ ] **SCOPE:** Should this be shared within a module, or across the entire app?

---

## 5. Examples

### Example 1: Configuration Values

❌ **Bad:**

```typescript
// user-service.ts
async function fetchUser(id: string) {
  const response = await fetch(`https://api.example.com/users/${id}`);
  return response.json();
}

// order-service.ts
async function fetchOrders(userId: string) {
  const response = await fetch(`https://api.example.com/orders?userId=${userId}`);
  return response.json();
}

// payment-service.ts
async function fetchPayments(orderId: string) {
  const response = await fetch(`https://api.example.com/payments?orderId=${orderId}`);
  return response.json();
}

// Problem: If API URL changes, you need to update ALL files
// Problem: Easy to have typos in different files
```

✅ **Good:**

```typescript
// config.ts - Single Source of Truth
export const API_CONFIG = {
  baseUrl: process.env.API_URL ?? 'https://api.example.com',
  timeout: 5000,
} as const;

// api-client.ts - Shared abstraction
export async function apiRequest<T>(endpoint: string): Promise<T> {
  const response = await fetch(`${API_CONFIG.baseUrl}${endpoint}`);
  return response.json();
}

// user-service.ts
import { apiRequest } from './api-client';
const fetchUser = (id: string) => apiRequest<User>(`/users/${id}`);

// order-service.ts
import { apiRequest } from './api-client';
const fetchOrders = (userId: string) => apiRequest<Order[]>(`/orders?userId=${userId}`);

// Benefits:
// - URL changes in ONE place
// - Consistent error handling, headers, timeout can be added centrally
// - Type-safe responses
```

### Example 2: Validation Logic

❌ **Bad:**

```typescript
// registration-form.tsx
const RegistrationForm = () => {
  const validateEmail = (email: string) => {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
  };

  const validatePassword = (password: string) => {
    return password.length >= 8 && /[A-Z]/.test(password) && /[0-9]/.test(password);
  };
  // ...
};

// profile-form.tsx
const ProfileForm = () => {
  const isEmailValid = (email: string) => {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  };
  // ...
};

// checkout-form.tsx
const CheckoutForm = () => {
  const checkEmail = (e: string) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(e);
  // ...
};

// Problems:
// - Same email regex in 3 places
// - If email validation rules change, need to update all
// - Inconsistent naming (validateEmail, isEmailValid, checkEmail)
```

✅ **Good:**

```typescript
// validators.ts - Single Source of Truth for validation rules
export const validators = {
  email: (email: string): boolean => {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
  },

  password: (password: string): boolean => {
    const minLength = 8;
    const hasUppercase = /[A-Z]/.test(password);
    const hasNumber = /[0-9]/.test(password);
    return password.length >= minLength && hasUppercase && hasNumber;
  },

  phone: (phone: string): boolean => {
    return /^\+?[\d\s\-()]+$/.test(phone) && phone.replace(/\D/g, '').length >= 10;
  },
} as const;

// Optional: Validation messages also centralized
export const validationMessages = {
  email: 'Please enter a valid email address',
  password: 'Password must be at least 8 characters with uppercase and number',
  phone: 'Please enter a valid phone number',
} as const;

// registration-form.tsx
import { validators, validationMessages } from '@/lib/validators';

const RegistrationForm = () => {
  const [email, setEmail] = useState('');
  const isEmailValid = validators.email(email);
  // ...
};

// Benefits:
// - Change validation rules in ONE place
// - Consistent behavior across all forms
// - Easy to test validators in isolation
// - Validation messages also consistent
```

### Example 3: Type Definitions

❌ **Bad:**

```typescript
// user-list.tsx
type User = {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user' | 'guest';
  createdAt: Date;
};

// user-profile.tsx
interface UserData {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user' | 'guest';
  createdAt: Date;
}

// user-api.ts
type UserResponse = {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user' | 'guest';
  createdAt: string; // Oops! Inconsistency - string vs Date
};

// Problems:
// - Same structure defined 3 times
// - Easy to have subtle inconsistencies (createdAt: Date vs string)
// - If User shape changes, need to update all places
```

✅ **Good:**

```typescript
// types/user.ts - Single Source of Truth
export type UserRole = 'admin' | 'user' | 'guest';

export type User = {
  id: string;
  name: string;
  email: string;
  role: UserRole;
  createdAt: Date;
};

// For API responses where dates are strings
export type UserDTO = Omit<User, 'createdAt'> & {
  createdAt: string;
};

// Conversion helper
export const userFromDTO = (dto: UserDTO): User => ({
  ...dto,
  createdAt: new Date(dto.createdAt),
});

// user-list.tsx
import { User } from '@/types/user';

// user-profile.tsx
import { User } from '@/types/user';

// user-api.ts
import { UserDTO, userFromDTO } from '@/types/user';

const fetchUser = async (id: string): Promise<User> => {
  const response = await fetch(`/api/users/${id}`);
  const dto: UserDTO = await response.json();
  return userFromDTO(dto);
};
```

### Example 4: React Component Patterns

❌ **Bad:**

```tsx
// ProductCard.tsx
const ProductCard = ({ product }) => {
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const handleAddToCart = async () => {
    setIsLoading(true);
    setError(null);
    try {
      await addToCart(product.id);
    } catch (e) {
      setError(e.message);
    } finally {
      setIsLoading(false);
    }
  };
  // ...
};

// UserActions.tsx
const UserActions = ({ userId }) => {
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const handleFollow = async () => {
    setIsLoading(true);
    setError(null);
    try {
      await followUser(userId);
    } catch (e) {
      setError(e.message);
    } finally {
      setIsLoading(false);
    }
  };
  // ...
};

// Same async state management pattern repeated everywhere
```

✅ **Good:**

```tsx
// hooks/useAsyncAction.ts - Extract the pattern
type AsyncActionState<T> = {
  data: T | null;
  isLoading: boolean;
  error: string | null;
  execute: (...args: unknown[]) => Promise<void>;
  reset: () => void;
};

function useAsyncAction<T>(
  action: (...args: unknown[]) => Promise<T>
): AsyncActionState<T> {
  const [state, setState] = useState({
    data: null as T | null,
    isLoading: false,
    error: null as string | null,
  });

  const execute = async (...args: unknown[]) => {
    setState({ data: null, isLoading: true, error: null });
    try {
      const data = await action(...args);
      setState({ data, isLoading: false, error: null });
    } catch (e) {
      setState({ data: null, isLoading: false, error: e.message });
    }
  };

  const reset = () => setState({ data: null, isLoading: false, error: null });

  return { ...state, execute, reset };
}

// ProductCard.tsx - Clean and focused
const ProductCard = ({ product }) => {
  const { isLoading, error, execute } = useAsyncAction(() => addToCart(product.id));

  return (
    <div>
      <button onClick={execute} disabled={isLoading}>
        {isLoading ? 'Adding...' : 'Add to Cart'}
      </button>
      {error && <span className="error">{error}</span>}
    </div>
  );
};

// UserActions.tsx - Same pattern, no duplication
const UserActions = ({ userId }) => {
  const { isLoading, error, execute } = useAsyncAction(() => followUser(userId));
  // ...
};
```

### Example 5: Business Logic Duplication

❌ **Bad:**

```typescript
// checkout.ts
function calculateOrderTotal(items: CartItem[]): number {
  let subtotal = 0;
  for (const item of items) {
    subtotal += item.price * item.quantity;
  }

  // Apply 10% discount for orders over $100
  if (subtotal > 100) {
    subtotal = subtotal * 0.9;
  }

  // Add 8.5% tax
  const total = subtotal * 1.085;

  return Math.round(total * 100) / 100;
}

// cart-preview.ts
function getCartTotal(cartItems: CartItem[]): number {
  const subtotal = cartItems.reduce((sum, item) => sum + item.price * item.quantity, 0);

  // 10% off for orders over $100
  const discounted = subtotal > 100 ? subtotal * 0.9 : subtotal;

  // Tax is 8.5%
  return Math.round(discounted * 1.085 * 100) / 100;
}

// Problems:
// - Same business rules (discount threshold, tax rate) in multiple places
// - If discount changes to 15%, need to find all occurrences
// - Subtle implementation differences could cause inconsistent totals
```

✅ **Good:**

```typescript
// pricing/constants.ts
export const PRICING = {
  discountThreshold: 100,
  discountPercent: 0.1, // 10%
  taxRate: 0.085, // 8.5%
} as const;

// pricing/calculator.ts
import { PRICING } from './constants';

export function calculateSubtotal(items: CartItem[]): number {
  return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
}

export function applyDiscount(subtotal: number): number {
  if (subtotal > PRICING.discountThreshold) {
    return subtotal * (1 - PRICING.discountPercent);
  }
  return subtotal;
}

export function applyTax(amount: number): number {
  return amount * (1 + PRICING.taxRate);
}

export function calculateTotal(items: CartItem[]): number {
  const subtotal = calculateSubtotal(items);
  const discounted = applyDiscount(subtotal);
  const withTax = applyTax(discounted);
  return Math.round(withTax * 100) / 100;
}

// checkout.ts
import { calculateTotal } from '@/pricing/calculator';
const orderTotal = calculateTotal(items);

// cart-preview.ts
import { calculateTotal } from '@/pricing/calculator';
const cartTotal = calculateTotal(cartItems);

// Benefits:
// - Business rules defined ONCE
// - Easy to test each step independently
// - Change discount? Update ONE constant
// - Guaranteed consistency across the app
```

---

## 6. References

- Andy Hunt & Dave Thomas - ["The Pragmatic Programmer"](https://pragprog.com/titles/tpp20/the-pragmatic-programmer-20th-anniversary-edition/) - Origin of DRY principle
- [DRY Principle - Wikipedia](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)
- Martin Fowler - ["Refactoring"](https://martinfowler.com/books/refactoring.html) - Techniques for removing duplication
- Sandi Metz - ["The Wrong Abstraction"](https://sandimetz.com/blog/2016/1/20/the-wrong-abstraction) - Why premature DRY can be harmful
