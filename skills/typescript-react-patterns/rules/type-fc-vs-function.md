# FC vs Function Declaration

Choosing between `React.FC` and regular function declarations for component typing.

## Bad Example

```tsx
// Using React.FC with implicit children (outdated pattern)
import React, { FC } from 'react';

interface ButtonProps {
  label: string;
  onClick: () => void;
}

// FC used to include children implicitly (React 17 and earlier)
// In React 18+, FC no longer includes children automatically
const Button: FC<ButtonProps> = ({ label, onClick }) => {
  return <button onClick={onClick}>{label}</button>;
};

// FC makes it harder to use generics
const List: FC<{ items: string[] }> = ({ items }) => (
  <ul>
    {items.map((item) => (
      <li key={item}>{item}</li>
    ))}
  </ul>
);
```

## Good Example

```tsx
// Using regular function declarations with explicit return types
interface ButtonProps {
  label: string;
  onClick: () => void;
}

function Button({ label, onClick }: ButtonProps): React.ReactElement {
  return <button onClick={onClick}>{label}</button>;
}

// Arrow function with explicit typing
const Card = ({ title, children }: {
  title: string;
  children: React.ReactNode;
}): React.ReactElement => {
  return (
    <div className="card">
      <h2>{title}</h2>
      {children}
    </div>
  );
};

// Generic components are cleaner without FC
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
}

function List<T>({ items, renderItem }: ListProps<T>): React.ReactElement {
  return <ul>{items.map(renderItem)}</ul>;
}

// Named function export for better debugging
export function UserProfile({ user }: { user: User }): React.ReactElement {
  return <div>{user.name}</div>;
}
```

## Why

1. **Explicit over implicit**: Regular functions require explicit children prop declaration, making the component API clearer
2. **Generic support**: Regular functions work better with TypeScript generics
3. **Consistency**: Avoids confusion about React 17 vs 18 FC behavior regarding children
4. **Better debugging**: Named function declarations appear with their names in React DevTools and stack traces
5. **No defaultProps issues**: FC had issues with defaultProps typing that regular functions don't have
6. **Industry trend**: The React and TypeScript communities have moved away from FC
