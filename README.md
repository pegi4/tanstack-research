# 🚀 TanStack Start – Unique Functions & Features

## 🔹 Routing (TanStack Router)

### `createFileRoute()` → Type-Safe Route Definition
Defines a route in a completely type-safe way with params/search typed from definition.

**Example from `start2/src/routes/posts.$postId.tsx`:**
```typescript
export const Route = createFileRoute('/posts/$postId')({
  loader: ({ params: { postId } }) => fetchPost({ data: postId }),
  errorComponent: PostErrorComponent,
  component: PostComponent,
  notFoundComponent: () => {
    return <NotFound>Post not found</NotFound>
  },
})

function PostComponent() {
  const post = Route.useLoaderData() // ✨ Fully typed!
  // ...
}
```

### Link `preload="intent"` → Hover Prefetching
Prefetches data when user hovers a link, making navigation instant.

**Example from `start2/src/router.tsx`:**
```typescript
export function createRouter() {
  const router = createTanStackRouter({
    routeTree,
    defaultPreload: 'intent', // ✨ Auto-prefetch on hover!
    // ...
  })
}
```

### `Route.useParams()` / `Route.useSearch()` → Strongly Typed
Type-safe params and query strings directly from route definition.

**Example from `start2/src/routes/posts.$postId.tsx`:**
```typescript
function PostComponent() {
  const post = Route.useLoaderData() // postId is typed!
  // params.postId is automatically typed as string
}
```

### Nested Routes with Layouts
File-based nested layouts that compose beautifully.

**Example from `start2/src/routes/_pathlessLayout/_nested-layout.tsx`:**
```typescript
export const Route = createFileRoute('/_pathlessLayout/_nested-layout')({
  component: LayoutComponent,
})

function LayoutComponent() {
  return (
    <div>
      <div>I'm a nested layout</div>
      <div className="flex gap-2 border-b">
        <Link to="/route-a">Go to route A</Link>
        <Link to="/route-b">Go to route B</Link>
      </div>
      <Outlet /> {/* Child routes render here */}
    </div>
  )
}
```

---

## 🔹 Data Fetching (TanStack Query Integration)

### Works Seamlessly with Router Loaders + useQuery()
Built-in cache, retry, refetch-on-focus, invalidation all work together.

**Example from `start2/src/utils/posts.tsx`:**
```typescript
export const fetchPost = createServerFn()
  .validator((d: string) => d)
  .handler(async ({ data }) => {
    console.info(`Fetching post with id ${data}...`)
    const res = await fetch(`https://jsonplaceholder.typicode.com/posts/${data}`)
    // ... error handling
    return await res.json()
  })

export const fetchPosts = createServerFn().handler(async () => {
  console.info('Fetching posts...')
  const res = await fetch('https://jsonplaceholder.typicode.com/posts')
  return (await res.json()).slice(0, 10)
})
```

### Query Prefetch Tied to Router Navigation
Unlike plain Next.js links, TanStack Start prefetches both route + data together.

**Example from `start2/src/routes/posts.tsx`:**
```typescript
// When you hover over these links, both route AND data are prefetched!
<Link
  to="/posts/$postId"
  params={{ postId: post.id }}
  className="block py-1 text-blue-800 hover:text-blue-600"
>
  {post.title}
</Link>
```

---

## 🔹 Server Functions

### `createServerFn(method, fn)` → Server-Only Functions
Define server-only functions inside your project, similar to Next.js "Server Actions" but simpler.

**Example from `start/src/routes/demo.start.server-funcs.tsx`:**
```typescript
import { createServerFn } from '@tanstack/react-start'

// GET server function
const getTodos = createServerFn({
  method: 'GET',
}).handler(async () => await readTodos())

// POST server function with validation
const addTodo = createServerFn({ method: 'POST' })
  .validator((d: string) => d)
  .handler(async ({ data }) => {
    const todos = await readTodos()
    todos.push({ id: todos.length + 1, name: data })
    await fs.promises.writeFile(filePath, JSON.stringify(todos, null, 2))
    return todos
  })

// Usage in component
function Home() {
  const submitTodo = useCallback(async () => {
    todos = await addTodo({ data: todo }) // ✨ Direct server call!
    setTodo('')
    router.invalidate()
  }, [addTodo, todo])
}
```

### Automatic Serialization & Type Safety
Server functions automatically serialize results to client with full type safety.

**Example from `start2/src/routes/deferred.tsx`:**
```typescript
const personServerFn = createServerFn({ method: 'GET' })
  .validator((d: string) => d)
  .handler(({ data: name }) => {
    return { name, randomNumber: Math.floor(Math.random() * 100) }
  })

// Usage with deferred loading
export const Route = createFileRoute('/deferred')({
  loader: async () => {
    return {
      person: await personServerFn({ data: 'John Doe' }), // ✨ Awaited
      deferredPerson: slowServerFn({ data: 'Tanner Linsley' }), // ✨ Deferred
    }
  },
})
```

---

## 🔹 Streaming & SSR

### Full-Document SSR with Streaming
Out-of-the-box streaming SSR without React Server Components complexity.

**Example from `start2/src/routes/deferred.tsx`:**
```typescript
function Deferred() {
  const { deferredPerson, person } = Route.useLoaderData()

  return (
    <div>
      {/* This renders immediately */}
      <div>{person.name} - {person.randomNumber}</div>
      
      {/* This streams in when ready */}
      <Suspense fallback={<div>Loading person...</div>}>
        <Await
          promise={deferredPerson}
          children={(data) => (
            <div>{data.name} - {data.randomNumber}</div>
          )}
        />
      </Suspense>
    </div>
  )
}
```

### No React Server Components (RSC) → Simpler Mental Model
Unlike Next.js App Router, no client/server component boundaries to worry about.

---

## 🔹 API Routes

### `createServerFileRoute()` → Type-Safe API Endpoints
Create API endpoints with full type safety and method handling.

**Example from `start/src/routes/api.demo-names.ts`:**
```typescript
import { createServerFileRoute } from '@tanstack/react-start/server'

export const ServerRoute = createServerFileRoute('/api/demo-names').methods({
  GET: () => {
    return new Response(JSON.stringify(['Alice', 'Bob', 'Charlie']), {
      headers: { 'Content-Type': 'application/json' },
    })
  },
})
```

---

## 🔹 Other TanStack Ecosystem Pieces (Optional)

- **TanStack Table** → Headless, highly customizable data tables
- **TanStack Form** → Type-safe form state manager (new)
- **TanStack Virtual** → Virtual scrolling for huge lists/tables
- **TanStack Store** (alpha) → Simple state manager (Zustand alternative)
- **TanStack DB** (beta) → Query builder / typed DB layer

---

## 🆚 Why They Matter Compared to Next.js

| Feature | TanStack Start | Next.js |
|---------|----------------|---------|
| **Server Functions** | `createServerFn` = framework-agnostic, not tied to RSC | Server Actions tied to RSC boundaries |
| **Router + Data Prefetch** | Router + Query prefetch work together seamlessly | Route prefetch separate from data prefetch |
| **Type Safety** | Params + search strings are TypeScript-first | Next router less strict with types |
| **Streaming SSR** | Streaming without RSC complexity | Forces you to learn RSC boundaries |
| **Mental Model** | Client-first with server capabilities | Server-first with client hydration |

---

## ⚡️ TL;DR:
**Unique TanStack Start Power** = Router + Query + Server Functions working tightly together with strong TypeScript, all built on top of Vite's lightning-fast development experience.

### Project Examples:
- **`start/`** - Basic server functions and API routes demo
- **`start2/`** - Advanced routing, layouts, streaming, and error boundaries
