---
title: useMemo Typing
category: Hook Typing
priority: MEDIUM
---


Properly typing useMemo for memoized computed values.

## Bad Example

```tsx
// Missing type annotation - relies entirely on inference
const expensiveResult = useMemo(() => {
  return someExpensiveCalculation(data);
}, [data]);

// Using 'any' loses type safety
const config = useMemo<any>(() => ({
  theme: 'dark',
  features: ['a', 'b'],
}), []);

// Unnecessary useMemo for simple values
const double = useMemo(() => count * 2, [count]); // Simple math doesn't need memoization

// Missing dependencies causes stale values
const filteredItems = useMemo(() => {
  return items.filter(item => item.category === selectedCategory);
}, []); // Missing items and selectedCategory

// Returning different types based on condition
const value = useMemo(() => {
  if (isLoading) return null;
  return data.map(transform);
}, [isLoading, data]); // Return type is unclear
```

## Good Example

```tsx
import { useMemo, useState } from 'react';

// Basic useMemo with explicit type
interface ProcessedData {
  total: number;
  average: number;
  max: number;
  min: number;
}

const statistics = useMemo<ProcessedData>(() => {
  const total = numbers.reduce((sum, n) => sum + n, 0);
  return {
    total,
    average: total / numbers.length,
    max: Math.max(...numbers),
    min: Math.min(...numbers),
  };
}, [numbers]);

// Complex object memoization
interface ChartConfig {
  type: 'line' | 'bar' | 'pie';
  data: number[];
  labels: string[];
  options: {
    responsive: boolean;
    animations: boolean;
    legend: {
      position: 'top' | 'bottom' | 'left' | 'right';
    };
  };
}

function Chart({ data, labels, type }: { data: number[]; labels: string[]; type: ChartConfig['type'] }) {
  const chartConfig = useMemo<ChartConfig>(() => ({
    type,
    data,
    labels,
    options: {
      responsive: true,
      animations: true,
      legend: {
        position: 'top',
      },
    },
  }), [type, data, labels]);

  return <RenderChart config={chartConfig} />;
}

// Filtering and sorting with proper typing
interface Product {
  id: string;
  name: string;
  price: number;
  category: string;
  inStock: boolean;
}

type SortField = 'name' | 'price';
type SortDirection = 'asc' | 'desc';

function ProductList({ products }: { products: Product[] }) {
  const [searchTerm, setSearchTerm] = useState('');
  const [category, setCategory] = useState<string | null>(null);
  const [sortField, setSortField] = useState<SortField>('name');
  const [sortDirection, setSortDirection] = useState<SortDirection>('asc');

  const filteredAndSortedProducts = useMemo<Product[]>(() => {
    let result = products;

    // Filter by search term
    if (searchTerm) {
      const term = searchTerm.toLowerCase();
      result = result.filter((p) =>
        p.name.toLowerCase().includes(term)
      );
    }

    // Filter by category
    if (category) {
      result = result.filter((p) => p.category === category);
    }

    // Sort
    result = [...result].sort((a, b) => {
      const aVal = a[sortField];
      const bVal = b[sortField];
      const comparison = aVal < bVal ? -1 : aVal > bVal ? 1 : 0;
      return sortDirection === 'asc' ? comparison : -comparison;
    });

    return result;
  }, [products, searchTerm, category, sortField, sortDirection]);

  return (
    <ul>
      {filteredAndSortedProducts.map((product) => (
        <li key={product.id}>{product.name} - ${product.price}</li>
      ))}
    </ul>
  );
}

// Memoizing derived state for context
interface ThemeColors {
  primary: string;
  secondary: string;
  background: string;
  text: string;
  border: string;
}

interface ThemeContextValue {
  theme: 'light' | 'dark';
  colors: ThemeColors;
}

function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');

  const colors = useMemo<ThemeColors>(() => {
    if (theme === 'light') {
      return {
        primary: '#007bff',
        secondary: '#6c757d',
        background: '#ffffff',
        text: '#212529',
        border: '#dee2e6',
      };
    }
    return {
      primary: '#6ea8fe',
      secondary: '#adb5bd',
      background: '#212529',
      text: '#ffffff',
      border: '#495057',
    };
  }, [theme]);

  const contextValue = useMemo<ThemeContextValue>(
    () => ({ theme, colors }),
    [theme, colors]
  );

  return (
    <ThemeContext.Provider value={contextValue}>
      {children}
    </ThemeContext.Provider>
  );
}

// Memoizing expensive transformations
interface RawDataPoint {
  timestamp: string;
  value: number;
  metadata: Record<string, unknown>;
}

interface ChartDataPoint {
  x: Date;
  y: number;
  label: string;
}

function DataVisualization({ rawData }: { rawData: RawDataPoint[] }) {
  const chartData = useMemo<ChartDataPoint[]>(() => {
    return rawData.map((point) => ({
      x: new Date(point.timestamp),
      y: point.value,
      label: `Value: ${point.value}`,
    }));
  }, [rawData]);

  const aggregatedData = useMemo(() => {
    const byMonth = new Map<string, number[]>();

    chartData.forEach((point) => {
      const monthKey = `${point.x.getFullYear()}-${point.x.getMonth()}`;
      const existing = byMonth.get(monthKey) ?? [];
      byMonth.set(monthKey, [...existing, point.y]);
    });

    return Array.from(byMonth.entries()).map(([month, values]) => ({
      month,
      average: values.reduce((a, b) => a + b, 0) / values.length,
      count: values.length,
    }));
  }, [chartData]);

  return <Chart data={chartData} aggregated={aggregatedData} />;
}

// Memoizing regex and other reusable objects
function SearchHighlighter({ text, searchTerm }: { text: string; searchTerm: string }) {
  const searchRegex = useMemo<RegExp | null>(() => {
    if (!searchTerm) return null;
    try {
      return new RegExp(`(${searchTerm})`, 'gi');
    } catch {
      return null;
    }
  }, [searchTerm]);

  const highlightedParts = useMemo<Array<{ text: string; highlighted: boolean }>>(() => {
    if (!searchRegex) {
      return [{ text, highlighted: false }];
    }

    const parts: Array<{ text: string; highlighted: boolean }> = [];
    let lastIndex = 0;
    let match: RegExpExecArray | null;

    while ((match = searchRegex.exec(text)) !== null) {
      if (match.index > lastIndex) {
        parts.push({ text: text.slice(lastIndex, match.index), highlighted: false });
      }
      parts.push({ text: match[0], highlighted: true });
      lastIndex = match.index + match[0].length;
    }

    if (lastIndex < text.length) {
      parts.push({ text: text.slice(lastIndex), highlighted: false });
    }

    return parts;
  }, [text, searchRegex]);

  return (
    <span>
      {highlightedParts.map((part, i) =>
        part.highlighted ? (
          <mark key={i}>{part.text}</mark>
        ) : (
          <span key={i}>{part.text}</span>
        )
      )}
    </span>
  );
}

// Generic memoized selector pattern
function useMemoizedSelector<TState, TSelected>(
  state: TState,
  selector: (state: TState) => TSelected
): TSelected {
  return useMemo(() => selector(state), [state, selector]);
}

// Usage
interface AppState {
  users: User[];
  products: Product[];
  orders: Order[];
}

function UserList({ state }: { state: AppState }) {
  const activeUsers = useMemoizedSelector(
    state,
    (s) => s.users.filter((u) => u.isActive)
  );

  return <ul>{activeUsers.map((u) => <li key={u.id}>{u.name}</li>)}</ul>;
}
```

## Why

1. **Explicit types**: Clear return types make code more readable and catch errors
2. **Correct dependencies**: All values used inside must be in the dependency array
3. **Performance**: Only memoize expensive calculations, not simple operations
4. **Reference stability**: Memoized objects maintain reference equality
5. **Derived state**: Compute derived values from state without storing redundant data
6. **Context optimization**: Memoize context values to prevent unnecessary re-renders
