# Generic Hooks Typing

Creating flexible, reusable hooks with TypeScript generics.

## Bad Example

```tsx
// Using 'any' instead of generics
function useFetch(url: string): { data: any; loading: boolean } {
  const [data, setData] = useState<any>(null);
  // ...
  return { data, loading };
}

// Not constraining generic types when needed
function useList<T>(initial: T[]) {
  const [items, setItems] = useState(initial);

  const add = (item: T) => setItems([...items, item]);
  const remove = (item: T) => setItems(items.filter(i => i === item)); // Needs ID

  return { items, add, remove };
}

// Overly complex generics that are hard to understand
function useStore<
  T extends Record<string, unknown>,
  K extends keyof T,
  V extends T[K],
  A extends { type: string; payload: V }
>(initial: T): [T, (action: A) => void] {
  // Too many type parameters
}
```

## Good Example

```tsx
import { useState, useCallback, useEffect, useRef, useMemo } from 'react';

// Basic generic hook for data fetching
interface FetchState<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
}

interface UseFetchOptions<T> {
  initialData?: T;
  onSuccess?: (data: T) => void;
  onError?: (error: Error) => void;
  transform?: (raw: unknown) => T;
}

function useFetch<T>(
  url: string | null,
  options: UseFetchOptions<T> = {}
): FetchState<T> & { refetch: () => Promise<void> } {
  const { initialData = null, onSuccess, onError, transform } = options;

  const [state, setState] = useState<FetchState<T>>({
    data: initialData,
    loading: !!url,
    error: null,
  });

  const fetchData = useCallback(async () => {
    if (!url) {
      setState({ data: initialData, loading: false, error: null });
      return;
    }

    setState((prev) => ({ ...prev, loading: true, error: null }));

    try {
      const response = await fetch(url);
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      const raw = await response.json();
      const data = transform ? transform(raw) : (raw as T);

      setState({ data, loading: false, error: null });
      onSuccess?.(data);
    } catch (err) {
      const error = err instanceof Error ? err : new Error('Fetch failed');
      setState({ data: null, loading: false, error });
      onError?.(error);
    }
  }, [url, initialData, transform, onSuccess, onError]);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  return { ...state, refetch: fetchData };
}

// Usage with type inference
interface User {
  id: string;
  name: string;
  email: string;
}

const { data: user, loading, error } = useFetch<User>('/api/user/1');
// user is User | null

// Generic list management hook with proper constraints
interface Identifiable {
  id: string | number;
}

interface UseListActions<T> {
  add: (item: T) => void;
  remove: (id: T extends Identifiable ? T['id'] : number) => void;
  update: (id: T extends Identifiable ? T['id'] : number, updates: Partial<T>) => void;
  clear: () => void;
  set: (items: T[]) => void;
}

function useList<T extends Identifiable>(
  initialItems: T[] = []
): [T[], UseListActions<T>] {
  const [items, setItems] = useState<T[]>(initialItems);

  const actions: UseListActions<T> = useMemo(
    () => ({
      add: (item: T) => {
        setItems((prev) => [...prev, item]);
      },
      remove: (id) => {
        setItems((prev) => prev.filter((item) => item.id !== id));
      },
      update: (id, updates) => {
        setItems((prev) =>
          prev.map((item) => (item.id === id ? { ...item, ...updates } : item))
        );
      },
      clear: () => {
        setItems([]);
      },
      set: (newItems) => {
        setItems(newItems);
      },
    }),
    []
  );

  return [items, actions];
}

// Usage
interface Todo {
  id: string;
  text: string;
  completed: boolean;
}

const [todos, { add, remove, update }] = useList<Todo>([]);
add({ id: '1', text: 'Learn TypeScript', completed: false });
update('1', { completed: true });
remove('1');

// Generic form hook
type ValidationRule<T> = (value: T) => string | null;

interface UseFormOptions<T extends Record<string, unknown>> {
  initialValues: T;
  validationRules?: Partial<{ [K in keyof T]: ValidationRule<T[K]> }>;
  onSubmit: (values: T) => void | Promise<void>;
}

interface UseFormResult<T extends Record<string, unknown>> {
  values: T;
  errors: Partial<Record<keyof T, string>>;
  touched: Partial<Record<keyof T, boolean>>;
  isSubmitting: boolean;
  isValid: boolean;
  handleChange: <K extends keyof T>(field: K, value: T[K]) => void;
  handleBlur: <K extends keyof T>(field: K) => void;
  handleSubmit: (e?: React.FormEvent) => Promise<void>;
  reset: () => void;
  setFieldValue: <K extends keyof T>(field: K, value: T[K]) => void;
  setFieldError: <K extends keyof T>(field: K, error: string) => void;
}

function useForm<T extends Record<string, unknown>>({
  initialValues,
  validationRules = {},
  onSubmit,
}: UseFormOptions<T>): UseFormResult<T> {
  const [values, setValues] = useState<T>(initialValues);
  const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});
  const [touched, setTouched] = useState<Partial<Record<keyof T, boolean>>>({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const validateField = useCallback(
    <K extends keyof T>(field: K, value: T[K]): string | null => {
      const rule = validationRules[field];
      return rule ? rule(value) : null;
    },
    [validationRules]
  );

  const handleChange = useCallback(
    <K extends keyof T>(field: K, value: T[K]) => {
      setValues((prev) => ({ ...prev, [field]: value }));

      if (touched[field]) {
        const error = validateField(field, value);
        setErrors((prev) => ({ ...prev, [field]: error ?? undefined }));
      }
    },
    [touched, validateField]
  );

  const handleBlur = useCallback(
    <K extends keyof T>(field: K) => {
      setTouched((prev) => ({ ...prev, [field]: true }));

      const error = validateField(field, values[field]);
      setErrors((prev) => ({ ...prev, [field]: error ?? undefined }));
    },
    [values, validateField]
  );

  const validateAll = useCallback((): boolean => {
    const newErrors: Partial<Record<keyof T, string>> = {};
    let isValid = true;

    for (const key of Object.keys(values) as Array<keyof T>) {
      const error = validateField(key, values[key]);
      if (error) {
        newErrors[key] = error;
        isValid = false;
      }
    }

    setErrors(newErrors);
    return isValid;
  }, [values, validateField]);

  const handleSubmit = useCallback(
    async (e?: React.FormEvent) => {
      e?.preventDefault();

      if (!validateAll()) return;

      setIsSubmitting(true);
      try {
        await onSubmit(values);
      } finally {
        setIsSubmitting(false);
      }
    },
    [values, validateAll, onSubmit]
  );

  const reset = useCallback(() => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
  }, [initialValues]);

  const setFieldValue = useCallback(<K extends keyof T>(field: K, value: T[K]) => {
    setValues((prev) => ({ ...prev, [field]: value }));
  }, []);

  const setFieldError = useCallback(<K extends keyof T>(field: K, error: string) => {
    setErrors((prev) => ({ ...prev, [field]: error }));
  }, []);

  const isValid = Object.keys(errors).length === 0;

  return {
    values,
    errors,
    touched,
    isSubmitting,
    isValid,
    handleChange,
    handleBlur,
    handleSubmit,
    reset,
    setFieldValue,
    setFieldError,
  };
}

// Usage
interface LoginFormValues {
  email: string;
  password: string;
}

const form = useForm<LoginFormValues>({
  initialValues: { email: '', password: '' },
  validationRules: {
    email: (value) => (!value.includes('@') ? 'Invalid email' : null),
    password: (value) => (value.length < 8 ? 'Password too short' : null),
  },
  onSubmit: async (values) => {
    await api.login(values.email, values.password);
  },
});

// Generic selection hook
function useSelection<T extends Identifiable>() {
  const [selectedIds, setSelectedIds] = useState<Set<T['id']>>(new Set());

  const select = useCallback((id: T['id']) => {
    setSelectedIds((prev) => new Set(prev).add(id));
  }, []);

  const deselect = useCallback((id: T['id']) => {
    setSelectedIds((prev) => {
      const next = new Set(prev);
      next.delete(id);
      return next;
    });
  }, []);

  const toggle = useCallback((id: T['id']) => {
    setSelectedIds((prev) => {
      const next = new Set(prev);
      if (next.has(id)) {
        next.delete(id);
      } else {
        next.add(id);
      }
      return next;
    });
  }, []);

  const selectAll = useCallback((items: T[]) => {
    setSelectedIds(new Set(items.map((item) => item.id)));
  }, []);

  const deselectAll = useCallback(() => {
    setSelectedIds(new Set());
  }, []);

  const isSelected = useCallback(
    (id: T['id']) => selectedIds.has(id),
    [selectedIds]
  );

  return {
    selectedIds: Array.from(selectedIds),
    select,
    deselect,
    toggle,
    selectAll,
    deselectAll,
    isSelected,
    selectedCount: selectedIds.size,
  };
}

// Generic async state machine hook
type AsyncStatus = 'idle' | 'pending' | 'resolved' | 'rejected';

interface UseAsyncMachineResult<TData, TError = Error> {
  status: AsyncStatus;
  data: TData | null;
  error: TError | null;
  isIdle: boolean;
  isPending: boolean;
  isResolved: boolean;
  isRejected: boolean;
  resolve: (data: TData) => void;
  reject: (error: TError) => void;
  reset: () => void;
  start: () => void;
}

function useAsyncMachine<TData, TError = Error>(): UseAsyncMachineResult<TData, TError> {
  const [status, setStatus] = useState<AsyncStatus>('idle');
  const [data, setData] = useState<TData | null>(null);
  const [error, setError] = useState<TError | null>(null);

  const resolve = useCallback((newData: TData) => {
    setData(newData);
    setError(null);
    setStatus('resolved');
  }, []);

  const reject = useCallback((newError: TError) => {
    setError(newError);
    setData(null);
    setStatus('rejected');
  }, []);

  const reset = useCallback(() => {
    setData(null);
    setError(null);
    setStatus('idle');
  }, []);

  const start = useCallback(() => {
    setStatus('pending');
  }, []);

  return {
    status,
    data,
    error,
    isIdle: status === 'idle',
    isPending: status === 'pending',
    isResolved: status === 'resolved',
    isRejected: status === 'rejected',
    resolve,
    reject,
    reset,
    start,
  };
}
```

## Why

1. **Type safety without 'any'**: Generics preserve type information throughout the hook
2. **Constraints**: Use `extends` to limit generic types to specific shapes
3. **Inference**: Well-designed generics often don't need explicit type arguments
4. **Reusability**: Generic hooks work with any compatible type
5. **Conditional types**: Enable different behavior based on type structure
6. **Clear contracts**: Generic return types clearly communicate what hooks provide
