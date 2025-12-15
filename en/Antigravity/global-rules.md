# Global Coding Rules (Full Stack)

Applicable to: Next.js / React / HTML / CSS / TypeScript / Python / Shell / Vite / Tailwind CSS / shadcn/ui / Supabase / Upstash Redis / Docker / Kubernetes, etc.

---

## 1. General Principles

1. **Readability Over Clever Tricks**  
   - Code is written for humans first, machines second.
   - Avoid "showing off" with code; prioritize clear and intuitive implementations.

2. **Avoid Magic Numbers / Magic Strings**  
   - Extract all fixed values (thresholds, paths, keys, timeouts, environment names, etc.) as constants or configuration:
     - Examples: `MAX_LOGIN_ATTEMPTS`, `SESSION_TTL_SECONDS`, `REDIS_CACHE_PREFIX`.
   - Except for self-explanatory values like `0, 1, -1, ""`, never hardcode values directly in logic.

3. **Single Responsibility & Modularity**  
   - Each function, file, and module should do one clear thing.
   - Avoid "god objects" or oversized files (consider splitting if >300 lines).

4. **Explicit Over Implicit**  
   - Be explicit about types, dependencies, return values, and error handling.
   - Avoid "conventions in your head"; express intent through code and comments.

5. **Secure by Default**  
   - Assume all input is untrusted by default; default to closed permissions, then gradually open.
   - Prevent injection, XSS, CSRF, and privilege escalation.

6. **Testability**  
   - Function outputs should only depend on explicit inputs (parameters/dependency injection); avoid implicit global state.
   - Prioritize testable design over "barely working" design.

---

## 2. Code Style & Structure

1. **Unified Formatting Tools**  
   - Use `prettier` + `eslint` for unified style (TS/JS/React).
   - Use `black` / `ruff` for Python.
   - Use `shfmt` for Shell (if available).

2. **Naming Conventions**  
   - Variables/Functions: `camelCase`
   - Constants: `UPPER_SNAKE_CASE`
   - Types/Interfaces/Classes: `PascalCase`
   - File names:
     - React components: `ComponentName.tsx`
     - Utility functions: `some-util.ts`
   - Names should express business meaning; avoid arbitrary abbreviations: `userAccessToken` is better than `uat`.

3. **Function & Class Design**  
   - Functions should have ≤ 3 parameters; use object parameters if more.
   - Avoid long functions (consider splitting if >40 lines).
   - Decouple business logic from IO/framework layers (HTTP/DB/UI).

4. **Comments & Documentation**  
   - Use comments to explain "why this is done" rather than "what this line does".
   - Write brief comments for complex logic/algorithms/edge cases.
   - Use JSDoc / docstrings appropriately for public modules/utility methods.

---

## 3. TypeScript / JavaScript / Next.js / Vite

1. **Prefer TypeScript**  
   - Avoid `any` (unless extreme scenarios and must be explained with comments).
   - Prefer using `interface` / `type` to define data structures.
   - Avoid overusing `as` type assertions.

2. **Async & Error Handling**  
   - Use `async/await`; avoid callback hell and excessive `.then()`.
   - API calls must explicitly catch errors (`try/catch` or error boundaries).
   - User-facing errors should have clear messages without exposing internal details.

3. **Next.js Specific Rules**  
   - Clearly distinguish between **Server Components** and **Client Components**.
   - Prioritize data fetching logic on the server (`getServerSideProps`/`server actions`, etc.).
   - Avoid exposing sensitive information on the client (API keys, secrets, internal URLs, etc.).
   - Separate route handlers from page logic; keep handlers focused on request processing and validation.

4. **State Management**  
   - Prefer React built-ins (`useState`, `useReducer`, `useContext`) and server state.
   - Use global state management (like Zustand) only when necessary.
   - Avoid duplicate state: get data from a single source (API / props), don't copy in multiple places.

---

## 4. React / shadcn/ui / Tailwind CSS

1. **Component Design**  
   - Keep components "dumb": input props, output UI.
   - Extract business logic into hooks (`useXxx`); components are responsible for presentation.
   - Place reusable UI in `components/`; business-specific pages in `app/` or `pages/`.

2. **Tailwind CSS Rules**  
   - Use Tailwind atomic classes for basic styles.
   - Extract complex/repeated combinations into components or `className` utility functions.
   - Control class name length: consider extracting when overly long/repetitive.

3. **shadcn/ui**  
   - Prefer using existing components and customize via `className` rather than reinventing the wheel.
   - Maintain accessibility: don't remove `aria-*` / `role` / keyboard support.

---

## 5. HTML / CSS

1. Use semantic tags: `<button>` ≠ `<div onClick>`.
2. All interactive elements must be keyboard operable.
3. Use BEM or Tailwind; don't pile CSS classes chaotically.
4. Avoid inline styles unless extremely simple or one-off situations.

---

## 6. Python

1. Follow PEP8; use `black` for formatting.
2. Keep functions small with single responsibility.
3. Use `logging` instead of `print` for logging.
4. Isolate dependencies with virtual environments (`venv`/`poetry`/`pipenv`).
5. Data structures interacting with JS/TS layers should have clear contracts (schema/dto).

---

## 7. Shell Scripts

1. Use `set -euo pipefail` to improve reliability.
2. Always quote variables: `"$VAR"`.
3. Don't hardcode sensitive information in scripts; retrieve from environment variables or secret management.
4. Add confirmation or clear comments for dangerous operations (delete/overwrite).

---

## 8. Data & Storage (Supabase / Upstash / Redis)

1. **Supabase**  
   - All DB operations must define clear schemas (use Supabase types or Zod validation).
   - Prioritize RLS (Row Level Security) for permission control.
   - Only expose necessary public APIs; server-side operations belong on the server.

2. **Redis / Upstash**  
   - Key naming convention: `app:env:domain:entity:id`.
   - All cache items must have reasonable expiration times (TTL); avoid indefinite caching unless justified.
   - Distinguish cache from persistent data: Redis only stores lossy data.

---

## 9. Security Rules

1. All external input (HTTP requests, forms, queries, headers, webhooks) must be validated.
2. Never expose any confidential information on the frontend (keys, internal IPs, credentials).
3. Unify authentication/authorization logic (middleware / hooks); don't scatter across codebase.
4. Never log passwords, tokens, or sensitive personal data.
5. HTTPS/secure cookies/CSRF protection are default requirements.

---

## 10. Performance & Scalability

1. Avoid unnecessary repeated requests and computations; use caching appropriately.
2. Use pagination / virtual scrolling for large lists.
3. Next.js:
   - Use Incremental Static Regeneration (ISR) or caching to reduce SSR pressure.
   - Avoid heavy computation and large data processing on the client.
4. Database queries must avoid N+1; use join queries or batch operations.

---

## 11. Testing & Quality Assurance

1. New/modified core logic must have corresponding tests (at least one of: unit / integration / e2e).
2. Establish automated tests for critical paths (authentication, payment, orders, data writes).
3. Never commit obviously failing tests or commented-out tests.
4. Enforce code format checks and basic test passes in CI.

---

## 12. Docker & Kubernetes

1. **Docker**  
   - Use minimal base images (like `alpine`, `slim`).
   - Use multi-stage builds to reduce final image size.
   - Run containers as non-root users.
   - Inject configuration via environment variables/config files, not hardcoded in images.

2. **Kubernetes**  
   - All Pods must set resource requests/limits (`resources.requests/limits`).
   - Configure liveness/readiness probes for services.
   - Use ConfigMap/Secret to manage configuration and secrets; don't write directly into images or source code.
   - Manage infrastructure configuration (Deployment/Service/Ingress) as IaC (e.g., GitOps).

---

## 13. Git & Collaboration

1. One branch per task; use semantic branch names: `feature/xxx`, `fix/xxx`.
2. Commit messages should clearly express "what was done" and "why".
3. Never push directly to main/master (except automated processes).
4. Keep PRs small and frequent to ensure easy review.

---

> **Practical Principles:**  
> - When in doubt, lean towards: stricter types, more validation, smaller permissions, lower coupling.  
> - If violating the above rules, must have a clear reason and explain in code/PR.
