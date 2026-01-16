---
title: Polymorphic Component Typing
category: Generic Components
priority: MEDIUM
---


Creating type-safe polymorphic components that can render as different HTML elements.

## Bad Example

```tsx
// Using 'any' loses all type safety
interface BoxProps {
  as?: any;
  children?: React.ReactNode;
}

const Box = ({ as: Component = 'div', ...props }: BoxProps) => {
  return <Component {...props} />;
};

// No validation of props for the rendered element
<Box as="a">This should have href</Box> // No type error, but missing required href

// String union doesn't provide prop inference
interface TextProps {
  as?: 'h1' | 'h2' | 'h3' | 'p' | 'span';
  children: React.ReactNode;
}

const Text = ({ as: Component = 'p', children }: TextProps) => {
  return <Component>{children}</Component>;
};
// Can't pass element-specific props like htmlFor to label
```

## Good Example

```tsx
import React from 'react';

// Core polymorphic types
type AsProp<C extends React.ElementType> = {
  as?: C;
};

type PropsToOmit<C extends React.ElementType, P> = keyof (AsProp<C> & P);

// Props without ref
type PolymorphicComponentProps<
  C extends React.ElementType,
  Props = object
> = Props &
  AsProp<C> &
  Omit<React.ComponentPropsWithoutRef<C>, PropsToOmit<C, Props>>;

// Props with ref
type PolymorphicComponentPropsWithRef<
  C extends React.ElementType,
  Props = object
> = PolymorphicComponentProps<C, Props> & {
  ref?: PolymorphicRef<C>;
};

type PolymorphicRef<C extends React.ElementType> =
  React.ComponentPropsWithRef<C>['ref'];

// Simple polymorphic Box component
interface BoxOwnProps {
  padding?: 'none' | 'sm' | 'md' | 'lg';
  margin?: 'none' | 'sm' | 'md' | 'lg';
  display?: 'block' | 'flex' | 'grid' | 'inline';
}

type BoxProps<C extends React.ElementType = 'div'> = PolymorphicComponentProps<C, BoxOwnProps>;

function Box<C extends React.ElementType = 'div'>({
  as,
  padding = 'none',
  margin = 'none',
  display = 'block',
  className,
  style,
  ...rest
}: BoxProps<C>): React.ReactElement {
  const Component = as ?? 'div';

  const computedStyle: React.CSSProperties = {
    padding: padding !== 'none' ? `var(--spacing-${padding})` : undefined,
    margin: margin !== 'none' ? `var(--spacing-${margin})` : undefined,
    display,
    ...style,
  };

  return <Component className={className} style={computedStyle} {...rest} />;
}

// Usage with full type inference
<Box padding="md">Default div</Box>
<Box as="section" padding="lg">Section element</Box>
<Box as="a" href="/home">Link with href autocomplete</Box>
<Box as="button" onClick={() => {}}>Button with onClick</Box>

// Polymorphic component with forwardRef
interface TextOwnProps {
  variant?: 'heading' | 'body' | 'caption';
  weight?: 'normal' | 'medium' | 'bold';
  color?: 'primary' | 'secondary' | 'muted';
}

type TextProps<C extends React.ElementType = 'span'> =
  PolymorphicComponentPropsWithRef<C, TextOwnProps>;

type TextComponent = <C extends React.ElementType = 'span'>(
  props: TextProps<C>
) => React.ReactElement | null;

const Text: TextComponent = React.forwardRef(
  <C extends React.ElementType = 'span'>(
    {
      as,
      variant = 'body',
      weight = 'normal',
      color = 'primary',
      className,
      ...rest
    }: TextProps<C>,
    ref?: PolymorphicRef<C>
  ) => {
    const Component = as ?? 'span';

    const classes = [
      className,
      `text-${variant}`,
      `font-${weight}`,
      `color-${color}`,
    ].filter(Boolean).join(' ');

    return <Component ref={ref} className={classes} {...rest} />;
  }
);

// Usage with refs
const headingRef = React.useRef<HTMLHeadingElement>(null);
<Text as="h1" ref={headingRef} variant="heading">Heading</Text>

// Constrained polymorphic component (only certain elements allowed)
type HeadingLevel = 'h1' | 'h2' | 'h3' | 'h4' | 'h5' | 'h6';

interface HeadingOwnProps {
  level?: HeadingLevel;
}

type HeadingProps = HeadingOwnProps &
  Omit<React.HTMLAttributes<HTMLHeadingElement>, keyof HeadingOwnProps>;

function Heading({
  level = 'h2',
  className,
  ...rest
}: HeadingProps): React.ReactElement {
  const Component = level;
  return <Component className={`heading heading-${level} ${className ?? ''}`} {...rest} />;
}

// Polymorphic with discriminated unions
type ButtonVariant = 'solid' | 'outline' | 'ghost';

interface ButtonBaseProps {
  variant?: ButtonVariant;
  size?: 'sm' | 'md' | 'lg';
  isLoading?: boolean;
}

type ButtonAsButton = ButtonBaseProps &
  Omit<React.ButtonHTMLAttributes<HTMLButtonElement>, keyof ButtonBaseProps> & {
    as?: 'button';
  };

type ButtonAsLink = ButtonBaseProps &
  Omit<React.AnchorHTMLAttributes<HTMLAnchorElement>, keyof ButtonBaseProps> & {
    as: 'a';
  };

type ButtonProps = ButtonAsButton | ButtonAsLink;

function Button(props: ButtonProps): React.ReactElement {
  const {
    as = 'button',
    variant = 'solid',
    size = 'md',
    isLoading = false,
    className,
    children,
    ...rest
  } = props;

  const classes = `btn btn-${variant} btn-${size} ${className ?? ''}`;

  if (as === 'a') {
    return (
      <a className={classes} {...(rest as React.AnchorHTMLAttributes<HTMLAnchorElement>)}>
        {isLoading ? 'Loading...' : children}
      </a>
    );
  }

  return (
    <button
      className={classes}
      disabled={isLoading}
      {...(rest as React.ButtonHTMLAttributes<HTMLButtonElement>)}
    >
      {isLoading ? 'Loading...' : children}
    </button>
  );
}

// Usage with proper type narrowing
<Button onClick={() => console.log('clicked')}>Click me</Button>
<Button as="a" href="/home">Go home</Button>
```

## Why

1. **Full type safety**: Props are validated based on the rendered element
2. **Autocomplete**: IDE suggests valid props for each element type
3. **Flexibility**: Single component can render as any HTML element or custom component
4. **Ref support**: Properly typed refs match the rendered element
5. **Constraint options**: Can limit to specific elements when needed
6. **Component reuse**: Reduces need for wrapper components like LinkButton, SubmitButton, etc.
