---
trigger: glob
globs: app/**/*.{ts,tsx}, pages/**/*.{ts,tsx}, src/**/*.{ts,tsx}
---

# Rule: Next.js + TypeScript

你是 Next.js + TypeScript 专家，遵循官方的 App Router 与数据获取最佳实践。

## 核心强制规则

### 1. 零容忍魔法数/字符串 ⭐
- 所有字面值提取为 `UPPER_SNAKE_CASE` 常量（除 0/1/-1/空字符串/布尔值）
- 位置：独立 `constants.ts` 或 `config.ts` 文件，按功能分组
- API端点、路由路径、环境变量、UI配置项统一常量化
- 使用 `as const` 断言创建只读常量对象

### 2. TypeScript 严格模式
- 必须 `strict: true` 和 `strictNullChecks: true`
- 禁用 `any`，必要时用 `unknown` + 类型守卫
- 所有函数参数、返回值必须类型标注
- Props 类型显式声明，禁用过时的 `React.FC<>`
- 优先 `interface` 定义对象，`type` 用于联合类型和工具类型

### 3. App Router 结构
- 使用 Next.js 13+ App Router（`app/` 目录）
- 路由文件：`page.tsx`、`layout.tsx`、`loading.tsx`、`error.tsx`、`not-found.tsx`
- 路由分组：`(auth)/`、`(dashboard)/` 等
- API 路由：`app/api/*/route.ts`
- 组件：`components/ui/`（shadcn）、`components/features/`、`components/shared/`
- 工具：`lib/` 或 `utils/`
- 类型：`types/` 或同目录 `.types.ts`

### 4. 组件设计
- **Server Component 优先**：默认服务端组件，仅交互需要时 `'use client'`
- 单一职责，超过 150 行拆分
- Props 解构在函数签名
- 条件渲染用 `&&` 和三元，避免 if-else 嵌套
- Loading/Error 状态早期返回

### 5. 数据获取
- 服务端优先：`async` 组件中直接 `await fetch()`
- 使用 `async/await`，禁用 `.then().catch()` 链
- 明确缓存策略：`{ cache: 'force-cache' }` 或 `{ next: { revalidate: 3600 } }`
- 客户端数据：SWR 或 React Query
- 错误处理：`try/catch` 或 Error Boundary

## 状态管理

### 1. 状态层次
- **URL State**：searchParams、pathname（筛选、分页、标签页）
- **Local State**：`useState`（表单输入、UI 开关）
- **Server State**：SWR/React Query（服务端数据）
- **Global State**：Zustand（轻量全局）或 Context API（简单场景）

### 2. URL 状态模式
- 服务端组件接收 `searchParams` 作为 props
- 客户端组件用 `useRouter` + `useSearchParams` 修改 URL
- 使用 `URLSearchParams` API 更新参数

### 3. Zustand 模式
- 创建独立 store 文件：`lib/store/*-store.ts`
- 使用 `create<T>()` + TypeScript 接口
- 选择器模式避免不必要的重渲染

## API 路由

### 1. Route Handler
- 导出 `GET`、`POST`、`PUT`、`DELETE` 等命名函数
- 参数：`NextRequest`、动态路由 `{ params }`
- 返回：`NextResponse.json(data, { status })`
- 错误处理：try/catch + 适当 HTTP 状态码
- 验证：在处理前验证 body/params

### 2. 动态路由
- 文件：`app/api/users/[id]/route.ts`
- 类型：`{ params: { id: string } }`
- 路径参数通过 `params` 对象访问

## 错误处理

### 1. Error Boundary
- 文件：`error.tsx`（客户端组件 `'use client'`）
- Props：`error: Error`、`reset: () => void`
- 作用域：同目录及子路由

### 2. Not Found
- 文件：`not-found.tsx`
- 触发：`notFound()` 函数
- 服务端组件即可

## 性能优化

### 1. Image 组件
- 使用 `next/image`，必须指定 `width`、`height`
- 首屏图片：`priority`
- 占位符：`placeholder="blur"`
- 远程图片：配置 `remotePatterns`
- 响应式：`sizes` 属性

### 2. 动态导入
- 客户端组件延迟加载：`dynamic(() => import())`
- 配置：`{ loading, ssr: false }`
- 减少初始包体积

### 3. Metadata
- 静态：导出 `metadata` 对象
- 动态：导出 `generateMetadata` 函数
- 类型：`import { Metadata } from 'next'`
- 模板：`title.template`

## 环境变量

### 1. 命名
- 公开：`NEXT_PUBLIC_*` 前缀（浏览器可访问）
- 私有：无前缀（仅服务端）
- 使用 Zod 验证：`envSchema.parse(process.env)`

### 2. 访问
- 集中管理：`lib/env.ts`
- 运行时验证确保类型安全
- 开发、生产环境分离

## 代码组织

### 1. 导入顺序
1. React 和 Next.js
2. 第三方库
3. 本地组件（绝对路径 `@/components`）
4. 工具函数和类型
5. 样式和静态资源

### 2. 命名约定
- 组件：`PascalCase`
- 函数/变量：`camelCase`
- 常量：`UPPER_SNAKE_CASE`
- 类型/接口：`PascalCase`
- Hooks：`use` 前缀 + `camelCase`

### 3. 路径别名
- 配置 `tsconfig.json` 中 `paths`
- 使用 `@/*` 代替相对路径
- 按模块分组：`@/components/*`、`@/lib/*`、`@/types/*`

## 表单处理

### 1. Server Actions（推荐）
- 文件：`app/actions/*.ts`
- 声明：`'use server'`
- 返回：`{ error?: string }` 或 `redirect()`
- 重新验证：`revalidatePath()`, `revalidateTag()`
- 表单：`<form action={serverAction}>`

### 2. React Hook Form + Zod
- Schema：`z.object({})` 定义验证规则
- Resolver：`zodResolver(schema)`
- 类型推导：`z.infer<typeof schema>`
- 错误信息：`formState.errors`

## 测试

### 1. 单元测试
- 工具函数测试：Jest + TypeScript
- 位置：`__tests__/` 或同目录 `.test.ts`
- 覆盖核心业务逻辑

### 2. 组件测试
- React Testing Library
- 测试用户交互和可访问性
- 避免实现细节测试

## 质量标准

✅ 类型安全：所有函数、组件、API 有类型标注
✅ 服务端优先：优先 Server Component 和 Server Actions
✅ 性能：Next.js Image、动态导入、适当缓存
✅ SEO：metadata、语义化 HTML、结构化数据
✅ 错误处理：Error Boundary、try/catch、友好提示
✅ 可访问性：语义化、ARIA、键盘导航
✅ 代码复用：共享组件、工具函数、自定义 Hooks
✅ 测试覆盖：关键路径和组件
