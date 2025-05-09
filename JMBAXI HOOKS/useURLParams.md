# `useURLParams` Hook

## What is `useURLParams` Hook?

The `useURLParams` hook is a custom React hook designed for managing and synchronizing query parameters in the URL of a Next.js application. It simplifies handling search, filters, pagination, and sorting states by keeping them in sync with the browser's URL query string. This ensures a consistent user experience and makes the application state easily shareable via URLs.

## How to Use This Hook?

### Code

```javascript
import { generateQueryString } from "@/utilities/query";
import { isEmpty } from "lodash";
import { useRouter } from "next/router";
import { useEffect, useState } from "react";

const useURLParams = () => {
  const router = useRouter();
  const [search, setSearch] = useState("");
  const [filter, setFilter] = useState({});
  const [page, setPage] = useState(0);
  const [sortingFilter, setSortingFilter] = useState({
    field: "",
    order: "",
  });

  useEffect(() => {
    const { query } = router;
    const search = query.search || "";
    const filter = query.filter ? JSON.parse(query.filter) : {};
    const page = query.page ? parseInt(query.page, 10) : 0;
    const sortingFilter = query.sortingFilter
      ? JSON.parse(query.sortingFilter)
      : {};
    setSearch(search);
    setFilter(filter);
    setPage(page);
    setSortingFilter(sortingFilter);
  }, [router]);

  const updateQueryParams = (newParams) => {
    const updatedParams = {
      ...newParams,
      filter: !isEmpty(newParams?.filter)
        ? JSON.stringify(newParams?.filter)
        : null,
      page: newParams?.page ? newParams?.page : null,
      sortingFilter: !isEmpty(newParams?.sortingFilter)
        ? JSON.stringify(newParams?.sortingFilter)
        : null,
    };
    const queryParams = {
      ...router.query,
      ...updatedParams,
    };
    const updatedQuery = generateQueryString(queryParams);
    router.push({
      pathname: router.pathname,
      query: updatedQuery,
    });
  };

  return {
    filter,
    page,
    search,
    setFilter,
    setPage,
    setSearch,
    setSortingFilter,
    sortingFilter,
    updateQueryParams,
  };
};

export default useURLParams;
```

- # Import the hook and use it in any component that requires query parameter management. Implementation of this hook in Nextjs:

```javascript
import useURLParams from "@/hooks/useURLParams";

const MyComponent = () => {
  const {
    filter,
    page,
    search,
    sortingFilter,
    setSortingFilter,
    updateQueryParams,
  } = useURLParams();

  const applyFilters = () => {
    updateQueryParams({
      filter: { category: "electronics", brand: "Apple" },
      page: 1,
      sortingFilter: { field: "price", order: "asc" },
    });
  };

  return (
    <div>
      <button onClick={applyFilters}>Apply Filters</button>
      <p>Current Page: {page}</p>
      <p>Search Term: {search}</p>
      <p>Filters: {JSON.stringify(filter)}</p>
      <p>Sorting: {JSON.stringify(sortingFilter)}</p>
    </div>
  );
};

```
## How It Works

### Initialization:
- The hook reads the current query parameters from the URL using `router.query`.
- It initializes local states (`search`, `filter`, `page`, `sortingFilter`) based on these parameters.

### Sync with URL:
- When query parameters in the URL change, the hook updates the local state automatically using the `useEffect` hook.

### Update URL:
- The `updateQueryParams` function updates the query parameters in the URL while maintaining existing ones. Non-essential parameters (`filter`, `page`, `sortingFilter`) are handled appropriately to avoid unnecessary clutter.
## API Reference

### State Variables

#### `search`
- **Type**: `string`
- **Description**: Holds the current search term from the URL.
- **Default**: `""`

#### `filter`
- **Type**: `object`
- **Description**: Represents the filter object from the URL query parameters.
- **Default**: `{}`

#### `page`
- **Type**: `number`
- **Description**: Represents the current pagination page from the URL.
- **Default**: `0`

#### `sortingFilter`
- **Type**: `object`
- **Description**: Holds the sorting configuration with fields like `field` and `order`.
- **Default**: `{ field: "", order: "" }`

---

### Setters

#### `setSearch(search: string)`
- **Description**: Updates the local `search` state.
- **Parameter**:
  - `search` (string): New search term.

#### `setFilter(filter: object)`
- **Description**: Updates the local `filter` state.
- **Parameter**:
  - `filter` (object): New filter object.

#### `setPage(page: number)`
- **Description**: Updates the local `page` state.
- **Parameter**:
  - `page` (number): New pagination page.

#### `setSortingFilter(sortingFilter: object)`
- **Description**: Updates the local `sortingFilter` state.
- **Parameter**:
  - `sortingFilter` (object): New sorting configuration object.

## Function

#### `updateQueryParams(newParams: object)`
- **Description**: Updates the URL query string and syncs it with the browser history.
- **Parameter**:
  - `newParams` (object): New query parameters to be merged with the existing ones.
    - `filter` (object): Filter criteria, converted to a JSON string.
    - `page` (number): Pagination page.
    - `sortingFilter` (object): Sorting configuration, converted to a JSON string.
- **Example**:

```javascript
updateQueryParams({
  filter: { category: "books", author: "J.K. Rowling" },
  page: 2,
  sortingFilter: { field: "price", order: "desc" },
});
```

## Benefits
- Keeps URL query parameters and component state in sync.
- Enables sharing URLs with applied filters, pagination, and sorting configurations.
- Enhances the user experience with deep-linking and browser navigation support.

## Dependencies

- **`@/utilities/query`**: Contains the `generateQueryString` function, which converts query parameters into a URL query string.
- **`lodash`**: Used for checking if objects are empty.
- **`next/router`**: Used for accessing the query params.

