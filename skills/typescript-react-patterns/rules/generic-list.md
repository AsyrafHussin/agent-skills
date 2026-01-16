# generic-list

**Priority:** MEDIUM
**Category:** Generic Components

## Why It Matters

Generic components allow building reusable, type-safe components that work with any data type. The type flows through from the data to the render function, providing full type safety.

## Incorrect

```typescript
// ❌ Using any - no type safety
interface ListProps {
  items: any[]
  renderItem: (item: any) => React.ReactNode
}

function List({ items, renderItem }: ListProps) {
  return <ul>{items.map(renderItem)}</ul>
}

// No autocomplete, no type checking
<List
  items={users}
  renderItem={(item) => <li>{item.nmae}</li>}  // Typo not caught!
/>
```

## Correct

### Basic Generic List

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

// Usage - T is inferred from items
interface User {
  id: number
  name: string
  email: string
}

const users: User[] = [
  { id: 1, name: 'John', email: 'john@example.com' }
]

<List
  items={users}
  renderItem={(user) => (
    // user is typed as User
    <span>{user.name} - {user.email}</span>
  )}
  keyExtractor={(user) => user.id}
/>
```

### With Constraints

```typescript
// ✅ Constrain generic to have id property
interface HasId {
  id: string | number
}

interface ListProps<T extends HasId> {
  items: T[]
  renderItem: (item: T) => React.ReactNode
}

function List<T extends HasId>({ items, renderItem }: ListProps<T>) {
  return (
    <ul>
      {items.map((item) => (
        <li key={item.id}>
          {renderItem(item)}
        </li>
      ))}
    </ul>
  )
}

// keyExtractor no longer needed - uses item.id
<List
  items={users}
  renderItem={(user) => <UserCard user={user} />}
/>
```

### Generic Grid Component

```typescript
interface GridProps<T> {
  items: T[]
  columns: number
  renderItem: (item: T) => React.ReactNode
  keyExtractor: (item: T) => string | number
  emptyState?: React.ReactNode
}

function Grid<T>({
  items,
  columns,
  renderItem,
  keyExtractor,
  emptyState = <p>No items</p>,
}: GridProps<T>) {
  if (items.length === 0) {
    return <>{emptyState}</>
  }

  return (
    <div
      className="grid gap-4"
      style={{ gridTemplateColumns: `repeat(${columns}, 1fr)` }}
    >
      {items.map((item) => (
        <div key={keyExtractor(item)}>
          {renderItem(item)}
        </div>
      ))}
    </div>
  )
}

// Usage
<Grid
  items={products}
  columns={3}
  renderItem={(product) => <ProductCard product={product} />}
  keyExtractor={(product) => product.id}
  emptyState={<EmptyProducts />}
/>
```

### Generic Table Component

```typescript
interface Column<T> {
  key: keyof T | string
  header: string
  render?: (item: T) => React.ReactNode
}

interface TableProps<T> {
  data: T[]
  columns: Column<T>[]
  keyExtractor: (item: T) => string | number
}

function Table<T>({ data, columns, keyExtractor }: TableProps<T>) {
  return (
    <table>
      <thead>
        <tr>
          {columns.map((col) => (
            <th key={String(col.key)}>{col.header}</th>
          ))}
        </tr>
      </thead>
      <tbody>
        {data.map((item) => (
          <tr key={keyExtractor(item)}>
            {columns.map((col) => (
              <td key={String(col.key)}>
                {col.render
                  ? col.render(item)
                  : String(item[col.key as keyof T] ?? '')}
              </td>
            ))}
          </tr>
        ))}
      </tbody>
    </table>
  )
}

// Usage
<Table
  data={users}
  columns={[
    { key: 'name', header: 'Name' },
    { key: 'email', header: 'Email' },
    {
      key: 'actions',
      header: 'Actions',
      render: (user) => <EditButton userId={user.id} />,
    },
  ]}
  keyExtractor={(user) => user.id}
/>
```

## Arrow Function Generic Syntax

```typescript
// ✅ Arrow function with generic
const List = <T,>({ items, renderItem }: ListProps<T>) => {
  // Note the comma after T - needed in TSX files
  return <ul>{items.map(renderItem)}</ul>
}

// Or with extends
const List = <T extends HasId>({ items }: { items: T[] }) => {
  return <ul>{items.map(item => <li key={item.id} />)}</ul>
}
```
