---
name: component-patterns
description: Implement React component patterns including composition, custom hooks, render props, HOCs, and compound components. Use when building reusable React components, implementing design patterns, or refactoring component architecture.
---

# Component Patterns

Implement React component patterns for building reusable, maintainable components.

## Quick Start

Use composition for most cases, custom hooks for shared logic, and compound components for flexible APIs.

## Instructions

### Composition Pattern

The default pattern for building flexible components.

**Basic composition:**
```tsx
function Card({ children }) {
  return <div className="card">{children}</div>;
}

function CardHeader({ children }) {
  return <div className="card-header">{children}</div>;
}

function CardBody({ children }) {
  return <div className="card-body">{children}</div>;
}

// Usage
<Card>
  <CardHeader>Title</CardHeader>
  <CardBody>Content</CardBody>
</Card>
```

**With props:**
```tsx
interface ButtonProps {
  variant?: 'primary' | 'secondary';
  size?: 'sm' | 'md' | 'lg';
  children: React.ReactNode;
}

function Button({ variant = 'primary', size = 'md', children }: ButtonProps) {
  return (
    <button className={`btn btn-${variant} btn-${size}`}>
      {children}
    </button>
  );
}
```

### Custom Hooks Pattern

Extract and reuse stateful logic across components.

**Basic custom hook:**
```tsx
function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);
  
  const toggle = useCallback(() => {
    setValue(v => !v);
  }, []);
  
  return [value, toggle] as const;
}

// Usage
function Component() {
  const [isOpen, toggleOpen] = useToggle();
  return <button onClick={toggleOpen}>{isOpen ? 'Close' : 'Open'}</button>;
}
```

**Data fetching hook:**
```tsx
function useFetch<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);
  
  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(setData)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [url]);
  
  return { data, loading, error };
}
```

**Form hook:**
```tsx
function useForm<T>(initialValues: T) {
  const [values, setValues] = useState(initialValues);
  
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setValues(prev => ({
      ...prev,
      [e.target.name]: e.target.value
    }));
  };
  
  const reset = () => setValues(initialValues);
  
  return { values, handleChange, reset };
}
```

### Compound Components Pattern

Create flexible component APIs with implicit state sharing.

**Basic compound component:**
```tsx
interface TabsContextValue {
  activeTab: string;
  setActiveTab: (tab: string) => void;
}

const TabsContext = createContext<TabsContextValue | null>(null);

function Tabs({ children, defaultTab }: { children: React.ReactNode; defaultTab: string }) {
  const [activeTab, setActiveTab] = useState(defaultTab);
  
  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
}

function TabList({ children }: { children: React.ReactNode }) {
  return <div className="tab-list">{children}</div>;
}

function Tab({ id, children }: { id: string; children: React.ReactNode }) {
  const context = useContext(TabsContext);
  if (!context) throw new Error('Tab must be used within Tabs');
  
  const { activeTab, setActiveTab } = context;
  
  return (
    <button
      className={activeTab === id ? 'active' : ''}
      onClick={() => setActiveTab(id)}
    >
      {children}
    </button>
  );
}

function TabPanel({ id, children }: { id: string; children: React.ReactNode }) {
  const context = useContext(TabsContext);
  if (!context) throw new Error('TabPanel must be used within Tabs');
  
  if (context.activeTab !== id) return null;
  return <div className="tab-panel">{children}</div>;
}

// Attach sub-components
Tabs.List = TabList;
Tabs.Tab = Tab;
Tabs.Panel = TabPanel;

// Usage
<Tabs defaultTab="home">
  <Tabs.List>
    <Tabs.Tab id="home">Home</Tabs.Tab>
    <Tabs.Tab id="profile">Profile</Tabs.Tab>
  </Tabs.List>
  <Tabs.Panel id="home">Home content</Tabs.Panel>
  <Tabs.Panel id="profile">Profile content</Tabs.Panel>
</Tabs>
```

### Render Props Pattern

Provide render flexibility through function props.

**Basic render prop:**
```tsx
interface MouseTrackerProps {
  render: (position: { x: number; y: number }) => React.ReactNode;
}

function MouseTracker({ render }: MouseTrackerProps) {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  
  useEffect(() => {
    const handleMove = (e: MouseEvent) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };
    
    window.addEventListener('mousemove', handleMove);
    return () => window.removeEventListener('mousemove', handleMove);
  }, []);
  
  return <>{render(position)}</>;
}

// Usage
<MouseTracker
  render={({ x, y }) => (
    <div>Mouse at {x}, {y}</div>
  )}
/>
```

**Children as function:**
```tsx
interface DataProviderProps<T> {
  url: string;
  children: (data: T | null, loading: boolean) => React.ReactNode;
}

function DataProvider<T>({ url, children }: DataProviderProps<T>) {
  const { data, loading } = useFetch<T>(url);
  return <>{children(data, loading)}</>;
}

// Usage
<DataProvider url="/api/users">
  {(users, loading) => (
    loading ? <Spinner /> : <UserList users={users} />
  )}
</DataProvider>
```

### Higher-Order Component (HOC) Pattern

Wrap components to add functionality (legacy pattern, prefer hooks).

**Basic HOC:**
```tsx
function withLoading<P extends object>(
  Component: React.ComponentType<P>
) {
  return function WithLoadingComponent(props: P & { loading: boolean }) {
    const { loading, ...rest } = props;
    
    if (loading) return <Spinner />;
    return <Component {...(rest as P)} />;
  };
}

// Usage
const UserListWithLoading = withLoading(UserList);
<UserListWithLoading users={users} loading={loading} />
```

**HOC with additional props:**
```tsx
function withAuth<P extends object>(
  Component: React.ComponentType<P & { user: User }>
) {
  return function WithAuthComponent(props: P) {
    const { user, loading } = useAuth();
    
    if (loading) return <Spinner />;
    if (!user) return <Navigate to="/login" />;
    
    return <Component {...props} user={user} />;
  };
}
```

## Performance Optimization Patterns

### React.memo

Prevent unnecessary re-renders of expensive components.

```tsx
const ExpensiveComponent = React.memo(function ExpensiveComponent({ data }) {
  // Expensive rendering logic
  return <div>{/* Complex UI */}</div>;
});

// With custom comparison
const MemoizedComponent = React.memo(
  function Component({ user }) {
    return <div>{user.name}</div>;
  },
  (prevProps, nextProps) => prevProps.user.id === nextProps.user.id
);
```

### useMemo

Memoize expensive calculations.

```tsx
function Component({ items }) {
  const sortedItems = useMemo(() => {
    return items.sort((a, b) => a.value - b.value);
  }, [items]);
  
  const total = useMemo(() => {
    return items.reduce((sum, item) => sum + item.price, 0);
  }, [items]);
  
  return <div>{/* Use sortedItems and total */}</div>;
}
```

### useCallback

Memoize functions to prevent child re-renders.

```tsx
function Parent() {
  const [count, setCount] = useState(0);
  
  const handleClick = useCallback(() => {
    setCount(c => c + 1);
  }, []);
  
  return <MemoizedChild onClick={handleClick} />;
}

const MemoizedChild = React.memo(function Child({ onClick }) {
  return <button onClick={onClick}>Click</button>;
});
```

### Code Splitting

Split code to reduce initial bundle size.

```tsx
const LazyComponent = React.lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <LazyComponent />
    </Suspense>
  );
}

// Route-based splitting
const Dashboard = React.lazy(() => import('./pages/Dashboard'));
const Profile = React.lazy(() => import('./pages/Profile'));

<Routes>
  <Route path="/dashboard" element={
    <Suspense fallback={<Spinner />}>
      <Dashboard />
    </Suspense>
  } />
</Routes>
```

### Virtual Scrolling

Handle large lists efficiently.

```tsx
import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualList({ items }) {
  const parentRef = useRef<HTMLDivElement>(null);
  
  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50,
  });
  
  return (
    <div ref={parentRef} style={{ height: '400px', overflow: 'auto' }}>
      <div style={{ height: `${virtualizer.getTotalSize()}px`, position: 'relative' }}>
        {virtualizer.getVirtualItems().map(virtualItem => (
          <div
            key={virtualItem.key}
            style={{
              position: 'absolute',
              top: 0,
              left: 0,
              width: '100%',
              height: `${virtualItem.size}px`,
              transform: `translateY(${virtualItem.start}px)`,
            }}
          >
            {items[virtualItem.index].name}
          </div>
        ))}
      </div>
    </div>
  );
}
```

## Common Patterns

### Container/Presentational Pattern

Separate logic from presentation.

```tsx
// Presentational component
interface UserListProps {
  users: User[];
  onDelete: (id: string) => void;
}

function UserList({ users, onDelete }: UserListProps) {
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>
          {user.name}
          <button onClick={() => onDelete(user.id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
}

// Container component
function UserListContainer() {
  const { data: users, isLoading } = useQuery(['users'], fetchUsers);
  const deleteMutation = useMutation(deleteUser);
  
  if (isLoading) return <Spinner />;
  
  return <UserList users={users} onDelete={deleteMutation.mutate} />;
}
```

### Provider Pattern

Share data across component tree.

```tsx
interface ThemeContextValue {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextValue | null>(null);

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');
  
  const toggleTheme = useCallback(() => {
    setTheme(t => t === 'light' ? 'dark' : 'light');
  }, []);
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) throw new Error('useTheme must be used within ThemeProvider');
  return context;
}
```

### Controlled vs Uncontrolled Components

**Controlled (recommended):**
```tsx
function ControlledInput() {
  const [value, setValue] = useState('');
  
  return (
    <input
      value={value}
      onChange={e => setValue(e.target.value)}
    />
  );
}
```

**Uncontrolled:**
```tsx
function UncontrolledInput() {
  const inputRef = useRef<HTMLInputElement>(null);
  
  const handleSubmit = () => {
    console.log(inputRef.current?.value);
  };
  
  return <input ref={inputRef} />;
}
```

## Troubleshooting

**Unnecessary re-renders:**
- Use React DevTools Profiler to identify
- Wrap with React.memo
- Use useMemo/useCallback appropriately
- Check if state is lifted too high

**Props drilling:**
- Use Context API for deeply nested props
- Consider component composition
- Extract intermediate components

**Stale closures in hooks:**
- Add dependencies to useEffect/useCallback
- Use functional updates: `setState(prev => prev + 1)`
- Use useRef for mutable values

**Memory leaks:**
- Clean up subscriptions in useEffect
- Cancel pending requests on unmount
- Remove event listeners
