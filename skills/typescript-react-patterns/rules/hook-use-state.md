---
title: useState Typing
category: Hook Typing
priority: CRITICAL
---


Properly typing the useState hook for type-safe state management.

## Bad Example

```tsx
// TypeScript infers 'never[]' for empty array initialization
const [items, setItems] = useState([]);
// items is never[], can't add items later

// Initial value doesn't match intended type
const [user, setUser] = useState(null);
// user is always null, can't set to User object

// Using 'any' loses type safety
const [data, setData] = useState<any>({});

// Type assertion instead of proper typing
const [config, setConfig] = useState({} as Config);
// Dangerous - no runtime validation

// Overly complex union when simpler type would work
const [count, setCount] = useState<number | null | undefined>(0);
```

## Good Example

```tsx
import { useState } from 'react';

// Basic inference works for primitive initial values
const [count, setCount] = useState(0); // number
const [name, setName] = useState(''); // string
const [isActive, setIsActive] = useState(false); // boolean

// Explicit typing for empty arrays
interface User {
  id: string;
  name: string;
  email: string;
}

const [users, setUsers] = useState<User[]>([]);

// Add users with full type safety
setUsers([{ id: '1', name: 'John', email: 'john@example.com' }]);
setUsers((prev) => [...prev, { id: '2', name: 'Jane', email: 'jane@example.com' }]);

// Nullable state for data that may not exist yet
const [currentUser, setCurrentUser] = useState<User | null>(null);

// Check before accessing
if (currentUser) {
  console.log(currentUser.name); // TypeScript knows it's User here
}

// Union types for specific states
type LoadingState = 'idle' | 'loading' | 'success' | 'error';
const [status, setStatus] = useState<LoadingState>('idle');

// Complex object state with interface
interface FormState {
  values: {
    email: string;
    password: string;
  };
  errors: Record<string, string>;
  isSubmitting: boolean;
  isValid: boolean;
}

const initialFormState: FormState = {
  values: { email: '', password: '' },
  errors: {},
  isSubmitting: false,
  isValid: false,
};

const [formState, setFormState] = useState<FormState>(initialFormState);

// Partial updates with spread
setFormState((prev) => ({
  ...prev,
  values: { ...prev.values, email: 'new@email.com' },
}));

// Lazy initialization for expensive computations
interface AppConfig {
  theme: 'light' | 'dark';
  language: string;
  features: string[];
}

const [config, setConfig] = useState<AppConfig>(() => {
  // Only runs on initial render
  const saved = localStorage.getItem('config');
  if (saved) {
    return JSON.parse(saved) as AppConfig;
  }
  return {
    theme: 'light',
    language: 'en',
    features: [],
  };
});

// Discriminated union for state machines
type FetchState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };

const [fetchState, setFetchState] = useState<FetchState<User[]>>({ status: 'idle' });

// Type-safe state transitions
const fetchUsers = async () => {
  setFetchState({ status: 'loading' });
  try {
    const data = await api.getUsers();
    setFetchState({ status: 'success', data });
  } catch (error) {
    setFetchState({ status: 'error', error: error as Error });
  }
};

// Render based on state
switch (fetchState.status) {
  case 'idle':
    return <button onClick={fetchUsers}>Load Users</button>;
  case 'loading':
    return <Spinner />;
  case 'success':
    return <UserList users={fetchState.data} />; // data is User[]
  case 'error':
    return <Error message={fetchState.error.message} />; // error is Error
}

// Generic state hook pattern
function useToggle(initial: boolean = false): [boolean, () => void] {
  const [value, setValue] = useState(initial);
  const toggle = useCallback(() => setValue((v) => !v), []);
  return [value, toggle];
}

// Set state with callback receives correct type
const [items, setItems] = useState<string[]>([]);
setItems((prevItems) => {
  // prevItems is string[]
  return [...prevItems, 'new item'];
});
```

## Why

1. **Type inference**: Let TypeScript infer types for primitives with initial values
2. **Explicit generics**: Use explicit types for complex state, empty arrays, and nullable values
3. **Union types**: Enable exhaustive checking and proper type narrowing
4. **Lazy initialization**: Type the function return value for expensive initial computations
5. **Discriminated unions**: Model state machines with type-safe transitions
6. **Callback typing**: setState callbacks receive correctly typed previous state
