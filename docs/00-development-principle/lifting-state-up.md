# Lifting State Up

---

## 1. Overview

**Lifting State Up** is a React pattern where you move shared state from child components to their closest common ancestor. When multiple components need to reflect the same changing data, you "lift" that state to their shared parent, making the parent the single source of truth.

**Key Idea:** When sibling components need to share state, move it to their closest common parent. Parent owns the state; children receive it as props.

---

## 2. Why It Matters

### The Problem

When sibling components manage their own copies of shared state:

- Duplicate state becomes inconsistent ("I changed it here, why didn't it update there?")
- Components cannot communicate or stay synchronized
- State synchronization bugs are common and hard to debug
- Testing requires complex mocking of component interactions
- No single source of truth - which component has the "real" value?

### The Solution

Lifting state up provides:

- **Single source of truth** - One component owns the state
- **Automatic sync** - All children reflect the same value
- **Clear data flow** - State flows down, events flow up
- **Centralized logic** - Parent handles all state updates
- **Easy testing** - Test the parent's state logic in isolation

---

## 3. Core Concepts

### Before and After Lifting

```plaintext
┌─────────────────────────────────────────────────────────────────┐
│                    BEFORE: DUPLICATED STATE                      │
│                                                                  │
│   ┌─────────────────┐           ┌─────────────────┐             │
│   │    Child A      │           │    Child B      │             │
│   │                 │           │                 │             │
│   │  state: value   │           │  state: value   │             │
│   │  (copy 1)       │           │  (copy 2)       │             │
│   └─────────────────┘           └─────────────────┘             │
│                                                                  │
│   ❌ Two copies of "value"                                      │
│   ❌ They can get out of sync                                   │
│   ❌ No communication between A and B                           │
│   ❌ Which one is "correct"?                                    │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    AFTER: LIFTED STATE                           │
│                                                                  │
│                    ┌─────────────────┐                          │
│                    │     Parent      │                          │
│                    │                 │                          │
│                    │  state: value   │  ← Single source of truth│
│                    │                 │                          │
│                    └────────┬────────┘                          │
│                             │                                   │
│                    ┌────────┴────────┐                          │
│                    │                 │                          │
│              ┌─────▼─────┐     ┌─────▼─────┐                    │
│              │  Child A  │     │  Child B  │                    │
│              │           │     │           │                    │
│              │ props:    │     │ props:    │                    │
│              │   value   │     │   value   │                    │
│              │   onChange│     │   onChange│                    │
│              └───────────┘     └───────────┘                    │
│                                                                  │
│   ✅ One source of truth (Parent)                               │
│   ✅ Both children always show same value                       │
│   ✅ Changes go through Parent                                  │
│   ✅ Clear ownership and data flow                              │
└─────────────────────────────────────────────────────────────────┘
```

### Controlled vs Uncontrolled Components

```plaintext
┌─────────────────────────────────────────────────────────────────┐
│                  CONTROLLED COMPONENTS                           │
│                                                                  │
│   Component receives its value from props and reports changes   │
│   via callbacks. Parent is in control.                          │
│                                                                  │
│   function ControlledInput({ value, onChange }) {               │
│     return (                                                    │
│       <input                                                    │
│         value={value}           // Value from parent            │
│         onChange={(e) => onChange(e.target.value)}  // Tell parent │
│       />                                                        │
│     );                                                          │
│   }                                                             │
│                                                                  │
│   ✅ Parent has full control over the value                     │
│   ✅ Can validate, transform, or reject changes                 │
│   ✅ Easy to synchronize with other components                  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                 UNCONTROLLED COMPONENTS                          │
│                                                                  │
│   Component manages its own internal state.                     │
│   Parent cannot directly control or observe it.                 │
│                                                                  │
│   function UncontrolledInput() {                                │
│     const [value, setValue] = useState('');  // Own state       │
│                                                                  │
│     return (                                                    │
│       <input                                                    │
│         value={value}                                           │
│         onChange={(e) => setValue(e.target.value)}              │
│       />                                                        │
│     );                                                          │
│   }                                                             │
│                                                                  │
│   ⚠️ Parent cannot see or control the value                     │
│   ⚠️ Cannot synchronize with sibling components                │
│   ⚠️ Only use when isolation is intentional                     │
└─────────────────────────────────────────────────────────────────┘
```

### When to Lift State

```plaintext
┌─────────────────────────────────────────────────────────────────┐
│                    WHEN TO LIFT STATE                            │
│                                                                  │
│   LIFT STATE WHEN:                                              │
│   ────────────────                                              │
│   ✅ Two+ components need the SAME changing data                │
│   ✅ Changes in one component should reflect in another         │
│   ✅ Parent needs to know about or control children's state     │
│   ✅ You need to validate or transform input before accepting   │
│   ✅ State needs to be persisted or shared with external systems│
│                                                                  │
│   KEEP STATE LOCAL WHEN:                                        │
│   ──────────────────────                                        │
│   ⚪ Only one component uses the state                          │
│   ⚪ State is purely presentational (hover, focus, animations)  │
│   ⚪ Lifting would cause unnecessary re-renders                 │
│   ⚪ Component is meant to be a self-contained "widget"         │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms

| Term | Definition |
| --- | --- |
| Lifting State | Moving state from child to parent component |
| Controlled Component | Component whose value is controlled by its parent via props |
| Uncontrolled Component | Component that manages its own internal state |
| Common Ancestor | Closest parent component that contains all components needing the state |
| Single Source of Truth | One authoritative location for the state |

---

## 4. Workflow & Checklist

### Recommended Workflow

```plaintext
1. IDENTIFY        →   2. FIND            →   3. LIFT           →   4. CONNECT
   Shared State         Common Ancestor       State Up             Children

   "Which state        "What's the           "Move useState       "Pass value &
    do multiple         closest parent        to parent"           onChange as
    children need?"     that has both?"                            props"
```

### Checklist

When deciding whether to lift state, verify:

- [ ] **SHARED:** Do 2+ components need this same state?
- [ ] **ANCESTOR:** Have I identified the closest common parent?
- [ ] **LIFTED:** Is the state now in the parent component?
- [ ] **PROPS:** Am I passing the value down as props?
- [ ] **CALLBACKS:** Am I passing update functions for children to call?

---

## 5. Examples

### Example 1: Synchronized Inputs (Temperature Converter)

❌ **Bad:**

```tsx
// Each input manages its own state - they can't sync
function CelsiusInput() {
  const [celsius, setCelsius] = useState('');

  return (
    <div>
      <label>Celsius:</label>
      <input value={celsius} onChange={(e) => setCelsius(e.target.value)} />
    </div>
  );
}

function FahrenheitInput() {
  const [fahrenheit, setFahrenheit] = useState('');

  return (
    <div>
      <label>Fahrenheit:</label>
      <input value={fahrenheit} onChange={(e) => setFahrenheit(e.target.value)} />
    </div>
  );
}

function TemperatureConverter() {
  return (
    <div>
      <CelsiusInput />
      <FahrenheitInput />
      {/* These inputs are completely independent! */}
      {/* Changing one doesn't update the other */}
    </div>
  );
}
```

✅ **Good:**

```tsx
// Conversion utilities (pure functions)
const toCelsius = (f: string): string => {
  const fahrenheit = parseFloat(f);
  if (isNaN(fahrenheit)) return '';
  return ((fahrenheit - 32) * 5 / 9).toFixed(2);
};

const toFahrenheit = (c: string): string => {
  const celsius = parseFloat(c);
  if (isNaN(celsius)) return '';
  return ((celsius * 9 / 5) + 32).toFixed(2);
};

// Controlled input component
function TemperatureInput({
  scale,
  value,
  onChange,
}: {
  scale: 'celsius' | 'fahrenheit';
  value: string;
  onChange: (value: string) => void;
}) {
  return (
    <div>
      <label>{scale === 'celsius' ? 'Celsius' : 'Fahrenheit'}:</label>
      <input value={value} onChange={(e) => onChange(e.target.value)} />
    </div>
  );
}

// Parent owns the state - single source of truth
function TemperatureConverter() {
  const [temperature, setTemperature] = useState('');
  const [scale, setScale] = useState<'celsius' | 'fahrenheit'>('celsius');

  // Derive both values from the single source of truth
  const celsius = scale === 'fahrenheit' ? toCelsius(temperature) : temperature;
  const fahrenheit = scale === 'celsius' ? toFahrenheit(temperature) : temperature;

  const handleCelsiusChange = (value: string) => {
    setScale('celsius');
    setTemperature(value);
  };

  const handleFahrenheitChange = (value: string) => {
    setScale('fahrenheit');
    setTemperature(value);
  };

  return (
    <div>
      <TemperatureInput scale="celsius" value={celsius} onChange={handleCelsiusChange} />
      <TemperatureInput scale="fahrenheit" value={fahrenheit} onChange={handleFahrenheitChange} />

      {/* Both inputs are now synchronized! */}
      {/* Change one, the other updates automatically */}
    </div>
  );
}
```

### Example 2: Form with Cross-Field Validation

❌ **Bad:**

```tsx
// Each field validates itself - can't do cross-field validation
function EmailField() {
  const [email, setEmail] = useState('');
  const [error, setError] = useState('');

  const handleChange = (value: string) => {
    setEmail(value);
    // Can only validate this field in isolation
    if (!value.includes('@')) {
      setError('Invalid email');
    } else {
      setError('');
    }
  };

  return (
    <div>
      <input value={email} onChange={(e) => handleChange(e.target.value)} />
      {error && <span className="error">{error}</span>}
    </div>
  );
}

function ConfirmEmailField() {
  const [confirmEmail, setConfirmEmail] = useState('');
  // Problem: Can't check if this matches the email field!
  // We don't have access to that state!
}
```

✅ **Good:**

```tsx
// Form manages all field state - can do any validation
type FormValues = {
  email: string;
  confirmEmail: string;
  password: string;
  confirmPassword: string;
};

type FormErrors = Partial<Record<keyof FormValues, string>>;

// Pure validation function
function validateForm(values: FormValues): FormErrors {
  const errors: FormErrors = {};

  if (!values.email) {
    errors.email = 'Email is required';
  } else if (!values.email.includes('@')) {
    errors.email = 'Invalid email format';
  }

  if (values.confirmEmail !== values.email) {
    errors.confirmEmail = 'Emails must match';  // Cross-field validation!
  }

  if (values.password.length < 8) {
    errors.password = 'Password must be at least 8 characters';
  }

  if (values.confirmPassword !== values.password) {
    errors.confirmPassword = 'Passwords must match';  // Cross-field validation!
  }

  return errors;
}

// Controlled input component
function FormField({
  label,
  type = 'text',
  value,
  error,
  onChange,
}: {
  label: string;
  type?: string;
  value: string;
  error?: string;
  onChange: (value: string) => void;
}) {
  return (
    <div>
      <label>{label}</label>
      <input
        type={type}
        value={value}
        onChange={(e) => onChange(e.target.value)}
        className={error ? 'error' : ''}
      />
      {error && <span className="error">{error}</span>}
    </div>
  );
}

// Form component - owns all state
function RegistrationForm() {
  const [values, setValues] = useState<FormValues>({
    email: '',
    confirmEmail: '',
    password: '',
    confirmPassword: '',
  });

  const [touched, setTouched] = useState<Set<keyof FormValues>>(new Set());

  // Validate entire form (all fields available)
  const errors = validateForm(values);

  const handleChange = (field: keyof FormValues) => (value: string) => {
    setValues((prev) => ({ ...prev, [field]: value }));
    setTouched((prev) => new Set(prev).add(field));
  };

  const showError = (field: keyof FormValues) =>
    touched.has(field) ? errors[field] : undefined;

  const isValid = Object.keys(errors).length === 0;

  return (
    <form onSubmit={(e) => { e.preventDefault(); if (isValid) { submit(values); } }}>
      <FormField
        label="Email"
        value={values.email}
        error={showError('email')}
        onChange={handleChange('email')}
      />
      <FormField
        label="Confirm Email"
        value={values.confirmEmail}
        error={showError('confirmEmail')}
        onChange={handleChange('confirmEmail')}
      />
      <FormField
        label="Password"
        type="password"
        value={values.password}
        error={showError('password')}
        onChange={handleChange('password')}
      />
      <FormField
        label="Confirm Password"
        type="password"
        value={values.confirmPassword}
        error={showError('confirmPassword')}
        onChange={handleChange('confirmPassword')}
      />
      <button type="submit" disabled={!isValid}>
        Register
      </button>
    </form>
  );
}
```

### Example 3: Accordion (Only One Open at a Time)

❌ **Bad:**

```tsx
// Each accordion item manages its own open state
function AccordionItem({ title, children }: AccordionItemProps) {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div>
      <button onClick={() => setIsOpen(!isOpen)}>{title}</button>
      {isOpen && <div>{children}</div>}
    </div>
  );
}

function FAQ() {
  return (
    <div>
      {/* Problem: Multiple items can be open at once! */}
      {/* Can't enforce "only one open" without lifting state */}
      <AccordionItem title="Question 1">Answer 1</AccordionItem>
      <AccordionItem title="Question 2">Answer 2</AccordionItem>
      <AccordionItem title="Question 3">Answer 3</AccordionItem>
    </div>
  );
}
```

✅ **Good:**

```tsx
// Controlled accordion item
function AccordionItem({
  title,
  isOpen,
  onToggle,
  children,
}: {
  title: string;
  isOpen: boolean;
  onToggle: () => void;
  children: React.ReactNode;
}) {
  return (
    <div className="accordion-item">
      <button onClick={onToggle} className={isOpen ? 'active' : ''}>
        {title}
        <span>{isOpen ? '−' : '+'}</span>
      </button>
      {isOpen && <div className="accordion-content">{children}</div>}
    </div>
  );
}

// Parent controls which item is open
function Accordion({ items }: { items: Array<{ title: string; content: React.ReactNode }> }) {
  const [openIndex, setOpenIndex] = useState<number | null>(null);

  const handleToggle = (index: number) => {
    // Only one can be open at a time!
    setOpenIndex((prev) => (prev === index ? null : index));
  };

  return (
    <div className="accordion">
      {items.map((item, index) => (
        <AccordionItem
          key={index}
          title={item.title}
          isOpen={openIndex === index}
          onToggle={() => handleToggle(index)}
        >
          {item.content}
        </AccordionItem>
      ))}
    </div>
  );
}

// Usage
function FAQ() {
  const items = [
    { title: 'Question 1', content: 'Answer 1' },
    { title: 'Question 2', content: 'Answer 2' },
    { title: 'Question 3', content: 'Answer 3' },
  ];

  return <Accordion items={items} />;
}

// Now only one item can be open at a time!
// Parent controls the behavior completely.
```

### Example 4: Shopping Cart with Product List

```tsx
// A common pattern: list of items with a shared summary

// Product component - controlled, receives cart operations
function ProductCard({
  product,
  quantity,
  onAdd,
  onRemove,
}: {
  product: Product;
  quantity: number;
  onAdd: () => void;
  onRemove: () => void;
}) {
  return (
    <div className="product-card">
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      <div className="quantity-controls">
        <button onClick={onRemove} disabled={quantity === 0}>
          −
        </button>
        <span>{quantity}</span>
        <button onClick={onAdd}>+</button>
      </div>
    </div>
  );
}

// Cart summary - receives cart state as props
function CartSummary({ cart, products }: { cart: CartState; products: Product[] }) {
  const total = Object.entries(cart).reduce((sum, [productId, quantity]) => {
    const product = products.find((p) => p.id === productId);
    return sum + (product?.price ?? 0) * quantity;
  }, 0);

  const itemCount = Object.values(cart).reduce((sum, qty) => sum + qty, 0);

  return (
    <div className="cart-summary">
      <h2>Cart</h2>
      <p>{itemCount} items</p>
      <p>Total: ${total.toFixed(2)}</p>
    </div>
  );
}

// Parent owns cart state - single source of truth
type CartState = Record<string, number>;

function ShopPage({ products }: { products: Product[] }) {
  const [cart, setCart] = useState<CartState>({});

  const addToCart = (productId: string) => {
    setCart((prev) => ({
      ...prev,
      [productId]: (prev[productId] ?? 0) + 1,
    }));
  };

  const removeFromCart = (productId: string) => {
    setCart((prev) => {
      const current = prev[productId] ?? 0;
      if (current <= 1) {
        const { [productId]: _, ...rest } = prev;
        return rest;
      }
      return { ...prev, [productId]: current - 1 };
    });
  };

  return (
    <div className="shop-page">
      <div className="products">
        {products.map((product) => (
          <ProductCard
            key={product.id}
            product={product}
            quantity={cart[product.id] ?? 0}
            onAdd={() => addToCart(product.id)}
            onRemove={() => removeFromCart(product.id)}
          />
        ))}
      </div>

      <CartSummary cart={cart} products={products} />
    </div>
  );
}

// Benefits:
// ✅ ProductCards and CartSummary share the same cart state
// ✅ Add/remove in any ProductCard updates CartSummary instantly
// ✅ Single source of truth in ShopPage
// ✅ Easy to add features: save cart, sync with server, etc.
```

---

## 6. References

- [React Docs: Sharing State Between Components](https://react.dev/learn/sharing-state-between-components)
- [React Docs: Choosing the State Structure](https://react.dev/learn/choosing-the-state-structure)
- [React Docs: Extracting State Logic into a Reducer](https://react.dev/learn/extracting-state-logic-into-a-reducer)
- Kent C. Dodds - ["State Colocation"](https://kentcdodds.com/blog/state-colocation-will-make-your-react-app-faster)
