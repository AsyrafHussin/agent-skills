# TypeScript React Patterns

Type-safe patterns for React development with TypeScript.

## Overview

This skill provides comprehensive guidance for:
- Component props typing
- Custom hook typing
- Event handler types
- Ref typing patterns
- Generic components
- Context with TypeScript

## Categories

### 1. Component Typing (Critical)
Proper typing for component props, children, and default values.

### 2. Hook Typing (Critical)
useState, useRef, useReducer, and custom hooks with TypeScript.

### 3. Event Handling (High)
Form events, keyboard events, mouse events, and custom events.

### 4. Ref Typing (High)
DOM refs, mutable refs, callback refs, and forwarded refs.

### 5. Generic Components (Medium)
Building reusable components with TypeScript generics.

### 6. Context & State (Medium)
Typed context, providers, and state management.

### 7. Utility Types (Low)
React utility types and TypeScript helpers.

## Quick Start

```typescript
// Typed component with props
interface ButtonProps {
  variant: 'primary' | 'secondary'
  onClick?: () => void
  children: React.ReactNode
}

function Button({ variant, onClick, children }: ButtonProps) {
  return (
    <button className={`btn-${variant}`} onClick={onClick}>
      {children}
    </button>
  )
}
```

## Usage

This skill triggers automatically when:
- Writing TypeScript React components
- Fixing type errors
- Creating custom hooks
- Building generic components

## References

- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/)
