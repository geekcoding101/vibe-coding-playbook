---
trigger: glob
globs: app/**/*.{ts,tsx}, pages/**/*.{ts,tsx}, src/**/*.{ts,tsx}
---

# Rule: Next.js + TypeScript

You are a Next.js + TypeScript expert, following official App Router and data fetching best practices.

## Core Mandatory Rules

### 1. Zero Tolerance for Magic Numbers/Strings ⭐
- Extract all literal values as `UPPER_SNAKE_CASE` constants (except 0/1/-1/empty strings/booleans)
- Location: Separate `constants.ts` or `config.ts` files, grouped by functionality
- Unify API endpoints, route paths, environment variables, UI config items as constants
- Use `as const` assertions to create readonly constant objects

### 2. TypeScript Strict Mode
- Must enable `strict: true` and `strictNullChecks: true`
- Avoid `any`, use `unknown` + type guards when necessary
- All function parameters and return values must have type annotations
- Explicitly declare Props types, avoid deprecated `React.FC<>`
- Prefer `interface` for objects, `type` for unions and utility types

### 3. App Router Structure
- Use Next.js 13+ App Router (`app/` directory)
- Route files: `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`, `not-found.tsx`
- Route groups: `(auth)/`, `(dashboard)/`, etc.
- API routes: `app/api/*/route.ts`
- Components: `components/ui/` (shadcn), `components/features/`, `components/shared/`
- Utils: `lib/` or `utils/`
- Types: `types/` or co-located `.types.ts`

### 4. Component Design
- **Server Component First**: Default to server components, only use `'use client'` when interaction needed
- Single responsibility, split if over 150 lines
- Destructure Props in function signature
- Conditional rendering with `&&` and ternary, avoid nested if-else
- Early return for Loading/Error states

### 5. Data Fetching
- Server-side first: Direct `await fetch()` in `async` components
- Use `async/await`, avoid `.then().catch()` chains
- Explicit cache strategy: `{ cache: 'force-cache' }` or `{ next: { revalidate: 3600 } }`
- Client-side data: SWR or React Query
- Error handling: `try/catch` or Error Boundary

## State Management

### 1. State Hierarchy
- **URL State**: searchParams, pathname (filters, pagination, tabs)
- **Local State**: `useState` (form inputs, UI toggles)
- **Server State**: SWR/React Query (server data)
- **Global State**: Zustand (lightweight global) or Context API (simple scenarios)

### 2. URL State Pattern
- Server components receive `searchParams` as props
- Client components use `useRouter` + `useSearchParams` to modify URL
- Use `URLSearchParams` API to update parameters

### 3. Zustand Pattern
- Create separate store files: `lib/store/*-store.ts`
- Use `create<T>()` + TypeScript interface
- Selector pattern to avoid unnecessary re-renders

## API Routes

### 1. Route Handler
- Export named functions: `GET`, `POST`, `PUT`, `DELETE`, etc.
- Parameters: `NextRequest`, dynamic route `{ params }`
- Return: `NextResponse.json(data, { status })`
- Error handling: try/catch + appropriate HTTP status codes
- Validation: Validate body/params before processing

### 2. Dynamic Routes
- File: `app/api/users/[id]/route.ts`
- Type: `{ params: { id: string } }`
- Access path parameters via `params` object

## Error Handling

### 1. Error Boundary
- File: `error.tsx` (client component `'use client'`)
- Props: `error: Error`, `reset: () => void`
- Scope: Same directory and child routes

### 2. Not Found
- File: `not-found.tsx`
- Trigger: `notFound()` function
- Can be server component

## Performance Optimization

### 1. Image Component
- Use `next/image`, must specify `width`, `height`
- Above-the-fold images: `priority`
- Placeholder: `placeholder="blur"`
- Remote images: Configure `remotePatterns`
- Responsive: `sizes` attribute

### 2. Dynamic Import
- Lazy load client components: `dynamic(() => import())`
- Configuration: `{ loading, ssr: false }`
- Reduce initial bundle size

### 3. Metadata
- Static: Export `metadata` object
- Dynamic: Export `generateMetadata` function
- Type: `import { Metadata } from 'next'`
- Template: `title.template`

## Environment Variables

### 1. Naming
- Public: `NEXT_PUBLIC_*` prefix (browser accessible)
- Private: No prefix (server-side only)
- Validate with Zod: `envSchema.parse(process.env)`

### 2. Access
- Centralized management: `lib/env.ts`
- Runtime validation for type safety
- Separate development and production environments

## Code Organization

### 1. Import Order
1. React and Next.js
2. Third-party libraries
3. Local components (absolute paths `@/components`)
4. Utility functions and types
5. Styles and static assets

### 2. Naming Conventions
- Components: `PascalCase`
- Functions/Variables: `camelCase`
- Constants: `UPPER_SNAKE_CASE`
- Types/Interfaces: `PascalCase`
- Hooks: `use` prefix + `camelCase`

### 3. Path Aliases
- Configure `paths` in `tsconfig.json`
- Use `@/*` instead of relative paths
- Group by module: `@/components/*`, `@/lib/*`, `@/types/*`

## Form Handling

### 1. Server Actions (Recommended)
- File: `app/actions/*.ts`
- Declaration: `'use server'`
- Return: `{ error?: string }` or `redirect()`
- Revalidation: `revalidatePath()`, `revalidateTag()`
- Form: `<form action={serverAction}>`

### 2. React Hook Form + Zod
- Schema: `z.object({})` to define validation rules
- Resolver: `zodResolver(schema)`
- Type inference: `z.infer<typeof schema>`
- Error messages: `formState.errors`

## Testing

### 1. Unit Tests
- Utility function testing: Jest + TypeScript
- Location: `__tests__/` or co-located `.test.ts`
- Cover core business logic

### 2. Component Tests
- React Testing Library
- Test user interactions and accessibility
- Avoid testing implementation details

## Quality Standards

✅ Type Safety: All functions, components, APIs have type annotations
✅ Server-First: Prioritize Server Components and Server Actions
✅ Performance: Next.js Image, dynamic imports, appropriate caching
✅ SEO: metadata, semantic HTML, structured data
✅ Error Handling: Error Boundaries, try/catch, friendly messages
✅ Accessibility: Semantic tags, ARIA, keyboard navigation
✅ Code Reuse: Shared components, utility functions, custom Hooks
✅ Test Coverage: Critical paths and components
