# Server Components

Deep dive into React Server Components in Next.js App Router.

## What are Server Components

Server Components render on the server and send HTML to the client. They:
- Reduce client bundle size
- Access backend resources directly
- Keep sensitive data on server
- Improve initial page load

## Benefits

**Performance:**
- Zero JavaScript sent to client
- Faster initial load
- Better Core Web Vitals

**Security:**
- API keys stay on server
- Direct database access
- No client-side exposure

**Developer Experience:**
- Async/await in components
- Direct data fetching
- Simplified data flow

## Usage Patterns

### Direct Data Fetching

```typescript
async function ProductList() {
  const products = await db.products.findMany();
  
  return (
    <div>
      {products.map(p => (
        <ProductCard key={p.id} product={p} />
      ))}
    </div>
  );
}
```

### Database Access

```typescript
import { prisma } from '@/lib/prisma';

async function UserProfile({ userId }: { userId: string }) {
  const user = await prisma.user.findUnique({
    where: { id: userId },
    include: { posts: true },
  });
  
  return <div>{user.name}</div>;
}
```

### API Integration

```typescript
async function WeatherWidget() {
  const weather = await fetch('https://api.weather.com/current', {
    headers: {
      'Authorization': `Bearer ${process.env.WEATHER_API_KEY}`,
    },
  }).then(res => res.json());
  
  return <div>{weather.temperature}°</div>;
}
```

## Caching Strategies

### Default Caching

```typescript
// Cached indefinitely by default
const data = await fetch('https://api.example.com/data');
```

### Time-based Revalidation

```typescript
// Revalidate every hour
const data = await fetch('https://api.example.com/data', {
  next: { revalidate: 3600 }
});
```

### No Caching

```typescript
// Fresh data on every request
const data = await fetch('https://api.example.com/data', {
  cache: 'no-store'
});
```

### Tagged Caching

```typescript
// Tag for on-demand revalidation
const data = await fetch('https://api.example.com/data', {
  next: { tags: ['products'] }
});

// Revalidate elsewhere
import { revalidateTag } from 'next/cache';
revalidateTag('products');
```

## Composition Patterns

### Passing Data to Client Components

```typescript
// Server Component
async function Page() {
  const data = await fetchData();
  
  return <ClientComponent data={data} />;
}

// Client Component
'use client';
function ClientComponent({ data }) {
  const [state, setState] = useState(data);
  return <div>{state}</div>;
}
```

### Nesting Components

```typescript
// Server Component can contain Client Components
async function ServerParent() {
  const data = await fetchData();
  
  return (
    <div>
      <ServerChild data={data} />
      <ClientChild data={data} />
    </div>
  );
}
```

### Children Pattern

```typescript
// Pass Client Component as children to Server Component
<ServerComponent>
  <ClientComponent />
</ServerComponent>
```

## Limitations

**Cannot use:**
- React hooks (useState, useEffect, etc.)
- Event handlers (onClick, onChange)
- Browser APIs (localStorage, window)
- React Context (for state)

**Can use:**
- async/await
- Server-only APIs
- Direct database access
- Environment variables

## Best Practices

1. **Keep Server Components async**: Fetch data directly
2. **Minimize Client Components**: Use 'use client' sparingly
3. **Pass serializable props**: No functions or class instances
4. **Cache appropriately**: Use revalidate for dynamic data
5. **Handle errors**: Use error boundaries
6. **Type everything**: TypeScript for safety
