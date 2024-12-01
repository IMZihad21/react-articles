The **TanStack React Query** library simplifies API state management in React applications, offering a robust and highly configurable toolkit for handling asynchronous data fetching and caching. In this article, we explore how to set up and utilize React Query effectively, focusing on `useQuery` and `useMutation` hooks with examples of practical usage.

---

## Setting Up React Query

### Query Client Configuration

The `QueryClient` serves as the core of React Query, allowing you to define global behaviors for queries. Below is a setup with custom defaults for error handling and query behaviors.

```typescript
import { QueryClient, QueryFunctionContext } from "@tanstack/react-query";
import axios from "axios";
import BASE_URL from "./apiEndpoint";

const apiInstance = axios.create({
  baseURL: BASE_URL,
});

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      queryFn: async ({ queryKey, signal }: QueryFunctionContext) => {
        const { data } = await apiInstance(`${queryKey[0]}`, { signal });
        return data;
      },
    },
  },
});

export default apiInstance;
```

### Integrating React Query with Your App

Wrap your root component with the `QueryClientProvider` to provide the `QueryClient` context to your application.

```typescript
import ReactDOM from "react-dom/client";
import { QueryClientProvider } from "@tanstack/react-query";
import { queryClient } from "./lib/api/queryClient";
import App from "./App";

ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <QueryClientProvider client={queryClient}>
      <App />
    </QueryClientProvider>
  </React.StrictMode>
);
```

---

## Leveraging React Query Hooks

### Fetching Data with `useQuery`

The `useQuery` hook provides a straightforward way to fetch and cache data. Here’s an example of fetching a list of items:

```typescript
import { useQuery } from "@tanstack/react-query";

const ItemsList = () => {
  const { data, isLoading, isError } = useQuery({ queryKey: ["/items"] });

  if (isLoading) return <div>Loading...</div>;
  if (isError) return <div>Error loading items.</div>;

  return (
    <ul>
      {data.map((item: { id: number; name: string }) => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
};

export default ItemsList;
```

### Mutating Data with `useApiMutation`

Mutations handle data modifications such as creating, updating, or deleting resources. Here’s how you can implement a reusable mutation hook.

```typescript
import { useMutation, UseMutationResult } from "@tanstack/react-query";
import { AxiosError } from "axios";
import apiInstance from "../lib/api/queryClient";

interface MutationHookOptions<TData, TVariables> {
  endpoint: string;
  method?: "POST" | "PUT" | "DELETE";
  isMultiPart?: boolean;
}

function useApiMutation<TData = unknown, TVariables = unknown>({
  endpoint,
  method = "POST",
  isMultiPart = false,
}: MutationHookOptions<TData, TVariables>): UseMutationResult<TData, AxiosError, TVariables> {
  const mutationFn = async (variables: TVariables) => {
    const response = await apiInstance.request<TData>({
      url: endpoint,
      method,
      data: variables,
      headers: {
        "Content-Type": isMultiPart ? "multipart/form-data" : "application/json",
      },
    });
    return response.data;
  };

  return useMutation(mutationFn);
}

export default useApiMutation;
```

---

## Examples of `useApiMutation` Usage

**Create a New Item**

```typescript
const CreateItem = () => {
  const mutation = useApiMutation({
    endpoint: "/items",
    method: "POST",
  });

  const handleSubmit = () => {
    mutation.mutate({ name: "New Item" }, {
      onSuccess: () => alert("Item created!"),
    });
  };

  return <button onClick={handleSubmit}>Create Item</button>;
};
```

**Update an Existing Item**

```typescript
const UpdateItem = ({ id }: { id: number }) => {
  const mutation = useApiMutation({
    endpoint: `/items/${id}`,
    method: "PUT",
  });

  const handleUpdate = () => {
    mutation.mutate({ name: "Updated Item" }, {
      onSuccess: () => alert("Item updated!"),
    });
  };

  return <button onClick={handleUpdate}>Update Item</button>;
};
```

**Delete an Item**

```typescript
const DeleteItem = ({ id }: { id: number }) => {
  const mutation = useApiMutation({
    endpoint: `/items/${id}`,
    method: "DELETE",
  });

  const handleDelete = () => {
    mutation.mutate(undefined, {
      onSuccess: () => alert("Item deleted!"),
    });
  };

  return <button onClick={handleDelete}>Delete Item</button>;
};
```

---

## Conclusion

React Query significantly reduces boilerplate and simplifies API interactions in React applications. With features like caching, query invalidation, and mutation management, it ensures optimal performance and a seamless developer experience. By configuring a `QueryClient` and utilizing hooks like `useQuery` and `useMutation`, you can efficiently manage API calls with minimal effort.

Start implementing these patterns today and supercharge your React application's API handling!

