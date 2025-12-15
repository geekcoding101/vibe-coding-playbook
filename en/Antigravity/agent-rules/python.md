---
trigger: glob
globs: **/*.py
---

# Python

Concise version for LLM context

## Core Mandatory Rules

### 1. Zero Tolerance for Magic Numbers/Strings ⭐
- **All** literal values must be extracted as module-level named constants (except 0/1/-1/empty strings)
- Constant naming: `UPPER_SNAKE_CASE`
- Grouped organization: HTTP config, API endpoints, environment variables, display config, etc.
- Location: Top of file in `# CONSTANTS` section, outside classes

### 2. Pythonic Requirements
- Use `enumerate(items, 1)` instead of manual indexing
- Use `pathlib.Path` instead of string path concatenation
- Use `with` context managers for all resources
- Use `dict.get(key, default)` instead of `try/except KeyError`
- Prefer list/dict/set comprehensions, use generator expressions for large data
- Unified string formatting with f-strings, avoid `%` and `.format()`

### 3. Type System
- **Mandatory**: All public functions/methods must have type annotations
- Import: `from typing import Dict, List, Optional, Tuple, Any`
- Return values: Explicit type or `Optional[T]`, avoid bare `None`
- Parameters: Positional parameters + type, keyword parameters with defaults

### 4. Docstrings
- **Required**: Module-level, class-level, public functions/methods
- Format: Google docstring style
  - Single-line summary + blank line + details
  - `Args:` - parameter name: type + description
  - `Returns:` - return value type + description
  - `Raises:` (if applicable)
  - Private methods optional but recommended

### 5. Function Design Principles
- **Single Responsibility**: Each function does one thing, function name clearly expresses its purpose (e.g., `validate_input`, `parse_config`, `send_request`)
- **Function Reuse**: Extract common logic (parameter validation, logging, error handling, etc.) into functions to avoid code duplication

## Code Organization

### File Structure Template
```
#!/usr/bin/env python3
"""Module docstring"""
# Imports: standard library → third-party → local
# CONSTANTS section
# CLASSES section
# FUNCTIONS section
# main() + if __name__ == '__main__'
```

### Class Design
- Class constants as class attributes (config dictionaries, etc.)
- `__init__` accepts config parameters, initializes instance attributes
- Public methods: No prefix, complete docstring
- Private methods: Single underscore prefix `_method`
- Instance state stored in `self.stats` dictionary (statistics)

## Error Handling

### Exception Hierarchy
1. Catch specific exceptions first (`FileNotFoundError`, `ValueError`)
2. Then catch business exceptions
3. Finally `Exception` as fallback
4. Each layer: log + return/raise

### Graceful Degradation
- Cache miss/network errors return `None`, don't raise exceptions
- Caller checks `is not None` to continue flow
- Only raise exceptions for critical paths

## Output & Logging

### Log File Configuration
- **Logging**: Important operations must output to log file for troubleshooting; use `logging` module to configure dual output to file and console
- **Log file naming convention**: `<script_name>_<timestamp>.log` (e.g., `data_processor_2025-12-14_15-30-45.log`)
- **Timestamp format**: `datetime.now().strftime('%Y-%m-%d_%H-%M-%S')`
- **Command-line argument**: Support custom log file path via argument (e.g., `-l/--log-file`)
- **Dual output configuration**: Use `logging.FileHandler` + `logging.StreamHandler` to output to both file and console

### Colorful Output
- Define `Colors` class (ANSI escape code constants)
- Success: GREEN + ✓, Failure: RED + ✗, Warning: YELLOW
- All user output uses color markers

### Progress Display
- In loops: `print(f"[{i}/{total}] ...", end='\r')` to overwrite line
- On completion: `print(f"\n{Colors.GREEN}✓...{Colors.RESET}")` for newline

### Statistics Summary
- `print_summary()` method: Title line (bold + color) + separator + multi-line statistics
- Numbers highlighted with `{Colors.BOLD}{value}{Colors.RESET}`

## Command-Line Tools

### argparse Configuration
- `formatter_class=argparse.RawDescriptionHelpFormatter`
- `epilog` includes Examples + Notes
- Boolean switches: `--flag` (action='store_true') + `--no-flag` (dest='flag', action='store_false')
- Parameter defaults consistent in both `default=` and help text

### Parameter Validation
- Independent validation with `validate_args(args)` function
- File existence, numeric ranges, mutually exclusive parameters
- Validation failure: Colored error message + `sys.exit(1)`
- Override warning: YELLOW prompt but continue

## Environment Variables

- Use `os.getenv(ENV_NAME_CONST)` instead of hardcoded strings
- Required variable validation: `if not var: print(error) + sys.exit(1)`
- Optional variables: `os.getenv(NAME, 'default')`
- Boolean environment variables: `.lower() == 'true'`

## Naming Conventions

- Variables/Functions: `snake_case`
- Constants: `UPPER_SNAKE_CASE`
- Classes: `PascalCase`
- Private: `_leading_underscore`
- Modules: `lowercase_no_underscores` or `lower_with_under`

## HTTP/API

- Timeout: Constant `HTTP_TIMEOUT_SECONDS`
- Methods: Constants `HTTP_METHOD_GET/POST/DELETE`
- Headers: Dictionary with constant keys `HEADER_AUTHORIZATION`
- Path templates: `API_PATH_XXX = '/path/{}'`, use `.format()` to fill
- Encoding: `req.data = json.dumps(data).encode(JSON_ENCODING)`
- Response: `json.loads(response.read().decode(JSON_ENCODING))`

## Test-Friendly

- Dependency injection: logger/client etc. passed via `__init__` parameters, lazy initialization with defaults
- Configurable: All constants can be overridden via parameters (timeout, interval, etc.)
- Pure functions: Input → Output, no hidden global state

## Quality Standards

✅ Imports: standard library → third-party → local, separate groups with blank lines
✅ Explicit boolean checks: `is not None`, `len(x) > 0`, avoid implicit truthiness
✅ Resources: Unified `with` context managers, avoid manual `.close()`
✅ Logging: Use `logging` module, don't use `print()` for state recording
✅ Exit codes: Success `sys.exit(0)`, failure `sys.exit(1)`
