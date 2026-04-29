---
name: component-generator
description: >
  Analyze a design reference (Figma screenshot, URL, or existing code) and
  generate or modify production-grade reusable UI components for a Next.js +
  TypeScript + Tailwind codebase. Use when creating new shared components,
  refactoring existing ones, or adapting components to match updated designs.
---

# Component Generator Skill

Convert design references into well-architected shared components — not one-off JSX.

## Stack Assumptions

- Next.js + TypeScript + Tailwind v4
- `cn()` utility, `cva` for variants, `forwardRef` for form controls/buttons
- **Tailwind v4** — tokens consumed from the project's design system, not raw Figma values

## Core Principles

- Abstract the reusable model, don't imitate the screenshot
- Respect existing project conventions (naming, exports, folder structure, tokens)
- Generic APIs — no page-specific text, business rules, or single-screen assumptions
- Support real production states, not just what's visible in the design
- Accessibility is non-negotiable

---

## Phase 1 — Identify Task Type

State which applies before writing any code:

- **New component** — from design reference
- **Modification** — existing component, adding variants/states/API improvements
- **Refactor** — existing component that's too narrow, duplicated, or page-coupled
- **Component family** — design implies multiple related pieces (FormField + Input, Tabs root + item, etc.)

---

## Phase 2 — Analyze the Design

Extract the full component model, not just surface styling:

- **Purpose:** informational / interactive / structural / input-oriented; primitive or composed
- **Variants:** solid/outline/ghost, neutral/brand/destructive, size tiers
- **States:** default, hover, focus, active, disabled, loading, error, selected, expanded
- **Structure:** layout, spacing rhythm, border/radius, icon placement, content slots
- **Composition:** single primitive, wrapper + primitive, compound, or root+item family

---

## Phase 3 — Design the API

Expose the right props, not every imaginable one.

- `variant` — visual role changes, same structure
- `size` — height/padding/text changes systematically
- `status` / `state` — validation or feedback semantics
- Slot props (`leadingIcon`, `trailingIcon`) — when consumer needs control
- Controlled props — for selected value, open state, active item
- Always accept and forward `className` as escape hatch

Avoid: giant prop bags, business-domain names, leaking styling internals.

**When ambiguous:** state the assumption and a default. Ask only if two structurally different APIs are equally plausible.

---

## Phase 4 — Token Alignment

Use semantic classes throughout. If token names are unknown, use sensible placeholders and flag them.

- ✅ `bg-primary`, `text-foreground`, `border-border`, `focus-visible:ring-ring`
- ❌ `bg-[#5869F7]`, `text-[13px]`, `border-[#D0D5DD]`

---

## Phase 5 — Accessibility

- Semantic HTML (`<button>`, not `<div onClick>`)
- `focus-visible:ring-*` on every focusable element
- Icon-only buttons need `aria-label`
- Form controls need `<label htmlFor>`, `aria-describedby` for errors, `aria-invalid`
- Loading states need `aria-busy`; decorative icons need `aria-hidden`

---

## Phase 6 — Code Standards

```ts
// Import order
import * as React from "react";
import { cva, type VariantProps } from "class-variance-authority";
import { cn } from "@/lib/utils";
```

- `forwardRef` on all form controls and buttons
- Export both component and `variants` config so consumers can reuse class strings
- Named exports unless project uses default
- `displayName` on forwardRef components
- No `React.FC`, no `any`, no `as SomeType` casts

**File structure:**

```
components/common/ComponentName/
  ComponentName.tsx
  ComponentName.types.ts   ← only if props are complex
  index.ts
```

---

## Phase 7 — Modification Rules

- Preserve existing public API, stable behaviors, and valid accessibility
- Change: page-specific assumptions, hardcoded values, missing states, brittle API choices
- Explain what changed and why — identify any breaking API changes explicitly

---

## Output

For every component, produce:

1. Component file(s)
2. Props interface with JSDoc on non-obvious props
3. Usage example covering key variants and states
4. Any flagged assumptions or follow-up questions
