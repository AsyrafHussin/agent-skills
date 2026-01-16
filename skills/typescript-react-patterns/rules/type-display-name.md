# Display Name Pattern

Setting displayName for better debugging and DevTools integration.

## Bad Example

```tsx
// Anonymous arrow function - shows as "Anonymous" in DevTools
export const Button = ({ children }: { children: React.ReactNode }) => {
  return <button>{children}</button>;
};

// HOC without displayName - hard to identify wrapped component
function withLogger<P extends object>(Component: React.ComponentType<P>) {
  return (props: P) => {
    console.log('Rendering:', props);
    return <Component {...props} />;
  };
}

// Memo without displayName
const ExpensiveList = React.memo(({ items }: { items: string[] }) => {
  return (
    <ul>
      {items.map((item) => (
        <li key={item}>{item}</li>
      ))}
    </ul>
  );
});

// ForwardRef without displayName
const Input = React.forwardRef<HTMLInputElement, React.InputHTMLAttributes<HTMLInputElement>>(
  (props, ref) => <input ref={ref} {...props} />
);

// Context without displayName
const ThemeContext = React.createContext<'light' | 'dark'>('light');
```

## Good Example

```tsx
import React, { forwardRef, memo, createContext } from 'react';

// Named function export - automatically has displayName
export function Button({ children }: { children: React.ReactNode }): React.ReactElement {
  return <button>{children}</button>;
}
// Button.displayName is automatically "Button"

// Arrow function with explicit displayName
export const IconButton = ({ icon, label }: { icon: React.ReactNode; label: string }): React.ReactElement => {
  return (
    <button aria-label={label}>
      {icon}
    </button>
  );
};
IconButton.displayName = 'IconButton';

// HOC with proper displayName
function withLogger<P extends object>(
  WrappedComponent: React.ComponentType<P>
): React.FC<P> {
  const displayName = WrappedComponent.displayName ?? WrappedComponent.name ?? 'Component';

  const WithLogger: React.FC<P> = (props) => {
    console.log(`Rendering ${displayName}:`, props);
    return <WrappedComponent {...props} />;
  };

  WithLogger.displayName = `withLogger(${displayName})`;

  return WithLogger;
}

// Usage
const LoggedButton = withLogger(Button);
// LoggedButton.displayName is "withLogger(Button)"

// Memo with displayName
interface ListProps {
  items: string[];
  onItemClick?: (item: string) => void;
}

const ExpensiveList = memo<ListProps>(({ items, onItemClick }) => {
  return (
    <ul>
      {items.map((item) => (
        <li key={item} onClick={() => onItemClick?.(item)}>
          {item}
        </li>
      ))}
    </ul>
  );
});
ExpensiveList.displayName = 'ExpensiveList';

// ForwardRef with displayName
interface InputProps extends Omit<React.InputHTMLAttributes<HTMLInputElement>, 'size'> {
  size?: 'sm' | 'md' | 'lg';
  error?: string;
}

const Input = forwardRef<HTMLInputElement, InputProps>(
  ({ size = 'md', error, className, ...rest }, ref) => {
    return (
      <div className="input-wrapper">
        <input
          ref={ref}
          className={`input input-${size} ${error ? 'input-error' : ''} ${className ?? ''}`}
          aria-invalid={!!error}
          {...rest}
        />
        {error && <span className="error">{error}</span>}
      </div>
    );
  }
);
Input.displayName = 'Input';

// Context with displayName
interface ThemeContextValue {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextValue | null>(null);
ThemeContext.displayName = 'ThemeContext';

// Provider component with displayName
const ThemeProvider = ({ children }: { children: React.ReactNode }): React.ReactElement => {
  const [theme, setTheme] = React.useState<'light' | 'dark'>('light');

  const value: ThemeContextValue = {
    theme,
    toggleTheme: () => setTheme((t) => (t === 'light' ? 'dark' : 'light')),
  };

  return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>;
};
ThemeProvider.displayName = 'ThemeProvider';

// Compound component pattern with displayNames
interface TabsContextValue {
  activeTab: string;
  setActiveTab: (tab: string) => void;
}

const TabsContext = createContext<TabsContextValue | null>(null);
TabsContext.displayName = 'TabsContext';

interface TabsProps {
  defaultTab: string;
  children: React.ReactNode;
}

const Tabs = ({ defaultTab, children }: TabsProps): React.ReactElement => {
  const [activeTab, setActiveTab] = React.useState(defaultTab);

  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
};
Tabs.displayName = 'Tabs';

const TabList = ({ children }: { children: React.ReactNode }): React.ReactElement => {
  return <div role="tablist">{children}</div>;
};
TabList.displayName = 'Tabs.TabList';

const Tab = ({ id, children }: { id: string; children: React.ReactNode }): React.ReactElement => {
  const context = React.useContext(TabsContext);
  if (!context) throw new Error('Tab must be used within Tabs');

  return (
    <button
      role="tab"
      aria-selected={context.activeTab === id}
      onClick={() => context.setActiveTab(id)}
    >
      {children}
    </button>
  );
};
Tab.displayName = 'Tabs.Tab';

const TabPanel = ({ id, children }: { id: string; children: React.ReactNode }): React.ReactElement | null => {
  const context = React.useContext(TabsContext);
  if (!context) throw new Error('TabPanel must be used within Tabs');

  if (context.activeTab !== id) return null;

  return <div role="tabpanel">{children}</div>;
};
TabPanel.displayName = 'Tabs.TabPanel';

// Attach sub-components
Tabs.TabList = TabList;
Tabs.Tab = Tab;
Tabs.TabPanel = TabPanel;

// Helper function for creating displayName
function getDisplayName<P>(Component: React.ComponentType<P>): string {
  return Component.displayName ?? Component.name ?? 'Component';
}

// Usage in HOC
function withErrorBoundary<P extends object>(Component: React.ComponentType<P>) {
  const WithErrorBoundary = (props: P) => {
    // Error boundary logic
    return <Component {...props} />;
  };

  WithErrorBoundary.displayName = `withErrorBoundary(${getDisplayName(Component)})`;

  return WithErrorBoundary;
}
```

## Why

1. **Debugging**: Components appear with readable names in React DevTools
2. **Error messages**: Stack traces show meaningful component names
3. **HOC clarity**: Wrapped components show their hierarchy (e.g., "withLogger(Button)")
4. **Profiling**: React Profiler shows component names for performance analysis
5. **Testing**: Component names appear in test output and snapshots
6. **Documentation**: DisplayName serves as self-documentation in compound components
