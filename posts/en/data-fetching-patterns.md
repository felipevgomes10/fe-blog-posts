<!-- Data fetching patterns in React -->

<!-- 
    In today's web applications, efficient data fetching is crucial for delivering exceptional user experiences. This comprehensive guide explores advanced data fetching patterns using TanStack Query (formerly React Query), focusing on practical strategies to improve both perceived and actual application performance.
 -->

 <!-- https://utfs.io/a/oqi3glmmqm/WfWc1HX19bacfgrcXJZNz1yDPrIUm6tXVKWsRvwL7xh3c49n -->

## Introduction

***Data fetching patterns*** are strategies that can be leveraged to ***improve the overall application's perceived performance***. By using these patterns we give the users the feeling the application is quicker than it actually is. There are multiple ways this can be achieved in a client-side application, each one of them is equally important and serves its unique purpose of improving ***User Experience*** and even ***Developer Experience***.

## The ways of caching

![kylo](https://utfs.io/a/oqi3glmmqm/WfWc1HX19bac2DgDwOWICGfsXt1UAzZBmRJd7OWolP4TLjkq)

***In-Memory Caching*** is one of the most important mechanisms there is in client-side applications that can be used to improve the performance perceived by the users. One thing though that's worth mentioning about it is that ***in-memory caching is different than client caching***. While client caching is something enabled by default in modern browsers, the in-memory cache is ***controlled by the application itself*** and can be done manually or by using third-party libraries, also it can be persisted by using the native storage options available in the browsers.

In-Memory Caching is a strategy that consists of ***saving the fetched data in memory***, usually using a Map for efficiency. A simple way of implementing this would be:

```ts
// Simple in-memory cache implementation
const cache = new Map()

// Get cache
const getCachedData = (key) => {
  const item = cache.get(key)
  
  if (!item) return null
  
  const now = Date.now()
  
  if (now > item.expiry) {
    cache.delete(key)
    
    return null
  }
  
  return item.data
}

// Set cache in memory
const setCacheData = (key, data, ttlSeconds = 300) => {
  cache.set(key, {
    data,
    expiry: Date.now() + (ttlSeconds * 1000)
  })
}

// Usage
const loadData = async (id) => {
  const cachedData = getCachedData(id)

  if (cachedData) return cachedData

  const { data } = await myService.getById(id)
  setCacheData(id, data)

  return data
}
```

While this implementation may come across as enough, there are some situations that need to be taken into consideration like: ***error handling, robustness, performance, race conditions, stale data, state management and several other issues that may emerge from this***. So, a more convenient approach would be using something that's already being used and tested by millions of other developers like [Tanstack Query](https://tanstack.com/query/latest/docs/framework/react/overview).

## Tanstack Query

Basically, [Tanstack Query](https://tanstack.com/query/latest/docs/framework/react/overview) is ***wrapper around http requests that manages queries and mutations in client-side applications***. This tool ***makes it possible for developers to focus on building great experiences*** while leaving the complexity of dealing with the cache to the tool itself.

Even though Tanstack Query manages the cache by itself, it also ***provides a fine grained control over the cache stored in memory***. It has a series of built-in methods that allows the application to step in and take control of what's happening behind the scenes.

The basic setup of a Tanstack Query instance for a React application would be:

```tsx
import {
  useQuery,
  useMutation,
  useQueryClient,
  QueryClient,
  QueryClientProvider,
} from '@tanstack/react-query'
import { getTodos, postTodo } from '../my-api'

// Create a client
const queryClient = new QueryClient()

function App() {
  return (
    // Provide the client to your App
    <QueryClientProvider client={queryClient}>
      <Todos />
    </QueryClientProvider>
  )
}

function Todos() {
  // Access the client
  const queryClient = useQueryClient()

  // Queries
  const query = useQuery({ queryKey: ['todos'], queryFn: getTodos })

  // Mutations
  const mutation = useMutation({
    mutationFn: postTodo,
    onSuccess: () => {
      // Invalidate and refetch
      queryClient.invalidateQueries({ queryKey: ['todos'] })
    },
  })

  return (
    <div>
      <ul>{query.data?.map((todo) => <li key={todo.id}>{todo.title}</li>)}</ul>

      <button
        onClick={() => {
          mutation.mutate({
            id: Date.now(),
            title: 'Do Laundry',
          })
        }}
      >
        Add Todo
      </button>
    </div>
  )
}

render(<App />, document.getElementById('root'))

// Source: https://tanstack.com/query/latest/docs/framework/react/quick-start
```

The example above illustrates the basics of how to work with Tanstack Query. ***It covers the creation of a [stable query client](https://tanstack.com/query/latest/docs/eslint/stable-query-client), wrapping your application with a QueryClientProvider, accessing the query client from inside a component anywhere in the React tree, making a query, performing a mutation and invalidating the stale data of a specific query.***

## How does Tanstack Query cache work?

***Tanstack Query cache works around a concept called*** [***query key***](https://tanstack.com/query/latest/docs/framework/react/guides/query-keys). Every query must have a query key associated to it, this query key is simply an array of values that's used to distinguish one query from another. ***The query key is used as a key for that particular query inside the in-memory cache data structure***. With this key it's easy for the application to manipulate the data of a specific query.

```tsx
import { useQuery } from '@tanstack/react-query'
import { getUser } from '../my-api'

function User() {
  // Query key used to cache the data
  const queryKey = ['user', '1']
  
  const query = useQuery({ queryKey, queryFn: getTodos })

  return (
    <div>
	  <span>{query.data?.name}</span>
	  <span>{query.data?.address}</span>
    </div>
  )
}
```

Now, the data for that particular user is cached, and ***whenever the application needs this data it'll reach for its cached version and revalidate it in background***, which ***allows users to have a better experience as they don't need to wait for data they have already fetched before***. Besides, now cached data can be accessed and manipulated using:

- [`getQueryData`](https://tanstack.com/query/latest/docs/reference/QueryClient/#queryclientgetquerydata)
- [`getQueriesData`](https://tanstack.com/query/latest/docs/reference/QueryClient/#queryclientgetqueriesdata)
- [`setQueryData`](https://tanstack.com/query/latest/docs/reference/QueryClient/#queryclientsetquerydata)
- [`setQueriesData`](https://tanstack.com/query/latest/docs/reference/QueryClient/#queryclientsetqueriesdata)

It's also possible to [invalidate](https://tanstack.com/query/latest/docs/framework/react/guides/query-invalidation) queries based on their query keys:

- [`invalidateQueries`](https://tanstack.com/query/latest/docs/reference/QueryClient/#queryclientinvalidatequeries)

Another important aspect of caching is the ***stale data***. ***Data is considered to be stale when the application no longer have its most up-to-date version***. Tanstack Query has a stale time option, which is important for managing how long the data fetched by queries is considered fresh. ***The stale time defaults to 0 milliseconds, meaning that queries will be updated in background every time the component mounts or when the window is refocused***. The ***stale time can be customized to the application's needs*** to optimize performance and reduce unnecessary data fetching.

```ts
const { data } = useQuery({ 
  queryKey: ['todos'], 
  queryFn: fetchTodos, 
  staleTime: 60 * 1000, // 60 seconds 
});
```

In the example above every time a query with a query key set to `['todos']` is made within 60 seconds Tanstack Query will use the data stored inside the cache instead of calling the api again.

Alternatively, a default stale time can be set on a query client and shared with all queries unless it's overridden by the query itself.

```ts
const queryClient = new QueryClient({ 
  defaultOptions: { 
    queries: { 
      staleTime: 60 * 1000, // 60 seconds 
    }, 
  }, 
});
```

For more information about caching in Tanstack Query:

- [Important defaults](https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults)
- [Caching examples](https://tanstack.com/query/latest/docs/framework/react/guides/caching)
- [Query options](https://tanstack.com/query/latest/docs/framework/react/guides/query-options)

## Initial vs placeholder data

***The initial and placeholder data options are two different ways to inform how the query needs to behave when no data is stored in cache***. 

The ***initial data option provides to the query the data (not partial) that's going to be shown initially in the application, skipping the loading state and caching its value***, being useful whenever the data is somehow available already. However, as the stale time defaults to 0 milliseconds the query is going to be revalidated immediately, so to avoid this the following can be done:

```ts
// Show initialTodos immediately, but won't refetch until another interaction event is encountered after 1000 ms
const result = useQuery({
  queryKey: ['todos'],
  queryFn: () => fetch('/todos'),
  initialData: initialTodos,
  staleTime: 60 * 1000, // 1 minute
  // This could be 10 seconds ago or 10 minutes ago
  initialDataUpdatedAt: initialTodosUpdatedTimestamp, // eg. 1608412420052
})

// Source: https://tanstack.com/query/latest/docs/framework/react/guides/initial-query-data
```

By comparing the difference between `staleTime` and `initialDataUpdatedAt` the query knows if the initial data is fresh or not and needs to be revalidated. 

***Another option is to pass a function as the initial data to the query***. The function is going to be executed only once when the query is initialized.

```ts
const result = useQuery({
  queryKey: ['todo', todoId],
  queryFn: () => fetch(`/todos/${todoId}`),
  staleTime: 60 * 1000, // 1 minute
  initialData: () => queryClient.getQueryData(['todos'])?.find((d) => d.id === todoId), // Use a todo from the 'todos' query as the initial data for this todo query
  initialDataUpdatedAt: () => queryClient.getQueryState(['todos'])?.dataUpdatedAt // Get the freshness of the 'todos' query
})
```

One more important thing illustrated in the example above is ***how the cache from other queries can be used to populate another's query cache with the initial data option***. By using the query client and its built-in methods `getQueryData` and `getQueryState` it's possible to access another's query data and state, and use them however the application needs.

On the other hand, there's ***the placeholder data option which allows the queries to behave as if they already have data, similar to the initial data option, but it does not persist the data to the cache***. It's useful when there's enough partial or fake data that can be displayed beforehand while the actual data is fetched in background.

```ts
function Todos() {
  const result = useQuery({
    queryKey: ['todos'],
    queryFn: () => fetch('/todos'),
    placeholderData: placeholderTodos,
  })
}

// Source: https://tanstack.com/query/latest/docs/framework/react/guides/placeholder-query-data
```

Additionally, ***the function version of the placeholder data option have access to the previous data of that query, which means it's possible to avoid showing loading states while fetching with dynamic queries like: fetching something by its id, or a*** [***paginated query***](https://tanstack.com/query/latest/docs/framework/react/guides/paginated-queries#better-paginated-queries-with-placeholderdata).

```ts
const result = useQuery({
  queryKey: ['todos', id],
  queryFn: () => fetch(`/todos/${id}`),
  placeholderData: (previousData, previousQuery) => previousData,
})

// Source: https://tanstack.com/query/latest/docs/framework/react/guides/placeholder-query-data
```

or simply:

```ts
import { useQuery, keepPreviousData } from '@tanstack/react-query'

const result = useQuery({
  queryKey: ['todos', id],
  queryFn: () => fetch(`/todos/${id}`),
  placeholderData: keepPreviousData
})
```

## Prefetching data

Prefetching data basically means **fetching resources that will be possibly needed before they are actually needed**. There are some different prefetching patterns:

- Inside components
- Inside event handlers
- Via router (more about this later)

***When rendering components there are often cases where a child component that needs to fetch some piece of data needs its parent query to be finished loading so it can be rendered and then start fetching the data it actually needs***. This generates what's called [***request waterfall***](https://tanstack.com/query/latest/docs/framework/react/guides/request-waterfalls) and can lead to poor user experience and perceived performance. For example:

```tsx
function Article({ id }) {
  const { data: articleData, isPending } = useQuery({
    queryKey: ['article', id],
    queryFn: getArticleById,
  })

  if (isPending) {
    return 'Loading article...'
  }

  return (
    <>
      <ArticleHeader articleData={articleData} />
      <ArticleBody articleData={articleData} />
      <Comments id={id} />
    </>
  )
}

function Comments({ id }) {
  const { data, isPending } = useQuery({
    queryKey: ['article-comments', id],
    queryFn: getArticleCommentsById,
  })

  ...
}

// Source: https://tanstack.com/query/latest/docs/framework/react/guides/prefetching
```

This generates the following waterfall:

```
1. |> getArticleById()
2.   |> getArticleCommentsById()

Source: https://tanstack.com/query/latest/docs/framework/react/guides/prefetching
```

An approach to navigate this is to prefetch the data like:

```tsx
function Article({ id }) {
  const { data: articleData, isPending } = useQuery({
    queryKey: ['article', id],
    queryFn: getArticleById,
  })

  // Prefetch
  useQuery({
    queryKey: ['article-comments', id],
    queryFn: getArticleCommentsById,
    // Optional optimization to avoid rerenders when this query changes:
    notifyOnChangeProps: [],
  })

  if (isPending) {
    return 'Loading article...'
  }

  return (
    <>
      <ArticleHeader articleData={articleData} />
      <ArticleBody articleData={articleData} />
      <Comments id={id} />
    </>
  )
}

function Comments({ id }) {
  const { data, isPending } = useQuery({
    queryKey: ['article-comments', id],
    queryFn: getArticleCommentsById,
  })

  ...
}

// Source: https://tanstack.com/query/latest/docs/framework/react/guides/prefetching
```

Now the waterfall looks like:

```
1. |> getArticleById()
1. |> getArticleCommentsById()

Source: https://tanstack.com/query/latest/docs/framework/react/guides/prefetching
```

Also, it's possible to prefetch data inside the `queryFn` function of a query and for this it's possible to use the query client:

```ts
const queryClient = useQueryClient()

const { data: articleData, isPending } = useQuery({
  queryKey: ['article', id],
  queryFn: (...args) => {
    queryClient.prefetchQuery({
      queryKey: ['article-comments', id],
      queryFn: getArticleCommentsById,
    })

    return getArticleById(...args)
  },
})

// Source: https://tanstack.com/query/latest/docs/framework/react/guides/prefetching
```

Note that if the component is being suspended the application needs to use `usePrefectQuery` and `useSuspenseQuery`. For more information about this:

- [Prefetching](https://tanstack.com/query/latest/docs/framework/react/guides/prefetching)
- [`usePrefetchQuery`](https://tanstack.com/query/latest/docs/framework/react/reference/usePrefetchQuery)
- [`useSuspenseQuery`](https://tanstack.com/query/latest/docs/framework/react/reference/useSuspenseQuery)

Another way to prefetch data is by ***identifying the user's intentions with event handlers and then fetching the data that's possibly going to be needed***. It's possible to do this by using the [`prefetchQuery`](https://tanstack.com/query/latest/docs/reference/QueryClient/#queryclientprefetchquery) method.

***This method only populates the query cache and returns nothing.***

```tsx
function ShowDetailsButton() {
  const queryClient = useQueryClient()

  const prefetch = () => {
    queryClient.prefetchQuery({
      queryKey: ['details'],
      queryFn: getDetailsData,
      // Prefetch only fires when data is older than the staleTime,
      // so in a case like this you definitely want to set one
      staleTime: 60000,
    })
  }

  return (
    <button onMouseEnter={prefetch} onFocus={prefetch} onClick={...}>
      Show Details
    </button>
  )
}

// Source: https://tanstack.com/query/latest/docs/framework/react/guides/prefetching
```

By doing this ***anytime the user hovers over or focus on the element, the data is going to be prefetched***. It's important to set a stale time in this cases so the application does not make a lot of unnecessary requests to the server.

## Ensure query data

Tanstack query provides a ***method to populate the query client cache called [`ensureQueryData`](https://tanstack.com/query/latest/docs/reference/QueryClient#queryclientensurequerydata) that can be used to make sure the data of a given query is available when needed***. Ensuring the query data is there means ***the method is going to check if the query being made has cached data that's not stale and return it, if the query does not have cached data yet the `ensureQueryData` method calls the api internally, saves its response to the cache and returns the data***.

```ts
const data = await queryClient.ensureQueryData({ queryKey, queryFn })

// Source: https://tanstack.com/query/latest/docs/reference/QueryClient#queryclientensurequerydata
```

***This is useful when the application needs to prefetch data that may have been cached already***. For example, inside routes loaders as the page could be accessed directly via its URL and this would require all the data to be loaded from scratch inside the loader, or the page could be open by the user navigating to it using a link that may prefetch the data of its target page when the mouse is over it, which means the route loader does not need to prefetch the data again.

## Infinite queries

It's also possible to perform cursor-based queries with infinite loading using Tanstack Query. Using the [`useInfiniteQuery`](https://tanstack.com/query/latest/docs/framework/react/reference/useInfiniteQuery), [`useSuspenseInfiniteQuery`](https://tanstack.com/query/latest/docs/framework/react/reference/useSuspenseInfiniteQuery) and [`usePrefetchInfiniteQuery`](https://tanstack.com/query/latest/docs/framework/react/reference/usePrefetchInfiniteQuery) hooks it's possible to infinite-load any cursor-based query in any situation the application may face.

```tsx
import { useInfiniteQuery } from '@tanstack/react-query'

function Projects() {
  const fetchProjects = async ({ pageParam }) => {
    const res = await fetch('/api/projects?cursor=' + pageParam)
    return res.json()
  }

  const {
    data,
    error,
    fetchNextPage,
    hasNextPage,
    isFetching,
    isFetchingNextPage,
    status,
  } = useInfiniteQuery({
    queryKey: ['projects'],
    queryFn: fetchProjects,
    initialPageParam: 0,
    getNextPageParam: (lastPage, pages) => lastPage.nextCursor,
  })

  return status === 'pending' ? (
    <p>Loading...</p>
  ) : status === 'error' ? (
    <p>Error: {error.message}</p>
  ) : (
    <>
      {data.pages.map((group, i) => (
        <React.Fragment key={i}>
          {group.data.map((project) => (
            <p key={project.id}>{project.name}</p>
          ))}
        </React.Fragment>
      ))}
      <div>
        <button
          onClick={() => fetchNextPage()}
          disabled={!hasNextPage || isFetchingNextPage}
        >
          {isFetchingNextPage
            ? 'Loading more...'
            : hasNextPage
              ? 'Load More'
              : 'Nothing more to load'}
        </button>
      </div>
      <div>{isFetching && !isFetchingNextPage ? 'Fetching...' : null}</div>
    </>
  )
}

// Source: https://tanstack.com/query/latest/docs/framework/react/guides/infinite-queries
```

Also, ***it's possible to prefetch the data of infinite queries inside effects, callbacks and outside React*** using:

- [`prefetchInfiniteQuery`](https://tanstack.com/query/latest/docs/reference/QueryClient/#queryclientprefetchinfinitequery)
- [`ensureInfiniteQueryData`](https://tanstack.com/query/latest/docs/reference/QueryClient/#queryclientensureinfinitequerydata)

For more details on the infinite query options:

[Infinite query options](https://tanstack.com/query/latest/docs/framework/react/reference/infiniteQueryOptions)

## Devtools

Tanstack Query comes with a built-in component that helps developers debug the queries and mutations being made by the application, with this component it's possible to see the state of the queries and mutations, as well as manipulate data and trigger events manually. It's widely recommended to have this installed in the application.
The state of a query can be:

- Fresh
- Fetching
- Paused
- Stale
- Inactive

[Devtools](https://tanstack.com/query/latest/docs/framework/react/devtools)

## Routes loaders

Most ***modern client-side routers allows the applications to have a loader function that executes outside the framework being used, meaning that the loader is going to be executed before the framework render the page***.

In [React Router Dom](https://reactrouter.com/en/main) since version 6, it's possible to setup a loader for each individual route as the following:

```ts
createBrowserRouter([
  {
    path: "/teams/:teamId",
    loader: ({ params }) => {
      return fakeGetTeam(params.teamId)
    }
  }
])

// Source: https://reactrouter.com/en/main/route/loader
```

By using loaders ***it's possible to retrieve data from anywhere like external services, and make it available to the component being rendered in this loaders route***.

```ts
function loader({ request }) {
  const url = new URL(request.url)
  const searchTerm = url.searchParams.get("q")
  return searchProducts(searchTerm)
}

// Source: https://reactrouter.com/en/main/route/loader
```

Then it's possible to access the by using the [`useLoaderData`](https://reactrouter.com/en/main/hooks/use-loader-data) hook:

```ts
function SomeRoute() {
  const data = useLoaderData()
  // { some: "thing" }
}

// Source: https://reactrouter.com/en/main/route/loader
```

Another important thing to notice is that ***when lazy-loading components, React Router Dom is going to call the lazy and the loader functions at the same time***, so by the time the browser finishes downloading the javascript to render the page it's highly possible the loader has finished run meaning the data will be available almost instantly.
## Integrating routes loaders and Tanstack Query

Even though it's possible to use the router capabilities alone to make data available early in the application, it's even better to combine both and take advantage of their strengths.

That said, ***the application can leverage the way loaders work and the moment they are run in the life cycle to prefetch resources using Tanstack Query***.

```ts
import { createBrowserRouter } from 'react-router-dom'
import { QueryClient } from '@tanstack/react-query'
import { getUsers } from '@api/get-users'

const queryClient = new QueryClient()

createBrowserRouter([
  {
    path: "/some-route",
    loader: async () => {
      // Makes sure every query that has ['users'] as query key has cached data
      await queryClient.ensureQueryData({
        queryKey: ['users'],
        queryFn: getUsers
      })

	  // Needs to return something, even if it's null
      return null
    }
  }
])
```

## Good practices and performance

There are some good practices that are advised to be followed when using Tanstack Query. By making sure these practices are followed it's most likely the perceived performance, maintainability and UX of the application is going to be kept in good level.

## Awaiting promises

Regarding the performance of prefetching queries, ***it's important to pay attention to blocking code** as sometimes awaiting for a promise to be resolved is not required for the application.

```ts
// Non-blocking code
function prefetch() {
  queryClient.prefetchQuery(queryOptions)
}

// Blocking code
async function prefetch() {
  await queryClient.prefetchQuery(queryOptions)
}
```

There's nothing else after the `prefetchQuery` call so it's not necessarily needed to await the promise unless it's intentional.

Another important thing to keep in mind is to ***use promise concurrency when necessary***. Sometimes a function needs to prefetch more than one query at a time and this can lead to poor perceived performance and big loading times if the promises are not resolved concurrently.

```ts
// Bad - await for each promise to be resolved
function prefetch() {
  await queryClient.prefetchQuery(queryOptions1)
  await queryClient.prefetchQuery(queryOptions2)
  await queryClient.prefetchQuery(queryOptions3)
}

// Good - resolve promises concurrently
async function prefetch() {
  await Promise.all(
    queryClient.prefetchQuery(queryOptions1),
    queryClient.prefetchQuery(queryOptions2),
    queryClient.prefetchQuery(queryOptions3),
  )
}
```

Note that ***sometimes a query may need a piece of information that returns from another query so prefetching the query earlier or persisting the important values in places that can be accessed globally like the local storage or the URL may be good strategies***.

## Reusability and isolation

An extra good practice is to ***isolate queries in functions so that they can be reused easily in different places***. Also, it's important to ***remember to expose the important things while keeping a good level of encapsulation***. Tanstack Query has a helper function called [`queryOptions`](https://tanstack.com/query/latest/docs/framework/react/reference/queryOptions#queryoptions) that facilitates this:

```ts
// users-query-options.ts
import { getUsers } from "@/services/users-service/users-service"
import { keepPreviousData, queryOptions } from "@tanstack/react-query"

export const usersQueryOptions = queryOptions({
  queryKey: ["users"],
  queryFn: getUsers,
  placeholderData: keepPreviousData, // Avoids laggy data transitions
})

// query.types.ts
export type QueryParams<TData, TReturn> = {
  select?: (data: TData) => TReturn // Enables query mapping
  initialData?: TData | (() => TData)
}

// use-users.ts
import { usersQueryOptions } from "@/lib/react-query/users-query-options/users-query-options"
import { useQuery } from "@tanstack/react-query"

import type { QueryParams } from "@/lib/react-query/query.types"
import type { ApiUser } from "@/services/users-service/users-service"

export function useUsers<TReturn = ApiUser[]>({
  select,
  initialData,
}: QueryParams<ApiUser[], TReturn> = {}) {
  const { data, isLoading, isFetching, isPending, isError, error, refetch } = useQuery({
    ...usersQueryOptions,
    select,
    initialData,
  })

  return {
    data,
    isLoading,
    isFetching,
    isPending,
    isError,
    error,
    refetch,
  }
}
```

The custom React hook above is a good strategy to isolate queries. It exposes the `select` and `initialData` methods and returns only a subset of the `useQuery` return object. ***Note how the hook is typed using generics, allowing the returned data type to be inferred from the return of the `select` method which defaults to the original return type***, this is extremely important for type-safety.

Usage of the custom hook:

```tsx
// users.tsx
import { useUsers } from "@/hooks/use-users/use-users"
import { UserMapper } from "@/mappers/user-mapper/user-mapper"

const userMapper = UserMapper.create() // stable set of methods

export function Users() {
  const { data: users = [] } = useUsers() // data is inferred as an array of api users
  const { data: mappedUsers = [] } = useUsers({ select: userMapper.mapMany }) // data is inferred as an array of mapped users;
  const { data: usersCount = 0 } = useUsers({ select: userMapper.count }) // data is inferred as a number;

  return ...
}
```

Additionally, ***it's important to use stable functions on the `select` and `initialData` parameters*** as this optimizes the rendering process. Stable functions can be functions created or imported from outside React's context (hooks, components, etc...), or functions that were memoized by `useCallback`. For more information see:

[Render optimizations](https://tanstack.com/query/latest/docs/framework/react/guides/render-optimizations)

## Query client

Also, apart from the things mentioned above ***it's important to set default values for important options in the query client instance***.

```ts
import { QueryClient } from "@tanstack/react-query";

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: false, // Set this only in specific queries if needed
      refetchOnWindowFocus: false,
      staleTime: 1000 * 60, // 1 minute for every query, unless it's overriden by the query itself
    },
  }
});
```

## Linting

Tanstack query provides a linting tool to enforce the best practices while using it. For more details on how to se it up see:

[ESLint plugin query](https://tanstack.com/query/latest/docs/eslint/eslint-plugin-query)