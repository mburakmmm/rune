# Changelog

All notable changes to **Rune** will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [0.3.0] - 2026-10-26

### Added
- **Choices & Enum Validation (`rune.cli`)**:
  - `choices: list[str]` validation on `Option` and `Argument`.
  - Automatically rejects disallowed inputs with exit code 2 and lists valid options in error messages.
  - Automatically formats allowed choices in `--help` output: `[choices: val1, val2]`.
- **Environment Variable Sourced Defaults**:
  - `Option.env_var: str`: binds an environment variable to fallback when option is absent on CLI.
  - Strict precedence order: `CLI argument > Environment variable > Declared default`.
  - Displays `[env: VAR_NAME]` in `--help`.
- **Clustered Short Boolean Flags**:
  - Cluster short flags (e.g. `-xvf` automatically expands to `-x`, `-v`, `-f`).
- **Repeated / Multiple Options**:
  - `Option.multiple: bool`: allows repeated option invocations (e.g. `--include dir1 --include dir2`).
  - `CliContext.get_list(name: str) -> list[str]` retrieves all accumulated values.
- **Secret Option Redaction**:
  - `Option.is_secret: bool`: masks default values in `--help` as `[default: ******]` and prevents leaking in diagnostics.
- **Command & Subcommand Aliases**:
  - `Command.add_alias(alias: str)` allows defining command shortcuts (e.g. `git ci` -> `commit`).
  - Displays aliases in `--help` commands list: `commit (ci, co)`.
- **Test Suites**:
  - `tests/unit/cli_advanced_test.nox` covering choices, env vars, clustered flags, repeated options, secrets, and aliases.

---

## [0.2.0] - 2026-10-26

### Added
- **Text & Span Composition (`rune.text`)**:
  - `Span` model associating styled text segments.
  - `Text` container supporting fluent chaining (`append()`), multi-span rendering, and accurate visible cell width calculation.
- **Extended Color Palettes (`rune.core`)**:
  - ANSI 256-color support (`color_256`, `bg_256`).
  - 24-bit Truecolor RGB support (`color_rgb`, `bg_rgb`).
  - Hex color code resolution (`#RRGGBB` via `hex_to_rgb` and `hex_to_bg_rgb`).
- **Layout & Wrapping Utilities (`rune.core`)**:
  - `wrap_text(s, max_width)`: word-aware paragraph wrapping with hard splitting of oversized words.
  - `truncate(s, max_width, ellipsis)`: width-aware string truncation with customizable ellipsis.
  - Dynamic terminal dimension discovery from `COLUMNS` and `LINES` environment variables.
- **Table Multiline & Wrapped Rendering (`rune.table`)**:
  - Multiline cell support (`\n` in cell values).
  - Configurable `Column.max_width`: wraps cell contents automatically to fit constrained column widths.
  - Vertical border continuity across multiline sub-rows.
- **Test Suites**:
  - `tests/unit/text_test.nox`
  - `tests/unit/core_color_wrap_test.nox`
  - `tests/unit/table_multiline_test.nox`

---

## [0.1.0] - 2026-10-26

### Added
- **Core Module (`rune.core`)**:
  - `Style` and ANSI color management supporting 16 foreground and background colors.
  - SGR code generator with automatic reset codes to prevent style leakage.
  - `strip_ansi` and `sanitize_ansi` for stripping malicious escape codes and measuring visible cell length.
  - `Console` and `ConsoleOptions` supporting stdout and capture sinks for testing.
  - Environment-aware capability detection (`NO_COLOR`, `CLICOLOR_FORCE`, `TERM=dumb`, `CI`).
- **Render Module (`rune.render`)**:
  - `Rule`: horizontal dividers with custom characters, alignments, and titles.
  - `Panel`: customizable box borders (`rounded`, `square`, `double`, `ascii`) with title and padding.
  - `Table`: tabular layout with auto-width calculation, alignments (left, center, right), and borders.
  - `Tree`: hierarchical branching tree visualization with iterative stack execution.
- **Progress Module (`rune.progress`)**:
  - `Task` tracking completion, percentages, and status.
  - `ProgressBar` rendering with unicode block characters and ASCII fallback.
  - `Spinner` with unicode dots animation and non-TTY plain text status fallback.
- **Prompt Module (`rune.prompt`)**:
  - `prompt_text` with default fallback and non-interactive protection.
  - `confirm` for yes/no boolean input.
  - `select_choice` for numbered or keyword selection.
  - Explicit `PromptError` raised when password masking is attempted without runtime echo suppression.
- **CLI Framework (`rune.cli`)**:
  - Declarative `Argument` and `Option` specifications.
  - Subcommand tree routing (`Command.add_subcommand`).
  - Typed conversions for `str`, `int`, `float`, and `bool` flags.
  - Automatic rich `--help` generation and standard exit codes (0 for success, 2 for usage errors).
- **Test Suite**:
  - Unit tests covering core, rendering, progress, prompts, and CLI parsing.
  - End-to-end integration tests verifying cross-module interoperability.
