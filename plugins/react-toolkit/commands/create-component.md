---
description: Create a new React component with TypeScript, proper structure, and optional tests
allowed-tools: Write, Read, Bash(mkdir:*)
argument-hint: [component-name] [--type=functional|class] [--with-tests]
---

# Create React Component

Create a new React component with proper TypeScript structure.

## Context

- Current directory: !`pwd`
- Existing components: !`find src/components -name "*.tsx" -o -name "*.ts" 2>/dev/null | head -10`

## Your Task

Create a well-structured React component with TypeScript.

### Arguments

- `$1`: Component name (required) - PascalCase (e.g., UserProfile, TodoList)
- `$2`: Component type (optional) - `functional` (default) or `class`
- `$3`: Include tests (optional) - `--with-tests` to generate test file

### Steps

1. **Determine component location:**
   - Check if `src/components` exists
   - If not, check for `src/` directory
   - Create appropriate directory structure

2. **Create component file structure:**

   **For simple component:**
   ```
   src/components/ComponentName.tsx
   ```

   **For complex component with styles:**
   ```
   src/components/ComponentName/
   ├── ComponentName.tsx
   ├── ComponentName.module.css
   └── index.ts
   ```

3. **Generate functional component (default):**

   ```tsx
   interface ComponentNameProps {
     // Add props here
   }
   
   export function ComponentName({}: ComponentNameProps) {
     return (
       <div>
         <h1>ComponentName</h1>
       </div>
     );
   }
   ```

4. **For component with common props:**

   ```tsx
   interface ComponentNameProps {
     className?: string;
     children?: React.ReactNode;
   }
   
   export function ComponentName({ className, children }: ComponentNameProps) {
     return (
       <div className={className}>
         {children}
       </div>
     );
   }
   ```

5. **For component with state:**

   ```tsx
   import { useState } from 'react';
   
   interface ComponentNameProps {
     initialValue?: number;
   }
   
   export function ComponentName({ initialValue = 0 }: ComponentNameProps) {
     const [count, setCount] = useState(initialValue);
     
     return (
       <div>
         <p>Count: {count}</p>
         <button onClick={() => setCount(count + 1)}>Increment</button>
       </div>
     );
   }
   ```

6. **For component with data fetching:**

   ```tsx
   import { useQuery } from '@tanstack/react-query';
   
   interface ComponentNameProps {
     id: string;
   }
   
   export function ComponentName({ id }: ComponentNameProps) {
     const { data, isLoading, error } = useQuery({
       queryKey: ['item', id],
       queryFn: () => fetchItem(id),
     });
     
     if (isLoading) return <div>Loading...</div>;
     if (error) return <div>Error: {error.message}</div>;
     
     return (
       <div>
         <h1>{data.title}</h1>
       </div>
     );
   }
   ```

7. **If using module CSS, create ComponentName.module.css:**

   ```css
   .container {
     padding: 1rem;
   }
   
   .title {
     font-size: 1.5rem;
     font-weight: bold;
   }
   ```

   And import in component:
   ```tsx
   import styles from './ComponentName.module.css';
   
   export function ComponentName() {
     return (
       <div className={styles.container}>
         <h1 className={styles.title}>Title</h1>
       </div>
     );
   }
   ```

8. **Create index.ts for barrel export:**

   ```ts
   export { ComponentName } from './ComponentName';
   export type { ComponentNameProps } from './ComponentName';
   ```

9. **If --with-tests flag provided, create test file:**

   **ComponentName.test.tsx:**
   ```tsx
   import { render, screen } from '@testing-library/react';
   import { ComponentName } from './ComponentName';
   
   describe('ComponentName', () => {
     it('renders without crashing', () => {
       render(<ComponentName />);
       expect(screen.getByText('ComponentName')).toBeInTheDocument();
     });
     
     it('accepts className prop', () => {
       const { container } = render(<ComponentName className="custom" />);
       expect(container.firstChild).toHaveClass('custom');
     });
   });
   ```

   **For component with interactions:**
   ```tsx
   import { render, screen, fireEvent } from '@testing-library/react';
   import { ComponentName } from './ComponentName';
   
   describe('ComponentName', () => {
     it('increments count on button click', () => {
       render(<ComponentName />);
       const button = screen.getByRole('button', { name: /increment/i });
       
       fireEvent.click(button);
       
       expect(screen.getByText(/count: 1/i)).toBeInTheDocument();
     });
   });
   ```

10. **Provide usage example to user:**

    ```tsx
    import { ComponentName } from './components/ComponentName';
    
    function App() {
      return (
        <div>
          <ComponentName />
        </div>
      );
    }
    ```

### Examples

**Example 1: Simple component**
```
/create-component Button
```

Creates `src/components/Button.tsx` with basic functional component.

**Example 2: Component with tests**
```
/create-component UserCard --with-tests
```

Creates `src/components/UserCard/` directory with component and test files.

**Example 3: Complex component**
```
/create-component TodoList functional --with-tests
```

Creates full component structure with TypeScript, styles, and tests.

## Notes

- Use PascalCase for component names
- Default to functional components (hooks-based)
- Include TypeScript interfaces for props
- Add common props (className, children) when appropriate
- Create barrel exports (index.ts) for complex components
- Follow project's existing component structure if detected
- Use CSS modules for styling to avoid conflicts
- Include basic accessibility attributes (aria-labels, roles)
