# event-handler-types

**Priority:** HIGH
**Category:** Event Handling

## Why It Matters

Event handlers in React use synthetic events with specific types. Using the correct types provides autocomplete for event properties and catches errors at compile time.

## Event Type Pattern

```
React.[EventType]<[ElementType]>
```

## Incorrect

```typescript
// ❌ Using native Event instead of React event
const handleClick = (e: Event) => {
  // Missing React-specific properties
}

// ❌ Missing element type
const handleChange = (e: React.ChangeEvent) => {
  e.target.value  // Error: Property 'value' does not exist
}

// ❌ Wrong event type
const handleSubmit = (e: React.MouseEvent) => {  // Should be FormEvent
  e.preventDefault()
}
```

## Correct

### Mouse Events

```typescript
// ✅ Click events
const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {
  console.log(e.clientX, e.clientY)
  console.log(e.currentTarget.disabled)  // Button properties
}

// ✅ With generic element
const handleDivClick = (e: React.MouseEvent<HTMLDivElement>) => {
  console.log(e.currentTarget.className)
}

<button onClick={handleClick}>Click</button>
<div onClick={handleDivClick}>Click</div>
```

### Form Events

```typescript
// ✅ Form submit
const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
  e.preventDefault()
  const formData = new FormData(e.currentTarget)
}

// ✅ Input change
const handleInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  console.log(e.target.value)
  console.log(e.target.type)
  console.log(e.target.checked)  // For checkboxes
}

// ✅ Select change
const handleSelectChange = (e: React.ChangeEvent<HTMLSelectElement>) => {
  console.log(e.target.value)
  console.log(e.target.selectedOptions)
}

// ✅ Textarea change
const handleTextareaChange = (e: React.ChangeEvent<HTMLTextAreaElement>) => {
  console.log(e.target.value)
}
```

### Keyboard Events

```typescript
// ✅ Key press
const handleKeyDown = (e: React.KeyboardEvent<HTMLInputElement>) => {
  if (e.key === 'Enter') {
    e.preventDefault()
    // submit
  }
  if (e.key === 'Escape') {
    // close
  }
  console.log(e.ctrlKey, e.shiftKey, e.altKey)
}

<input onKeyDown={handleKeyDown} />
```

### Focus Events

```typescript
// ✅ Focus and blur
const handleFocus = (e: React.FocusEvent<HTMLInputElement>) => {
  console.log('Focused:', e.target.name)
}

const handleBlur = (e: React.FocusEvent<HTMLInputElement>) => {
  console.log('Blurred:', e.target.value)
}

<input onFocus={handleFocus} onBlur={handleBlur} />
```

### Drag Events

```typescript
const handleDragStart = (e: React.DragEvent<HTMLDivElement>) => {
  e.dataTransfer.setData('text/plain', 'data')
}

const handleDrop = (e: React.DragEvent<HTMLDivElement>) => {
  e.preventDefault()
  const data = e.dataTransfer.getData('text/plain')
}
```

### Touch Events

```typescript
const handleTouchStart = (e: React.TouchEvent<HTMLDivElement>) => {
  const touch = e.touches[0]
  console.log(touch.clientX, touch.clientY)
}
```

## Inline vs Defined Handlers

```typescript
// ✅ Inline - types inferred automatically
<button onClick={(e) => {
  // e is React.MouseEvent<HTMLButtonElement>
  console.log(e.clientX)
}}>
  Click
</button>

// ✅ Defined - must type explicitly
const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {
  console.log(e.clientX)
}
<button onClick={handleClick}>Click</button>
```

## Event Handler Props

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

## Common Event Types Reference

| Event | Type | Common Elements |
|-------|------|-----------------|
| click | `MouseEvent` | button, div, a |
| change | `ChangeEvent` | input, select, textarea |
| submit | `FormEvent` | form |
| keydown/keyup | `KeyboardEvent` | input, div |
| focus/blur | `FocusEvent` | input, button |
| drag | `DragEvent` | any draggable |
| touch | `TouchEvent` | any touchable |
