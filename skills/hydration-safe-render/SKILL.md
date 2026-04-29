---
name: hydration-safe-render
description: >
  Eliminate hydration mismatches and visual flickering when rendering content
  that depends on client-only storage (localStorage, cookies, sessionStorage).
  Use when a component reads client-side state during render — theme toggles,
  user preferences, auth state, feature flags, or any value that differs
  between server and client. Generates the correct inline script pattern so
  the DOM is updated synchronously before React hydrates.
tags: rendering, ssr, hydration, localStorage, flicker, next.js, react
---

# Hydration-Safe Render Skill

Prevent SSR breakage and post-hydration flickering by injecting a synchronous inline script that updates the DOM before React hydrates — the same technique used by Next.js themes, shadcn/ui, and most production apps that persist user preferences.

---

## When to Use

- Theme / dark mode toggles stored in `localStorage`
- User preferences (language, layout, font size)
- Auth state from cookies that affects initial render
- Feature flags persisted client-side
- Any value that differs between server and client and affects visible layout

---

## Why Other Approaches Break

**Accessing `localStorage` directly during render:**

```tsx
// WRONG — throws on server: "localStorage is not defined"
const theme = localStorage.getItem('theme') || 'light'
```

**Reading in `useEffect`:**

```tsx
// WRONG — renders with default first, then corrects after hydration = flash
useEffect(() => {
  const stored = localStorage.getItem('theme')
  if (stored) setTheme(stored)
}, [])
```

The component mounts with the default value (`light`), paints, then `useEffect` fires and updates it — producing a visible flash of the wrong state on every page load.

---

## The Correct Pattern

Inject a `<script>` tag that runs synchronously before the browser paints. The script reads `localStorage` and sets the correct class/attribute on the wrapper element — so React hydrates onto a DOM that already has the right value.

### Theme wrapper

```tsx
// components/ThemeWrapper.tsx
export function ThemeWrapper({ children }: { children: React.ReactNode }) {
  return (
    <>
      <div id="theme-wrapper">
        {children}
      </div>
      <script
        dangerouslySetInnerHTML={{
          __html: `
            (function() {
              try {
                var theme = localStorage.getItem('theme') || 'light';
                var el = document.getElementById('theme-wrapper');
                if (el) el.className = theme;
              } catch (e) {}
            })();
          `,
        }}
      />
    </>
  )
}
```

### Next.js App Router — inject at the `<html>` level

For dark mode that sets a class on `<html>` itself (the standard Next.js pattern):

```tsx
// app/layout.tsx
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" suppressHydrationWarning>
      <head />
      <body>
        <script
          dangerouslySetInnerHTML={{
            __html: `
              (function() {
                try {
                  var theme = localStorage.getItem('theme');
                  var prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
                  var resolved = theme || (prefersDark ? 'dark' : 'light');
                  document.documentElement.classList.add(resolved);
                } catch (e) {}
              })();
            `,
          }}
        />
        {children}
      </body>
    </html>
  )
}
```

> `suppressHydrationWarning` on `<html>` suppresses the expected mismatch on the `class` attribute — this is intentional and safe here because the script reconciles the value before React touches it.

### Cookie-based preference (works with SSR / middleware)

When the value is available server-side via cookies, read it directly in the Server Component — no inline script needed:

```tsx
// app/layout.tsx (Server Component)
import { cookies } from 'next/headers'

export default function RootLayout({ children }: { children: React.ReactNode }) {
  const theme = cookies().get('theme')?.value ?? 'light'

  return (
    <html lang="en" className={theme}>
      <body>{children}</body>
    </html>
  )
}
```

Prefer this over the inline script approach when the value is already in a cookie — no client-side script, no flash, fully SSR-rendered.

---

## Phase 1 — Identify the Storage Source

Ask or infer:

| Source | Approach |
|---|---|
| `localStorage` | Inline script on client (script pattern above) |
| Cookie (readable server-side) | Read in Server Component via `cookies()` |
| Cookie (client-only, HttpOnly) | Inline script reading `document.cookie` |
| `sessionStorage` | Inline script (same pattern, replace `localStorage`) |
| `window.matchMedia` only | Inline script with `matchMedia` check, no storage read |

---

## Phase 2 — Generate the Implementation

Based on the source:

1. **Script content** — reads the value, applies it to the correct DOM target (`document.documentElement`, a wrapper `div`, `data-*` attribute, or CSS variable)
2. **Wrapper component or layout placement** — where the script lives (layout root vs. component-level)
3. **`suppressHydrationWarning`** — add only to the element whose attribute the script mutates
4. **React state hook** — if the value also needs to be reactive after mount, provide a `useTheme`-style hook that reads the already-applied DOM value rather than re-reading storage

### Reactive hook pattern (post-hydration updates)

```tsx
// hooks/useTheme.ts
import { useEffect, useState } from 'react'

export function useTheme() {
  // Read the value already applied by the inline script — no flash
  const [theme, setTheme] = useState<string>(() => {
    if (typeof window === 'undefined') return 'light'
    return document.documentElement.classList.contains('dark') ? 'dark' : 'light'
  })

  function toggle() {
    const next = theme === 'dark' ? 'light' : 'dark'
    document.documentElement.classList.replace(theme, next)
    localStorage.setItem('theme', next)
    setTheme(next)
  }

  return { theme, toggle }
}
```

---

## Phase 3 — Checklist Before Outputting

- [ ] Script is wrapped in an IIFE and has a `try/catch` — never let it crash the page
- [ ] `suppressHydrationWarning` is on the element the script mutates, not globally
- [ ] Script is placed **before** the content it affects in the DOM
- [ ] No direct `localStorage` access outside the script or a `typeof window` guard
- [ ] Cookie approach used instead of script where the value is server-readable
- [ ] Reactive hook initializes from the DOM state, not from a fresh `localStorage` read

---

## Output Order

1. **Inline script / layout change** — the core fix
2. **Reactive hook** (if needed for toggle/update behavior)
3. **Assumptions** — storage key name, default value, target element, whether dark mode uses class or `data-theme`
