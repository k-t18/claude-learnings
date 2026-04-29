---
name: design-system-setup
description: >
  Set up, audit, or normalize a design system for a Next.js or React project
  using Tailwind v4. Use when the user wants to extract design tokens from
  Figma or screenshots, sync design to code, standardize colors/typography/
  spacing, create semantic tokens, introduce theming, or reconcile design drift.
---

# Design System Setup Skill

Establish a token architecture that makes future frontend work fast, consistent, and scalable.

## Stack Assumptions

- **Next.js or React**, App Router or routes folder
- **Tailwind v4** — tokens via CSS variables and `@theme`
- Single source of truth preferred; no parallel config layers

## Core Principles

- Audit before generating — never assume project structure
- Prefer structured design data over screenshots; respect existing conventions
- Separate confirmed tokens from inferred ones from fallbacks
- Semantic tokens in components, not raw palette values

---

## Phase 1 — Audit

Check the repo for: global stylesheet, Tailwind config, PostCSS config, token JSON, font setup, TypeScript token exports.

Flag any existing drift: raw hex values in components, duplicate semantic names, palette used directly instead of semantically, missing dark mode structure.

**Output:** detected stack, existing files worth preserving, gaps, recommended path.

---

## Phase 2 — Extract Tokens

**Source priority:** structured API/MCP → token JSON → existing code → screenshots → verbal description.

**Foundation tokens** (primitive values): color palettes, type scale, font families/weights/line-heights/tracking, spacing, border radius, shadows, opacity, motion, breakpoints.

**Semantic tokens** (purpose, not appearance): background/surface, foreground/text, muted, border, ring, accent, card, primary, secondary, success/warning/danger/info, disabled, overlay.

**Component tokens** (infer cautiously from consistent patterns): button/input/badge/card/table/toast states.

**Interaction states to identify:** default, hover, active, focus, disabled, selected, error, loading.

**Theme modes:** extract each explicitly if present (light, dark, high-contrast). Do not invent a mode system unless asked.

---

## Phase 3 — Normalize

Convert designer naming into stable engineering naming. Avoid: `blue2`, `main`, `text dark 2`. Separate palette names from semantic names.

**Three layers:**

- Foundation: `color.palette.blue.500`, `space.4`, `radius.md`
- Semantic: `color.text.primary`, `color.surface.canvas`, `color.border.default`
- Component: `button.primary.bg.default`, `input.border.focus`

**Confidence levels on every token:** Confirmed / Inferred / Assumed / Missing.

---

## Phase 4 — Token Summary

Before writing files, produce this table structure:

```md
### Environment

- Framework / Router / Tailwind version / Existing strategy / Recommended path

### Foundation Tokens

| Token | Value | Source | Confidence |

### Semantic Tokens

| Token | Maps To | Confidence |

### Missing / Ambiguous
```

If confidence is high, proceed. Pause only for ambiguity that would materially break the system.

---

## Phase 5 — Implementation

**Tailwind v4:** use `@theme` for token definition, CSS variables for runtime theming. Keep them consistent — no parallel layers.

**Existing token system:** extend or normalize it. Do not duplicate sources of truth.

**Generate only what's needed.** Possible outputs: global CSS variables, `@theme` block, token JSON, TypeScript constants, font config, usage docs.

---

## Phase 6 — Generation Rules

- **One source of truth.** If multiple artifacts exist, their relationship must be explicit.
- **Semantic utilities in components** — `bg-primary`, `text-muted-foreground`, not raw palette.
- **Dark mode:** implement if in source. If absent, mark as "pending design input" — do not fabricate.
- **Fonts:** reuse existing loading strategy. Don't conflict with runtime font loading.
- **Spacing:** extend default scale unless replacement is required.
- **Accessibility check before finalizing:** text contrast, focus visibility, disabled-state clarity, state distinguishability. Report issues explicitly.

---

## Phase 7 — Output Order

1. Audit summary
2. Token summary table
3. Recommended approach
4. Generated/updated files
5. What changed and why
6. Usage guidance for future development
7. Open issues or follow-up suggestions
8. `design-system.md` at project root with usage examples

---

## Checklist

- [ ] Stack and existing token strategy detected
- [ ] Foundation + semantic tokens established; confidence levels noted
- [ ] Single source of truth; Tailwind v4 integration correct
- [ ] Fonts aligned; dark mode handled per source availability
- [ ] Semantic utilities usable in components
- [ ] Missing/ambiguous tokens documented
- [ ] Accessibility risks reviewed

## Pitfalls

- Don't duplicate token sources or silently invent dark mode
- Don't treat screenshot values as confirmed
- Don't preserve messy designer naming in code
- Don't flatten all three token layers into one
- Don't overwrite existing system files destructively
