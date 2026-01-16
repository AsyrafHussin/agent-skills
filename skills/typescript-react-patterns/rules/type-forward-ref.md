---
title: ForwardRef Typing
category: Ref Typing
priority: HIGH
---


Properly typing components that forward refs to child elements.

## Bad Example

```tsx
// Missing ref type annotation
const Input = React.forwardRef((props, ref) => {
  return <input ref={ref} {...props} />;
});

// Incorrect ref type
interface ButtonProps {
  variant: 'primary' | 'secondary';
}

const Button = React.forwardRef<HTMLDivElement, ButtonProps>((props, ref) => {
  // Ref type doesn't match the actual element
  return <button ref={ref as any}>{props.variant}</button>;
});

// Not exposing ref at all for imperative handle
const ComplexInput = React.forwardRef((props: InputProps, ref) => {
  const inputRef = React.useRef<HTMLInputElement>(null);

  // ref is ignored, parent can't access input
  return <input ref={inputRef} {...props} />;
});
```

## Good Example

```tsx
import React, { forwardRef, useRef, useImperativeHandle } from 'react';

// Basic forwardRef with proper typing
interface InputProps extends Omit<React.InputHTMLAttributes<HTMLInputElement>, 'size'> {
  label: string;
  error?: string;
  size?: 'sm' | 'md' | 'lg';
}

const Input = forwardRef<HTMLInputElement, InputProps>(
  ({ label, error, size = 'md', className, ...rest }, ref) => {
    return (
      <div className={`input-wrapper input-${size}`}>
        <label>{label}</label>
        <input
          ref={ref}
          className={`input ${error ? 'input-error' : ''} ${className ?? ''}`}
          aria-invalid={!!error}
          {...rest}
        />
        {error && <span className="error-message">{error}</span>}
      </div>
    );
  }
);

Input.displayName = 'Input';

// Usage
const MyForm = () => {
  const inputRef = useRef<HTMLInputElement>(null);

  const focusInput = () => {
    inputRef.current?.focus();
  };

  return (
    <form>
      <Input ref={inputRef} label="Email" type="email" />
      <button type="button" onClick={focusInput}>Focus Input</button>
    </form>
  );
};

// ForwardRef with useImperativeHandle for custom methods
interface FormInputHandle {
  focus: () => void;
  clear: () => void;
  getValue: () => string;
  validate: () => boolean;
}

interface FormInputProps {
  name: string;
  label: string;
  required?: boolean;
  pattern?: RegExp;
}

const FormInput = forwardRef<FormInputHandle, FormInputProps>(
  ({ name, label, required = false, pattern }, ref) => {
    const inputRef = useRef<HTMLInputElement>(null);
    const [error, setError] = React.useState<string | null>(null);

    useImperativeHandle(ref, () => ({
      focus: () => {
        inputRef.current?.focus();
      },
      clear: () => {
        if (inputRef.current) {
          inputRef.current.value = '';
          setError(null);
        }
      },
      getValue: () => {
        return inputRef.current?.value ?? '';
      },
      validate: () => {
        const value = inputRef.current?.value ?? '';

        if (required && !value) {
          setError('This field is required');
          return false;
        }

        if (pattern && !pattern.test(value)) {
          setError('Invalid format');
          return false;
        }

        setError(null);
        return true;
      },
    }));

    return (
      <div className="form-input">
        <label htmlFor={name}>{label}</label>
        <input ref={inputRef} id={name} name={name} aria-invalid={!!error} />
        {error && <span className="error">{error}</span>}
      </div>
    );
  }
);

FormInput.displayName = 'FormInput';

// Usage with imperative handle
const RegistrationForm = () => {
  const emailRef = useRef<FormInputHandle>(null);
  const passwordRef = useRef<FormInputHandle>(null);

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();

    const isEmailValid = emailRef.current?.validate() ?? false;
    const isPasswordValid = passwordRef.current?.validate() ?? false;

    if (isEmailValid && isPasswordValid) {
      console.log('Email:', emailRef.current?.getValue());
      // Submit form
    }
  };

  const handleReset = () => {
    emailRef.current?.clear();
    passwordRef.current?.clear();
  };

  return (
    <form onSubmit={handleSubmit}>
      <FormInput
        ref={emailRef}
        name="email"
        label="Email"
        required
        pattern={/^[^\s@]+@[^\s@]+\.[^\s@]+$/}
      />
      <FormInput
        ref={passwordRef}
        name="password"
        label="Password"
        required
      />
      <button type="submit">Register</button>
      <button type="button" onClick={handleReset}>Reset</button>
    </form>
  );
};

// Generic forwardRef component
interface SelectOption<T> {
  value: T;
  label: string;
}

interface SelectProps<T> {
  options: SelectOption<T>[];
  value?: T;
  onChange: (value: T) => void;
  placeholder?: string;
}

// Helper type for generic forwardRef
type GenericForwardRefComponent = <T>(
  props: SelectProps<T> & { ref?: React.ForwardedRef<HTMLSelectElement> }
) => React.ReactElement;

const Select: GenericForwardRefComponent = forwardRef(
  <T,>(
    { options, value, onChange, placeholder }: SelectProps<T>,
    ref: React.ForwardedRef<HTMLSelectElement>
  ) => {
    return (
      <select
        ref={ref}
        value={String(value)}
        onChange={(e) => {
          const selected = options.find((opt) => String(opt.value) === e.target.value);
          if (selected) onChange(selected.value);
        }}
      >
        {placeholder && <option value="">{placeholder}</option>}
        {options.map((option) => (
          <option key={String(option.value)} value={String(option.value)}>
            {option.label}
          </option>
        ))}
      </select>
    );
  }
) as GenericForwardRefComponent;

// ForwardRef with polymorphic component
type PolymorphicRef<C extends React.ElementType> = React.ComponentPropsWithRef<C>['ref'];

interface BoxProps<C extends React.ElementType = 'div'> {
  as?: C;
  padding?: 'sm' | 'md' | 'lg';
}

type BoxComponent = <C extends React.ElementType = 'div'>(
  props: BoxProps<C> &
    Omit<React.ComponentPropsWithRef<C>, keyof BoxProps<C>>
) => React.ReactElement | null;

const Box: BoxComponent = forwardRef(
  <C extends React.ElementType = 'div'>(
    { as, padding, ...rest }: BoxProps<C>,
    ref: PolymorphicRef<C>
  ) => {
    const Component = as ?? 'div';
    return <Component ref={ref} data-padding={padding} {...rest} />;
  }
) as BoxComponent;
```

## Why

1. **Type safety**: Proper typing ensures ref type matches the actual DOM element
2. **Imperative API**: `useImperativeHandle` enables custom methods with type safety
3. **Display name**: Setting displayName improves React DevTools debugging
4. **Generic support**: Special patterns enable generic forwardRef components
5. **Flexibility**: Can expose native element ref or custom imperative handle
6. **Parent control**: Enables parent components to imperatively control children
