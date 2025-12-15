---
trigger: glob
globs: **/*.py
---

# Python

精简版供 LLM 上下文使用

## 核心强制规则

### 1. 零容忍魔法数/字符串 ⭐
- **所有** 字面值必须提取为模块级具名常量（除 0/1/-1/空字符串）
- 常量命名：`UPPER_SNAKE_CASE`
- 分组组织：HTTP配置、API端点、环境变量、显示配置等
- 位置：文件顶部 `# CONSTANTS` 区域，类外

### 2. Pythonic 强制要求
- 使用 `enumerate(items, 1)` 替代手动索引
- 使用 `pathlib.Path` 替代字符串路径拼接
- 使用 `with` 上下文管理器处理所有资源
- 使用 `dict.get(key, default)` 替代 `try/except KeyError`
- 优先列表/字典/集合推导式，大数据用生成器表达式
- 字符串格式化统一 f-string，禁用 `%` 和 `.format()`

### 3. 类型系统
- **强制**：所有公共函数/方法必须类型标注
- 导入：`from typing import Dict, List, Optional, Tuple, Any`
- 返回值：明确类型或 `Optional[T]`，禁用裸 `None`
- 参数：位置参数 + 类型，关键字参数带默认值

### 4. 文档字符串
- **必需**：模块级、类级、公共函数/方法
- 格式：Google docstring style
  - 单行简述 + 空行 + 详述
  - `Args:` - 参数名: 类型 + 说明
  - `Returns:` - 返回值类型 + 说明
  - `Raises:` (如适用)
  - 私有方法可选但推荐

### 5. 函数设计原则
- **单一职责**：每个函数只做一件事，函数名清晰表达其用途（如 `validate_input`, `parse_config`, `send_request`）
- **函数复用**：将通用逻辑（参数校验、日志输出、错误处理等）抽取为函数，避免重复代码

## 代码组织

### 文件结构模板
```
#!/usr/bin/env python3
"""模块docstring"""
# 导入：标准库 → 第三方 → 本地
# CONSTANTS 区
# CLASSES 区
# FUNCTIONS 区
# main() + if __name__ == '__main__'
```

### 类设计
- 类常量作为类属性（字典配置等）
- `__init__` 接受配置参数，初始化实例属性
- 公共方法：无前缀，完整docstring
- 私有方法：单下划线前缀 `_method`
- 实例状态存储在 `self.stats` 字典（统计）

## 错误处理

### 异常层次
1. 先捕获具体异常（`FileNotFoundError`, `ValueError`）
2. 再捕获业务异常
3. 最后 `Exception` 兜底
4. 每层 log + 返回/抛出

### 优雅降级
- 缓存miss/网络错误返回 `None`，不抛异常
- 调用方检查 `is not None` 继续流程
- 关键路径才抛异常中断

## 输出与日志

### 日志文件配置
- **日志记录**：重要操作需输出到日志文件，便于事后排查；使用 `logging` 模块配置文件和控制台双输出
- **日志文件命名规范**：`<script_name>_<timestamp>.log`（如 `data_processor_2025-12-14_15-30-45.log`）
- **时间戳格式**：`datetime.now().strftime('%Y-%m-%d_%H-%M-%S')`
- **命令行参数**：支持通过参数（如 `-l/--log-file`）自定义日志文件路径
- **双输出配置**：使用 `logging.FileHandler` + `logging.StreamHandler` 同时输出到文件和控制台

### 彩色输出
- 定义 `Colors` 类（ANSI转义码常量）
- 成功：GREEN + ✓，失败：RED + ✗，警告：YELLOW
- 所有用户输出用彩色标记

### 进度显示
- 循环中：`print(f"[{i}/{total}] ...", end='\r')` 覆盖行
- 完成后：`print(f"\n{Colors.GREEN}✓...{Colors.RESET}")`换行

### 统计摘要
- `print_summary()` 方法：标题行（粗体+颜色）+ 分隔符 + 多行统计
- 数字用 `{Colors.BOLD}{value}{Colors.RESET}` 高亮

## 命令行工具

### argparse 配置
- `formatter_class=argparse.RawDescriptionHelpFormatter`
- `epilog` 包含 Examples + Notes
- 布尔开关：`--flag` (action='store_true') + `--no-flag` (dest='flag', action='store_false')
- 参数默认值在 `default=` 和 help 文本中一致

### 参数验证
- `validate_args(args)` 函数独立验证
- 文件存在性、数值范围、互斥参数
- 验证失败：彩色错误信息 + `sys.exit(1)`
- 覆盖警告：YELLOW 提示但继续

## 环境变量

- 使用 `os.getenv(ENV_NAME_CONST)` 而非硬编码字符串
- 必需变量验证：`if not var: print(error) + sys.exit(1)`
- 可选变量：`os.getenv(NAME, 'default')`
- 布尔环境变量：`.lower() == 'true'`

## 命名约定

- 变量/函数：`snake_case`
- 常量：`UPPER_SNAKE_CASE`
- 类：`PascalCase`
- 私有：`_leading_underscore`
- 模块：`lowercase_no_underscores` 或 `lower_with_under`

## HTTP/API

- 超时：常量化 `HTTP_TIMEOUT_SECONDS`
- 方法：常量 `HTTP_METHOD_GET/POST/DELETE`
- Headers：字典用常量键 `HEADER_AUTHORIZATION`
- 路径模板：`API_PATH_XXX = '/path/{}'`, 用 `.format()` 填充
- 编码：`req.data = json.dumps(data).encode(JSON_ENCODING)`
- 响应：`json.loads(response.read().decode(JSON_ENCODING))`

## 测试友好

- 依赖注入：logger/client等通过 `__init__` 参数传入，默认值惰性初始化
- 可配置：所有常量可通过参数覆盖（timeout, interval等）
- 纯函数：输入 → 输出，无隐藏全局状态

## 质量标准

✅ 导入：标准库 → 第三方 → 本地，每组用空行分隔
✅ 布尔判断明确：`is not None`, `len(x) > 0`，避免隐式truthiness
✅ 资源：统一 `with` 上下文管理器，禁手动 `.close()`
✅ 日志：使用 `logging` 模块，不用 `print()` 记录状态
✅ 退出码：成功 `sys.exit(0)`，失败 `sys.exit(1)`
