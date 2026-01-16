# useContext Typing

Properly typing Context and useContext for type-safe global state.

## Bad Example

```tsx
// Untyped context - no type safety
const AppContext = React.createContext(undefined);

// Using 'any' for context value
const ThemeContext = React.createContext<any>({ theme: 'light' });

// Default value that doesn't match actual usage
interface User {
  id: string;
  name: string;
}

const UserContext = React.createContext<User>({
  id: '',
  name: '',
}); // Fake default encourages using context outside provider

// Not handling missing provider
const AuthContext = React.createContext<AuthContextValue | undefined>(undefined);

function useAuth() {
  const context = React.useContext(AuthContext);
  return context; // Could be undefined, but callers don't know
}
```

## Good Example

```tsx
import React, { createContext, useContext, useState, useCallback, useMemo } from 'react';

// Pattern 1: Context with null default and custom hook
interface User {
  id: string;
  name: string;
  email: string;
}

interface AuthContextValue {
  user: User | null;
  isAuthenticated: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

const AuthContext = createContext<AuthContextValue | null>(null);
AuthContext.displayName = 'AuthContext';

// Custom hook with error handling
function useAuth(): AuthContextValue {
  const context = useContext(AuthContext);

  if (context === null) {
    throw new Error('useAuth must be used within an AuthProvider');
  }

  return context;
}

// Provider component
function AuthProvider({ children }: { children: React.ReactNode }): React.ReactElement {
  const [user, setUser] = useState<User | null>(null);

  const login = useCallback(async (email: string, password: string) => {
    const response = await api.login(email, password);
    setUser(response.user);
  }, []);

  const logout = useCallback(() => {
    setUser(null);
  }, []);

  const value = useMemo<AuthContextValue>(
    () => ({
      user,
      isAuthenticated: user !== null,
      login,
      logout,
    }),
    [user, login, logout]
  );

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

// Usage in component
function UserProfile(): React.ReactElement {
  const { user, logout } = useAuth(); // Guaranteed to be non-null

  if (!user) {
    return <div>Please log in</div>;
  }

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
      <button onClick={logout}>Logout</button>
    </div>
  );
}

// Pattern 2: Multiple contexts for separation of concerns
interface ThemeContextValue {
  theme: 'light' | 'dark';
  colors: {
    primary: string;
    background: string;
    text: string;
  };
}

interface ThemeActionsContextValue {
  setTheme: (theme: 'light' | 'dark') => void;
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextValue | null>(null);
const ThemeActionsContext = createContext<ThemeActionsContextValue | null>(null);

function useTheme(): ThemeContextValue {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}

function useThemeActions(): ThemeActionsContextValue {
  const context = useContext(ThemeActionsContext);
  if (!context) {
    throw new Error('useThemeActions must be used within ThemeProvider');
  }
  return context;
}

function ThemeProvider({ children }: { children: React.ReactNode }): React.ReactElement {
  const [theme, setThemeState] = useState<'light' | 'dark'>('light');

  const colors = useMemo(() => ({
    primary: theme === 'light' ? '#007bff' : '#6ea8fe',
    background: theme === 'light' ? '#ffffff' : '#212529',
    text: theme === 'light' ? '#212529' : '#ffffff',
  }), [theme]);

  const themeValue = useMemo<ThemeContextValue>(
    () => ({ theme, colors }),
    [theme, colors]
  );

  const actionsValue = useMemo<ThemeActionsContextValue>(
    () => ({
      setTheme: setThemeState,
      toggleTheme: () => setThemeState((t) => (t === 'light' ? 'dark' : 'light')),
    }),
    []
  );

  return (
    <ThemeContext.Provider value={themeValue}>
      <ThemeActionsContext.Provider value={actionsValue}>
        {children}
      </ThemeActionsContext.Provider>
    </ThemeContext.Provider>
  );
}

// Pattern 3: Generic context factory
function createSafeContext<T>(displayName: string) {
  const Context = createContext<T | null>(null);
  Context.displayName = displayName;

  function useContextSafe(): T {
    const context = useContext(Context);
    if (context === null) {
      throw new Error(`use${displayName} must be used within ${displayName}Provider`);
    }
    return context;
  }

  return [Context.Provider, useContextSafe] as const;
}

// Usage of factory
interface NotificationContextValue {
  notifications: Notification[];
  addNotification: (notification: Omit<Notification, 'id'>) => void;
  removeNotification: (id: string) => void;
}

const [NotificationProvider, useNotifications] = createSafeContext<NotificationContextValue>('Notification');

// Pattern 4: Context with reducer
interface CartState {
  items: CartItem[];
  total: number;
}

interface CartItem {
  id: string;
  name: string;
  price: number;
  quantity: number;
}

type CartAction =
  | { type: 'ADD_ITEM'; payload: Omit<CartItem, 'quantity'> }
  | { type: 'REMOVE_ITEM'; payload: string }
  | { type: 'UPDATE_QUANTITY'; payload: { id: string; quantity: number } }
  | { type: 'CLEAR' };

interface CartContextValue {
  state: CartState;
  dispatch: React.Dispatch<CartAction>;
  // Computed values and convenience methods
  itemCount: number;
  addItem: (item: Omit<CartItem, 'quantity'>) => void;
  removeItem: (id: string) => void;
}

const CartContext = createContext<CartContextValue | null>(null);

function cartReducer(state: CartState, action: CartAction): CartState {
  switch (action.type) {
    case 'ADD_ITEM': {
      const existing = state.items.find((i) => i.id === action.payload.id);
      if (existing) {
        return {
          ...state,
          items: state.items.map((i) =>
            i.id === action.payload.id ? { ...i, quantity: i.quantity + 1 } : i
          ),
          total: state.total + action.payload.price,
        };
      }
      return {
        ...state,
        items: [...state.items, { ...action.payload, quantity: 1 }],
        total: state.total + action.payload.price,
      };
    }
    case 'REMOVE_ITEM': {
      const item = state.items.find((i) => i.id === action.payload);
      if (!item) return state;
      return {
        ...state,
        items: state.items.filter((i) => i.id !== action.payload),
        total: state.total - item.price * item.quantity,
      };
    }
    case 'UPDATE_QUANTITY': {
      const item = state.items.find((i) => i.id === action.payload.id);
      if (!item) return state;
      const diff = action.payload.quantity - item.quantity;
      return {
        ...state,
        items: state.items.map((i) =>
          i.id === action.payload.id ? { ...i, quantity: action.payload.quantity } : i
        ),
        total: state.total + item.price * diff,
      };
    }
    case 'CLEAR':
      return { items: [], total: 0 };
    default:
      return state;
  }
}

function CartProvider({ children }: { children: React.ReactNode }): React.ReactElement {
  const [state, dispatch] = React.useReducer(cartReducer, { items: [], total: 0 });

  const value = useMemo<CartContextValue>(
    () => ({
      state,
      dispatch,
      itemCount: state.items.reduce((sum, item) => sum + item.quantity, 0),
      addItem: (item) => dispatch({ type: 'ADD_ITEM', payload: item }),
      removeItem: (id) => dispatch({ type: 'REMOVE_ITEM', payload: id }),
    }),
    [state]
  );

  return <CartContext.Provider value={value}>{children}</CartContext.Provider>;
}

function useCart(): CartContextValue {
  const context = useContext(CartContext);
  if (!context) {
    throw new Error('useCart must be used within CartProvider');
  }
  return context;
}
```

## Why

1. **Null default with error boundary**: Using `null` and throwing errors prevents silent failures
2. **Custom hooks**: Encapsulate context access and error handling
3. **Separated contexts**: Split state and actions to prevent unnecessary re-renders
4. **Factory pattern**: Reduces boilerplate for creating typed contexts
5. **Memoization**: Provider values should be memoized to prevent unnecessary re-renders
6. **DisplayName**: Improves debugging in React DevTools
