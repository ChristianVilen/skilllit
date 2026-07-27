# 3. Server-Side Performance

**Impact: HIGH**

Optimizing server-side rendering and data fetching eliminates server-side waterfalls and reduces response times.

### 3.1 Cross-Request LRU Caching

**Impact: HIGH (caches across requests)**

`React.cache()` only works within one request. For data shared across sequential requests (user clicks button A then button B), use an LRU cache.

**Implementation:**

```typescript
import { LRUCache } from 'lru-cache'

const cache = new LRUCache<string, any>({
  max: 1000,
  ttl: 5 * 60 * 1000, // 5 minutes
})

export async function getUser(id: string) {
  const cached = cache.get(id)
  if (cached) return cached

  const user = await db.user.findUnique({ where: { id } })
  cache.set(id, user)
  return user
}

// Request 1: DB query, result cached
// Request 2: cache hit, no DB query
```

Use when sequential user actions hit multiple endpoints needing the same data within seconds.

**Note on serverless environments:** In traditional serverless, each invocation runs in isolation, so consider Redis for cross-process caching. In environments with persistent instances, LRU caching can be especially effective as multiple concurrent requests can share the same cache.

Reference: [https://github.com/isaacs/node-lru-cache](https://github.com/isaacs/node-lru-cache)

### 3.2 Minimize Data Transfer to Client

**Impact: HIGH (reduces data transfer size)**

When passing data from server to client (via API responses or SSR), only send the fields that the client actually needs. This reduces payload size and improves load times.

**Incorrect: sends all 50 fields**

```tsx
// API endpoint
export async function GET() {
  const user = await fetchUser() // 50 fields
  return Response.json({ user })
}

// Client component
function Profile() {
  const { data } = useSWR('/api/user', fetcher)
  return <div>{data.user.name}</div> // uses 1 field
}
```

**Correct: sends only 1 field**

```tsx
// API endpoint
export async function GET() {
  const user = await fetchUser()
  return Response.json({ name: user.name })
}

// Client component
function Profile() {
  const { data } = useSWR('/api/user', fetcher)
  return <div>{data.name}</div>
}
```

### 3.3 Parallel Data Fetching with Component Composition

**Impact: CRITICAL (eliminates server-side waterfalls)**

When fetching data in nested components, structure your code to allow parallel fetching rather than sequential. Use Promise.all() or start fetches early.

**Incorrect: Sequential fetching in parent**

```tsx
function Page() {
  const [header, setHeader] = useState(null)
  const [sidebar, setSidebar] = useState(null)

  useEffect(() => {
    async function loadData() {
      const headerData = await fetchHeader()
      setHeader(headerData)
      const sidebarData = await fetchSidebarItems()
      setSidebar(sidebarData)
    }
    loadData()
  }, [])

  return (
    <div>
      <div>{header}</div>
      <nav>{sidebar?.map(renderItem)}</nav>
    </div>
  )
}
```

**Correct: Parallel fetching**

```tsx
function Page() {
  const [header, setHeader] = useState(null)
  const [sidebar, setSidebar] = useState(null)

  useEffect(() => {
    async function loadData() {
      const [headerData, sidebarData] = await Promise.all([fetchHeader(), fetchSidebarItems()])
      setHeader(headerData)
      setSidebar(sidebarData)
    }
    loadData()
  }, [])

  return (
    <div>
      <div>{header}</div>
      <nav>{sidebar?.map(renderItem)}</nav>
    </div>
  )
}
```

**Alternative: Component-level data fetching with SWR**

```tsx
function Header() {
  const { data } = useSWR('/api/header', fetcher)
  return <div>{data}</div>
}

function Sidebar() {
  const { data } = useSWR('/api/sidebar', fetcher)
  return <nav>{data?.map(renderItem)}</nav>
}

function Page() {
  return (
    <div>
      <Header />
      <Sidebar />
    </div>
  )
}
```

With SWR, both components fetch in parallel automatically.

### 3.4 Per-Request Deduplication with React.cache()

**Impact: MEDIUM (deduplicates within request)**

Use `React.cache()` for server-side request deduplication. This is useful in SSR scenarios where the same data might be fetched multiple times during a single render.

**Usage:**

```typescript
import { cache } from 'react'

export const getCurrentUser = cache(async () => {
  const session = await auth()
  if (!session?.user?.id) return null
  return await db.user.findUnique({
    where: { id: session.user.id },
  })
})
```

Within a single request, multiple calls to `getCurrentUser()` execute the query only once.

