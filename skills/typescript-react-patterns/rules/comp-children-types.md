---
title: Component Children Types
category: Component Typing
priority: CRITICAL
---

# comp-children-types

## Why It Matters

`children` is one of the most commonly used props. Using the wrong type causes type errors or allows invalid usage. Choose the right type based on what your component accepts.

## Children Type Options

| Type | Accepts | Use Case |
|------|---------|----------|
| `React.ReactNode` | Anything renderable | Most components |
| `React.ReactElement` | JSX elements only | When you need element props |
| `React.ReactElement[]` | Array of elements | Tabs, lists |
| `string` | Text only | Text-only components |
| `(data: T) => ReactNode` | Render prop | Data fetching, renderless |

## Incorrect

```typescript
// ❌ Too restrictive - won't accept strings or numbers
interface CardProps {
  children: React.ReactElement
}

// This fails:
<Card>Hello</Card>  // Error: string is not ReactElement
<Card>{42}</Card>   // Error: number is not ReactElement

// ❌ No type - any is implied
interface CardProps {
  children: any
}

// ❌ JSX.Element - React Native incompatible
interface CardProps {
  children: JSX.Element
}
```

## Correct

### ReactNode (Most Common)

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
<Card title="List">{items.map(i => <Item key={i.id} />)}</Card>
<Card title="Maybe">{showContent && <Content />}</Card>
<Card title="Empty">{null}</Card>
```

### ReactElement (JSX Only)

```typescript
// ✅ When you need to access element props
interface TabsProps {
  children: React.ReactElement<TabProps> | React.ReactElement<TabProps>[]
}

interface TabProps {
  label: string
  children: React.ReactNode
}

function Tabs({ children }: TabsProps) {
  const tabs = React.Children.toArray(children) as React.ReactElement<TabProps>[]

  return (
    <div>
      <div className="tab-list">
        {tabs.map((tab, i) => (
          <button key={i}>{tab.props.label}</button>
        ))}
      </div>
      <div className="tab-panels">
        {children}
      </div>
    </div>
  )
}

// Usage
<Tabs>
  <Tab label="Profile">Profile content</Tab>
  <Tab label="Settings">Settings content</Tab>
</Tabs>
```

### Render Props

```typescript
// ✅ Function as children (render prop)
interface DataFetcherProps<T> {
  url: string
  children: (data: T, loading: boolean, error: Error | null) => React.ReactNode
}

function DataFetcher<T>({ url, children }: DataFetcherProps<T>) {
  const [data, setData] = useState<T | null>(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState<Error | null>(null)

  // fetch logic...

  return <>{children(data as T, loading, error)}</>
}

// Usage
<DataFetcher<User[]> url="/api/users">
  {(users, loading, error) => {
    if (loading) return <Spinner />
    if (error) return <Error message={error.message} />
    return <UserList users={users} />
  }}
</DataFetcher>
```

### PropsWithChildren Utility

```typescript
import { PropsWithChildren } from 'react'

// ✅ Shorthand for adding children
interface CardProps {
  title: string
}

function Card({ title, children }: PropsWithChildren<CardProps>) {
  return (
    <div>
      <h2>{title}</h2>
      {children}
    </div>
  )
}

// Equivalent to:
interface CardProps {
  title: string
  children?: React.ReactNode
}
```

### Required vs Optional Children

```typescript
// ✅ Required children
interface ContainerProps {
  children: React.ReactNode  // Required, but can be null/undefined at runtime
}

// ✅ Truly required (must provide content)
interface ContainerProps {
  children: NonNullable<React.ReactNode>
}

// ✅ Optional children (explicit)
interface ContainerProps {
  children?: React.ReactNode
}
```

## Impact

- Correct types prevent runtime errors
- Better autocomplete and documentation
- Catches invalid children at compile time
