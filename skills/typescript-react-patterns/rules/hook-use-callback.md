# useCallback Typing

Properly typing useCallback for memoized function references.

## Bad Example

```tsx
// Missing dependency array type inference
const handleClick = useCallback((id) => {
  console.log(id); // id is implicitly 'any'
}, []);

// Incorrect return type inference
const fetchData = useCallback(async () => {
  const data = await api.getData();
  return data; // Return type not enforced
});

// Dependencies not aligned with closure usage
const [count, setCount] = useState(0);
const increment = useCallback(() => {
  setCount(count + 1); // Stale closure - count not in deps
}, []); // Missing count dependency

// Using 'any' for event parameter
const handleChange = useCallback((e: any) => {
  setValue(e.target.value);
}, []);
```

## Good Example

```tsx
import { useCallback, useState } from 'react';

// Basic callback with explicit parameter types
const handleClick = useCallback((id: string) => {
  console.log('Clicked:', id);
}, []);

// Callback with event typing
const handleInputChange = useCallback(
  (event: React.ChangeEvent<HTMLInputElement>) => {
    const value = event.target.value;
    console.log('Input changed:', value);
  },
  []
);

// Callback with return type
const calculateTotal = useCallback(
  (items: CartItem[]): number => {
    return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  },
  []
);

// Async callback with proper typing
interface User {
  id: string;
  name: string;
  email: string;
}

const fetchUser = useCallback(
  async (userId: string): Promise<User> => {
    const response = await fetch(`/api/users/${userId}`);
    if (!response.ok) {
      throw new Error('Failed to fetch user');
    }
    return response.json();
  },
  []
);

// Callback using state with proper dependencies
function Counter() {
  const [count, setCount] = useState(0);
  const [step, setStep] = useState(1);

  // Use functional update to avoid stale closure
  const increment = useCallback(() => {
    setCount((prevCount) => prevCount + step);
  }, [step]); // Only depends on step, count uses functional update

  // When you need the current value in the callback
  const logAndIncrement = useCallback(() => {
    console.log('Current count:', count);
    setCount((prevCount) => prevCount + 1);
  }, [count]); // count needed for console.log

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+{step}</button>
    </div>
  );
}

// Generic callback type
type AsyncCallback<TArgs extends unknown[], TReturn> = (
  ...args: TArgs
) => Promise<TReturn>;

// Callback passed to child with proper typing
interface ItemListProps {
  items: Item[];
  onItemSelect: (item: Item) => void;
  onItemDelete: (id: string) => Promise<void>;
}

function ItemContainer() {
  const [items, setItems] = useState<Item[]>([]);

  const handleSelect = useCallback((item: Item) => {
    console.log('Selected:', item.name);
  }, []);

  const handleDelete = useCallback(async (id: string): Promise<void> => {
    await api.deleteItem(id);
    setItems((prev) => prev.filter((item) => item.id !== id));
  }, []);

  return (
    <ItemList
      items={items}
      onItemSelect={handleSelect}
      onItemDelete={handleDelete}
    />
  );
}

// Callback with multiple parameters
interface FormData {
  name: string;
  email: string;
  message: string;
}

type FormField = keyof FormData;

const handleFieldChange = useCallback(
  (field: FormField, value: string) => {
    setFormData((prev) => ({ ...prev, [field]: value }));
  },
  []
);

// Callback returning cleanup function
const subscribeToUpdates = useCallback(
  (channel: string, onMessage: (msg: Message) => void): (() => void) => {
    const subscription = socket.subscribe(channel, onMessage);
    return () => subscription.unsubscribe();
  },
  []
);

// Callback with conditional logic and proper typing
type SortDirection = 'asc' | 'desc';
type SortField = 'name' | 'date' | 'price';

const handleSort = useCallback(
  (field: SortField, direction: SortDirection) => {
    setItems((prev) => {
      const sorted = [...prev].sort((a, b) => {
        const aVal = a[field];
        const bVal = b[field];
        const comparison = aVal < bVal ? -1 : aVal > bVal ? 1 : 0;
        return direction === 'asc' ? comparison : -comparison;
      });
      return sorted;
    });
  },
  []
);

// Debounced callback with types
function useDebouncedCallback<T extends (...args: Parameters<T>) => ReturnType<T>>(
  callback: T,
  delay: number
): T {
  const timeoutRef = useRef<ReturnType<typeof setTimeout>>();

  return useCallback(
    ((...args: Parameters<T>) => {
      if (timeoutRef.current) {
        clearTimeout(timeoutRef.current);
      }
      timeoutRef.current = setTimeout(() => {
        callback(...args);
      }, delay);
    }) as T,
    [callback, delay]
  );
}

// Usage
const debouncedSearch = useDebouncedCallback(
  (query: string) => {
    fetchSearchResults(query);
  },
  300
);

// Callback with external dependencies explicitly typed
interface ApiClient {
  get: <T>(url: string) => Promise<T>;
  post: <T>(url: string, data: unknown) => Promise<T>;
}

function useDataFetcher(apiClient: ApiClient) {
  const fetchData = useCallback(
    async <T>(endpoint: string): Promise<T> => {
      return apiClient.get<T>(endpoint);
    },
    [apiClient]
  );

  const postData = useCallback(
    async <T>(endpoint: string, data: unknown): Promise<T> => {
      return apiClient.post<T>(endpoint, data);
    },
    [apiClient]
  );

  return { fetchData, postData };
}

// Memoized callback for optimized child renders
const MemoizedChild = React.memo<{ onClick: () => void }>(({ onClick }) => {
  console.log('Child rendered');
  return <button onClick={onClick}>Click me</button>;
});

function Parent() {
  const [count, setCount] = useState(0);

  // Without useCallback, MemoizedChild would re-render on every Parent render
  const handleClick = useCallback(() => {
    setCount((c) => c + 1);
  }, []);

  return (
    <div>
      <p>Count: {count}</p>
      <MemoizedChild onClick={handleClick} />
    </div>
  );
}
```

## Why

1. **Type inference**: Explicit parameter types enable autocomplete and catch errors
2. **Return type safety**: Declaring return types ensures callbacks return expected values
3. **Proper dependencies**: Functional updates avoid stale closures without adding state to deps
4. **Event typing**: Proper React event types for form and DOM events
5. **Memoization stability**: Stable references prevent unnecessary child re-renders
6. **Generic callbacks**: Support for reusable typed callback patterns
