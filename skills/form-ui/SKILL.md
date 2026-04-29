---
name: form-ui
description: >
  Build a fully styled form UI component using PrimeReact inputs, React Hook
  Form Controller pattern, and Tailwind spacing tokens. Use when the user
  provides a Figma MCP link and a field list. Generates the form component with
  typed props, Controller-wrapped PrimeReact inputs, and a void onSubmit
  placeholder — ready for the form-api-integration skill to wire up. Does not
  generate Zod schema or mutation logic.
---

# Form UI Skill

Build the visual form component from a Figma reference. API wiring is handled separately by the `form-api-integration` skill.

## Inputs Required

- **Figma MCP link** — drives layout, field order, labels, input types, spacing
- **Field list** — field names and types (from API payload or user description)

If either is missing, ask before generating.

---

## Phase 1 — Read the Figma Reference

From the Figma MCP link, extract:

- Form title and submit button label
- Field order and visual grouping
- Layout structure (single column, two column, sections)
- Labels, placeholders, helper text, any visible validation hints
- Input type per field — use the mapping below if not explicit

**PrimeReact input mapping:**

| Field type        | Component       |
| ----------------- | --------------- |
| Short text        | `InputText`     |
| Long text         | `InputTextarea` |
| Number            | `InputNumber`   |
| Date / date range | `Calendar`      |
| Single select     | `Dropdown`      |
| Multi select      | `MultiSelect`   |
| Boolean / toggle  | `Checkbox`      |
| Password          | `Password`      |

When Figma and the field list conflict, prefer Figma for input type, prefer field list for naming.

---

## Phase 2 — Generate the Form Component

### File structure

```
src/components/[feature]/
  [Feature]Form.tsx
```

### Props interface

Always expose `onSubmit` as a typed void prop — never hardcode submission logic:

```ts
export interface FeatureFormProps {
  onSubmit: (values: FeatureFormValues) => void;
  isSubmitting?: boolean;
}
```

### Form values type

Derive field types from the field list. Place inline in the component file:

```ts
export interface FeatureFormValues {
  name: string;
  amount: number | undefined;
  category: string;
  date: Date | undefined;
  notes?: string;
}
```

### Component shell

```tsx
import { useForm, Controller } from 'react-hook-form';
import { InputText } from 'primereact/inputtext';
import { Button } from 'primereact/button';
// ... other PrimeReact imports

export interface FeatureFormValues { ... }

export interface FeatureFormProps {
  onSubmit: (values: FeatureFormValues) => void;
  isSubmitting?: boolean;
}

export function FeatureForm({ onSubmit, isSubmitting = false }: FeatureFormProps) {
  const { control, handleSubmit, formState: { errors } } = useForm<FeatureFormValues>({
    defaultValues: {
      name: '',
      amount: undefined,
      // ...
    },
  });

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="flex flex-col gap-6">
      {/* fields */}
      <Button
        type="submit"
        label="Submit"
        loading={isSubmitting}
        disabled={isSubmitting}
      />
    </form>
  );
}
```

---

## Phase 3 — Controller Patterns

Always use `Controller` — never `register`. PrimeReact inputs are uncontrolled.

**InputText:**

```tsx
<Controller
  name="name"
  control={control}
  render={({ field }) => (
    <div className="flex flex-col gap-1">
      <label htmlFor="name" className="text-sm font-medium text-foreground">
        Name
      </label>
      <InputText
        id="name"
        {...field}
        className={errors.name ? "p-invalid" : ""}
      />
      {errors.name && <small className="p-error">{errors.name.message}</small>}
    </div>
  )}
/>
```

**InputNumber:**

```tsx
<Controller
  name="amount"
  control={control}
  render={({ field }) => (
    <div className="flex flex-col gap-1">
      <label htmlFor="amount">Amount</label>
      <InputNumber
        id="amount"
        value={field.value}
        onValueChange={(e) => field.onChange(e.value)}
        className={errors.amount ? "p-invalid" : ""}
      />
      {errors.amount && (
        <small className="p-error">{errors.amount.message}</small>
      )}
    </div>
  )}
/>
```

**Dropdown:**

```tsx
<Controller
  name="category"
  control={control}
  render={({ field }) => (
    <div className="flex flex-col gap-1">
      <label htmlFor="category">Category</label>
      <Dropdown
        id="category"
        value={field.value}
        onChange={(e) => field.onChange(e.value)}
        options={[]} {/* consumer provides */}
        className={errors.category ? 'p-invalid' : ''}
      />
      {errors.category && <small className="p-error">{errors.category.message}</small>}
    </div>
  )}
/>
```

**Calendar:**

```tsx
<Calendar
  id="date"
  value={field.value}
  onChange={(e) => field.onChange(e.value)}
  className={errors.date ? "p-invalid" : ""}
/>
```

**MultiSelect:**

```tsx
<MultiSelect
  id="tags"
  value={field.value}
  onChange={(e) => field.onChange(e.value)}
  options={[]} {/* consumer provides */}
  className={errors.tags ? 'p-invalid' : ''}
/>
```

**Checkbox:**

```tsx
<Checkbox
  inputId="active"
  checked={field.value}
  onChange={(e) => field.onChange(e.checked)}
/>
```

**InputTextarea:**

```tsx
<InputTextarea
  id="notes"
  {...field}
  rows={4}
  className={errors.notes ? "p-invalid" : ""}
/>
```

---

## Phase 4 — Layout

Derive layout entirely from Figma — spacing, columns, section grouping.

- Use Tailwind spacing tokens for gaps and padding (`gap-4`, `gap-6`, `p-6`)
- Single column: `flex flex-col gap-6`
- Two column: `grid grid-cols-2 gap-4`
- Sections: wrap in `<div className="flex flex-col gap-4">` with an optional section label
- Never use PrimeReact layout utilities (`p-fluid`, `p-grid`)

---

## Output Order

1. **`[Feature]Form.tsx`** — complete component
2. **Handoff note** — lists `FeatureFormValues` type and `onSubmit` prop signature for the `form-api-integration` skill
3. **Assumptions** — inferred input types, layout decisions, any fields skipped
