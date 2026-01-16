# comp-props-interface

**Priority:** CRITICAL
**Category:** Component Typing

## Why It Matters

Consistent typing strategy makes code predictable and maintainable. Interfaces are preferred for props because they're extendable, provide better error messages, and align with React's composition model.

## Incorrect

```typescript
// ❌ Using type when interface is better
type ButtonProps = {
  label: string
  onClick: () => void
}

// ❌ Inline types - not reusable
function Button({ label, onClick }: { label: string; onClick: () => void }) {
  return <button onClick={onClick}>{label}</button>
}

// ❌ No typing
function Card(props) {
  return <div>{props.title}</div>
}
```

## Correct

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

function IconButton({
  label,
  onClick,
  disabled,
  icon,
  iconPosition = 'left',
}: IconButtonProps) {
  return (
    <button onClick={onClick} disabled={disabled}>
      {iconPosition === 'left' && icon}
      {label}
      {iconPosition === 'right' && icon}
    </button>
  )
}
```

## When to Use `type`

```typescript
// ✅ Use type for unions
type ButtonVariant = 'primary' | 'secondary' | 'danger'
type Size = 'sm' | 'md' | 'lg'

// ✅ Use type for computed/mapped types
type ButtonState = 'idle' | 'loading' | 'success' | 'error'

// ✅ Use type for props that can't be interface
type PropsWithRequiredChildren = {
  children: React.ReactNode
} & React.HTMLAttributes<HTMLDivElement>

interface ButtonProps {
  variant: ButtonVariant
  size: Size
}
```

## Extending HTML Element Props

```typescript
// ✅ Extend native button props
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant: 'primary' | 'secondary'
  isLoading?: boolean
}

function Button({
  variant,
  isLoading,
  children,
  disabled,
  ...props
}: ButtonProps) {
  return (
    <button
      className={`btn-${variant}`}
      disabled={disabled || isLoading}
      {...props}
    >
      {isLoading ? 'Loading...' : children}
    </button>
  )
}

// Now supports all native button attributes
<Button variant="primary" type="submit" aria-label="Submit form">
  Submit
</Button>
```

## Props with Generics

```typescript
// ✅ Generic interface
interface SelectProps<T> {
  options: T[]
  value: T
  onChange: (value: T) => void
  getLabel: (option: T) => string
}

function Select<T>({ options, value, onChange, getLabel }: SelectProps<T>) {
  return (
    <select
      value={getLabel(value)}
      onChange={(e) => {
        const selected = options.find((o) => getLabel(o) === e.target.value)
        if (selected) onChange(selected)
      }}
    >
      {options.map((option) => (
        <option key={getLabel(option)} value={getLabel(option)}>
          {getLabel(option)}
        </option>
      ))}
    </select>
  )
}
```

## Pattern Summary

| Use Case | Recommendation |
|----------|----------------|
| Component props | `interface` |
| Union types | `type` |
| Extending HTML elements | `interface extends` |
| Mapped/computed types | `type` |
| Generic props | `interface<T>` |
