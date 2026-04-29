---
name: api-integration
description: >
  Generate API integration code for a React + TypeScript + TanStack Query
  codebase. Use when the user wants to integrate a new API endpoint or modify
  an existing one. Requires endpoint info (curl, response shape, or existing
  service). Generates: service file and custom hook. Never modifies apiClient.ts,
  useApiQuery.ts, or useApiMutation.ts — those are fixed infrastructure.
---

# API Integration Skill

Generate a service file and custom hook for a given API endpoint.

## Fixed Infrastructure (never modify)

- `src/services/api/apiClient.ts` — `api.get/post/put/patch/delete`, `ApiError`
- `src/hooks/api/useApiQuery.ts` — `useApiQuery`
- `src/hooks/api/useApiMutation.ts` — `useApiMutation`

## Output Files

```
src/
  services/[domain]/[feature]/
    [feature].api.ts          ← service: endpoint functions
  hooks/[feature]/
    use[Feature].ts           ← custom hook: TanStack Query logic
```

---

## Phase 1 — Parse the Request

Extract from the user's prompt:

- **Endpoint(s):** URL, method, path params, query params, request body shape
- **Response shape:** from curl output, sample JSON, or described structure
- **Feature name:** derive from the endpoint or resource name
- **Domain folder:** infer from URL path (e.g. `/transactions/invoices` → `services/transactions/invoices/`)
- **Operation type:** query (GET) or mutation (POST/PUT/PATCH/DELETE)
- **Existing service:** if provided, extend it rather than recreate

If endpoint info is missing or ambiguous, ask before generating.

---

## Phase 2 — Generate the Service File

### Naming

- File: `[feature].api.ts`
- Export: `const [feature]Api = { ... }`
- Function names: `getList`, `getById`, `create`, `update`, `remove`

### Pattern

```ts
import { api } from "../../api/apiClient";
import type { ApiResponse } from "../../api/apiClient";

export interface FeatureItem {
  // typed from response shape
}

export interface CreateFeaturePayload {
  // typed from request body
}

export const featureApi = {
  getList: (): Promise<ApiResponse<FeatureItem[]>> =>
    api.get("/api/method/..."),

  getById: (id: string): Promise<ApiResponse<FeatureItem>> =>
    api.get(`/api/method/.../${id}`),

  create: (payload: CreateFeaturePayload): Promise<ApiResponse<FeatureItem>> =>
    api.post("/api/method/...", payload),

  update: (
    id: string,
    payload: Partial<CreateFeaturePayload>,
  ): Promise<ApiResponse<FeatureItem>> =>
    api.put(`/api/method/.../${id}`, payload),

  remove: (id: string): Promise<ApiResponse<void>> =>
    api.delete(`/api/method/.../${id}`),
};
```

- Include only the methods implied by the user's request
- Type request/response from provided shape; use `unknown` + comment if shape is unclear
- Path params as function args, query params as an optional object arg

---

## Phase 3 — Generate the Custom Hook

### Naming

- File: `use[Feature].ts`
- Export: `function use[Feature]()`

### Query hook pattern

```ts
import { useApiQuery } from "../api/useApiQuery";
import {
  featureApi,
  type FeatureItem,
} from "../../services/domain/feature/feature.api";

export function useFeature() {
  const query = useApiQuery({
    queryKey: ["feature"],
    queryFn: featureApi.getList,
    showErrorToast: true,
  });

  return {
    items: query.data,
    isLoading: query.isLoading,
    error: query.error,
  };
}
```

### Mutation hook pattern

```ts
import { useApiMutation } from "../api/useApiMutation";
import { useQueryClient } from "@tanstack/react-query";
import {
  featureApi,
  type CreateFeaturePayload,
} from "../../services/domain/feature/feature.api";

export function useCreateFeature() {
  const queryClient = useQueryClient();

  return useApiMutation({
    mutationFn: featureApi.create,
    successMessage: "Feature created successfully",
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["feature"] });
    },
  });
}
```

### Rules

- Query keys are ad hoc per feature — derive from resource name
- Cache invalidation lives in `onSuccess` inside the hook
- Toast calls use `showSuccessToast` / `showErrorToast` flags — never implement toast logic directly
- Expose a clean return shape from the hook; don't return the raw query object
- If multiple operations exist (list + create + update), group into one hook file with named exports per operation, or one combined hook — use judgment based on cohesion

---

## Phase 4 — Types

- Derive TypeScript interfaces from the provided response/request shapes
- Place types in the service file unless they're shared — then suggest a `[feature].types.ts`
- Never use `any`; use `unknown` with a comment if shape is genuinely unclear
- For paginated responses, wrap in a `PaginatedResponse<T>` shape if the API returns count/total

---

## Output Order

1. **Service file** (`[feature].api.ts`) with interfaces and endpoint functions
2. **Custom hook** (`use[Feature].ts`)
3. **Usage example** — minimal snippet showing the hook in a component
4. **Assumptions** — list any inferred decisions (endpoint shape, type names, query key, folder path)

refer API_ARCHITECTURE.md
