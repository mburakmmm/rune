# Changelog

All notable changes to **Rune** will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
