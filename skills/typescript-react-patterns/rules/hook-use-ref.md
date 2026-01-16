---
title: useRef Typing
category: Hook Typing
priority: HIGH
---


Properly typing useRef for DOM elements, mutable values, and instance variables.

## Bad Example

```tsx
// Missing type parameter - ref.current is undefined
const inputRef = useRef();
inputRef.current.focus(); // Error: Object is possibly undefined

// Wrong element type
const buttonRef = useRef<HTMLInputElement>(null);
<button ref={buttonRef}>Click</button> // Type error at runtime issues

// Using ref for mutable value but initialized with null
const countRef = useRef<number>(null);
countRef.current += 1; // Error: Cannot assign to current because it's readonly

// Forgetting null for DOM refs
const divRef = useRef<HTMLDivElement>();
<div ref={divRef} /> // Type mismatch
```

## Good Example

```tsx
import { useRef, useEffect } from 'react';

// DOM element refs - initialize with null
const inputRef = useRef<HTMLInputElement>(null);
const buttonRef = useRef<HTMLButtonElement>(null);
const divRef = useRef<HTMLDivElement>(null);
const canvasRef = useRef<HTMLCanvasElement>(null);

// Usage with null check
function FocusInput() {
  const inputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    // Null check required because ref might not be attached yet
    inputRef.current?.focus();
  }, []);

  const handleClick = () => {
    if (inputRef.current) {
      inputRef.current.select();
    }
  };

  return (
    <>
      <input ref={inputRef} type="text" />
      <button onClick={handleClick}>Select All</button>
    </>
  );
}

// Mutable value refs - don't use null, current is writable
const renderCount = useRef<number>(0);
const previousValue = useRef<string>('');
const timerId = useRef<ReturnType<typeof setTimeout>>();
const intervalId = useRef<ReturnType<typeof setInterval>>();

function usePrevious<T>(value: T): T | undefined {
  const ref = useRef<T>();

  useEffect(() => {
    ref.current = value; // Writable because not initialized with null
  }, [value]);

  return ref.current;
}

// Instance variable for tracking state without re-renders
function Timer() {
  const [count, setCount] = useState(0);
  const intervalRef = useRef<ReturnType<typeof setInterval>>();
  const isRunning = useRef(false);

  const start = () => {
    if (isRunning.current) return;
    isRunning.current = true;

    intervalRef.current = setInterval(() => {
      setCount((c) => c + 1);
    }, 1000);
  };

  const stop = () => {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
      isRunning.current = false;
    }
  };

  useEffect(() => {
    return () => {
      if (intervalRef.current) {
        clearInterval(intervalRef.current);
      }
    };
  }, []);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
    </div>
  );
}

// Callback ref for dynamic refs
function DynamicRefs() {
  const itemRefs = useRef<Map<string, HTMLLIElement>>(new Map());

  const setItemRef = (id: string) => (el: HTMLLIElement | null) => {
    if (el) {
      itemRefs.current.set(id, el);
    } else {
      itemRefs.current.delete(id);
    }
  };

  const scrollToItem = (id: string) => {
    const el = itemRefs.current.get(id);
    el?.scrollIntoView({ behavior: 'smooth' });
  };

  const items = ['a', 'b', 'c'];

  return (
    <ul>
      {items.map((id) => (
        <li key={id} ref={setItemRef(id)}>
          Item {id}
        </li>
      ))}
    </ul>
  );
}

// Ref for storing latest callback without stale closure
function useLatestCallback<T extends (...args: unknown[]) => unknown>(callback: T): T {
  const callbackRef = useRef(callback);

  useEffect(() => {
    callbackRef.current = callback;
  }, [callback]);

  return useCallback(
    ((...args) => callbackRef.current(...args)) as T,
    []
  );
}

// Complex object refs
interface FormInstance {
  validate: () => boolean;
  reset: () => void;
  getValues: () => Record<string, unknown>;
  setValues: (values: Record<string, unknown>) => void;
}

function useFormRef() {
  const formRef = useRef<FormInstance | null>(null);

  const registerForm = (instance: FormInstance) => {
    formRef.current = instance;
  };

  return { formRef, registerForm };
}

// Video/Audio element refs
function VideoPlayer({ src }: { src: string }) {
  const videoRef = useRef<HTMLVideoElement>(null);

  const play = () => videoRef.current?.play();
  const pause = () => videoRef.current?.pause();
  const seek = (time: number) => {
    if (videoRef.current) {
      videoRef.current.currentTime = time;
    }
  };

  return (
    <div>
      <video ref={videoRef} src={src} />
      <button onClick={play}>Play</button>
      <button onClick={pause}>Pause</button>
      <button onClick={() => seek(0)}>Restart</button>
    </div>
  );
}

// Canvas ref with 2D context
function Canvas() {
  const canvasRef = useRef<HTMLCanvasElement>(null);
  const contextRef = useRef<CanvasRenderingContext2D | null>(null);

  useEffect(() => {
    const canvas = canvasRef.current;
    if (canvas) {
      contextRef.current = canvas.getContext('2d');
    }
  }, []);

  const draw = () => {
    const ctx = contextRef.current;
    if (ctx) {
      ctx.fillStyle = 'blue';
      ctx.fillRect(10, 10, 100, 100);
    }
  };

  return (
    <>
      <canvas ref={canvasRef} width={400} height={400} />
      <button onClick={draw}>Draw</button>
    </>
  );
}

// Generic ref wrapper hook
function useRefState<T>(initialValue: T) {
  const ref = useRef(initialValue);
  const [, forceUpdate] = useState({});

  const setValue = useCallback((value: T | ((prev: T) => T)) => {
    ref.current = typeof value === 'function'
      ? (value as (prev: T) => T)(ref.current)
      : value;
    forceUpdate({});
  }, []);

  return [ref, setValue] as const;
}
```

## Why

1. **Null for DOM refs**: DOM refs must be initialized with `null` because elements don't exist during initial render
2. **Non-null for mutable values**: Refs storing values (not elements) should omit `null` for writable `current`
3. **Proper element types**: Match ref type to the actual HTML element for correct method access
4. **ReturnType for timers**: Use `ReturnType<typeof setTimeout>` for cross-platform compatibility
5. **No re-renders**: Refs don't trigger re-renders, perfect for values that change frequently
6. **Callback refs**: Use function refs for dynamic collections of elements
