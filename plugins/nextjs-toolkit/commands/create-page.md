---
description: Create a new Next.js page with appropriate rendering strategy and boilerplate
allowed-tools: Read, Write, Bash(mkdir:*)
argument-hint: <page-path> [--ssr|--ssg|--isr]
---

# Create Page

Generate a new Next.js page with the appropriate rendering strategy.

## Context

- Current directory: !`pwd`
- Next.js version: !`cat package.json | grep '"next"' || echo "Not found"`
- App Router: !`[ -d "app" ] && echo "Yes" || echo "No"`

## Your Task

Create a new Next.js page with proper structure and rendering strategy.

### Arguments

- `$1`: Page path (e.g., "products/[id]", "blog", "dashboard")
- `$2`: Optional flag for rendering strategy:
  - `--ssr`: Server-Side Rendering
  - `--ssg`: Static Site Generation
  - `--isr`: Incremental Static Regeneration
  - (default: SSG for App Router, SSG for Pages Router)

### Steps

1. **Detect Next.js structure**
   - Check if using App Router (app/ directory) or Pages Router (pages/ directory)
   - Determine Next.js version from package.json

2. **Parse page path**
   - Extract route segments
   - Identify dynamic segments ([id], [...slug])
   - Determine if nested route

3. **Create directory structure**
   - For App Router: `app/<path>/page.tsx`
   - For Pages Router: `pages/<path>.tsx` or `pages/<path>/index.tsx`
   - Create parent directories as needed

4. **Generate page component**
   
   **For App Router:**
   - Server Component by default
   - Add 'use client' if interactive
   - Include metadata export
   - Add loading.tsx and error.tsx if appropriate
   
   **For Pages Router:**
   - Add getStaticProps for SSG/ISR
   - Add getServerSideProps for SSR
   - Add getStaticPaths for dynamic routes

5. **Add TypeScript types**
   - Props interface
   - Params type for dynamic routes
   - Metadata type for App Router

6. **Provide usage instructions**
   - How to navigate to the page
   - How to pass data
   - How to customize rendering
   - Next steps for implementation

### Output Format

```markdown
# Page Created: /<path>

## Files Created
- Page: app/<path>/page.tsx (or pages/<path>.tsx)
- Loading: app/<path>/loading.tsx (if App Router)
- Error: app/<path>/error.tsx (if App Router)

## Rendering Strategy
- Method: [SSG/SSR/ISR]
- Rationale: [Why this strategy was chosen]

## Usage

### Navigation
\`\`\`typescript
import Link from 'next/link';
<Link href="/<path>">Go to page</Link>
\`\`\`

### Data Fetching
[Instructions based on rendering strategy]

## Next Steps
1. Implement data fetching logic
2. Add UI components
3. Configure metadata/SEO
4. Test on development server
5. Optimize performance
```

## Examples

**Create a static about page:**
```
/create-page about
```

**Create a dynamic product page with SSG:**
```
/create-page products/[id] --ssg
```

**Create a user dashboard with SSR:**
```
/create-page dashboard --ssr
```

**Create a blog with ISR:**
```
/create-page blog/[slug] --isr
```

**Expected output:**
- Complete page boilerplate
- Appropriate rendering strategy
- TypeScript types
- Loading and error states (App Router)
- Usage examples
- Next steps
