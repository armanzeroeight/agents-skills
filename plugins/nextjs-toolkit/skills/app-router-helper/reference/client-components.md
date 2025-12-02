# Client Components

Guide to Client Components in Next.js App Router.

## What are Client Components

Client Components render on the client and enable interactivity. Mark with 'use client' directive.

## When to Use

**Required for:**
- Event handlers (onClick, onChange, onSubmit)
- React hooks (useState, useEffect, useContext)
- Browser APIs (localStorage, window, document)
- Third-party libraries requiring client
- Custom hooks

## Basic Usage

```typescript
'use client';

import { useState } from 'react';

export function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}
```

## Common Patterns

### Form Handling

```typescript
'use client';

export function ContactForm() {
  const [status, setStatus] = useState('idle');
  
  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    setStatus('submitting');
    
    const formData = new FormData(e.target as HTMLFormElement);
    await fetch('/api/contact', {
      method: 'POST',
      body: formData,
    });
    
    setStatus('success');
  }
  
  return (
    <form onSubmit={handleSubmit}>
      <input name="email" type="email" required />
      <button disabled={status === 'submitting'}>
        Submit
      </button>
    </form>
  );
}
```

### Data Fetching with SWR

```typescript
'use client';

import useSWR from 'swr';

export function UserProfile() {
  const { data, error } = useSWR('/api/user', fetcher);
  
  if (error) return <div>Failed to load</div>;
  if (!data) return <div>Loading...</div>;
  
  return <div>Hello {data.name}</div>;
}
```

### Local Storage

```typescript
'use client';

import { useEffect, useState } from 'react';

export function ThemeToggle() {
  const [theme, setTheme] = useState('light');
  
  useEffect(() => {
    const saved = localStorage.getItem('theme');
    if (saved) setTheme(saved);
  }, []);
  
  const toggle = () => {
    const newTheme = theme === 'light' ? 'dark' : 'light';
    setTheme(newTheme);
    localStorage.setItem('theme', newTheme);
  };
  
  return <button onClick={toggle}>Toggle Theme</button>;
}
```

## Optimization

### Minimize Client Components

```typescript
// Bad: Entire page is Client Component
'use client';
export default function Page() {
  const [state, setState] = useState();
  return <div>...</div>;
}

// Good: Only interactive part is Client Component
export default function Page() {
  return (
    <div>
      <StaticContent />
      <InteractiveWidget />
    </div>
  );
}
```

### Code Splitting

```typescript
import dynamic from 'next/dynamic';

const HeavyComponent = dynamic(() => import('./HeavyComponent'), {
  loading: () => <p>Loading...</p>,
  ssr: false,
});
```

## Best Practices

1. **Use sparingly**: Default to Server Components
2. **Push 'use client' down**: Only mark interactive parts
3. **Memoize callbacks**: Use useCallback for child props
4. **Memoize values**: Use useMemo for expensive computations
5. **Handle loading states**: Show feedback during async operations
