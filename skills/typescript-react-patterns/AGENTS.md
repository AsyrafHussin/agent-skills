# TypeScript React Patterns

**Version 1.0.0**
TypeScript + React Best Practices
January 2026

> **Note:**
> This document is primarily for AI agents and LLMs to follow when writing,
> maintaining, or refactoring TypeScript + React codebases. Developers
> may also find it useful, but guidance here is optimized for automation
> and consistency in AI-assisted workflows.

---

## Abstract

Comprehensive TypeScript patterns for React development, designed for AI agents and developers. Contains 25+ rules across 7 categories covering component typing, hook patterns, event handling, refs, generics, context, and utility types. Each rule includes detailed type annotations, incorrect vs correct examples, and practical usage patterns to ensure type-safe, maintainable React codebases.

---

## Table of Contents

1. [Component Typing](#1-component-typing) — **CRITICAL**
   - 1.1 [Component Props Interface](#11-component-props-interface)
   - 1.2 [Component Children Types](#12-component-children-types)
   - 1.3 [Props Interface Definition](#13-props-interface-definition)
   - 1.4 [Children Prop Typing](#14-children-prop-typing)
   - 1.5 [Rest Props Typing](#15-rest-props-typing)
   - 1.6 [FC vs Function Declaration](#16-fc-vs-function-declaration)
   - 1.7 [Default Props Typing](#17-default-props-typing)

2. [Hook Typing](#2-hook-typing) — **CRITICAL**
   - 2.1 [useState Hook Typing](#21-usestate-hook-typing)
   - 2.2 [useRef Hook Typing](#22-useref-hook-typing)
   - 2.3 [Custom Hooks Typing](#23-custom-hooks-typing)
   - 2.4 [useReducer Typing](#24-usereducer-typing)
   - 2.5 [Generic Hooks Typing](#25-generic-hooks-typing)
   - 2.6 [useCallback Typing](#26-usecallback-typing)
   - 2.7 [useMemo Typing](#27-usememo-typing)
   - 2.8 [useContext Typing](#28-usecontext-typing)

3. [Event Handling](#3-event-handling) — **HIGH**
   - 3.1 [Event Handler Types](#31-event-handler-types)
   - 3.2 [Click Event Handler Typing](#32-click-event-handler-typing)

4. [Ref Typing](#4-ref-typing) — **HIGH**
   - 4.1 [ForwardRef Typing](#41-forwardref-typing)

5. [Generic Components](#5-generic-components) — **MEDIUM**
   - 5.1 [Generic List Component](#51-generic-list-component)
   - 5.2 [Polymorphic Component Typing](#52-polymorphic-component-typing)

6. [Context & State](#6-context--state) — **MEDIUM**
   - 6.1 [Create Typed Context](#61-create-typed-context)

7. [Utility Types](#7-utility-types) — **LOW**
   - 7.1 [Display Name Pattern](#71-display-name-pattern)

---

## 1. Component Typing

**Impact: CRITICAL**

Fundamental patterns for typing React component props. Every component needs proper prop typing for type safety and developer experience.

### 1.1 Component Props Interface

**Impact: CRITICAL (foundation for all components)**

Use `interface` for component props, `type` for unions. Interfaces are extendable and provide better error messages.

**Incorrect: using type when interface is better**

```typescript
// ❌ Using type limits extensibility
type ButtonProps = {
  label: string
  onClick: () => void
}

// ❌ Inline types - not reusable
function Button({ label, onClick }: { label: string; onClick: () => void }) {
  return <button onClick={onClick}>{label}</button>
}
```

**Correct: interface for props**

```typescript
// ✅ Interface for component props
interface ButtonProps {
  label: string
  onClick: () => void
  disabled?: boolean
}

function Button({ label, onClick, disabled = false }: ButtonProps) {
  return (
    <button onClick={onClick} disabled={disabled}>
      {label}
    </button>
  )
}

// ✅ Extending interfaces
interface IconButtonProps extends ButtonProps {
  icon: React.ReactNode
  iconPosition?: 'left' | 'right'
}
```

**When to use `type`:**

```typescript
// ✅ Use type for unions
type ButtonVariant = 'primary' | 'secondary' | 'danger'
type Size = 'sm' | 'md' | 'lg'

// ✅ Use type for intersection with HTML props
type PropsWithRequiredChildren = {
  children: React.ReactNode
} & React.HTMLAttributes<HTMLDivElement>
```

### 1.2 Component Children Types

**Impact: CRITICAL (most commonly used prop)**

Choose the right children type: `ReactNode` for flexibility, `ReactElement` for element-specific needs.

**Incorrect: wrong children type**

```typescript
// ❌ Too restrictive - won't accept strings or numbers
interface CardProps {
  children: React.ReactElement
}

<Card>Hello</Card>  // Error: string is not ReactElement
```

**Correct: ReactNode for most cases**

```typescript
// ✅ Accepts anything React can render
interface CardProps {
  title: string
  children: React.ReactNode
}

function Card({ title, children }: CardProps) {
  return (
    <div className="card">
      <h2>{title}</h2>
      {children}
    </div>
  )
}

// All valid:
<Card title="Welcome">Hello</Card>
<Card title="Count">{42}</Card>
<Card title="User"><UserProfile /></Card>
```

**ReactElement for element-specific needs:**

```typescript
// ✅ When you need to access element props
interface TabsProps {
  children: React.ReactElement<TabProps> | React.ReactElement<TabProps>[]
}

function Tabs({ children }: TabsProps) {
  const tabs = React.Children.toArray(children) as React.ReactElement<TabProps>[]

  return (
    <div>
      {tabs.map((tab, i) => (
        <button key={i}>{tab.props.label}</button>
      ))}
    </div>
  )
}
```

**Render props pattern:**

```typescript
// ✅ Function as children
interface DataFetcherProps<T> {
  url: string
  children: (data: T, loading: boolean, error: Error | null) => React.ReactNode
}

<DataFetcher<User[]> url="/api/users">
  {(users, loading, error) => {
    if (loading) return <Spinner />
    if (error) return <Error message={error.message} />
    return <UserList users={users} />
  }}
</DataFetcher>
```

---

## 2. Hook Typing

**Impact: CRITICAL**

Essential patterns for typing React hooks. Proper hook typing prevents runtime errors and provides better IDE support.

### 2.1 useState Hook Typing

**Impact: CRITICAL (most frequently used hook)**

Explicitly type useState when the initial value doesn't represent all possible states.

**Incorrect: type inference issues**

```typescript
// ❌ Inferred as null, can't set to User
const [user, setUser] = useState(null)
setUser({ id: 1, name: 'John' })  // Error!

// ❌ Inferred as never[] - can't add typed items
const [items, setItems] = useState([])
setItems([{ id: 1 }])  // Error!
```

**Correct: explicit union types**

```typescript
// ✅ Explicit union type for nullable state
interface User {
  id: number
  name: string
  email: string
}

const [user, setUser] = useState<User | null>(null)

// Both work:
setUser({ id: 1, name: 'John', email: 'john@example.com' })
setUser(null)

// ✅ Typed array
interface Todo {
  id: number
  text: string
  done: boolean
}

const [todos, setTodos] = useState<Todo[]>([])
```

**Discriminated unions for state machines:**

```typescript
// ✅ Type-safe state machine
type RequestState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error }

const [state, setState] = useState<RequestState<User>>({ status: 'idle' })

switch (state.status) {
  case 'success':
    return <UserCard user={state.data} />  // data is typed!
  case 'error':
    return <Error message={state.error.message} />
}
```

### 2.2 useRef Hook Typing

**Impact: CRITICAL (DOM access and mutable values)**

useRef has two patterns: DOM refs (nullable) and mutable values (non-nullable).

**Incorrect: wrong ref typing**

```typescript
// ❌ Missing element type
const inputRef = useRef(null)
inputRef.current.focus()  // Error: possibly null

// ❌ Wrong initial value for DOM ref
const inputRef = useRef<HTMLInputElement>()  // undefined, not null
```

**Correct: DOM element refs**

```typescript
// ✅ DOM ref - pass null, type the element
const inputRef = useRef<HTMLInputElement>(null)
const buttonRef = useRef<HTMLButtonElement>(null)
const divRef = useRef<HTMLDivElement>(null)

function Form() {
  const inputRef = useRef<HTMLInputElement>(null)

  const focusInput = () => {
    inputRef.current?.focus()  // Optional chaining
  }

  return <input ref={inputRef} />
}
```

**Mutable value refs:**

```typescript
// ✅ Mutable ref - non-null initial value
function Timer() {
  const intervalRef = useRef<number | undefined>(undefined)
  const countRef = useRef(0)  // Inferred as MutableRefObject<number>

  useEffect(() => {
    intervalRef.current = window.setInterval(() => {
      countRef.current++  // No null check needed
    }, 1000)

    return () => clearInterval(intervalRef.current)
  }, [])
}
```

### 2.3 Custom Hooks Typing

**Impact: CRITICAL (reusable logic)**

Type custom hook parameters and return values explicitly for clarity and type safety.

**Correct: explicit types**

```typescript
// ✅ Clear parameter and return types
interface UseUserOptions {
  onError?: (error: Error) => void
  retry?: number
}

interface UseUserReturn {
  user: User | null
  loading: boolean
  error: Error | null
  refetch: () => Promise<void>
}

function useUser(id: string, options?: UseUserOptions): UseUserReturn {
  const [user, setUser] = useState<User | null>(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState<Error | null>(null)

  const refetch = async () => {
    // implementation
  }

  return { user, loading, error, refetch }
}
```

---

## 3. Event Handling

**Impact: HIGH**

Correct event types provide autocomplete and catch property access errors.

### 3.1 Event Handler Types

**Impact: HIGH (every interactive component)**

Use `React.MouseEvent<T>`, `React.ChangeEvent<T>`, etc. with proper element types.

**Pattern: `React.[EventType]<[ElementType]>`**

**Incorrect: wrong event types**

```typescript
// ❌ Using native Event instead of React event
const handleClick = (e: Event) => {
  // Missing React-specific properties
}

// ❌ Missing element type
const handleChange = (e: React.ChangeEvent) => {
  e.target.value  // Error: Property 'value' does not exist
}
```

**Correct: proper event types**

```typescript
// ✅ Mouse events
const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {
  console.log(e.clientX, e.clientY)
  console.log(e.currentTarget.disabled)
}

// ✅ Form events
const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
  e.preventDefault()
  const formData = new FormData(e.currentTarget)
}

const handleInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  console.log(e.target.value)
  console.log(e.target.checked)  // For checkboxes
}

// ✅ Keyboard events
const handleKeyDown = (e: React.KeyboardEvent<HTMLInputElement>) => {
  if (e.key === 'Enter') {
    e.preventDefault()
  }
  console.log(e.ctrlKey, e.shiftKey, e.altKey)
}
```

**Event handler props:**

```typescript
// ✅ Typing event handler props
interface ButtonProps {
  onClick?: React.MouseEventHandler<HTMLButtonElement>
  onFocus?: React.FocusEventHandler<HTMLButtonElement>
}

// Equivalent to:
interface ButtonProps {
  onClick?: (e: React.MouseEvent<HTMLButtonElement>) => void
  onFocus?: (e: React.FocusEvent<HTMLButtonElement>) => void
}
```

---

## 4. Ref Typing

**Impact: HIGH**

Proper ref typing is critical for DOM manipulation and imperative handles.

### 4.1 ForwardRef Typing

**Impact: HIGH (reusable components)**

Type forwardRef with proper element types for components that expose DOM refs.

**Correct: typed forwardRef**

```typescript
// ✅ forwardRef with proper types
interface InputProps {
  label: string
  error?: string
}

const Input = forwardRef<HTMLInputElement, InputProps>(
  ({ label, error }, ref) => {
    return (
      <div>
        <label>{label}</label>
        <input ref={ref} />
        {error && <span>{error}</span>}
      </div>
    )
  }
)

Input.displayName = 'Input'

// Usage
function Form() {
  const inputRef = useRef<HTMLInputElement>(null)

  return <Input ref={inputRef} label="Email" />
}
```

---

## 5. Generic Components

**Impact: MEDIUM**

Type-safe reusable components that work with any data type.

### 5.1 Generic List Component

**Impact: MEDIUM (common pattern)**

Use generics to create type-safe list components.

**Correct: generic list**

```typescript
// ✅ Generic list component
interface ListProps<T> {
  items: T[]
  renderItem: (item: T, index: number) => React.ReactNode
  keyExtractor: (item: T) => string | number
}

function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={keyExtractor(item)}>
          {renderItem(item, index)}
        </li>
      ))}
    </ul>
  )
}

// Usage - T is inferred
<List
  items={users}
  renderItem={(user) => <UserCard user={user} />}
  keyExtractor={(user) => user.id}
/>
```

---

## 6. Context & State

**Impact: MEDIUM**

Type-safe global state with context.

### 6.1 Create Typed Context

**Impact: MEDIUM (common pattern)**

Use null default with custom hook that throws if used outside provider.

**Correct: typed context pattern**

```typescript
// ✅ Null default + custom hook
interface AuthContextType {
  user: User | null
  login: (email: string, password: string) => Promise<void>
  logout: () => void
  isLoading: boolean
}

const AuthContext = createContext<AuthContextType | null>(null)

export function useAuth(): AuthContextType {
  const context = useContext(AuthContext)

  if (context === null) {
    throw new Error('useAuth must be used within an AuthProvider')
  }

  return context  // TypeScript knows this is AuthContextType
}

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null)
  const [isLoading, setIsLoading] = useState(true)

  const login = async (email: string, password: string) => {
    // implementation
  }

  const logout = () => setUser(null)

  return (
    <AuthContext.Provider value={{ user, login, logout, isLoading }}>
      {children}
    </AuthContext.Provider>
  )
}
```

---

## 7. Utility Types

**Impact: LOW**

Specialized type patterns for edge cases.

### 7.1 Display Name Pattern

**Impact: LOW (debugging aid)**

Set displayName for better DevTools integration.

```typescript
const MyComponent = () => <div>Content</div>
MyComponent.displayName = 'MyComponent'
```

---

## References

1. [https://react.dev](https://react.dev)
2. [https://react.dev/learn/typescript](https://react.dev/learn/typescript)
3. [https://www.typescriptlang.org](https://www.typescriptlang.org)
4. [https://www.typescriptlang.org/docs/handbook/react.html](https://www.typescriptlang.org/docs/handbook/react.html)
5. [https://github.com/typescript-cheatsheets/react](https://github.com/typescript-cheatsheets/react)
6. [https://react-typescript-cheatsheet.netlify.app](https://react-typescript-cheatsheet.netlify.app)
