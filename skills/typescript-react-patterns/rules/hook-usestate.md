# hook-usestate

**Priority:** CRITICAL
**Category:** Hook Typing

## Why It Matters

useState infers types from initial values. When the initial value doesn't represent all possible states (like `null` for async data), explicit typing prevents runtime errors.

## Incorrect

```typescript
// ❌ Inferred as null, can't set to User
const [user, setUser] = useState(null)
setUser({ id: 1, name: 'John' })  // Error!

// ❌ Inferred as never[] - can't add typed items
const [items, setItems] = useState([])
setItems([{ id: 1 }])  // Error!

// ❌ Inferred as string, can't set undefined
const [search, setSearch] = useState('')
setSearch(undefined)  // Error if you need undefined state
```

## Correct

### Nullable State

```typescript
interface User {
  id: number
  name: string
  email: string
}

// ✅ Explicit union type for nullable state
const [user, setUser] = useState<User | null>(null)

// Now both work:
setUser({ id: 1, name: 'John', email: 'john@example.com' })
setUser(null)

// Access with null check
if (user) {
  console.log(user.name)  // TypeScript knows user is User here
}
```

### Array State

```typescript
interface Todo {
  id: number
  text: string
  done: boolean
}

// ✅ Typed array
const [todos, setTodos] = useState<Todo[]>([])

// Add item
setTodos(prev => [...prev, { id: Date.now(), text: 'New', done: false }])

// Update item
setTodos(prev =>
  prev.map(todo =>
    todo.id === id ? { ...todo, done: !todo.done } : todo
  )
)

// Remove item
setTodos(prev => prev.filter(todo => todo.id !== id))
```

### Object State

```typescript
interface FormData {
  name: string
  email: string
  age: number
}

// ✅ Initial value matches type - inference works
const [form, setForm] = useState<FormData>({
  name: '',
  email: '',
  age: 0,
})

// Update single field
setForm(prev => ({ ...prev, name: 'John' }))

// ✅ Partial updates helper
const updateForm = <K extends keyof FormData>(
  field: K,
  value: FormData[K]
) => {
  setForm(prev => ({ ...prev, [field]: value }))
}

updateForm('name', 'John')
updateForm('age', 25)
```

### Union State

```typescript
// ✅ Discriminated union for state machines
type RequestState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error }

const [state, setState] = useState<RequestState<User>>({ status: 'idle' })

// Usage
switch (state.status) {
  case 'idle':
    return <button onClick={fetch}>Load</button>
  case 'loading':
    return <Spinner />
  case 'success':
    return <UserCard user={state.data} />  // data is typed!
  case 'error':
    return <Error message={state.error.message} />
}
```

### Lazy Initialization

```typescript
// ✅ Type annotation with lazy init
const [state, setState] = useState<ComplexState>(() => {
  // Expensive computation only runs once
  return computeInitialState()
})

// ✅ Reading from storage
const [theme, setTheme] = useState<'light' | 'dark'>(() => {
  const saved = localStorage.getItem('theme')
  return (saved as 'light' | 'dark') || 'light'
})
```

### Undefined vs Null

```typescript
// ✅ Use undefined for "not yet set"
const [selectedId, setSelectedId] = useState<number | undefined>(undefined)

// ✅ Use null for "explicitly empty"
const [user, setUser] = useState<User | null>(null)

// Convention:
// - undefined: "nothing selected yet"
// - null: "deliberately cleared" or "no user logged in"
```

## Common Patterns

```typescript
// Boolean state - inference works fine
const [isOpen, setIsOpen] = useState(false)

// String state - inference works fine
const [search, setSearch] = useState('')

// Number state - inference works fine
const [count, setCount] = useState(0)

// Complex state - always type explicitly
const [data, setData] = useState<DataType | null>(null)
const [items, setItems] = useState<ItemType[]>([])
const [state, setState] = useState<StateUnion>({ status: 'idle' })
```
