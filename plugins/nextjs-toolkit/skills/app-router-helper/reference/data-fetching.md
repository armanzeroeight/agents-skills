# Data Fetching

Advanced data fetching strategies in Next.js App Router.

## Fetch API Extensions

Next.js extends the native fetch API with caching and revalidation.

### Cache Options

```typescript
// Cached indefinitely (default)
fetch('https://api.example.com/data');

// No caching
fetch('https://api.example.com/data', { cache: 'no-store' });

// Force cache
fetch('https://api.example.com/data', { cache: 'force-cache' });
```

### Revalidation

```typescript
// Time-based revalidation
fetch('https://api.example.com/data', {
  next: { revalidate: 3600 } // seconds
});

// Tag-based revalidation
fetch('https://api.example.com/data', {
  next: { tags: ['products'] }
});
```

## Patterns

### Parallel Fetching

```typescript
async function Page() {
  const [user, posts, comments] = await Promise.all([
    fetchUser(),
    fetchPosts(),
    fetchComments(),
  ]);
  
  return <div>...</div>;
}
```

### Sequential Fetching

```typescript
async function Page() {
  const user = await fetchUser();
  const posts = await fetchUserPosts(user.id);
  const comments = await fetchPostComments(posts[0].id);
  
  return <div>...</div>;
}
```

### Streaming with Suspense

```typescript
import { Suspense } from 'react';

export default function Page() {
  return (
    <div>
      <Suspense fallback={<UserSkeleton />}>
        <UserProfile />
      </Suspense>
      <Suspense fallback={<PostsSkeleton />}>
        <UserPosts />
      </Suspense>
    </div>
  );
}
```

## Revalidation Strategies

### On-Demand Revalidation

```typescript
// app/api/revalidate/route.ts
import { revalidatePath, revalidateTag } from 'next/cache';

export async function POST(request: Request) {
  // Revalidate specific path
  revalidatePath('/products');
  
  // Revalidate by tag
  revalidateTag('products');
  
  return Response.json({ revalidated: true });
}
```

### Webhook Integration

```typescript
// app/api/webhook/route.ts
export async function POST(request: Request) {
  const body = await request.json();
  
  if (body.event === 'product.updated') {
    revalidateTag('products');
  }
  
  return Response.json({ received: true });
}
```

## Error Handling

```typescript
async function Page() {
  try {
    const data = await fetchData();
    return <Content data={data} />;
  } catch (error) {
    return <ErrorMessage error={error} />;
  }
}
```

## Best Practices

1. **Fetch where needed**: Co-locate data fetching with components
2. **Use parallel fetching**: Improve performance with Promise.all
3. **Cache appropriately**: Balance freshness and performance
4. **Handle errors**: Always handle fetch failures
5. **Use Suspense**: Stream content for better UX
6. **Revalidate strategically**: Use tags for targeted updates
