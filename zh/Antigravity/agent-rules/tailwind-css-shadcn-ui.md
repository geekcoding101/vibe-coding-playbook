---
trigger: glob
globs: app/**/*.{tsx,jsx}, components/**/*.{tsx,jsx}
---

# Rule: Tailwind CSS + shadcn/ui

你负责在使用 Tailwind + shadcn/ui 的 React 代码中，保证样式可读、可复用、可维护。

## 核心强制规则

### 1. Tailwind 使用原则 ⭐
- 原子类优先，禁止内联 `style={{}}`
- 禁止硬编码颜色（`#3b82f6`），使用 CSS 变量（`bg-primary`）或 Tailwind 预定义色
- 响应式移动端优先：`sm:` `md:` `lg:` `xl:` `2xl:`
- 间距统一：Tailwind 系统（`p-4` `gap-6`），禁止任意值 `p-[13px]`（除极特殊情况）
- 颜色系统：语义化命名（`primary` `secondary` `destructive` `muted` `accent`）

### 2. Class 组织
- 使用 `cn()` 工具函数合并类名
- 顺序：布局 → 尺寸 → 间距 → 颜色 → 文字 → 边框 → 效果 → 动画
- 单元素超过 6 个类名考虑换行或提取变体
- 条件类名：`cn('base', condition && 'conditional', className)`

### 3. 颜色系统
- CSS 变量定义主题色：`app/globals.css` 中 `:root` 和 `.dark`
- HSL 格式：`--primary: 221.2 83.2% 53.3%`
- 使用 `bg-primary` `text-primary-foreground` 等语义类
- 深色模式：`dark:` 前缀自动适配

### 4. shadcn/ui 组件
- 按需安装：`npx shadcn-ui@latest add button card dialog`
- 组件位置：`components/ui/`
- 直接编辑源码（非 node_modules）
- 变体系统：`class-variance-authority` (cva)
- 组合优于继承

## 响应式设计

### 1. 断点系统
- 默认断点：`sm: 640px` `md: 768px` `lg: 1024px` `xl: 1280px` `2xl: 1536px`
- 移动端优先：基础类 → `md:` → `lg:` 等
- 所有布局必须适配移动端

### 2. 布局模式
- Container：`container mx-auto px-4 md:px-6 lg:px-8`
- Grid：`grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-4`
- Flex：`flex flex-col sm:flex-row items-start sm:items-center gap-4`

## 组件模式

### 1. shadcn 组件结构
- 基础组件：`Card`、`CardHeader`、`CardTitle`、`CardContent`、`CardFooter`
- 表单组件：`Form`、`FormField`、`FormItem`、`FormLabel`、`FormControl`、`FormMessage`
- 对话框：`Dialog`、`DialogContent`、`DialogHeader`、`DialogTitle`、`DialogFooter`
- 使用 `React.forwardRef` 传递 ref
- 使用 `cn()` 合并 className

### 2. 变体定义
- 使用 `cva()` 定义变体
- 配置：base classes + variants + defaultVariants
- 类型：`VariantProps<typeof variants>`
- 导出：变体函数和组件

## 动画和过渡

### 1. Tailwind 动画
- 内置：`animate-spin` `animate-pulse` `animate-bounce`
- 过渡：`transition-all duration-300 ease-in-out`
- Hover 效果：`hover:scale-105 hover:shadow-lg`
- Active 状态：`active:scale-95`

### 2. 自定义动画
- 配置：`tailwind.config.ts` 中 `keyframes` 和 `animation`
- 使用：`animate-[name]`
- 常用：accordion、slide-in、fade-in 等

## 深色模式

### 1. 实现
- Provider：`next-themes` 的 `ThemeProvider`
- 属性：`attribute="class"`
- 策略：`defaultTheme="system"` `enableSystem`
- HTML：`<html suppressHydrationWarning>`

### 2. 样式
- 使用 `dark:` 前缀
- 所有颜色相关类必须提供深色变体
- 配合 CSS 变量自动适配

### 3. 切换器
- Hook：`useTheme()` from `next-themes`
- 图标：Sun/Moon (lucide-react)
- 过渡动画：`transition-all` + `rotate` + `scale`

## 常用布局

### 1. 侧边栏布局
- Container：`flex min-h-screen`
- 侧边栏：`hidden md:flex w-64 flex-col border-r`
- 主内容：`flex flex-1 flex-col`
- Header：`flex h-16 items-center justify-between border-b`

### 2. 居中容器
- 水平垂直居中：`flex min-h-screen items-center justify-center`
- 限制宽度：`w-full max-w-md`
- 背景渐变：`bg-gradient-to-br from-gray-50 to-gray-100`

### 3. Sticky Header
- 固定：`sticky top-0 z-50`
- 背景模糊：`bg-background/95 backdrop-blur`
- 支持检测：`supports-[backdrop-filter]:bg-background/60`

## 图标系统

### 1. Lucide React
- 导入：`import { Home, Settings } from 'lucide-react'`
- 尺寸：`h-4 w-4` (16px)、`h-5 w-5` (20px)、`h-6 w-6` (24px)
- 颜色：`text-muted-foreground`、`text-destructive`
- Loading：`animate-spin` on `Loader2`

### 2. 尺寸约定
- `h-3 w-3`：角标
- `h-4 w-4`：按钮内、表单
- `h-5 w-5`：导航、列表
- `h-6 w-6`：标题旁
- `h-8 w-8`：大图标按钮

## 实用工具类

### 1. 文本截断
- 单行：`truncate`
- 多行：`line-clamp-2` `line-clamp-3`
- 需要 `@tailwindcss/line-clamp` 插件或 Tailwind v3.3+

### 2. 滚动优化
- 平滑：`scroll-smooth`
- 隐藏滚动条：`scrollbar-hide` 或 `[&::-webkit-scrollbar]:hidden`
- 自定义滚动条：`scrollbar-thin` + `scrollbar-thumb-*` + `scrollbar-track-*`

### 3. 焦点样式
- 移除默认：`focus-visible:outline-none`
- 自定义 Ring：`focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2`
- 应用到所有交互元素

## 性能优化

### 1. 类名优化
- 使用 `cn()` 避免字符串拼接
- 提取重复类为变体
- 条件类名用 Boolean 短路

### 2. PurgeCSS
- 配置 `content` 覆盖所有文件
- 扫描路径：`./app/**/*.{js,ts,jsx,tsx}`, `./components/**/*.{js,ts,jsx,tsx}`
- 生产构建自动移除未使用样式

### 3. 变体复用
- 使用 `cva()` 定义共享变体
- 避免每个组件重复相同类名组合

## 可访问性

### 1. 语义化
- 使用正确的 HTML 标签（`button`、`nav`、`main`）
- ARIA 属性：`aria-label`、`aria-pressed`、`aria-expanded`
- Screen reader：`sr-only` 类隐藏视觉但保留语义

### 2. 键盘导航
- 确保 `tabIndex` 合理
- 交互元素可通过键盘访问
- Focus 样式清晰可见

### 3. 颜色对比度
- 文本和背景对比度至少 4.5:1（WCAG AA）
- 使用 Tailwind 预定义颜色确保对比度
- 避免低对比度组合（`text-gray-400` on `bg-gray-200`）

## 质量标准

✅ 原子类优先，禁用自定义 CSS
✅ 颜色系统：CSS 变量 + 语义化命名
✅ 响应式：移动端优先，全屏幕适配
✅ 深色模式：所有颜色提供 `dark:` 变体
✅ 组件复用：shadcn/ui + 变体扩展
✅ 可访问性：语义化、ARIA、键盘导航
✅ 性能：类名优化、PurgeCSS
✅ 动画流畅：适当过渡提升体验
