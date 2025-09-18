# 🚀 TanStack Start – Unique Functions & Features

## Introduction

TanStack Start is a full-stack React framework built on TanStack Router, with a client-first approach and fast Vite-powered development.

**Key Philosophy:**
- **Type-safe everything** - From routing to server functions, full TypeScript coverage
- **Client-first development** - Familiar React patterns without complex server/client boundaries  
- **Vite-powered** - Instant HMR and blazing fast builds
- **Framework-agnostic server functions** - Not tied to React Server Components

See real code examples from demo projects (`start/` and `start2/`) showing TanStack Start's features, with direct comparisons to Next.js.

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

## 🔹 Data Fetching (Router Loaders + Server Functions)

### Route Loaders with Server Functions
TanStack Start uses route loaders that can call server functions, providing automatic caching and prefetching.

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

**Used in route loaders from `start2/src/routes/posts.$postId.tsx`:**
```typescript
export const Route = createFileRoute('/posts/$postId')({
  loader: ({ params: { postId } }) => fetchPost({ data: postId }),
  component: PostComponent,
})

function PostComponent() {
  const post = Route.useLoaderData() // ✨ Data from server function!
  return <div>{post.title}</div>
}
```

### Route + Data Prefetch Together
Unlike plain Next.js links, TanStack Start prefetches both route AND data when you hover.

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

### TanStack Query Integration (Optional)
While these projects don't use it, TanStack Start **can** work with TanStack Query for client-side caching, but it's not required since server functions + route loaders provide similar benefits.

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

## 🆚 Practical Comparison with Next.js

| Area | TanStack Start | Next.js |
|------|----------------|---------|
| **Dev Speed** | Vite, blazing fast HMR | Webpack/Turbopack, slower on larger projects |
| **Server Functions** | `createServerFn`, simple mental model | Server Actions, tied to RSC complexity |
| **Data Fetching** | Query-first, router-integrated | RSC/Server Actions + Query (if you need cache) |
| **Type Safety** | Params + search strings are TypeScript-first | Next router less strict with types |
| **Ecosystem** | Still very young, fewer templates | Huge ecosystem, backed by Vercel |
| **Streaming SSR** | Streaming without RSC complexity | Forces you to learn RSC boundaries |
| **Mental Model** | Client-first with server capabilities | Server-first with client hydration |
| **Best Use Case** | SaaS/dashboard, dev teams wanting TS-first | SEO-heavy public sites, image-rich content |

---

## ⚡️ TL;DR:
**Unique TanStack Start Power** = Router + Query + Server Functions working tightly together with strong TypeScript, all built on top of Vite's lightning-fast development experience.

### Project Examples:
- **`start/`** - Basic server functions and API routes demo
- **`start2/`** - Advanced routing, layouts, streaming, and error boundaries

---

## 🎯 Recommendation

**Choose TanStack Start if:**
- You're building a **startup SaaS/dashboard** application
- Your dev team prioritizes **TypeScript-first** development
- You want **blazing fast development** with Vite
- You prefer a **simpler mental model** without RSC complexity
- You're comfortable with a **younger ecosystem**

**Choose Next.js if:**
- You're building **SEO-heavy public sites** or **image-rich content**
- You need **production-level optimizations** out of the box
- You want **extensive ecosystem support** and templates
- You require **edge middleware** and advanced deployment features
- You need the **safety of a mature, battle-tested framework**

**Bottom Line:** If you're building a startup SaaS/dashboard → TanStack Start is cleaner, faster, and more developer-friendly. If you need production-level SEO, image optimization, and edge middleware → Next.js is the safe choice.
