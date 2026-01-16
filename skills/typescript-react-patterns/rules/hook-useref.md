---
title: useRef Hook Typing
category: Hook Typing
priority: CRITICAL
---

# hook-useref

## Why It Matters

useRef has two distinct use cases with different typing: DOM element refs (nullable) and mutable value storage (non-nullable). Using the wrong pattern causes type errors or requires unnecessary null checks.

## The Two Patterns

| Use Case | Initial Value | Type | `.current` |
|----------|--------------|------|------------|
| DOM ref | `null` | `RefObject<T>` | Readonly, nullable |
| Mutable value | actual value | `MutableRefObject<T>` | Mutable, non-nullable |

## Incorrect

```typescript
// ❌ Missing element type
const inputRef = useRef(null)
inputRef.current.focus()  // Error: possibly null

// ❌ Wrong initial value for DOM ref
const inputRef = useRef<HTMLInputElement>()  // undefined, not null
<input ref={inputRef} />  // Type error

// ❌ Treating mutable ref as nullable
const countRef = useRef<number>(0)
if (countRef.current !== null) {  // Unnecessary check
  countRef.current++
}
```

## Correct

### DOM Element Refs

```typescript
// ✅ DOM ref - pass null, type the element
const inputRef = useRef<HTMLInputElement>(null)
const buttonRef = useRef<HTMLButtonElement>(null)
const divRef = useRef<HTMLDivElement>(null)

function Form() {
  const inputRef = useRef<HTMLInputElement>(null)

  const focusInput = () => {
    // Optional chaining because ref might not be attached yet
    inputRef.current?.focus()
  }

  useEffect(() => {
    // Focus on mount
    inputRef.current?.focus()
  }, [])

  return <input ref={inputRef} />
}
```

### Common DOM Element Types

```typescript
// Form elements
const inputRef = useRef<HTMLInputElement>(null)
const textareaRef = useRef<HTMLTextAreaElement>(null)
const selectRef = useRef<HTMLSelectElement>(null)
const formRef = useRef<HTMLFormElement>(null)

// Container elements
const divRef = useRef<HTMLDivElement>(null)
const sectionRef = useRef<HTMLElement>(null)

// Media elements
const videoRef = useRef<HTMLVideoElement>(null)
const audioRef = useRef<HTMLAudioElement>(null)
const canvasRef = useRef<HTMLCanvasElement>(null)

// SVG elements
const svgRef = useRef<SVGSVGElement>(null)
```

### Mutable Value Refs

```typescript
// ✅ Mutable ref - pass actual initial value
function Timer() {
  // Non-null initial value makes .current non-nullable
  const intervalRef = useRef<number | undefined>(undefined)
  const countRef = useRef(0)  // Inferred as MutableRefObject<number>

  useEffect(() => {
    intervalRef.current = window.setInterval(() => {
      countRef.current++  // No null check needed
    }, 1000)

    return () => {
      clearInterval(intervalRef.current)
    }
  }, [])
}
```

### Storing Previous Value

```typescript
// ✅ Store previous props/state
function usePrevious<T>(value: T): T | undefined {
  const ref = useRef<T | undefined>(undefined)

  useEffect(() => {
    ref.current = value
  }, [value])

  return ref.current
}

// Usage
function Counter({ count }: { count: number }) {
  const prevCount = usePrevious(count)

  return (
    <p>
      Current: {count}, Previous: {prevCount ?? 'none'}
    </p>
  )
}
```

### Storing Callbacks

```typescript
// ✅ Store latest callback without re-renders
function useEventCallback<T extends (...args: any[]) => any>(fn: T): T {
  const ref = useRef<T>(fn)

  useEffect(() => {
    ref.current = fn
  }, [fn])

  return useCallback(
    ((...args) => ref.current(...args)) as T,
    []
  )
}

// Usage
function Chat({ onMessage }: { onMessage: (msg: string) => void }) {
  const stableOnMessage = useEventCallback(onMessage)

  useEffect(() => {
    socket.on('message', stableOnMessage)
    return () => socket.off('message', stableOnMessage)
  }, [stableOnMessage])  // Never changes
}
```

### Multiple Refs (Callback Refs)

```typescript
// ✅ Callback ref for dynamic elements
function List({ items }: { items: Item[] }) {
  const itemRefs = useRef<Map<string, HTMLLIElement>>(new Map())

  const setRef = (id: string) => (el: HTMLLIElement | null) => {
    if (el) {
      itemRefs.current.set(id, el)
    } else {
      itemRefs.current.delete(id)
    }
  }

  const scrollToItem = (id: string) => {
    itemRefs.current.get(id)?.scrollIntoView()
  }

  return (
    <ul>
      {items.map(item => (
        <li key={item.id} ref={setRef(item.id)}>
          {item.name}
        </li>
      ))}
    </ul>
  )
}
```

## Pattern Summary

```typescript
// DOM element: null initial, nullable current
const domRef = useRef<HTMLElement>(null)
domRef.current?.method()

// Mutable value: typed initial, non-nullable current
const valueRef = useRef<number>(0)
valueRef.current++  // No null check

// Maybe undefined value
const maybeRef = useRef<Timer | undefined>(undefined)
```
