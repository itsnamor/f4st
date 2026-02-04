# Immutability

---

## 1. Overview

**Immutability** is the principle that once data is created, it cannot be changed. Instead of modifying existing data, you create new data structures with the desired changes. This eliminates an entire class of bugs related to unexpected state changes and makes your code predictable.

**Key Idea:** Never modify data in place. Always create new copies with changes applied. The original remains untouched.

---

## 2. Why It Matters

### The Problem

When data is mutable (can be changed):

- Unexpected state changes cause mysterious, hard-to-track bugs
- Shared mutable state leads to race conditions in concurrent code
- Debugging requires tracking ALL possible mutation points across the codebase
- Change detection is expensive (requires deep comparison of objects)
- Undo/redo functionality becomes complex to implement
- You can never trust that data hasn't been modified elsewhere

### The Solution

Following Immutability provides:

- **Predictable state** - Data only changes when you explicitly create new versions
- **Safe sharing** - Pass data freely without fear of it being modified
- **Easy debugging** - Previous states are preserved; you can inspect history
- **Cheap change detection** - Reference equality (`===`) tells you if data changed
- **Time-travel debugging** - Keep all previous states for replay
- **Concurrent safety** - No race conditions on immutable data

---

## 3. Core Concepts

### Mutable vs Immutable Operations

```plaintext
┌─────────────────────────────────────────────────────────────────┐
│                    MUTABLE OPERATIONS (AVOID)                    │
│                                                                  │
│   These MODIFY the original data:                               │
│                                                                  │
│   Arrays:                                                       │
│   ❌ array.push(item)      - adds to end                        │
│   ❌ array.pop()           - removes from end                   │
│   ❌ array.shift()         - removes from start                 │
│   ❌ array.unshift(item)   - adds to start                      │
│   ❌ array.splice(i, n)    - removes/inserts elements           │
│   ❌ array.sort()          - sorts in place                     │
│   ❌ array.reverse()       - reverses in place                  │
│   ❌ array[i] = value      - direct assignment                  │
│                                                                  │
│   Objects:                                                      │
│   ❌ object.key = value    - direct property assignment         │
│   ❌ delete object.key     - removes property                   │
│   ❌ Object.assign(obj, {})- mutates first argument             │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                  IMMUTABLE OPERATIONS (USE THESE)                │
│                                                                  │
│   These return NEW data, leaving the original unchanged:        │
│                                                                  │
│   Arrays:                                                       │
│   ✅ [...array, item]           - add to end                    │
│   ✅ [item, ...array]           - add to start                  │
│   ✅ array.slice(0, -1)         - remove from end               │
│   ✅ array.slice(1)             - remove from start             │
│   ✅ array.filter(fn)           - remove by condition           │
│   ✅ array.map(fn)              - transform elements            │
│   ✅ array.concat(other)        - combine arrays                │
│   ✅ [...array].sort()          - sort a copy                   │
│   ✅ array.toSorted()           - sort (ES2023)                 │
│   ✅ array.toReversed()         - reverse (ES2023)              │
│   ✅ array.toSpliced(i, n)      - splice (ES2023)               │
│                                                                  │
│   Objects:                                                      │
│   ✅ { ...object, key: value }  - update/add property           │
│   ✅ { ...object }              - shallow copy                  │
│   ✅ Object.assign({}, obj, {}) - merge into new object         │
│   ✅ structuredClone(obj)       - deep copy                     │
└─────────────────────────────────────────────────────────────────┘
```

### Structural Sharing

```plaintext
┌─────────────────────────────────────────────────────────────────┐
│                    STRUCTURAL SHARING                            │
│                                                                  │
│   Immutable updates don't copy everything - they SHARE          │
│   unchanged parts with the original.                            │
│                                                                  │
│   Original State:                                               │
│   ┌──────────────────────────────────────┐                      │
│   │  state = {                           │                      │
│   │    user: { name: 'Alice', age: 25 }, │ ←─┐                  │
│   │    posts: [post1, post2, post3],     │ ←─┼── Shared!        │
│   │    settings: { theme: 'dark' }       │   │                  │
│   │  }                                   │   │                  │
│   └──────────────────────────────────────┘   │                  │
│                                              │                  │
│   After updating settings.theme:             │                  │
│   ┌──────────────────────────────────────┐   │                  │
│   │  newState = {                        │   │                  │
│   │    user: state.user,                 │ ──┘ (same reference) │
│   │    posts: state.posts,               │ ──┘ (same reference) │
│   │    settings: { theme: 'light' }      │ ← NEW object         │
│   │  }                                   │                      │
│   └──────────────────────────────────────┘                      │
│                                                                  │
│   Only the changed path gets new objects.                       │
│   Memory efficient! Performance efficient!                      │
└─────────────────────────────────────────────────────────────────┘
```

### Reference Equality for Change Detection

```plaintext
┌─────────────────────────────────────────────────────────────────┐
│                CHEAP CHANGE DETECTION                            │
│                                                                  │
│   With immutability, checking if something changed is O(1):     │
│                                                                  │
│   // Mutable world - need deep comparison 😢                    │
│   function hasChanged(prev, next) {                             │
│     return JSON.stringify(prev) !== JSON.stringify(next);       │
│     // Slow! O(n) where n is the size of the object             │
│   }                                                             │
│                                                                  │
│   // Immutable world - reference comparison 😊                  │
│   function hasChanged(prev, next) {                             │
│     return prev !== next;                                       │
│     // Fast! O(1) constant time                                 │
│   }                                                             │
│                                                                  │
│   This is why React's reconciliation is fast!                   │
│   If reference is the same, skip re-rendering.                  │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms

| Term | Definition |
| --- | --- |
| Immutable | Cannot be changed after creation |
| Mutable | Can be changed after creation |
| Structural Sharing | Reusing unchanged parts of data structures across versions |
| Persistent Data Structure | Data structure that preserves previous versions when modified |
| Copy-on-Write | Creating copies only when modifications are needed |

---

## 4. Workflow & Checklist

### Recommended Workflow

```plaintext
1. RECEIVE        →   2. TRANSFORM       →   3. RETURN         →   4. ORIGINAL
   Data                 Create Copy           New Version          Unchanged

   "Get the            "Spread, map,         "Return the          "Old data is
    current state"      filter, etc."         new object"          still valid"
```

### Checklist

Before writing code that modifies data, verify:

- [ ] **SPREAD:** Am I using spread operators (`...`) for object/array updates?
- [ ] **METHODS:** Am I using immutable array methods (`map`, `filter`, `concat`)?
- [ ] **AVOID:** Am I avoiding mutating methods (`push`, `pop`, `splice`, `sort`)?
- [ ] **CONST:** Am I using `const` for variable declarations?
- [ ] **NESTED:** For deeply nested updates, should I use a library like Immer?

---

## 5. Examples

### Example 1: Object Updates

❌ **Bad:**

```typescript
// Mutating the original object
function updateUser(user: User, updates: Partial<User>): User {
  user.name = updates.name ?? user.name;       // Mutation!
  user.email = updates.email ?? user.email;    // Mutation!
  user.updatedAt = new Date();                 // Mutation!
  return user;
}

const user = { id: '1', name: 'Alice', email: 'alice@example.com', updatedAt: null };
const updated = updateUser(user, { name: 'Alicia' });

console.log(user.name);       // 'Alicia' - Original was DESTROYED!
console.log(user === updated); // true - Same object!

// Problems:
// - Caller's data was unexpectedly modified
// - Can't compare old vs new (they're the same object)
// - React won't detect the change (same reference)
```

✅ **Good:**

```typescript
// Creating a new object with the updates
function updateUser(user: User, updates: Partial<User>): User {
  return {
    ...user,
    ...updates,
    updatedAt: new Date(),
  };
}

const user = { id: '1', name: 'Alice', email: 'alice@example.com', updatedAt: null };
const updated = updateUser(user, { name: 'Alicia' });

console.log(user.name);        // 'Alice' - Original preserved!
console.log(updated.name);     // 'Alicia' - New value in new object
console.log(user === updated); // false - Different objects

// Benefits:
// - Original data unchanged
// - Can compare old vs new easily
// - React detects the change (different reference)
```

### Example 2: Array Operations

❌ **Bad:**

```typescript
// Mutating arrays
function addTodo(todos: Todo[], todo: Todo): Todo[] {
  todos.push(todo); // Mutation!
  return todos;
}

function removeTodo(todos: Todo[], id: string): Todo[] {
  const index = todos.findIndex((t) => t.id === id);
  if (index !== -1) {
    todos.splice(index, 1); // Mutation!
  }
  return todos;
}

function updateTodo(todos: Todo[], id: string, updates: Partial<Todo>): Todo[] {
  const todo = todos.find((t) => t.id === id);
  if (todo) {
    Object.assign(todo, updates); // Mutation!
  }
  return todos;
}

// All these functions destroy the original array!
```

✅ **Good:**

```typescript
// Immutable array operations
function addTodo(todos: Todo[], todo: Todo): Todo[] {
  return [...todos, todo];
}

function removeTodo(todos: Todo[], id: string): Todo[] {
  return todos.filter((t) => t.id !== id);
}

function updateTodo(todos: Todo[], id: string, updates: Partial<Todo>): Todo[] {
  return todos.map((t) => (t.id === id ? { ...t, ...updates } : t));
}

// Insert at specific position
function insertTodo(todos: Todo[], index: number, todo: Todo): Todo[] {
  return [...todos.slice(0, index), todo, ...todos.slice(index)];
}

// Move item within array
function moveTodo(todos: Todo[], fromIndex: number, toIndex: number): Todo[] {
  const result = [...todos];
  const [removed] = result.splice(fromIndex, 1);
  result.splice(toIndex, 0, removed);
  return result;
  // Note: We created a copy first with [...todos], so original is safe
}

// Usage
const todos = [
  { id: '1', text: 'Learn React', done: false },
  { id: '2', text: 'Learn TypeScript', done: true },
];

const newTodos = addTodo(todos, { id: '3', text: 'Learn Immutability', done: false });

console.log(todos.length);    // 2 - Original unchanged
console.log(newTodos.length); // 3 - New array with new item
```

### Example 3: Nested Object Updates

❌ **Bad:**

```typescript
// Mutating nested objects
function updateUserAddress(user: User, city: string): User {
  user.address.city = city; // Deep mutation!
  return user;
}

const user = {
  name: 'Alice',
  address: { street: '123 Main St', city: 'Boston', zip: '02101' },
};

const updated = updateUserAddress(user, 'New York');

console.log(user.address.city); // 'New York' - Original was mutated!
```

✅ **Good:**

```typescript
// Immutable nested updates - spread at each level
function updateUserAddress(user: User, city: string): User {
  return {
    ...user,
    address: {
      ...user.address,
      city,
    },
  };
}

const user = {
  name: 'Alice',
  address: { street: '123 Main St', city: 'Boston', zip: '02101' },
};

const updated = updateUserAddress(user, 'New York');

console.log(user.address.city);    // 'Boston' - Original preserved!
console.log(updated.address.city); // 'New York' - New nested object

// For deeply nested updates, consider using Immer:
import { produce } from 'immer';

const updated = produce(user, (draft) => {
  draft.address.city = 'New York';
  // Looks like mutation, but Immer handles immutability!
});
```

### Example 4: React State Updates

❌ **Bad:**

```tsx
// Mutating React state directly
function TodoList() {
  const [todos, setTodos] = useState<Todo[]>([]);

  const addTodo = (text: string) => {
    todos.push({ id: Date.now().toString(), text, done: false }); // Mutation!
    setTodos(todos); // Same reference - React won't re-render!
  };

  const toggleTodo = (id: string) => {
    const todo = todos.find((t) => t.id === id);
    if (todo) {
      todo.done = !todo.done; // Mutation!
    }
    setTodos(todos); // Same reference - React won't re-render!
  };

  // This component will NOT update correctly!
}
```

✅ **Good:**

```tsx
// Proper immutable React state updates
function TodoList() {
  const [todos, setTodos] = useState<Todo[]>([]);

  const addTodo = (text: string) => {
    setTodos((prev) => [...prev, { id: Date.now().toString(), text, done: false }]);
  };

  const toggleTodo = (id: string) => {
    setTodos((prev) => prev.map((t) => (t.id === id ? { ...t, done: !t.done } : t)));
  };

  const removeTodo = (id: string) => {
    setTodos((prev) => prev.filter((t) => t.id !== id));
  };

  // React detects changes correctly because references change!
}

// Or with useReducer for complex state:
type Action =
  | { type: 'ADD'; text: string }
  | { type: 'TOGGLE'; id: string }
  | { type: 'REMOVE'; id: string };

function todoReducer(state: Todo[], action: Action): Todo[] {
  switch (action.type) {
    case 'ADD':
      return [...state, { id: Date.now().toString(), text: action.text, done: false }];
    case 'TOGGLE':
      return state.map((t) => (t.id === action.id ? { ...t, done: !t.done } : t));
    case 'REMOVE':
      return state.filter((t) => t.id !== action.id);
    default:
      return state;
  }
}
```

### Example 5: Redux-Style State Management

```typescript
// Immutable state updates in Redux pattern

type State = {
  users: Record<string, User>;
  posts: Post[];
  ui: {
    isLoading: boolean;
    selectedUserId: string | null;
  };
};

type Action =
  | { type: 'SET_LOADING'; payload: boolean }
  | { type: 'SELECT_USER'; payload: string }
  | { type: 'ADD_USER'; payload: User }
  | { type: 'UPDATE_USER'; payload: { id: string; updates: Partial<User> } }
  | { type: 'ADD_POST'; payload: Post }
  | { type: 'REMOVE_POST'; payload: string };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'SET_LOADING':
      return {
        ...state,
        ui: { ...state.ui, isLoading: action.payload },
      };

    case 'SELECT_USER':
      return {
        ...state,
        ui: { ...state.ui, selectedUserId: action.payload },
      };

    case 'ADD_USER':
      return {
        ...state,
        users: { ...state.users, [action.payload.id]: action.payload },
      };

    case 'UPDATE_USER':
      return {
        ...state,
        users: {
          ...state.users,
          [action.payload.id]: {
            ...state.users[action.payload.id],
            ...action.payload.updates,
          },
        },
      };

    case 'ADD_POST':
      return {
        ...state,
        posts: [...state.posts, action.payload],
      };

    case 'REMOVE_POST':
      return {
        ...state,
        posts: state.posts.filter((p) => p.id !== action.payload),
      };

    default:
      return state;
  }
}

// Every action returns a NEW state object
// Old state is never modified
// React/Redux can detect changes with simple reference checks
```

---

## 6. References

- [Immutable Object - Wikipedia](https://en.wikipedia.org/wiki/Immutable_object)
- [Immer.js](https://immerjs.github.io/immer/) - Write mutable code, get immutable updates
- Kyle Simpson - ["Functional-Light JavaScript"](https://github.com/getify/Functional-Light-JS)
- [React Docs: Updating Objects in State](https://react.dev/learn/updating-objects-in-state)
- [React Docs: Updating Arrays in State](https://react.dev/learn/updating-arrays-in-state)
