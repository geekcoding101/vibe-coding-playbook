---
trigger: glob
globs: **/*.sh
---

# Rule: Shell Scripts

## Script Requirements

- **Argument Parsing**: Use `getopts` to support command-line arguments, provide clear `-h/--help` documentation.
- **Log File**: Output important operations to log file for troubleshooting; use `tee` to output to both console and file simultaneously.
  - Log file naming convention: `<script_name>_<timestamp>.log` (e.g., `backup_2025-12-14_15-30-45.log`)
  - Timestamp format: `$(/bin/date +%Y-%m-%d_%H-%M-%S)`
  - Support custom log file path via command-line argument (e.g., `-l/--log-file`)
- **Colorful Output**: Use ANSI color codes in console logs to distinguish different levels (e.g., `INFO` green, `WARN` yellow, `ERROR` red) for better readability.
- **Function Reuse**: Extract common logic (input validation, logging, error handling, etc.) into functions to avoid code duplication.
- **Single Responsibility**: Each function should do one thing; function names should clearly express their purpose (e.g., `validate_input`, `log_info`, `cleanup_temp_files`).
- **Complete Comments**: Document purpose, dependencies, and usage examples at the script header; add comments for key functions and complex logic.

## Security & Robustness

- Always quote variables: `"$VAR"` to avoid word splitting and glob expansion issues.
- Add path validation or explanatory comments for dangerous commands like `rm`, `mv` to prevent accidental deletion of important files.

## Maintainability

- Avoid piling complex business logic in Shell scripts; when logic becomes complex, consider rewriting in Python / TS or other languages.
- Don't hardcode passwords, tokens, or secrets in scripts; inject them from environment variables or secret management.

When you see unquoted variables or suspected hardcoded secret information, automatically point it out and provide a secure version.
