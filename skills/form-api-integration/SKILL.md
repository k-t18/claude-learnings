---
name: form-api-integration
description: >
  Wire a form component (built by the form-ui skill) to an API endpoint using
  Zod validation and useApiMutation. Use after the form-ui skill has generated
  the form component and FeatureFormValues type. Requires the existing form
  component, curl request, and response structure. Generates: Zod schema,
  mutation hook, service file (if needed), and a wired parent component that
  connects everything. Never modifies apiClient.ts, useApiQuery.ts, or
  useApiMutation.ts.
---

# Form API Integration Skill

Wire an existing form component to an API. Assumes the `form-ui` skill has already run and produced a `[Feature]Form.tsx` with a typed `onSubmit` prop and `FeatureFormValues` interface.

## Fixed Infrastructure (never modify)

- `src/services/api/apiClient.ts`
- `src/hooks/api/useApiMutation.ts`

## Inputs Required

- **Existing form component** — from `form-ui` skill, must have `FeatureFormValues` and `onSubmit: (values: FeatureFormValues) => void`
- **curl + response structure** — for service file generation
- **Existing service file** — if one exists, extend it; otherwise generate it

If any are missing, ask before generating.

---

## Phase 1 — Parse the API

From curl + response:

- Endpoint URL, HTTP method, request body shape
- Response shape for typing
- Whether a service file exists or needs to be created
- Map `FeatureFormValues` fields to API payload fields — flag any mismatches

---

## Phase 2 — Generate the Zod Schema

Import `FeatureFormValues` from the form component as the base type reference.
Schema field order must match the form's field list.

```ts
// [feature]Form.schema.ts
import { z } from "zod";
import type { FeatureFormValues } from "../../components/feature/FeatureForm";

export const featureFormSchema = z.object({
  name: z.string().min(1, "Name is required"),
  amount: z
    .number({ invalid_type_error: "Amount must be a number" })
    .positive(),
  category: z.string().min(1, "Category is required"),
  date: z.date({ required_error: "Date is required" }),
  notes: z.string().optional(),
}) satisfies z.ZodType<FeatureFormValues>;

export type FeatureFormSchema = z.infer<typeof featureFormSchema>;
```

**Rules:**

- Required strings: `z.string().min(1, 'Field is required')` — never bare `z.string()`
- Numbers from `InputNumber`: `z.number()` not `z.string()`
- Dates from `Calendar`: `z.date()`
- Optional fields: `.optional()` or `.nullable()` based on API shape
- Use `satisfies z.ZodType<FeatureFormValues>` to catch schema/type mismatches at compile time
- Add `.email()` / `.min()` / `.max()` only when implied by field name or API constraints

---

## Phase 3 — Service File

If no service file exists, generate one. If one exists, add only the missing method.

```ts
// services/[domain]/[feature]/[feature].api.ts
import { api } from "../../api/apiClient";
import type { ApiResponse } from "../../api/apiClient";

export interface FeatureItem {
  // typed from response shape
}

export interface CreateFeaturePayload {
  // typed from request body / FeatureFormValues
}

export const featureApi = {
  create: (payload: CreateFeaturePayload): Promise<ApiResponse<FeatureItem>> =>
    api.post("/api/method/...", payload),
};
```

---

## Phase 4 — Mutation Hook

```ts
// hooks/[feature]/useCreateFeature.ts
import { useApiMutation } from "../api/useApiMutation";
import { useQueryClient } from "@tanstack/react-query";
import { featureApi } from "../../services/domain/feature/feature.api";

export function useCreateFeature() {
  const queryClient = useQueryClient();

  return useApiMutation({
    mutationFn: featureApi.create,
    showSuccessToast: true,
    successMessage: "Created successfully",
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["feature"] });
    },
  });
}
```

---

## Phase 5 — Wired Parent Component

Create a wrapper that owns the mutation and passes `onSubmit` + `isSubmitting` down to the form:

```tsx
// [Feature]FormContainer.tsx
import { zodResolver } from "@hookform/resolvers/zod";
import { FeatureForm } from "./FeatureForm";
import {
  featureFormSchema,
  type FeatureFormSchema,
} from "./featureForm.schema";
import { useCreateFeature } from "../../hooks/feature/useCreateFeature";

export function FeatureFormContainer() {
  const createFeature = useCreateFeature();

  const handleSubmit = (values: FeatureFormSchema) => {
    createFeature.mutate(values);
  };

  return (
    <FeatureForm
      onSubmit={handleSubmit}
      isSubmitting={createFeature.isPending}
      resolver={zodResolver(featureFormSchema)}
    />
  );
}
```

> **Note:** Pass `resolver` as a prop into `FeatureForm` so the form component stays schema-agnostic. Update `FeatureFormProps` to accept `resolver?: Resolver<FeatureFormValues>` if not already present.

**After successful submit:** form reset is called inside `FeatureForm` via `onSuccess` callback passed from the container, or the mutation hook's `onSuccess` triggers a reset if the form exposes a `reset` ref. Keep reset logic in the form component — the container only handles submission.

---

## File Structure

```
src/
  services/[domain]/[feature]/
    [feature].api.ts
  hooks/[feature]/
    use[Feature].ts
  components/[feature]/
    [Feature]Form.tsx               ← from form-ui skill (unchanged)
    [Feature]FormContainer.tsx      ← new: owns mutation + wiring
    [feature]Form.schema.ts         ← new: Zod schema
```

---

## Output Order

1. **Zod schema** (`[feature]Form.schema.ts`)
2. **Service file** — new or diff if extending existing
3. **Mutation hook** (`use[Feature].ts`)
4. **Container component** (`[Feature]FormContainer.tsx`)
5. **Any changes needed to `[Feature]Form.tsx`** — only if `resolver` prop needs adding
6. **Assumptions** — field mapping, query key, folder path, any payload transform needed
