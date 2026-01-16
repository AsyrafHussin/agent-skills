# Children Prop Typing

Properly typing the children prop for different component use cases.

## Bad Example

```tsx
// Using 'any' for children
interface CardProps {
  children: any;
}

// Using incorrect types
interface LayoutProps {
  children: JSX.Element; // Too restrictive - doesn't allow strings, numbers, arrays
}

// Not handling optional children correctly
interface ContainerProps {
  children: React.ReactNode; // Should be optional if component works without children
}

function Container({ children }: ContainerProps) {
  return <div>{children}</div>; // Error if children not provided
}

// Expecting specific children but using ReactNode
interface TabsProps {
  children: React.ReactNode; // Can't enforce Tab children
}
```

## Good Example

```tsx
// Basic children with ReactNode - most flexible
interface CardProps {
  title: string;
  children: React.ReactNode;
}

function Card({ title, children }: CardProps): React.ReactElement {
  return (
    <div className="card">
      <h2>{title}</h2>
      <div className="card-content">{children}</div>
    </div>
  );
}

// Optional children for components that work without them
interface AlertProps {
  message: string;
  children?: React.ReactNode;
}

function Alert({ message, children }: AlertProps): React.ReactElement {
  return (
    <div className="alert">
      <p>{message}</p>
      {children && <div className="alert-actions">{children}</div>}
    </div>
  );
}

// Single element child using ReactElement
interface TooltipProps {
  content: string;
  children: React.ReactElement;
}

function Tooltip({ content, children }: TooltipProps): React.ReactElement {
  return (
    <div className="tooltip-wrapper">
      {React.cloneElement(children, {
        'aria-describedby': 'tooltip',
      })}
      <span id="tooltip" role="tooltip">
        {content}
      </span>
    </div>
  );
}

// Render function children (function as child)
interface DataFetcherProps<T> {
  url: string;
  children: (data: T | null, loading: boolean, error: Error | null) => React.ReactNode;
}

function DataFetcher<T>({ url, children }: DataFetcherProps<T>): React.ReactElement {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  // fetch logic...

  return <>{children(data, loading, error)}</>;
}

// Usage
<DataFetcher<User[]> url="/api/users">
  {(users, loading, error) => {
    if (loading) return <Spinner />;
    if (error) return <Error message={error.message} />;
    return <UserList users={users!} />;
  }}
</DataFetcher>

// Strictly typed children for specific child components
interface TabProps {
  label: string;
  children: React.ReactNode;
}

function Tab({ label, children }: TabProps): React.ReactElement {
  return <div role="tabpanel">{children}</div>;
}

interface TabsProps {
  children: React.ReactElement<TabProps> | React.ReactElement<TabProps>[];
}

function Tabs({ children }: TabsProps): React.ReactElement {
  const tabs = React.Children.toArray(children) as React.ReactElement<TabProps>[];

  return (
    <div className="tabs">
      <div role="tablist">
        {tabs.map((tab, index) => (
          <button key={index} role="tab">
            {tab.props.label}
          </button>
        ))}
      </div>
      {tabs}
    </div>
  );
}

// PropsWithChildren utility type
import { PropsWithChildren } from 'react';

interface PanelProps {
  title: string;
  collapsible?: boolean;
}

function Panel({ title, collapsible, children }: PropsWithChildren<PanelProps>): React.ReactElement {
  return (
    <section>
      <header>{title}</header>
      <div>{children}</div>
    </section>
  );
}
```

## Why

1. **ReactNode for flexibility**: `React.ReactNode` accepts strings, numbers, elements, arrays, fragments, and null
2. **ReactElement for single elements**: Use when you need to clone or inspect the child element
3. **Function children for data passing**: Enables powerful render prop patterns with type safety
4. **Explicit optionality**: Mark children as optional when component works without them
5. **Typed children for compound components**: Ensures only valid child components are passed
6. **PropsWithChildren utility**: Cleaner syntax when extending existing prop interfaces
