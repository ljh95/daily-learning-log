https://tkdodo.eu/blog/the-query-options-api#queryoptions

```ts
const todoQueries = {
  all: () => ["todos"],
  lists: () => [...todoQueries.all(), "list"],
  list: (filters: string) =>
    queryOptions({
      queryKey: [...todoQueries.lists(), filters],
      queryFn: () => fetchTodos(filters),
    }),
  details: () => [...todoQueries.all(), "detail"],
  detail: (id: number) =>
    queryOptions({
      queryKey: [...todoQueries.details(), id],
      queryFn: () => fetchTodo(id),
      staleTime: 5000,
    }),
};
```
