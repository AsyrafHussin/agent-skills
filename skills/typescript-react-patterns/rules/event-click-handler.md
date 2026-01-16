# Click Event Handler Typing

Properly typing click event handlers for various React elements.

## Bad Example

```tsx
// Using 'any' for event parameter
const handleClick = (e: any) => {
  console.log(e.target.value);
};

// Missing event type entirely
const onClick = (e) => {
  e.preventDefault();
};

// Using wrong event type
const handleButtonClick = (e: React.MouseEvent<HTMLDivElement>) => {
  // Should be HTMLButtonElement for button clicks
};

// Not handling synthetic events properly
const handleLinkClick = (e: MouseEvent) => {
  // Should be React.MouseEvent, not DOM MouseEvent
  e.preventDefault();
};
```

## Good Example

```tsx
import React, { useCallback } from 'react';

// Button click with proper typing
const handleButtonClick = (event: React.MouseEvent<HTMLButtonElement>) => {
  console.log('Button clicked');
  console.log('Button text:', event.currentTarget.textContent);
};

// Div/container click
const handleContainerClick = (event: React.MouseEvent<HTMLDivElement>) => {
  // Stop propagation to parent handlers
  event.stopPropagation();
  console.log('Container clicked at:', event.clientX, event.clientY);
};

// Link click with preventDefault
const handleLinkClick = (event: React.MouseEvent<HTMLAnchorElement>) => {
  event.preventDefault();
  const href = event.currentTarget.href;
  console.log('Navigating to:', href);
  // Custom navigation logic
};

// Generic click handler for reusable components
type ClickHandler<T extends HTMLElement = HTMLElement> = (
  event: React.MouseEvent<T>
) => void;

// Click with data attribute
interface ItemProps {
  id: string;
  name: string;
  onClick: (id: string) => void;
}

function Item({ id, name, onClick }: ItemProps): React.ReactElement {
  const handleClick = useCallback(
    (event: React.MouseEvent<HTMLLIElement>) => {
      event.stopPropagation();
      onClick(id);
    },
    [id, onClick]
  );

  return <li onClick={handleClick}>{name}</li>;
}

// Click handler with multiple possible elements
const handleAnyClick = (
  event: React.MouseEvent<HTMLButtonElement | HTMLAnchorElement>
) => {
  // Works for both button and anchor elements
  event.preventDefault();
  console.log('Element clicked:', event.currentTarget.tagName);
};

// Using event properties safely
const handleClickWithCoordinates = (event: React.MouseEvent<HTMLDivElement>) => {
  const { clientX, clientY, pageX, pageY, screenX, screenY } = event;

  console.log('Client coordinates:', clientX, clientY);
  console.log('Page coordinates:', pageX, pageY);
  console.log('Screen coordinates:', screenX, screenY);

  // Check for modifier keys
  if (event.ctrlKey || event.metaKey) {
    console.log('Ctrl/Cmd + Click');
  }
  if (event.shiftKey) {
    console.log('Shift + Click');
  }
  if (event.altKey) {
    console.log('Alt + Click');
  }
};

// Double click handler
const handleDoubleClick = (event: React.MouseEvent<HTMLDivElement>) => {
  console.log('Double clicked!');
};

// Right click (context menu) handler
const handleContextMenu = (event: React.MouseEvent<HTMLDivElement>) => {
  event.preventDefault(); // Prevent default context menu
  console.log('Right click at:', event.clientX, event.clientY);
  // Show custom context menu
};

// Click handler factory for list items
function createClickHandler<T extends { id: string }>(
  item: T,
  callback: (item: T) => void
): React.MouseEventHandler<HTMLElement> {
  return (event) => {
    event.stopPropagation();
    callback(item);
  };
}

// Usage in component
interface Product {
  id: string;
  name: string;
  price: number;
}

function ProductList({
  products,
  onSelect,
}: {
  products: Product[];
  onSelect: (product: Product) => void;
}): React.ReactElement {
  return (
    <ul>
      {products.map((product) => (
        <li key={product.id} onClick={createClickHandler(product, onSelect)}>
          {product.name} - ${product.price}
        </li>
      ))}
    </ul>
  );
}

// Component with optional click handler
interface CardProps {
  title: string;
  children: React.ReactNode;
  onClick?: React.MouseEventHandler<HTMLDivElement>;
  isClickable?: boolean;
}

function Card({
  title,
  children,
  onClick,
  isClickable = !!onClick,
}: CardProps): React.ReactElement {
  return (
    <div
      className={`card ${isClickable ? 'clickable' : ''}`}
      onClick={onClick}
      role={isClickable ? 'button' : undefined}
      tabIndex={isClickable ? 0 : undefined}
    >
      <h2>{title}</h2>
      {children}
    </div>
  );
}

// Accessible click handler (keyboard support)
function AccessibleButton({
  onClick,
  children,
}: {
  onClick: () => void;
  children: React.ReactNode;
}): React.ReactElement {
  const handleClick = (event: React.MouseEvent<HTMLDivElement>) => {
    onClick();
  };

  const handleKeyDown = (event: React.KeyboardEvent<HTMLDivElement>) => {
    if (event.key === 'Enter' || event.key === ' ') {
      event.preventDefault();
      onClick();
    }
  };

  return (
    <div
      role="button"
      tabIndex={0}
      onClick={handleClick}
      onKeyDown={handleKeyDown}
    >
      {children}
    </div>
  );
}

// Using MouseEventHandler type alias
type ButtonClickHandler = React.MouseEventHandler<HTMLButtonElement>;

const myButtonHandler: ButtonClickHandler = (event) => {
  console.log('Button:', event.currentTarget.name);
};

// Forwarding click events
interface WrapperProps {
  children: React.ReactElement;
  onBeforeClick?: () => boolean; // Return false to prevent click
}

function ClickWrapper({ children, onBeforeClick }: WrapperProps): React.ReactElement {
  const handleClick = (event: React.MouseEvent) => {
    if (onBeforeClick && !onBeforeClick()) {
      event.preventDefault();
      event.stopPropagation();
      return;
    }

    // Forward to child's onClick if it exists
    const childOnClick = children.props.onClick;
    if (childOnClick) {
      childOnClick(event);
    }
  };

  return React.cloneElement(children, { onClick: handleClick });
}
```

## Why

1. **Type safety**: Proper event types catch errors like accessing wrong properties
2. **Element matching**: Event type should match the actual HTML element
3. **React vs DOM events**: Use `React.MouseEvent`, not native `MouseEvent`
4. **currentTarget vs target**: `currentTarget` is typed, `target` needs assertion
5. **Handler type aliases**: `React.MouseEventHandler<T>` simplifies function signatures
6. **Accessibility**: Click handlers on non-button elements need keyboard support
