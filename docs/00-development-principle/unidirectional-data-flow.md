# Unidirectional Data Flow

---

## 1. Overview

**Unidirectional Data Flow** is an architectural pattern where data flows in a single, predictable direction through your application. Instead of data being modified from multiple places, changes follow a clear cycle: Action → State → View → Action. This makes state changes predictable and debugging straightforward.

**Key Idea:** Data flows ONE way. State is the single source of truth. Views read state and dispatch actions; they never modify state directly.

---

## 2. Why It Matters

### The Problem

When data can flow in multiple directions (bidirectional):

- State changes can come from anywhere, making bugs hard to trace
- Two-way bindings create circular dependencies and infinite loops
- Unpredictable update cascades - one change triggers many others
- Difficult to understand the current state of the application
- Race conditions when multiple sources try to update the same state
- "Who changed this?" becomes impossible to answer

### The Solution

Following Unidirectional Data Flow provides:

- **Single source of truth** - State lives in one place
- **Predictable changes** - All updates go through the same path
- **Easy debugging** - Can trace any state change back to its action
- **Time-travel debugging** - Can replay actions to reproduce bugs
- **Clear mental model** - Everyone understands how data moves

---

## 3. Core Concepts

### The Data Flow Cycle

```plaintext
┌─────────────────────────────────────────────────────────────────┐
│                 UNIDIRECTIONAL DATA FLOW CYCLE                   │
│                                                                  │
│                    ┌─────────────────┐                          │
│                    │                 │                          │
│            ┌───────│      STATE      │◀──────┐                  │
│            │       │ (Single Source  │       │                  │
│            │       │   of Truth)     │       │                  │
│            │       └─────────────────┘       │                  │
│            │                                 │                  │
│            ▼                                 │                  │
│   ┌─────────────────┐               ┌─────────────────┐         │
│   │                 │               │                 │         │
│   │      VIEW       │               │     REDUCER     │         │
│   │  (Renders UI    │               │ (Computes new   │         │
│   │   from state)   │               │  state from     │         │
│   │                 │               │  action)        │         │
│   └────────┬────────┘               └────────▲────────┘         │
│            │                                 │                  │
│            │        ┌─────────────────┐      │                  │
│            │        │                 │      │                  │
│            └───────▶│     ACTION      │──────┘                  │
│                     │ (Describes what │                         │
│                     │    happened)    │                         │
│                     └─────────────────┘                         │
│                                                                  │
│   1. State determines what View renders                         │
│   2. View can only dispatch Actions (not modify state)          │
│   3. Reducer processes Action and produces new State            │
│   4. New State triggers View re-render                          │
│   5. Repeat...                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Unidirectional vs Bidirectional

```plaintext
┌─────────────────────────────────────────────────────────────────┐
│              BIDIRECTIONAL (TWO-WAY BINDING)                     │
│                                                                  │
│              ┌───────────┐                                      │
│              │   Model   │                                      │
│              └─────┬─────┘                                      │
│                    │                                            │
│              ┌─────┴─────┐                                      │
│              │  ▲     │  │                                      │
│              │  │     ▼  │                                      │
│              └───────────┘                                      │
│              ┌───────────┐                                      │
│              │   View    │                                      │
│              └───────────┘                                      │
│                                                                  │
│   ❌ Model can update View                                      │
│   ❌ View can directly update Model                             │
│   ❌ Multiple components can update same Model                  │
│   ❌ Circular updates are possible                              │
│   ❌ "Who changed what?" is hard to answer                      │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│              UNIDIRECTIONAL (ONE-WAY FLOW)                       │
│                                                                  │
│              ┌───────────┐                                      │
│              │   State   │────────────┐                         │
│              └───────────┘            │                         │
│                    ▲                  ▼                         │
│                    │            ┌───────────┐                   │
│              ┌───────────┐      │   View    │                   │
│              │  Reducer  │      └─────┬─────┘                   │
│              └─────▲─────┘            │                         │
│                    │                  ▼                         │
│              ┌─────┴─────┐      ┌───────────┐                   │
│              │  Action   │◀─────│ dispatch  │                   │
│              └───────────┘      └───────────┘                   │
│                                                                  │
│   ✅ State flows DOWN to View                                   │
│   ✅ Actions flow UP from View                                  │
│   ✅ Reducer is the ONLY way to change state                    │
│   ✅ Every change is traceable                                  │
│   ✅ Predictable, debuggable                                    │
└─────────────────────────────────────────────────────────────────┘
```

### The Three Principles

```plaintext
┌─────────────────────────────────────────────────────────────────┐
│              THREE PRINCIPLES OF UNIDIRECTIONAL FLOW             │
│                                                                  │
│   1. SINGLE SOURCE OF TRUTH                                     │
│   ────────────────────────                                      │
│   All application state lives in ONE store.                     │
│   There's only ONE place to look for current state.             │
│                                                                  │
│   2. STATE IS READ-ONLY                                         │
│   ────────────────────                                          │
│   Views can READ state but never MODIFY it directly.            │
│   The only way to change state is to dispatch an action.        │
│                                                                  │
│   3. CHANGES ARE MADE WITH PURE FUNCTIONS                       │
│   ──────────────────────────────────────                        │
│   Reducers are pure: (state, action) => newState                │
│   No side effects, predictable, testable.                       │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms

| Term | Definition |
| --- | --- |
| Action | A plain object describing what happened (e.g., `{ type: 'ADD_TODO', text: 'Buy milk' }`) |
| Reducer | A pure function that takes current state and action, returns new state |
| Store | Container holding the application state tree |
| Dispatch | The function used to send actions to the reducer |
| State | The single source of truth for your application data |

---

## 4. Workflow & Checklist

### Recommended Workflow

```plaintext
1. USER            →   2. DISPATCH        →   3. REDUCE         →   4. RENDER
   Interacts            Action                 New State             View

   "User clicks        "dispatch({           "reducer(state,      "View re-renders
    a button"           type: 'CLICK' })"      action) → new"       with new state"
```

### Checklist

Before implementing state management, verify:

- [ ] **SINGLE STORE:** Is there ONE source of truth for this state?
- [ ] **READ-ONLY:** Do views only READ state, never modify directly?
- [ ] **ACTIONS:** Are all state changes triggered by dispatched actions?
- [ ] **PURE REDUCERS:** Are reducers pure functions without side effects?
- [ ] **TRACEABLE:** Can I trace any state change back to its action?

---

## 5. Examples

### Example 1: Two-Way Binding Problems

❌ **Bad:**

```typescript
// Two-way binding - state can be modified from multiple places
class UserFormController {
  user = { name: '', email: '' };

  // View can directly mutate state
  template = `
    <input [(ngModel)]="user.name">
    <input [(ngModel)]="user.email">
  `;

  // API response can also mutate state
  async loadUser(id: string) {
    this.user = await fetchUser(id);  // Direct mutation
  }

  // Other components can mutate too
  updateFromExternalSource(name: string) {
    this.user.name = name;  // Direct mutation
  }

  // Even validation might mutate
  validateAndFix() {
    if (!this.user.email.includes('@')) {
      this.user.email = '';  // Direct mutation
    }
  }
}

// Problems:
// - State can change from 4+ different places
// - No way to track what caused a change
// - Circular updates can occur
// - Race conditions between API and user input
```

✅ **Good:**

```typescript
// Unidirectional - all changes go through actions and reducer
type UserState = {
  name: string;
  email: string;
  isLoading: boolean;
  error: string | null;
};

type UserAction =
  | { type: 'SET_NAME'; payload: string }
  | { type: 'SET_EMAIL'; payload: string }
  | { type: 'LOAD_USER_START' }
  | { type: 'LOAD_USER_SUCCESS'; payload: User }
  | { type: 'LOAD_USER_ERROR'; payload: string }
  | { type: 'CLEAR_EMAIL' };

function userReducer(state: UserState, action: UserAction): UserState {
  switch (action.type) {
    case 'SET_NAME':
      return { ...state, name: action.payload };
    case 'SET_EMAIL':
      return { ...state, email: action.payload };
    case 'LOAD_USER_START':
      return { ...state, isLoading: true, error: null };
    case 'LOAD_USER_SUCCESS':
      return {
        ...state,
        isLoading: false,
        name: action.payload.name,
        email: action.payload.email,
      };
    case 'LOAD_USER_ERROR':
      return { ...state, isLoading: false, error: action.payload };
    case 'CLEAR_EMAIL':
      return { ...state, email: '' };
    default:
      return state;
  }
}

// React component - only dispatches actions
function UserForm() {
  const [state, dispatch] = useReducer(userReducer, initialState);

  return (
    <form>
      <input
        value={state.name}
        onChange={(e) => dispatch({ type: 'SET_NAME', payload: e.target.value })}
      />
      <input
        value={state.email}
        onChange={(e) => dispatch({ type: 'SET_EMAIL', payload: e.target.value })}
      />
    </form>
  );
}

// Benefits:
// - ALL changes go through dispatch → reducer
// - Can log every action to see what happened
// - Easy to trace: "email changed because SET_EMAIL was dispatched"
// - Reducer is pure, testable
```

### Example 2: Child-to-Parent Communication

❌ **Bad:**

```tsx
// Child directly mutating parent state
function Child({ parentState }: { parentState: { count: number } }) {
  const handleClick = () => {
    parentState.count += 1;  // Direct mutation of parent state!
  };

  return <button onClick={handleClick}>Add</button>;
}

function Parent() {
  const [state] = useState({ count: 0 });

  // State might change without Parent knowing!
  return (
    <div>
      <p>Count: {state.count}</p>
      <Child parentState={state} />
    </div>
  );
}

// Problems:
// - Parent doesn't control its own state
// - Mutations are invisible to React
// - Component won't re-render correctly
```

✅ **Good:**

```tsx
// Child communicates via callbacks (actions)
function Child({ onIncrement }: { onIncrement: () => void }) {
  return <button onClick={onIncrement}>Add</button>;
}

function Parent() {
  const [count, setCount] = useState(0);

  // Parent controls all state changes
  const handleIncrement = () => setCount((c) => c + 1);

  return (
    <div>
      <p>Count: {count}</p>
      <Child onIncrement={handleIncrement} />
    </div>
  );
}

// Data flows DOWN (count)
// Events flow UP (onIncrement callback)
// State changes happen in ONE place (Parent)
```

### Example 3: Complex State with useReducer

```tsx
// Full example of unidirectional flow in React

// 1. Define State Shape
type TodoState = {
  todos: Todo[];
  filter: 'all' | 'active' | 'completed';
  editingId: string | null;
};

// 2. Define Actions (all possible state changes)
type TodoAction =
  | { type: 'ADD_TODO'; payload: string }
  | { type: 'TOGGLE_TODO'; payload: string }
  | { type: 'DELETE_TODO'; payload: string }
  | { type: 'EDIT_TODO'; payload: { id: string; text: string } }
  | { type: 'SET_FILTER'; payload: 'all' | 'active' | 'completed' }
  | { type: 'START_EDITING'; payload: string }
  | { type: 'STOP_EDITING' }
  | { type: 'CLEAR_COMPLETED' };

// 3. Reducer (pure function - single place where state changes)
function todoReducer(state: TodoState, action: TodoAction): TodoState {
  switch (action.type) {
    case 'ADD_TODO':
      return {
        ...state,
        todos: [
          ...state.todos,
          { id: crypto.randomUUID(), text: action.payload, completed: false },
        ],
      };

    case 'TOGGLE_TODO':
      return {
        ...state,
        todos: state.todos.map((todo) =>
          todo.id === action.payload ? { ...todo, completed: !todo.completed } : todo
        ),
      };

    case 'DELETE_TODO':
      return {
        ...state,
        todos: state.todos.filter((todo) => todo.id !== action.payload),
      };

    case 'EDIT_TODO':
      return {
        ...state,
        todos: state.todos.map((todo) =>
          todo.id === action.payload.id ? { ...todo, text: action.payload.text } : todo
        ),
        editingId: null,
      };

    case 'SET_FILTER':
      return { ...state, filter: action.payload };

    case 'START_EDITING':
      return { ...state, editingId: action.payload };

    case 'STOP_EDITING':
      return { ...state, editingId: null };

    case 'CLEAR_COMPLETED':
      return {
        ...state,
        todos: state.todos.filter((todo) => !todo.completed),
      };

    default:
      return state;
  }
}

// 4. Components only read state and dispatch actions
function TodoApp() {
  const [state, dispatch] = useReducer(todoReducer, initialState);

  // Derived state (computed from source of truth)
  const visibleTodos = useMemo(() => {
    switch (state.filter) {
      case 'active':
        return state.todos.filter((t) => !t.completed);
      case 'completed':
        return state.todos.filter((t) => t.completed);
      default:
        return state.todos;
    }
  }, [state.todos, state.filter]);

  return (
    <div>
      <TodoInput onAdd={(text) => dispatch({ type: 'ADD_TODO', payload: text })} />

      <TodoList
        todos={visibleTodos}
        editingId={state.editingId}
        onToggle={(id) => dispatch({ type: 'TOGGLE_TODO', payload: id })}
        onDelete={(id) => dispatch({ type: 'DELETE_TODO', payload: id })}
        onStartEdit={(id) => dispatch({ type: 'START_EDITING', payload: id })}
        onSaveEdit={(id, text) => dispatch({ type: 'EDIT_TODO', payload: { id, text } })}
        onCancelEdit={() => dispatch({ type: 'STOP_EDITING' })}
      />

      <TodoFilters
        current={state.filter}
        onChange={(filter) => dispatch({ type: 'SET_FILTER', payload: filter })}
        onClearCompleted={() => dispatch({ type: 'CLEAR_COMPLETED' })}
      />
    </div>
  );
}

// 5. Child components are "dumb" - just render and call callbacks
function TodoList({ todos, onToggle, onDelete, ...props }: TodoListProps) {
  return (
    <ul>
      {todos.map((todo) => (
        <TodoItem
          key={todo.id}
          todo={todo}
          onToggle={() => onToggle(todo.id)}
          onDelete={() => onDelete(todo.id)}
          {...props}
        />
      ))}
    </ul>
  );
}

// Benefits of this pattern:
// ✅ All state in one place (reducer)
// ✅ All changes are explicit (actions)
// ✅ Easy to add logging: log every action
// ✅ Easy to add undo: store previous states
// ✅ Easy to test: reducer is pure function
// ✅ Easy to debug: replay actions to reproduce bugs
```

### Example 4: Debugging with Action Logging

```typescript
// One of the biggest benefits of unidirectional flow: debugging

// Middleware to log all actions
function createLoggingReducer<S, A>(reducer: (state: S, action: A) => S) {
  return (state: S, action: A): S => {
    console.group(`Action: ${(action as any).type}`);
    console.log('Previous State:', state);
    console.log('Action:', action);

    const newState = reducer(state, action);

    console.log('New State:', newState);
    console.groupEnd();

    return newState;
  };
}

// Now you can see EXACTLY what happened:
// Action: ADD_TODO
//   Previous State: { todos: [] }
//   Action: { type: 'ADD_TODO', payload: 'Buy milk' }
//   New State: { todos: [{ id: '123', text: 'Buy milk', completed: false }] }

// Time-travel debugging
const stateHistory: State[] = [];

function createHistoryReducer<S, A>(reducer: (state: S, action: A) => S) {
  return (state: S, action: A): S => {
    const newState = reducer(state, action);
    stateHistory.push(newState);
    return newState;
  };
}

// Now you can:
// - Go back to any previous state
// - Replay actions to reproduce bugs
// - Export action history for bug reports
```

---

## 6. References

- [Flux Architecture](https://facebook.github.io/flux/) - Original pattern from Facebook
- [Redux Documentation](https://redux.js.org/) - Most popular implementation
- [React useReducer](https://react.dev/reference/react/useReducer) - Built-in React hook
- Dan Abramov - ["You Might Not Need Redux"](https://medium.com/@dan_abramov/you-might-not-need-redux-be46360cf367)
