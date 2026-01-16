# Props Interface Definition

Defining component props using TypeScript interfaces with proper naming conventions and organization.

## Bad Example

```tsx
// Using inline types - hard to reuse and maintain
const UserCard = (props: { name: string; email: string; age: number; onEdit: () => void }) => {
  return <div>{props.name}</div>;
};

// Using type alias with Props suffix but no component prefix
type Props = {
  title: string;
  subtitle?: string;
};

// Mixing optional and required without clear organization
interface CardProps {
  onClose?: () => void;
  title: string;
  children: React.ReactNode;
  subtitle?: string;
  isOpen: boolean;
  onOpen?: () => void;
}

// Exporting props that should be internal
export interface InternalComponentProps {
  _internalState: boolean;
  publicProp: string;
}
```

## Good Example

```tsx
// Named interface with component prefix
interface UserCardProps {
  name: string;
  email: string;
  age: number;
  onEdit: () => void;
}

function UserCard({ name, email, age, onEdit }: UserCardProps): React.ReactElement {
  return (
    <div className="user-card">
      <h3>{name}</h3>
      <p>{email}</p>
      <p>Age: {age}</p>
      <button onClick={onEdit}>Edit</button>
    </div>
  );
}

// Organized props: required first, then optional, grouped by purpose
interface ModalProps {
  // Required props
  isOpen: boolean;
  title: string;
  children: React.ReactNode;

  // Optional content props
  subtitle?: string;
  footer?: React.ReactNode;

  // Optional callbacks
  onOpen?: () => void;
  onClose?: () => void;
}

// Extending HTML element props for wrapper components
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  isLoading?: boolean;
}

// Composing interfaces for complex components
interface BaseInputProps {
  label: string;
  error?: string;
  hint?: string;
}

interface TextInputProps extends BaseInputProps {
  type: 'text' | 'email' | 'password';
  value: string;
  onChange: (value: string) => void;
}

interface NumberInputProps extends BaseInputProps {
  type: 'number';
  value: number;
  onChange: (value: number) => void;
  min?: number;
  max?: number;
}

type InputProps = TextInputProps | NumberInputProps;

// Export only public-facing props
export interface DataTableProps<T> {
  data: T[];
  columns: Column<T>[];
  onRowClick?: (item: T) => void;
}

// Keep internal props private
interface DataTableInternalProps<T> extends DataTableProps<T> {
  _sortState: SortState;
  _filterState: FilterState;
}
```

## Why

1. **Maintainability**: Named interfaces are easier to find, update, and refactor
2. **Reusability**: Interfaces can be imported and extended by other components
3. **Documentation**: Props interface serves as inline documentation for component API
4. **Organization**: Grouping required/optional and by purpose improves readability
5. **Type safety**: Extending HTML element types ensures compatibility with native props
6. **Encapsulation**: Separating public and internal props maintains clean APIs
