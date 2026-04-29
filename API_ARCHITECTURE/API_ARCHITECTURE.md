# API Architecture Diagram

## 🏗️ System Architecture

```
┌────────────────────────────────────────────────────────────────────┐
│                          React Component                           │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  import { useInvoices } from './hooks/useInvoices';         │   │
│  │                                                             │   │
│  │  function InvoicesPage() {                                  │   │
│  │    const { invoices, createInvoice } = useInvoices();       │   │
│  │    // Use data in JSX                                       │   │
│  │  }                                                          │   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────────────┬───────────────────────────────────┘
                                 │
                                 ↓
┌───────────────────────────────────────────────────────────────────┐
│                       Custom Hook Layer                           │
│  ┌────────────────────────────────────────────────────────────┐   │
│  │  src/hooks/useInvoices.ts                                  │   │
│  │  ┌─────────────────────────────────────────────────────┐   │   │
│  │  │  export function useInvoices() {                    │   │   │
│  │  │    const { data } = useApiQuery({                   │   │   │
│  │  │      queryKey: ['invoices'],                        │   │   │
│  │  │      queryFn: invoicesService.getList,              │   │   │
│  │  │    });                                              │   │   │
│  │  │    return { invoices: data };                       │   │   │
│  │  │  }                                                  │   │   │
│  │  └─────────────────────────────────────────────────────┘   │   │
│  └────────────────────────────────────────────────────────────┘   │
└────────────────────────────────┬──────────────────────────────────┘
                                 │
                                 ↓
┌─────────────────────────────────────────────────────────────────────┐
│                   TanStack Query Wrapper Layer                      │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │  src/hooks/api/useApiQuery.ts                              │     │
│  │  ┌──────────────────────────────────────────────────────┐  │     │
│  │  │  export function useApiQuery(options) {              │  │     │
│  │  │    const query = useQuery({                          │  │     │
│  │  │      queryKey: options.queryKey,                     │  │     │
│  │  │      queryFn: options.queryFn,                       │  │     │
│  │  │    });                                               │  │     │
│  │  │                                                      │  │     │
│  │  │    // Handle success                                 │  │     │
│  │  │    if (query.isSuccess && options.onSuccess) {       │  │     │
│  │  │      options.onSuccess(query.data);                  │  │     │
│  │  │      // Show toast notification                      │  │     │
│  │  │    }                                                 │  │     │
│  │  │                                                      │  │     │
│  │  │    // Handle error                                   │  │     │
│  │  │    if (query.isError && options.onError) {          │ │    │
│  │  │      options.onError(query.error);                  │ │    │
│  │  │      // Show error toast                            │ │    │
│  │  │    }                                                 │ │    │
│  │  │                                                      │ │    │
│  │  │    return query;                                    │ │    │
│  │  │  }                                                   │ │    │
│  │  └──────────────────────────────────────────────────────┘ │    │
│  └────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────┐    │
│  │  src/hooks/api/useApiMutation.ts                           │    │
│  │  - Wraps useMutation                                       │    │
│  │  - Adds success/error handling                             │    │
│  │  - Shows toast notifications                               │    │
│  └────────────────────────────────────────────────────────────┘    │
└────────────────────────────────┬────────────────────────────────────┘
                                 │
                                 ↓
┌─────────────────────────────────────────────────────────────────────┐
│                    TanStack Query Core                               │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  @tanstack/react-query (v5.90.7)                            │   │
│  │  ┌─────────────────────────────────────────────────────┐   │   │
│  │  │  • useQuery - GET requests                          │   │   │
│  │  │  • useMutation - POST/PUT/DELETE requests            │   │   │
│  │  │  • QueryClient - Cache management                    │   │   │
│  │  │                                                       │   │   │
│  │  │  Features:                                           │   │   │
│  │  │  ✓ Automatic caching                                │   │   │
│  │  │  ✓ Background refetching                            │   │   │
│  │  │  ✓ Request deduplication                            │   │   │
│  │  │  ✓ Optimistic updates                               │   │   │
│  │  │  ✓ Automatic retries                                │   │   │
│  │  │  ✓ Loading/error states                             │   │   │
│  │  └─────────────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────────────┬────────────────────────────────────┘
                                 │
                                 ↓
┌─────────────────────────────────────────────────────────────────────┐
│                         Service Layer                                │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  src/services/transactions/invoices/invoices.service.ts      │   │
│  │  ┌─────────────────────────────────────────────────────┐   │   │
│  │  │  export const invoicesService = {                    │   │   │
│  │  │    getList: () =>                                    │   │   │
│  │  │      api.get<Invoice[]>('/api/method/get_list'),    │   │   │
│  │  │                                                      │   │   │
│  │  │    create: (data) =>                                │   │   │
│  │  │      api.post<Invoice>('/api/method/create', data), │   │   │
│  │  │                                                      │   │   │
│  │  │    update: (id, data) =>                            │   │   │
│  │  │      api.put<Invoice>(`/api/method/${id}`, data),   │   │   │
│  │  │                                                      │   │   │
│  │  │    delete: (id) =>                                  │   │   │
│  │  │      api.delete(`/api/method/${id}`),               │   │   │
│  │  │  };                                                  │   │   │
│  │  └─────────────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────────────┬────────────────────────────────────┘
                                 │
                                 ↓
┌─────────────────────────────────────────────────────────────────────┐
│                       API Client Layer                               │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  src/services/api/apiClient.ts                               │   │
│  │  ┌─────────────────────────────────────────────────────┐   │   │
│  │  │  export const api = {                                │   │   │
│  │  │    get: (endpoint, options) =>                      │   │   │
│  │  │      apiFetch(endpoint, { method: 'GET' }),         │   │   │
│  │  │                                                      │   │   │
│  │  │    post: (endpoint, body, options) =>               │   │   │
│  │  │      apiFetch(endpoint, {                           │   │   │
│  │  │        method: 'POST',                              │   │   │
│  │  │        body: JSON.stringify(body),                  │   │   │
│  │  │      }),                                            │   │   │
│  │  │    // ... put, patch, delete                        │   │   │
│  │  │  };                                                  │   │   │
│  │  │                                                      │   │   │
│  │  │  async function apiFetch(endpoint, options) {       │   │   │
│  │  │    const token = getAuthToken();                    │   │   │
│  │  │    const headers = {                                │   │   │
│  │  │      'Content-Type': 'application/json',            │   │   │
│  │  │      'Authorization': `token ${token}`,             │   │   │
│  │  │    };                                               │   │   │
│  │  │                                                      │   │   │
│  │  │    const response = await fetch(url, {              │   │   │
│  │  │      ...options,                                    │   │   │
│  │  │      headers,                                       │   │   │
│  │  │    });                                              │   │   │
│  │  │                                                      │   │   │
│  │  │    const data = await response.json();              │   │   │
│  │  │                                                      │   │   │
│  │  │    if (!response.ok || data.status === 'error') {   │   │   │
│  │  │      throw new ApiError(data.message, ...);         │   │   │
│  │  │    }                                                 │   │   │
│  │  │                                                      │   │   │
│  │  │    return data.data; // Unwrap response            │   │   │
│  │  │  }                                                   │   │   │
│  │  └─────────────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────────────┬────────────────────────────────────┘
                                 │
                                 ↓
┌─────────────────────────────────────────────────────────────────────┐
│                       Native Fetch API                               │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  Browser's built-in fetch()                                  │   │
│  │  • Makes HTTP requests                                       │   │
│  │  • Returns Response object                                   │   │
│  │  • Handles network communication                             │   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────────────┬────────────────────────────────────┘
                                 │
                                 ↓
┌─────────────────────────────────────────────────────────────────────┐
│                        Backend API                                   │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  http://<staging_url>.8848digitalerp.com                       │   │
│  │  /api/method/app_name.api.transactions_api...            │   │
│  │                                                               │   │
│  │  Returns:                                                     │   │
│  │  {                                                            │   │
│  │    status: 'success' | 'error',                              │   │
│  │    data: [...],                                              │   │
│  │    message: 'Request processed successfully',                │   │
│  │    timestamp: '2025-11-24 ...',                              │   │
│  │    error_code?: 'PERMISSION_DENIED'                          │   │
│  │  }                                                            │   │
│  └─────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

## 🔄 Request Flow (Success Case)

```
1. Component calls hook
   InvoicesPage → useInvoices()

2. Hook calls TanStack Query wrapper
   useInvoices() → useApiQuery({ queryFn: invoicesService.getList })

3. TanStack Query wrapper calls TanStack Query core
   useApiQuery() → useQuery({ queryFn })

4. TanStack Query calls service
   useQuery() → invoicesService.getList()

5. Service calls API client
   invoicesService.getList() → api.get('/endpoint')

6. API client calls fetch
   api.get() → fetch(url, { headers: {...} })

7. Fetch makes HTTP request
   fetch() → Network Request → Backend API

8. Backend responds
   { status: 'success', data: [...], message: '...' }

9. API client unwraps response
   apiFetch() → Returns data.data (just the array)

10. TanStack Query caches result
    useQuery() → Stores in cache with key ['invoices']

11. TanStack Query wrapper fires success callback
    useApiQuery() → options.onSuccess(data)
                 → Show success toast

12. Hook returns data
    useInvoices() → { invoices: [...], isLoading: false }

13. Component re-renders with data
    InvoicesPage → Displays invoice list
```

## ❌ Error Flow

```
1-7. [Same as success case up to network request]

8. Backend responds with error
   { status: 'error', message: 'Permission denied', error_code: 'PERMISSION_DENIED' }

9. API client throws ApiError
   apiFetch() → throw new ApiError(message, statusCode, errorCode)

10. TanStack Query catches error
    useQuery() → Sets error state

11. TanStack Query wrapper fires error callback
    useApiQuery() → options.onError(error)
                 → Show error toast

12. Hook returns error
    useInvoices() → { invoices: undefined, isLoading: false, error: ApiError }

13. Component displays error
    InvoicesPage → Shows error message
```

## 🎯 Key Benefits of This Architecture

### 1. **Separation of Concerns**

- **Component**: UI logic only
- **Hook**: Business logic and state management
- **Service**: API endpoint definitions
- **API Client**: HTTP communication
- **TanStack Query**: Caching and data synchronization

### 2. **Reusability**

- Services can be used in multiple hooks
- Hooks can be used in multiple components
- API client is used by all services

### 3. **Type Safety**

- TypeScript types flow through all layers
- Compile-time error checking
- Autocomplete in IDEs

### 4. **Testability**

- Each layer can be tested independently
- Easy to mock dependencies
- Clear boundaries

### 5. **Maintainability**

- Single source of truth for each concern
- Easy to update authentication logic (one place)
- Easy to add new endpoints (add to service)
- Easy to change UI (component only)

## 📊 Data Flow Summary

```
Component ←────────── Hook ←────────── TanStack Query
    ↓                   ↓                     ↓
 Render UI         Business Logic        Cache + State
                        ↓                     ↓
                   Service ←────────── API Client
                        ↓                     ↓
                 API Endpoints         HTTP Layer
                                            ↓
                                      Native Fetch
                                            ↓
                                       Backend API
```

## 🚀 Why This Works Well

1. **TanStack Query** handles caching, refetching, and state management
2. **API Client** handles HTTP, auth, and error normalization
3. **Services** define clear API contracts
4. **Hooks** compose functionality for components
5. **Components** focus on UI rendering

This architecture scales well as your application grows! 🎉
