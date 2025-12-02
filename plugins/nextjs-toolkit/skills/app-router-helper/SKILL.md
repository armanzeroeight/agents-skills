---
name: app-router-helper
description: Implement Next.js App Router patterns including Server Components, Client Components, layouts, route organization, and data fetching. Use when building with App Router, organizing routes, or implementing modern Next.js patterns. Trigger words include "App Router", "Server Component", "Client Component", "layout", "app directory".
---

# App Router Helper

Implement Next.js App Router patterns for modern React applications.

## Quick Start

App Router (Next.js 13+) uses file-system routing in the `app/` directory with Server Components by default.

**Key concepts:**
- Server Components (default): Render on server, reduce bundle size
- Client Components ('use client'): Interactive, use hooks
- Layouts: Shared UI across routes
- Loading/Error: Automatic UI states

## Instructions

### Step 1: Understand File Structure

**Basic structure:**
```
app/
├── layout.tsx          # Root layout (required)
├── page.tsx            # Home page (/)
├── loading.tsx         # Loading UI
├── error.tsx           # Error UI
├── not-found.tsx       # 404 page
└── about/
    └── page.tsx        # About page (/about)
```

**Special files:**
- `layout.tsx`: Shared UI, doesn't re-render
- `page.tsx`: Unique UI for route
- `loading.tsx`: Loading state (Suspense boundary)
- `error.tsx`: Error boundary
- `template.tsx`: Re-renders on navigation
- `route.ts`: API endpoint

### Step 2: Create Layouts

**Root layout (required):**
```typescript
// app/layout.tsx
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body>
        <Header />
        {children}
        <Footer />
      </body>
    </html>
  );
}
```

**Nested layouts:**
```typescript
// app/dashboard/layout.tsx
export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <div>
      <Sidebar />
      <main>{children}</main>
    </div>
  );
}
```

**Layouts persist across navigation and don't re-render.**

### Step 3: Server vs Client Components

**Server Component (default):**
```typescript
// app/products/page.tsx
// No 'use client' = Server Component

async function ProductsPage() {
  // Can fetch data directly
  const products = await db.products.findMany();
  
  return (
    <div>
      {products.map(p => (
        <ProductCard key={p.id} product={p} />
      ))}
    </div>
  );
}

export default ProductsPage;
```

**Client Component:**
```typescript
// app/components/AddToCart.tsx
'use client';

import { useState } from 'react';

export function AddToCart({ productId }: { productId: string }) {
  const [count, setCount] = useState(1);
  
  const handleAdd = () => {
    // Client-side logic
    addToCart(productId, count);
  };
  
  return (
    <div>
      <button onClick={() => setCount(count - 1)}>-</button>
      <span>{count}</span>
      <button onClick={() => setCount(count + 1)}>+</button>
      <button onClick={handleAdd}>Add to Cart</button>
    </div>
  );
}
```

**When to use 'use client':**
- Event handlers (onClick, onChange)
- React hooks (useState, useEffect, useContext)
- Browser APIs (localStorage, window)
- Third-party libraries requiring client

### Step 4: Implement Data Fetching

**Server Component data fetching:**
```typescript
// app/posts/page.tsx
async function PostsPage() {
  // Fetch in Server Component
  const posts = await fetch('https://api.example.com/posts', {
    next: { revalidate: 3600 } // Cache for 1 hour
  }).then(res => res.json());
  
  return <PostList posts={posts} />;
}
```

**Parallel data fetching:**
```typescript
async function Page() {
  // Fetch in parallel
  const [user, posts] = await Promise.all([
    fetchUser(),
    fetchPosts(),
  ]);
  
  return (
    <div>
      <UserProfile user={user} />
      <PostList posts={posts} />
    </div>
  );
}
```

**Sequential data fetching:**
```typescript
async function Page() {
  const user = await fetchUser();
  const posts = await fetchUserPosts(user.id); // Depends on user
  
  return <div>...</div>;
}
```

### Step 5: Organize Routes

**Route groups (don't affect URL):**
```
app/
├── (marketing)/
│   ├── layout.tsx      # Marketing layout
│   ├── about/
│   │   └── page.tsx    # /about
│   └── contact/
│       └── page.tsx    # /contact
└── (shop)/
    ├── layout.tsx      # Shop layout
    └── products/
        └── page.tsx    # /products
```

**Dynamic routes:**
```
app/
└── products/
    └── [id]/
        └── page.tsx    # /products/123
```

```typescript
// app/products/[id]/page.tsx
export default function ProductPage({
  params,
}: {
  params: { id: string }
}) {
  return <div>Product {params.id}</div>;
}
```

**Catch-all routes:**
```
app/
└── docs/
    └── [...slug]/
        └── page.tsx    # /docs/a, /docs/a/b, /docs/a/b/c
```

### Step 6: Handle Loading and Errors

**Loading UI:**
```typescript
// app/dashboard/loading.tsx
export default function Loading() {
  return <div>Loading dashboard...</div>;
}
```

**Error handling:**
```typescript
// app/dashboard/error.tsx
'use client'; // Error components must be Client Components

export default function Error({
  error,
  reset,
}: {
  error: Error;
  reset: () => void;
}) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={reset}>Try again</button>
    </div>
  );
}
```

**Not found:**
```typescript
// app/not-found.tsx
export default function NotFound() {
  return <div>404 - Page Not Found</div>;
}
```

## Common Patterns

### Streaming with Suspense

```typescript
import { Suspense } from 'react';

export default function Page() {
  return (
    <div>
      <h1>Dashboard</h1>
      <Suspense fallback={<Skeleton />}>
        <SlowComponent />
      </Suspense>
      <FastComponent />
    </div>
  );
}
```

### Parallel Routes

```
app/
└── dashboard/
    ├── layout.tsx
    ├── @analytics/
    │   └── page.tsx
    ├── @team/
    │   └── page.tsx
    └── page.tsx
```

```typescript
// app/dashboard/layout.tsx
export default function Layout({
  children,
  analytics,
  team,
}: {
  children: React.ReactNode;
  analytics: React.ReactNode;
  team: React.ReactNode;
}) {
  return (
    <div>
      {children}
      <div className="grid">
        {analytics}
        {team}
      </div>
    </div>
  );
}
```

### Intercepting Routes

```
app/
└── photos/
    ├── [id]/
    │   └── page.tsx
    └── (.)[id]/
        └── page.tsx    # Intercepts /photos/[id]
```

### Metadata

```typescript
// app/products/[id]/page.tsx
import { Metadata } from 'next';

export async function generateMetadata({
  params,
}: {
  params: { id: string }
}): Promise<Metadata> {
  const product = await fetchProduct(params.id);
  
  return {
    title: product.name,
    description: product.description,
    openGraph: {
      images: [product.image],
    },
  };
}
```

### API Routes

```typescript
// app/api/products/route.ts
import { NextResponse } from 'next/server';

export async function GET(request: Request) {
  const products = await db.products.findMany();
  return NextResponse.json(products);
}

export async function POST(request: Request) {
  const body = await request.json();
  const product = await db.products.create({ data: body });
  return NextResponse.json(product, { status: 201 });
}
```

**Dynamic API routes:**
```typescript
// app/api/products/[id]/route.ts
export async function GET(
  request: Request,
  { params }: { params: { id: string } }
) {
  const product = await db.products.findUnique({
    where: { id: params.id }
  });
  return NextResponse.json(product);
}
```

## Advanced

For detailed patterns:
- [Server Components](reference/server-components.md) - Deep dive into Server Components
- [Client Components](reference/client-components.md) - Client Component patterns
- [Data Fetching](reference/data-fetching.md) - Advanced data fetching strategies

## Troubleshooting

**"use client" not working:**
- Must be at top of file
- Check for Server Component imports
- Verify no async in Client Components

**Data not updating:**
- Check cache configuration
- Use revalidatePath or revalidateTag
- Verify fetch cache settings

**Layout not applying:**
- Ensure layout.tsx exists
- Check file naming (must be exact)
- Verify export default

**Hydration errors:**
- Server and client HTML must match
- Avoid using browser APIs in Server Components
- Check for dynamic content (dates, random)

## Best Practices

1. **Default to Server Components**: Only use 'use client' when needed
2. **Fetch data where needed**: Co-locate data fetching with components
3. **Use layouts**: Share UI and avoid re-renders
4. **Implement loading states**: Use loading.tsx and Suspense
5. **Handle errors**: Add error.tsx boundaries
6. **Optimize metadata**: Use generateMetadata for SEO
7. **Stream content**: Use Suspense for better UX
8. **Type everything**: Use TypeScript for params and props
