---
title: Use SVGs as React Components
impact: HIGH
impactDescription: Better styling and integration
tags: asset, svg, components, react, vite
---

## Use SVGs as React Components

**Impact: HIGH (Better styling and integration)**

SVGs can be used as images or as React components. Using them as components enables styling with CSS, dynamic colors, and better integration with React.

## Incorrect

```typescript
// Using SVG as image - limited styling options
function Logo() {
  return <img src="/logo.svg" alt="Logo" className="w-8 h-8" />
}

// Inline SVG everywhere - duplicated code
function Icon() {
  return (
    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
      <path d="M12 2L2 7l10 5 10-5-10-5zM2 17l10 5 10-5M2 12l10 5 10-5" />
    </svg>
  )
}
```

## Correct

Install vite-plugin-svgr:

```bash
npm install vite-plugin-svgr -D
```

Configure Vite:

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import svgr from 'vite-plugin-svgr'

export default defineConfig({
  plugins: [
    react(),
    svgr({
      // Export as React component by default
      exportAsDefault: false,
      // SVG options
      svgrOptions: {
        plugins: ['@svgr/plugin-svgo', '@svgr/plugin-jsx'],
        svgoConfig: {
          plugins: [
            {
              name: 'removeViewBox',
              active: false, // Keep viewBox for scaling
            },
          ],
        },
      },
    }),
  ],
})
```

Usage:

```typescript
// Import as React component
import { ReactComponent as Logo } from './assets/logo.svg'
// Or with default export config
import Logo from './assets/logo.svg?react'

// Import as URL (for img src)
import logoUrl from './assets/logo.svg'

function Header() {
  return (
    <header>
      {/* As component - fully styleable */}
      <Logo className="w-8 h-8 text-blue-600 hover:text-blue-700" />

      {/* As image */}
      <img src={logoUrl} alt="Logo" className="w-8 h-8" />
    </header>
  )
}
```

## TypeScript Support

```typescript
// src/vite-env.d.ts
/// <reference types="vite/client" />
/// <reference types="vite-plugin-svgr/client" />

// Or manually declare
declare module '*.svg?react' {
  import type { FunctionComponent, SVGProps } from 'react'
  const content: FunctionComponent<SVGProps<SVGSVGElement>>
  export default content
}

declare module '*.svg' {
  const content: string
  export default content
}
```

## Dynamic SVG Colors

```typescript
// SVG component inherits currentColor
import SearchIcon from './assets/search.svg?react'

function SearchButton({ active }: { active: boolean }) {
  return (
    <button className={active ? 'text-blue-600' : 'text-gray-400'}>
      {/* Icon color follows text color */}
      <SearchIcon className="w-5 h-5" />
      Search
    </button>
  )
}
```

## Icon Component Pattern

```typescript
// components/Icon.tsx
import type { SVGProps, FunctionComponent } from 'react'

// Import all icons
import HomeIcon from '@/assets/icons/home.svg?react'
import SettingsIcon from '@/assets/icons/settings.svg?react'
import UserIcon from '@/assets/icons/user.svg?react'

const icons = {
  home: HomeIcon,
  settings: SettingsIcon,
  user: UserIcon,
} as const

type IconName = keyof typeof icons

interface IconProps extends SVGProps<SVGSVGElement> {
  name: IconName
  size?: number
}

export function Icon({ name, size = 24, className, ...props }: IconProps) {
  const IconComponent = icons[name]
  return (
    <IconComponent
      width={size}
      height={size}
      className={className}
      {...props}
    />
  )
}

// Usage
<Icon name="home" size={20} className="text-gray-600" />
```

## Impact

- SVGs styleable with Tailwind/CSS
- Dynamic colors without multiple SVG files
- Better tree-shaking of unused icons
