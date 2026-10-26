# Changelog

All notable changes to **Rune** will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [0.8.0] - 2026-10-26

### Added
- **Performance Benchmark Suite (`benchmarks/`)**:
  - `benchmarks/bench_table.nox`: benchmarks table layout performance up to 1,000 rows (sub-3ms execution).
  - `benchmarks/bench_cli.nox`: benchmarks CLI argument parsing up to 1,000 options (sub-2ms execution).
  - `benchmarks/bench_render.nox`: benchmarks syntax highlighting (800 lines), markdown parsing (700 lines), and tree layout (1,000 nodes).
  - `benchmarks/run_benchmarks.nox`: unified runner executing all benchmarks and presenting results in a live Rune table.
- **Allocation Bounds Enforcement**:
  - Table column count capped at 100 to prevent denial-of-service via huge dimension allocations.
  - Table padding clamped within [0, 20].
  - Tree rendering bounded to 10,000 nodes to prevent memory exhaustion and recursion hangs.
- **Test Suites**:
  - `tests/unit/benchmark_test.nox` verifying dimension limits and bounds enforcement.

---

## [0.7.0] - 2026-10-26

### Added
- **Shell Autocompletion (`rune.cli`)**:
  - `generate_bash_completion(cmd: Command) -> str`: generates Bash completion script using `compgen`, extracting subcommands, subcommand aliases, options, and option choices.
  - `generate_fish_completion(cmd: Command) -> str`: generates native Fish completion script with subcommands, descriptions, short/long flags, and `-a` choice arguments.
  - `generate_zsh_completion(cmd: Command) -> str`: generates Zsh completion script with `_arguments` and `_describe` commands.
  - `generate_completion(cmd: Command, shell: str) -> str`: unified completion dispatcher for `bash`, `fish`, and `zsh`.
  - `sanitize_shell_str(s: str) -> str`: sanitizes all shell identifiers and text to prevent command injection vulnerabilities.
- **Unit Tests**:
  - `tests/unit/completion_test.nox` covering Bash, Fish, Zsh script generation, subcommand extraction, option choices, and shell injection protection.

---

## [0.6.0] - 2026-10-26

### Added
- **Syntax Highlighting (`rune.render`)**:
  - `Syntax(code, language, line_numbers, start_line)`: fast, pure Nox lexical scanner for syntax highlighting.
  - Highlights language keywords (`def`, `class`, `return`, `if`, etc.), builtins/types (`int`, `str`, `list`, etc.), string literals, numbers, comments, and operators.
  - Optional line numbering with customizable start line and padded gutter (`│` / `|`).
  - `highlight_code(code, language, line_numbers, options) -> str`.
- **Terminal Markdown (`rune.render`)**:
  - `Markdown(content)`: terminal markdown renderer supporting headings (`#`, `##`, `###`), bulleted and numbered lists, blockquotes (`>`), horizontal rules (`---`), and inline styling (bold, italic, code).
  - Code blocks (```` ```nox ````) rendered inside styled panels with syntax highlighting.
  - Full plain-text and ASCII fallback when ANSI color or Unicode is absent.
  - `render_markdown(content, options) -> str`.
- **Test Suites**:
  - `tests/unit/syntax_test.nox`: tests tokenization, line numbers, ASCII fallback, color stripping, and ANSI injection sanitization.
  - `tests/unit/markdown_test.nox`: tests headings, lists, blockquotes, code blocks, rules, and plain/ASCII fallback.

---

## [0.5.0] - 2026-10-26

### Added
- **Validated Integer Input (`prompt_int`)**:
  - `prompt_int(question, default_val, min_val, max_val, max_retries, console)`: validates integer format with whitespace trimming.
  - Range validation with `min_val` and `max_val` bounds checks.
  - Interactive retry loop with clear error feedback, bounded by `max_retries` before raising `PromptError`.
- **Multi-Select Prompt (`select_multiple`)**:
  - `select_multiple(question, choices, default_selected, console)`: allows selecting zero, one, or multiple items.
  - Accepts comma-separated numbers (e.g. `1, 3`), exact item names, or keyword `all`.
  - Gracefully falls back to `default_selected` when enter is pressed without input.
- **Unit Tests**:
  - `tests/unit/prompt_advanced_test.nox` testing numeric range validation, retry logic, and multi-selection parsing.

---

## [0.4.0] - 2026-10-26

### Added
- **Monotonic ETA & Transfer Speed Math (`rune.progress`)**:
  - `Task.speed() -> float`: calculates items/bytes per second.
  - `Task.eta_seconds() -> int` and `Task.format_eta() -> str`: safe, division-by-zero guarded remaining time estimate.
  - `format_duration(seconds: int) -> str`: formats seconds as `mm:ss` or `hh:mm:ss`.
  - `format_bytes(bytes: int) -> str`: formats byte counts into `B`, `KB`, `MB`, `GB`.
- **Terminal Rate Limiter (`ProgressBar.should_render`)**:
  - `ProgressBar.min_interval_ms: int` (default 50ms): bounds terminal redraws to prevent CPU & stdout flooding.
- **Multi-Task Live Progress Display**:
  - `ProgressBar.render_all(options, show_eta)`: renders all registered tasks in a single unified block with percentage, progress bar, ETA, and elapsed time upon completion.
- **Test Suites**:
  - `tests/unit/progress_live_test.nox` covering duration formatting, byte formatting, ETA calculation, multi-task layout, and rate limiting logic.

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
