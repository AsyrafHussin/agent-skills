# ctx-create

**Priority:** MEDIUM
**Category:** Context & State

## Why It Matters

Context requires proper typing for both the value and the default. A common pattern is using `null` as default with a custom hook that throws if used outside the provider.

## Incorrect

```typescript
// ❌ No typing - any is inferred
const AuthContext = createContext(undefined)

// ❌ Unsafe access - might be undefined
const AuthContext = createContext<AuthContextType | undefined>(undefined)
const auth = useContext(AuthContext)
auth.user  // Error: possibly undefined

// ❌ Fake default value - misleading
const AuthContext = createContext<AuthContextType>({
  user: null,
  login: () => {},  // Fake implementation
  logout: () => {},
})
```

## Correct

### Null Default with Custom Hook

```typescript
// ✅ Proper pattern: null default + custom hook
interface User {
  id: number
  name: string
  email: string
}

interface AuthContextType {
  user: User | null
  login: (email: string, password: string) => Promise<void>
  logout: () => void
  isLoading: boolean
}

// Create context with null - indicates "not yet provided"
const AuthContext = createContext<AuthContextType | null>(null)

// Custom hook with runtime check
export function useAuth(): AuthContextType {
  const context = useContext(AuthContext)

  if (context === null) {
    throw new Error('useAuth must be used within an AuthProvider')
  }

  return context  // TypeScript knows this is AuthContextType, not null
}

// Provider component
export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null)
  const [isLoading, setIsLoading] = useState(true)

  const login = async (email: string, password: string) => {
    setIsLoading(true)
    try {
      const user = await authApi.login(email, password)
      setUser(user)
    } finally {
      setIsLoading(false)
    }
  }

  const logout = () => {
    setUser(null)
  }

  return (
    <AuthContext.Provider value={{ user, login, logout, isLoading }}>
      {children}
    </AuthContext.Provider>
  )
}
```

### Usage

```typescript
// ✅ Safe usage in components
function UserProfile() {
  const { user, logout } = useAuth()  // Typed, never null

  if (!user) {
    return <Navigate to="/login" />
  }

  return (
    <div>
      <h1>{user.name}</h1>
      <button onClick={logout}>Logout</button>
    </div>
  )
}

// ✅ App setup
function App() {
  return (
    <AuthProvider>
      <Router>
        <Routes>
          <Route path="/profile" element={<UserProfile />} />
        </Routes>
      </Router>
    </AuthProvider>
  )
}
```

### Multiple Contexts Pattern

```typescript
// theme-context.tsx
interface ThemeContextType {
  theme: 'light' | 'dark'
  toggleTheme: () => void
}

const ThemeContext = createContext<ThemeContextType | null>(null)

export function useTheme() {
  const context = useContext(ThemeContext)
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider')
  }
  return context
}

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<'light' | 'dark'>('light')

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light')
  }

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  )
}
```

### Context with Reducer

```typescript
interface State {
  count: number
  step: number
}

type Action =
  | { type: 'INCREMENT' }
  | { type: 'DECREMENT' }
  | { type: 'SET_STEP'; payload: number }

interface CounterContextType {
  state: State
  dispatch: React.Dispatch<Action>
}

const CounterContext = createContext<CounterContextType | null>(null)

function counterReducer(state: State, action: Action): State {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + state.step }
    case 'DECREMENT':
      return { ...state, count: state.count - state.step }
    case 'SET_STEP':
      return { ...state, step: action.payload }
    default:
      return state
  }
}

export function useCounter() {
  const context = useContext(CounterContext)
  if (!context) {
    throw new Error('useCounter must be used within CounterProvider')
  }
  return context
}

export function CounterProvider({ children }: { children: React.ReactNode }) {
  const [state, dispatch] = useReducer(counterReducer, {
    count: 0,
    step: 1,
  })

  return (
    <CounterContext.Provider value={{ state, dispatch }}>
      {children}
    </CounterContext.Provider>
  )
}
```

## Pattern Benefits

- **Type safety**: Context value is properly typed
- **Runtime safety**: Hook throws clear error if used incorrectly
- **No fake defaults**: Null indicates "not provided" clearly
- **Clean API**: Custom hook provides nice interface
