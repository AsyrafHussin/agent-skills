---
title: Use Dynamic Imports for Heavy Components
impact: CRITICAL
impactDescription: 30-50% reduction in initial bundle
tags: split, dynamic-imports, lazy-loading, code-splitting, react
---

## Use Dynamic Imports for Heavy Components

**Impact: CRITICAL (30-50% reduction in initial bundle)**

Heavy components like charts, editors, and complex forms shouldn't be loaded until needed. Dynamic imports allow loading code on-demand, reducing initial bundle size.

## Incorrect

```typescript
// All heavy libraries loaded upfront
import { Chart } from 'chart.js'
import ReactQuill from 'react-quill'
import { PDFViewer } from '@react-pdf/renderer'
import MonacoEditor from '@monaco-editor/react'

function Dashboard() {
  const [showChart, setShowChart] = useState(false)

  return (
    <div>
      <button onClick={() => setShowChart(true)}>Show Chart</button>
      {showChart && <Chart data={data} />}
    </div>
  )
}
```

**Problem:** Chart.js, React Quill, PDF renderer, and Monaco are all loaded even if never used.

## Correct

```typescript
import { lazy, Suspense, useState } from 'react'

// Lazy load heavy components
const Chart = lazy(() => import('./components/Chart'))
const Editor = lazy(() => import('./components/Editor'))
const PDFViewer = lazy(() => import('./components/PDFViewer'))

function Dashboard() {
  const [showChart, setShowChart] = useState(false)
  const [showEditor, setShowEditor] = useState(false)

  return (
    <div>
      <button onClick={() => setShowChart(true)}>Show Chart</button>
      <button onClick={() => setShowEditor(true)}>Show Editor</button>

      {showChart && (
        <Suspense fallback={<ChartSkeleton />}>
          <Chart data={data} />
        </Suspense>
      )}

      {showEditor && (
        <Suspense fallback={<EditorSkeleton />}>
          <Editor />
        </Suspense>
      )}
    </div>
  )
}
```

## Conditional Dynamic Import

```typescript
// Load library only when feature is activated
async function exportToPDF() {
  // pdf-lib is only loaded when user clicks export
  const { PDFDocument } = await import('pdf-lib')

  const pdfDoc = await PDFDocument.create()
  // ... generate PDF
}

function ExportButton() {
  const [loading, setLoading] = useState(false)

  const handleExport = async () => {
    setLoading(true)
    await exportToPDF()
    setLoading(false)
  }

  return (
    <button onClick={handleExport} disabled={loading}>
      {loading ? 'Generating...' : 'Export PDF'}
    </button>
  )
}
```

## Preload on Interaction

```typescript
const HeavyModal = lazy(() => import('./HeavyModal'))

function ModalTrigger() {
  const [isOpen, setIsOpen] = useState(false)

  // Preload when user shows intent
  const preload = () => {
    import('./HeavyModal')
  }

  return (
    <>
      <button
        onClick={() => setIsOpen(true)}
        onMouseEnter={preload}
        onFocus={preload}
      >
        Open Settings
      </button>

      {isOpen && (
        <Suspense fallback={<ModalSkeleton />}>
          <HeavyModal onClose={() => setIsOpen(false)} />
        </Suspense>
      )}
    </>
  )
}
```

## Feature Flag Based Loading

```typescript
// Only load admin features for admin users
function App({ user }) {
  const AdminPanel = user.isAdmin
    ? lazy(() => import('./AdminPanel'))
    : null

  return (
    <div>
      <MainContent />
      {AdminPanel && (
        <Suspense fallback={<Loading />}>
          <AdminPanel />
        </Suspense>
      )}
    </div>
  )
}
```

## Heavy Library Examples

Libraries that should typically be dynamically imported:
- Chart libraries (Chart.js, Recharts, D3)
- Rich text editors (React Quill, TipTap, Slate)
- Code editors (Monaco, CodeMirror)
- PDF libraries (react-pdf, pdf-lib)
- Date pickers with locales
- Map libraries (Mapbox, Google Maps)
- Markdown renderers

## Impact

- Initial bundle can be 50%+ smaller
- Faster Time to Interactive
- Better user experience on slow connections
