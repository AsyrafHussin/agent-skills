# Rule Sections

## Priority Levels

| Level | Description | When to Apply |
|-------|-------------|---------------|
| CRITICAL | Essential for type-safe React code | Always |
| HIGH | Common patterns for most projects | Most components |
| MEDIUM | Advanced patterns for reusable code | When building component libraries |
| LOW | Specialized utilities and edge cases | Specific scenarios |

## Section Overview

### 1. Component Typing (CRITICAL)

Fundamental patterns for typing React component props, children, and HTML element extensions. These are the building blocks of every typed component.

**Key patterns:**
- Interface vs type for props
- Children typing (ReactNode, ReactElement, render props)
- Extending HTML element props
- Default props patterns
- Rest/spread props

### 2. Hook Typing (CRITICAL)

Essential patterns for typing React hooks. Every React application uses these hooks, making proper typing crucial for safety and maintainability.

**Key patterns:**
- useState with nullable and union types
- useRef for DOM elements and mutable values
- useEffect cleanup typing
- useReducer with discriminated unions
- Custom hook return types
- Generic hooks

### 3. Event Handling (HIGH)

Correct event types for form inputs, buttons, and interactive elements. Proper event typing provides autocomplete and catches property access errors.

**Key patterns:**
- MouseEvent types (click, double-click, context menu)
- FormEvent types (submit, change)
- KeyboardEvent types
- FocusEvent types
- Event handler type aliases
- Synthetic event vs native events

### 4. Ref Typing (HIGH)

Patterns for useRef, forwardRef, and ref callbacks. Critical for DOM manipulation and accessing child component instances.

**Key patterns:**
- DOM element refs (nullable)
- Mutable value refs (non-nullable)
- Callback refs
- forwardRef with generic types
- useImperativeHandle typing
- Multiple refs pattern

### 5. Generic Components (MEDIUM)

Building type-safe, reusable components with TypeScript generics. Essential for component libraries and data-agnostic components.

**Key patterns:**
- Generic list/grid components
- Generic form components
- Generic select/dropdown
- Generic table components
- Constrained generics (extends)
- Arrow function generic syntax

### 6. Context & State (MEDIUM)

Creating and consuming typed React context. Important for global state management and theme providers.

**Key patterns:**
- Typed context with null default
- Custom hooks with runtime checks
- Context with useReducer
- Multiple context composition
- Provider component patterns

### 7. Utility Types (LOW)

React and TypeScript utility types for advanced patterns. Used for specific scenarios requiring type manipulation.

**Key patterns:**
- Built-in React types (ComponentProps, ElementType)
- Pick/Omit for props
- Discriminated unions for state machines
- Type assertions and guards
- Polymorphic "as" prop pattern

## Rule Organization

Rules follow the naming convention: `{category-prefix}-{rule-name}.md`

**Category prefixes:**
- `comp-` - Component Typing
- `hook-` - Hook Typing
- `event-` - Event Handling
- `ref-` - Ref Typing (includes forwardRef)
- `generic-` - Generic Components
- `ctx-` - Context & State
- `util-` - Utility Types

**Priority distribution:**
- CRITICAL: 8 rules (Component & Hook fundamentals)
- HIGH: 6 rules (Events, Refs, useReducer)
- MEDIUM: 8 rules (Generics, Context, advanced patterns)
- LOW: 3 rules (Display name, specialized utilities)
