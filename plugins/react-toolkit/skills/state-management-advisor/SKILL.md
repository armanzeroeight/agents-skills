---
name: state-management-advisor
description: Choose and implement React state management solutions including Context, Zustand, Redux Toolkit, TanStack Query, and Jotai. Use when selecting state management, implementing global state, or managing server state in React applications.
---

# State Management Advisor

Choose and implement the right state management solution for React applications.

## Quick Start

Use local state by default, Context for simple global state, Zustand for complex client state, and TanStack Query for server state.

## Instructions

### Decision Tree

**Start here:**
1. Is it server data (from API)? → Use TanStack Query
2. Is it used in one component only? → Use useState
3. Is it simple global state (theme, auth)? → Use Context
4. Is it complex client state? → Use Zustand or Redux Toolkit

### Local State (useState)

For component-specific state.

**Basic usage:**
```tsx
function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

**With objects:**
```tsx
function Form() {
  const [formData, setFormData] = useState({
    name: '',
    email: ''
  });
  
  const handleChange = (field: string, value: string) => {
    setFormData(prev => ({ ...prev, [field]: value }));
  };
  
  return (
    <form>
      <input
        value={formData.name}
        onChange={e => handleChange('name', e.target.value)}
      />
    </form>
  );
}
```

### React Context

For simple global state shared across components.

**Setup:**
```tsx
interface AuthContextValue {
  user: User | null;
  login: (credentials: Credentials) => Promise<void>;
  logout: () => void;
}

const AuthContext = createContext<AuthContextValue | null>(null);

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  
  const login = async (credentials: Credentials) => {
    const user = await api.login(credentials);
    setUser(user);
  };
  
  const logout = () => {
    setUser(null);
  };
  
  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) throw new Error('useAuth must be used within AuthProvider');
  return context;
}
```

**Usage:**
```tsx
function App() {
  return (
    <AuthProvider>
      <Router />
    </AuthProvider>
  );
}

function Profile() {
  const { user, logout } = useAuth();
  return <div>{user?.name} <button onClick={logout}>Logout</button></div>;
}
```

**Optimization - Split contexts:**
```tsx
// Separate frequently changing data
const ThemeContext = createContext<Theme>(null);
const ThemeUpdateContext = createContext<(theme: Theme) => void>(null);

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  return (
    <ThemeContext.Provider value={theme}>
      <ThemeUpdateContext.Provider value={setTheme}>
        {children}
      </ThemeUpdateContext.Provider>
    </ThemeContext.Provider>
  );
}
```

### Zustand

Lightweight global state with minimal boilerplate.

**Installation:**
```bash
npm install zustand
```

**Basic store:**
```tsx
import { create } from 'zustand';

interface CounterStore {
  count: number;
  increment: () => void;
  decrement: () => void;
}

const useCounterStore = create<CounterStore>((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
}));

// Usage
function Counter() {
  const count = useCounterStore((state) => state.count);
  const increment = useCounterStore((state) => state.increment);
  
  return <button onClick={increment}>Count: {count}</button>;
}
```

**With async actions:**
```tsx
interface UserStore {
  users: User[];
  loading: boolean;
  fetchUsers: () => Promise<void>;
}

const useUserStore = create<UserStore>((set) => ({
  users: [],
  loading: false,
  fetchUsers: async () => {
    set({ loading: true });
    const users = await api.fetchUsers();
    set({ users, loading: false });
  },
}));
```

**With middleware:**
```tsx
import { persist } from 'zustand/middleware';

const useStore = create(
  persist(
    (set) => ({
      token: null,
      setToken: (token) => set({ token }),
    }),
    {
      name: 'auth-storage',
    }
  )
);
```

### Redux Toolkit

For complex state with many interactions.

**Installation:**
```bash
npm install @reduxjs/toolkit react-redux
```

**Setup store:**
```tsx
import { configureStore, createSlice } from '@reduxjs/toolkit';

const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment: (state) => {
      state.value += 1;
    },
    decrement: (state) => {
      state.value -= 1;
    },
  },
});

export const { increment, decrement } = counterSlice.actions;

export const store = configureStore({
  reducer: {
    counter: counterSlice.reducer,
  },
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

**Provider:**
```tsx
import { Provider } from 'react-redux';

function App() {
  return (
    <Provider store={store}>
      <Router />
    </Provider>
  );
}
```

**Usage:**
```tsx
import { useSelector, useDispatch } from 'react-redux';

function Counter() {
  const count = useSelector((state: RootState) => state.counter.value);
  const dispatch = useDispatch();
  
  return (
    <button onClick={() => dispatch(increment())}>
      Count: {count}
    </button>
  );
}
```

**Async with createAsyncThunk:**
```tsx
import { createAsyncThunk } from '@reduxjs/toolkit';

export const fetchUsers = createAsyncThunk(
  'users/fetch',
  async () => {
    const response = await api.fetchUsers();
    return response.data;
  }
);

const usersSlice = createSlice({
  name: 'users',
  initialState: { data: [], loading: false },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchUsers.pending, (state) => {
        state.loading = true;
      })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        state.data = action.payload;
        state.loading = false;
      });
  },
});
```

### TanStack Query (React Query)

For server state management.

**Installation:**
```bash
npm install @tanstack/react-query
```

**Setup:**
```tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient();

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <Router />
    </QueryClientProvider>
  );
}
```

**Fetching data:**
```tsx
import { useQuery } from '@tanstack/react-query';

function Users() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['users'],
    queryFn: fetchUsers,
  });
  
  if (isLoading) return <Spinner />;
  if (error) return <Error message={error.message} />;
  
  return <UserList users={data} />;
}
```

**Mutations:**
```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

function CreateUser() {
  const queryClient = useQueryClient();
  
  const mutation = useMutation({
    mutationFn: createUser,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
  
  return (
    <button onClick={() => mutation.mutate({ name: 'John' })}>
      Create User
    </button>
  );
}
```

**With parameters:**
```tsx
function User({ id }: { id: string }) {
  const { data } = useQuery({
    queryKey: ['user', id],
    queryFn: () => fetchUser(id),
  });
  
  return <div>{data?.name}</div>;
}
```

**Optimistic updates:**
```tsx
const mutation = useMutation({
  mutationFn: updateUser,
  onMutate: async (newUser) => {
    await queryClient.cancelQueries({ queryKey: ['users'] });
    const previousUsers = queryClient.getQueryData(['users']);
    
    queryClient.setQueryData(['users'], (old) => [...old, newUser]);
    
    return { previousUsers };
  },
  onError: (err, newUser, context) => {
    queryClient.setQueryData(['users'], context.previousUsers);
  },
  onSettled: () => {
    queryClient.invalidateQueries({ queryKey: ['users'] });
  },
});
```

### Jotai

Atomic state management.

**Installation:**
```bash
npm install jotai
```

**Basic atoms:**
```tsx
import { atom, useAtom } from 'jotai';

const countAtom = atom(0);

function Counter() {
  const [count, setCount] = useAtom(countAtom);
  
  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

**Derived atoms:**
```tsx
const countAtom = atom(0);
const doubleCountAtom = atom((get) => get(countAtom) * 2);

function Display() {
  const [doubleCount] = useAtom(doubleCountAtom);
  return <div>Double: {doubleCount}</div>;
}
```

**Async atoms:**
```tsx
const usersAtom = atom(async () => {
  const response = await fetch('/api/users');
  return response.json();
});

function Users() {
  const [users] = useAtom(usersAtom);
  return <UserList users={users} />;
}
```

## Comparison Matrix

| Solution | Best For | Pros | Cons |
|----------|----------|------|------|
| useState | Local state | Simple, built-in | Limited to component |
| Context | Simple global state | Built-in, no deps | Can cause re-renders |
| Zustand | Complex client state | Minimal, fast | Smaller ecosystem |
| Redux Toolkit | Large apps, teams | Powerful, DevTools | Verbose, learning curve |
| TanStack Query | Server state | Caching, sync | Not for client state |
| Jotai | Atomic state | Fine-grained, modern | Smaller ecosystem |

## Common Patterns

### Combining Solutions

**TanStack Query + Zustand:**
```tsx
// Server state with TanStack Query
const { data: users } = useQuery(['users'], fetchUsers);

// Client state with Zustand
const selectedUserId = useStore((state) => state.selectedUserId);
```

**Context + TanStack Query:**
```tsx
function AuthProvider({ children }) {
  const { data: user } = useQuery(['me'], fetchCurrentUser);
  
  return (
    <AuthContext.Provider value={{ user }}>
      {children}
    </AuthContext.Provider>
  );
}
```

### Form State

**Controlled with useState:**
```tsx
function Form() {
  const [values, setValues] = useState({ name: '', email: '' });
  
  const handleSubmit = (e) => {
    e.preventDefault();
    api.submit(values);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        value={values.name}
        onChange={e => setValues(prev => ({ ...prev, name: e.target.value }))}
      />
    </form>
  );
}
```

**With form library:**
```tsx
import { useForm } from 'react-hook-form';

function Form() {
  const { register, handleSubmit } = useForm();
  
  const onSubmit = (data) => api.submit(data);
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('name')} />
    </form>
  );
}
```

## Troubleshooting

**Context causing re-renders:**
- Split into multiple contexts
- Use useMemo for context value
- Consider Zustand instead

**Stale data in TanStack Query:**
- Adjust staleTime and cacheTime
- Use refetchInterval for polling
- Invalidate queries after mutations

**Redux boilerplate:**
- Use Redux Toolkit (not legacy Redux)
- Use createSlice for reducers
- Use createAsyncThunk for async

**State updates not reflecting:**
- Check if mutating state directly
- Use functional updates
- Verify dependencies in useEffect
