# Global Coding Rules（Full Stack）

适用范围：Next.js / React / HTML / CSS / TypeScript / Python / Shell / Vite / Tailwind CSS / shadcn/ui / Supabase / Upstash Redis / Docker / Kubernetes 等。

---

## 1. 通用原则

1. **可读性优先于聪明技巧**  
   - 代码首先写给人看，其次才是给机器执行。
   - 避免“炫技式”写法，优先清晰、直观的实现。

2. **避免 Magic Number / Magic String**  
   - 所有固定值（阈值、路径、key、超时、环境名等）提取为常量或配置：
     - 如：`MAX_LOGIN_ATTEMPTS`、`SESSION_TTL_SECONDS`、`REDIS_CACHE_PREFIX`。
   - 除 `0, 1, -1, ""` 等自解释值外，禁止直接硬编码在逻辑中。

3. **单一职责 & 模块化**  
   - 每个函数、文件、模块只做一件清晰的事。
   - 避免“上帝对象”或超大文件（>300 行考虑拆分）。

4. **显式优于隐式**  
   - 明确类型、依赖、返回值和错误处理。
   - 避免“约定在脑子里”，用代码和注释表达意图。

5. **缺省安全（Secure by default）**  
   - 默认认为输入不可信，默认关闭权限，再逐步开放。
   - 防注入、防 XSS、防 CSRF、防越权。

6. **可测试性**  
   - 函数输出只依赖显式输入（参数/依赖注入），避免隐式全局状态。
   - 可单测的设计优先于勉强能跑的设计。

---

## 2. 代码风格与结构

1. **统一格式化工具**  
   - 使用 `prettier` + `eslint`（TS/JS/React）统一风格。
   - Python 使用 `black` / `ruff`。
   - Shell 使用 `shfmt`（如有）。

2. **命名规范**  
   - 变量/函数：`camelCase`
   - 常量：`UPPER_SNAKE_CASE`
   - 类型/接口/类：`PascalCase`
   - 文件名：
     - React 组件：`ComponentName.tsx`
     - 工具函数：`some-util.ts`
   - 命名表达业务含义，不用缩写乱写：`userAccessToken` 优于 `uat`.

3. **函数与类设计**  
   - 函数参数 ≤ 3 个，超过用对象参数。
   - 避免长函数（>40 行考虑拆分）。
   - 业务逻辑与 IO/框架层（HTTP/DB/UI）解耦。

4. **注释与文档**  
   - 用注释解释“为什么这样做”，而不是“这行代码做了什么”。
   - 对复杂逻辑/算法/边界条件写简要注释。
   - 公共模块/工具方法适当使用 JSDoc / docstring。

---

## 3. TypeScript / JavaScript / Next.js / Vite

1. **首选 TypeScript**  
   - 禁用 `any`（除非极端场景且必须加注释说明）。
   - 优先使用 `interface` / `type` 定义数据结构。
   - 避免滥用 `as` 类型断言。

2. **异步与错误处理**  
   - 使用 `async/await`，避免回调地狱和过多 `.then()`.
   - API 调用必须显式捕获错误（`try/catch` 或错误边界）。
   - 对用户可见错误要有清晰提示，不暴露内部细节。

3. **Next.js 特定规则**  
   - 清晰区分 **Server Components** 和 **Client Components**。
   - 数据获取逻辑优先放在 server（`getServerSideProps`/`server actions`等）。
   - 避免在客户端暴露敏感信息（API key、密钥、内部 URL 等）。
   - Route handler 与页面逻辑分离，保持 handler 只做请求处理与校验。

4. **状态管理**  
   - 优先使用 React 内置（`useState`, `useReducer`, `useContext`）和 server state。
   - 全局状态管理（如 Zustand）只在必要时使用。
   - 避免重复状态：从单一源获取数据（API / props），不要多处 copy。

---

## 4. React / shadcn/ui / Tailwind CSS

1. **组件设计**  
   - 组件保持“傻瓜化”：输入 props，输出 UI。
   - 将业务逻辑抽到 hooks（`useXxx`）中，组件负责展示。
   - 可复用的 UI 放入 `components/`，业务页面特定放在 `app/` 或 `pages/`。

2. **Tailwind CSS 规则**  
   - 基础样式使用 Tailwind 原子类。
   - 复杂/重复的组合抽成组件或 `className` 工具函数。
   - 控制类名长度：超长/重复时考虑提取。

3. **shadcn/ui**  
   - 优先使用已有组件并通过 `className` 定制，而不是重造轮子。
   - 保持可访问性：不要删除 `aria-*` / `role` / keyboard 支持。

---

## 5. HTML / CSS

1. 语义化标签：`<button>` ≠ `<div onClick>`.
2. 所有交互元素必须可键盘操作。
3. 使用 BEM 或 Tailwind，不混乱堆 CSS class。
4. 避免行内样式，除非极简单、一次性情况。

---

## 6. Python

1. 遵循 PEP8，使用 `black` 格式化。
2. 函数短小，单一职责。
3. 使用 `logging` 而不是 `print` 记录日志。
4. 虚拟环境隔离依赖（`venv`/`poetry`/`pipenv`）。
5. 与 JS/TS 层交互的数据结构要有清晰契约（schema/dto）。

---

## 7. Shell 脚本

1. 使用 `set -euo pipefail` 提高可靠性。
2. 变量一律加双引号：`"$VAR"`。
3. 不在脚本中硬编码敏感信息，从环境变量或密钥管理中获取。
4. 对危险操作（删除/覆盖）加确认或清晰注释。

---

## 8. 数据与存储（Supabase / Upstash / Redis）

1. **Supabase**  
   - 所有 DB 操作必须定义清晰的 schema（使用 Supabase types 或 Zod 校验）。
   - 权限控制优先使用 RLS（Row Level Security）。
   - 只暴露必要的 public API；服务端操作放在 server 端。

2. **Redis / Upstash**  
   - key 命名规范：`app:env:domain:entity:id`。
   - 所有缓存项必须设置合理过期时间（TTL），禁止无限期缓存除非确有理由。
   - 区分缓存与持久数据：Redis 只存可丢失数据。

---

## 9. 安全规则

1. 所有外部输入（HTTP 请求、表单、query、headers、webhook）必须做校验。
2. 不在前端暴露任何机密信息（密钥、内部 IP、凭证）。
3. 认证/授权逻辑统一封装（middleware / hooks），不要分散在各处。
4. 禁止在日志中记录密码、token、敏感个人数据。
5. HTTPS/安全 Cookie/CSRF 防护为默认要求。

---

## 10. 性能与可扩展性

1. 避免不必要的重复请求和重复计算，合理使用缓存。
2. 大列表使用分页 / 虚拟滚动。
3. Next.js：
   - 使用增量静态生成（ISR）或缓存减少 SSR 压力。
   - 避免在客户端做大计算和大数据处理。
4. 数据库查询必须避免 N+1，使用关联查询或批量操作。

---

## 11. 测试与质量保证

1. 新增/修改核心逻辑，必须有相应测试（单元 / 集成 / e2e 中至少一种）。
2. 对关键路径（认证、支付、订单、数据写入）建立自动化测试。
3. 禁止提交带有明显失败的测试或被注释掉的测试。
4. CI 中强制代码格式检查和基础测试通过。

---

## 12. Docker & Kubernetes

1. **Docker**  
   - 使用精简基础镜像（如 `alpine`、`slim`）。
   - 分阶段构建（multi-stage build），减小最终镜像体积。
   - 容器内以非 root 用户运行。
   - 配置通过环境变量/配置文件注入，而不是写死在镜像中。

2. **Kubernetes**  
   - 所有 Pod 必须设置资源请求/限制（`resources.requests/limits`）。
   - 为服务配置 liveness/readiness probe。
   - 使用 ConfigMap/Secret 管理配置与密钥，不直接写入镜像或源码。
   - 将 infra 配置（Deployment/Service/Ingress）作为 IaC 管理（如 GitOps）。

---

## 13. Git 与协作

1. 一事一分支，分支名语义化：`feature/xxx`、`fix/xxx`。
2. Commit message 清晰表达“做了什么”和“为什么”。
3. 禁止在 main/master 上直接推送（除自动化流程）。
4. PR 尽量小而频繁，保证易于 review。

---

> **实践原则：**  
> - 不确定时，倾向于：类型更严格、校验更多、权限更小、耦合更低。  
> - 若违反上述规则，必须有清晰理由并在代码/PR 中说明。
