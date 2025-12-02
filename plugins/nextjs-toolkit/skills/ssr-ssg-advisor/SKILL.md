---
name: ssr-ssg-advisor
description: Choose optimal Next.js rendering strategy (SSR, SSG, ISR, CSR) based on content type, update frequency, and performance requirements. Use when deciding how to render pages, optimizing performance, or implementing data fetching. Trigger words include "rendering", "SSR", "SSG", "ISR", "static", "server-side".
---

# SSR/SSG Advisor

Choose the optimal rendering strategy for Next.js pages based on requirements.

## Quick Start

**Decision criteria:**
- **SSG**: Static content, pre-render at build → Best performance
- **ISR**: Static with updates, revalidate periodically → Balance of both
- **SSR**: Dynamic per-request, personalized → Fresh data
- **CSR**: Client-side only, highly interactive → User-specific

## Instructions

### Step 1: Analyze Content Requirements

**Ask these questions:**
1. Does content change per user? (personalization)
2. How frequently does content update?
3. Is SEO critical?
4. What's the acceptable data freshness?
5. How many pages need to be generated?

### Step 2: Choose Rendering Strategy

**Static Site Generation (SSG):**
```typescript
// pages/products/[id].tsx
export async function getStaticProps({ params }) {
  const product = await fetchProduct(params.id);
  
  return {
    props: { product },
    // Optional: revalidate for ISR
    // revalidate: 60, // seconds
  };
}

export async function getStaticPaths() {
  const products = await fetchAllProducts();
  
  return {
    paths: products.map(p => ({ params: { id: p.id } })),
    fallback: 'blocking', // or false, or true
  };
}
```

**When to use SSG:**
- Marketing pages
- Blog posts
- Documentation
- Product catalogs (if manageable size)
- Any content that doesn't change often

**Server-Side Rendering (SSR):**
```typescript
// pages/dashboard.tsx
export async function getServerSideProps(context) {
  const session = await getSession(context);
  const data = await fetchUserData(session.userId);
  
  return {
    props: { data },
  };
}
```

**When to use SSR:**
- User dashboards
- Personalized content
- Real-time data
- Content requiring authentication
- Frequently changing data

**Incremental Static Regeneration (ISR):**
```typescript
// pages/blog/[slug].tsx
export async function getStaticProps({ params }) {
  const post = await fetchPost(params.slug);
  
  return {
    props: { post },
    revalidate: 60, // Regenerate every 60 seconds
  };
}
```

**When to use ISR:**
- Blog with frequent updates
- Product pages with price changes
- News sites
- Content that updates periodically
- Large sites where full rebuild is slow

**Client-Side Rendering (CSR):**
```typescript
'use client'; // App Router

import { useEffect, useState } from 'react';

function Dashboard() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetch('/api/user-data')
      .then(res => res.json())
      .then(setData);
  }, []);
  
  return <div>{data ? <Content data={data} /> : <Loading />}</div>;
}
```

**When to use CSR:**
- Highly interactive UIs
- User-specific data (after auth)
- Real-time updates
- SEO not required
- Data behind authentication

### Step 3: Implement Data Fetching

**App Router (Next.js 13+):**
```typescript
// app/products/page.tsx
async function ProductsPage() {
  // SSG: cached by default
  const products = await fetch('https://api.example.com/products');
  
  // ISR: revalidate periodically
  const products = await fetch('https://api.example.com/products', {
    next: { revalidate: 3600 } // 1 hour
  });
  
  // SSR: no caching
  const products = await fetch('https://api.example.com/products', {
    cache: 'no-store'
  });
  
  return <ProductList products={products} />;
}
```

**Pages Router:**
```typescript
// getStaticProps: SSG/ISR
// getServerSideProps: SSR
// useEffect + fetch: CSR
```

### Step 4: Configure Caching and Revalidation

**On-demand revalidation:**
```typescript
// app/api/revalidate/route.ts
import { revalidatePath, revalidateTag } from 'next/cache';

export async function POST(request) {
  const path = request.nextUrl.searchParams.get('path');
  
  if (path) {
    revalidatePath(path);
    return Response.json({ revalidated: true });
  }
  
  return Response.json({ revalidated: false });
}
```

**Tagged caching:**
```typescript
// Fetch with tags
const data = await fetch('https://api.example.com/products', {
  next: { tags: ['products'] }
});

// Revalidate by tag
revalidateTag('products');
```

### Step 5: Handle Fallback Strategies

**getStaticPaths fallback options:**
```typescript
export async function getStaticPaths() {
  return {
    paths: [...],
    fallback: false,    // 404 for non-pre-rendered paths
    // fallback: true,  // Generate on-demand, show loading
    // fallback: 'blocking', // Generate on-demand, wait for page
  };
}
```

## Common Patterns

### Hybrid Approach

```typescript
// Mix strategies in same app
// - SSG for marketing pages
// - ISR for blog posts
// - SSR for user dashboard
// - CSR for interactive features

// app/layout.tsx (SSG)
export default function RootLayout({ children }) {
  return <html><body>{children}</body></html>;
}

// app/blog/[slug]/page.tsx (ISR)
async function BlogPost({ params }) {
  const post = await fetch(`/api/posts/${params.slug}`, {
    next: { revalidate: 60 }
  });
  return <Article post={post} />;
}

// app/dashboard/page.tsx (SSR)
async function Dashboard() {
  const data = await fetch('/api/user', { cache: 'no-store' });
  return <DashboardContent data={data} />;
}
```

### Optimistic UI with ISR

```typescript
// Show stale data immediately, revalidate in background
export async function getStaticProps() {
  const data = await fetchData();
  
  return {
    props: { data },
    revalidate: 1, // Revalidate every second
  };
}
```

### Conditional Rendering

```typescript
// Different rendering based on route
export async function getServerSideProps(context) {
  const { preview } = context;
  
  if (preview) {
    // SSR for preview mode
    const data = await fetchDraftContent();
    return { props: { data, preview: true } };
  }
  
  // Redirect to SSG version
  return {
    redirect: {
      destination: '/static-version',
      permanent: false,
    },
  };
}
```

## Decision Matrix

| Requirement | SSG | ISR | SSR | CSR |
|------------|-----|-----|-----|-----|
| SEO Critical | ✅ | ✅ | ✅ | ❌ |
| Fast TTFB | ✅ | ✅ | ❌ | ❌ |
| Fresh Data | ❌ | ⚠️ | ✅ | ✅ |
| Personalized | ❌ | ❌ | ✅ | ✅ |
| Large Scale | ⚠️ | ✅ | ✅ | ✅ |
| Build Time | ❌ | ✅ | ✅ | ✅ |

✅ = Excellent, ⚠️ = Acceptable, ❌ = Poor

## Performance Considerations

**SSG:**
- Fastest: Pre-rendered at build time
- Best for CDN caching
- Long build times for large sites
- Stale data until next build

**ISR:**
- Fast initial load (cached)
- Automatic updates
- Best of both worlds
- Slight delay for revalidation

**SSR:**
- Always fresh data
- Slower TTFB
- Higher server load
- Can't be cached at CDN edge

**CSR:**
- Slow initial load
- No SEO benefits
- Reduces server load
- Best for authenticated content

## Troubleshooting

**Build taking too long:**
- Use ISR instead of SSG
- Reduce number of pre-rendered paths
- Use fallback: 'blocking'

**Stale data showing:**
- Reduce revalidate time
- Implement on-demand revalidation
- Consider SSR for critical data

**High server costs:**
- Move from SSR to ISR where possible
- Implement proper caching
- Use CDN for static assets

**SEO issues:**
- Avoid CSR for public content
- Use SSG or SSR
- Implement proper metadata

## Best Practices

1. **Start with SSG**: Default to static, move to dynamic only when needed
2. **Use ISR for updates**: Better than full SSR for most cases
3. **Combine strategies**: Different pages can use different methods
4. **Cache aggressively**: Use CDN and browser caching
5. **Monitor performance**: Track TTFB, build times, server load
6. **Implement fallbacks**: Handle errors and loading states
7. **Test thoroughly**: Verify behavior in production
