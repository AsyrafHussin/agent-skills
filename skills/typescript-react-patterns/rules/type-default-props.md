# Default Props Typing

Modern approaches to typing default props in React with TypeScript.

## Bad Example

```tsx
// Using deprecated defaultProps static property
interface ButtonProps {
  variant?: 'primary' | 'secondary';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
}

const Button: React.FC<ButtonProps> = ({ variant, size, disabled, children }) => {
  return (
    <button className={`btn-${variant} btn-${size}`} disabled={disabled}>
      {children}
    </button>
  );
};

// This pattern is deprecated and will be removed
Button.defaultProps = {
  variant: 'primary',
  size: 'md',
  disabled: false,
};

// Default values not reflected in type system
interface CardProps {
  elevation: number;
  rounded: boolean;
}

function Card({ elevation, rounded }: CardProps) {
  // TypeScript doesn't know about defaults, may cause issues
  return <div style={{ boxShadow: `0 ${elevation}px ${elevation * 2}px rgba(0,0,0,0.1)` }} />;
}

Card.defaultProps = {
  elevation: 2,
  rounded: true,
};
```

## Good Example

```tsx
// Using default parameters in destructuring (preferred modern approach)
interface ButtonProps {
  variant?: 'primary' | 'secondary';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  children: React.ReactNode;
}

function Button({
  variant = 'primary',
  size = 'md',
  disabled = false,
  children,
}: ButtonProps): React.ReactElement {
  return (
    <button className={`btn-${variant} btn-${size}`} disabled={disabled}>
      {children}
    </button>
  );
}

// Extracting defaults for reuse and testing
interface CardProps {
  elevation?: number;
  rounded?: boolean;
  children: React.ReactNode;
}

const cardDefaults: Required<Pick<CardProps, 'elevation' | 'rounded'>> = {
  elevation: 2,
  rounded: true,
};

function Card({
  elevation = cardDefaults.elevation,
  rounded = cardDefaults.rounded,
  children,
}: CardProps): React.ReactElement {
  return (
    <div
      className={rounded ? 'rounded' : ''}
      style={{ boxShadow: `0 ${elevation}px ${elevation * 2}px rgba(0,0,0,0.1)` }}
    >
      {children}
    </div>
  );
}

// Complex default objects with proper typing
interface FormFieldProps {
  name: string;
  label?: string;
  validation?: {
    required?: boolean;
    minLength?: number;
    maxLength?: number;
    pattern?: RegExp;
  };
  styles?: {
    container?: React.CSSProperties;
    label?: React.CSSProperties;
    input?: React.CSSProperties;
  };
}

const defaultValidation: Required<NonNullable<FormFieldProps['validation']>> = {
  required: false,
  minLength: 0,
  maxLength: Infinity,
  pattern: /.*/,
};

const defaultStyles: Required<NonNullable<FormFieldProps['styles']>> = {
  container: {},
  label: {},
  input: {},
};

function FormField({
  name,
  label = name,
  validation = {},
  styles = {},
}: FormFieldProps): React.ReactElement {
  const mergedValidation = { ...defaultValidation, ...validation };
  const mergedStyles = { ...defaultStyles, ...styles };

  return (
    <div style={mergedStyles.container}>
      <label style={mergedStyles.label}>{label}</label>
      <input
        name={name}
        required={mergedValidation.required}
        minLength={mergedValidation.minLength}
        maxLength={mergedValidation.maxLength}
        pattern={mergedValidation.pattern.source}
        style={mergedStyles.input}
      />
    </div>
  );
}

// Using satisfies for type-safe defaults
interface ThemeProps {
  colors?: {
    primary?: string;
    secondary?: string;
    background?: string;
  };
  spacing?: {
    small?: number;
    medium?: number;
    large?: number;
  };
}

const themeDefaults = {
  colors: {
    primary: '#007bff',
    secondary: '#6c757d',
    background: '#ffffff',
  },
  spacing: {
    small: 8,
    medium: 16,
    large: 24,
  },
} satisfies Required<{
  colors: Required<NonNullable<ThemeProps['colors']>>;
  spacing: Required<NonNullable<ThemeProps['spacing']>>;
}>;

function ThemeProvider({
  colors = themeDefaults.colors,
  spacing = themeDefaults.spacing,
  children,
}: ThemeProps & { children: React.ReactNode }): React.ReactElement {
  const theme = {
    colors: { ...themeDefaults.colors, ...colors },
    spacing: { ...themeDefaults.spacing, ...spacing },
  };

  return (
    <ThemeContext.Provider value={theme}>
      {children}
    </ThemeContext.Provider>
  );
}
```

## Why

1. **Default parameters are type-aware**: TypeScript understands the values inside the function
2. **No deprecation concerns**: `defaultProps` is deprecated for function components
3. **Better tree-shaking**: Default parameters can be optimized by bundlers
4. **Explicit defaults**: Makes component behavior clear when reading the code
5. **Testable defaults**: Extracted default objects can be imported in tests
6. **Merged defaults**: Complex objects can use spread for partial overrides
