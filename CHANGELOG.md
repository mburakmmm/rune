# Changelog

All notable changes to **Rune** will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.4.2] - 2026-10-26

### Added & Hardened (GPT 5.6 Audit Hardening for Event Batching, Global Clipping, and Robust Setup)
- **[P1] Event Batching in `AppRunner`**:
  - `process_input_queue()` now processes all pending queued events in batch by invoking `app.update(ev)` sequentially and executing a single `render_frame()` at the end of the batch. This eliminates redundant frame redraws when processing high-volume keystroke or stream bursts.
- **[P1] Global 2D Clipping Engine (`ScreenBuffer.set_clip` / `Frame.set_clip`)**:
  - Added native clipping rectangle support to `ScreenBuffer` (`clip_rect: Rect | None`).
  - `set()`, `write_string()`, and `fill_rect()` strictly respect the active clipping rectangle. Content originating from any widget (`TableWidget`, `Tabs`, `Paragraph`, etc.) is guaranteed never to spill outside its designated bounding area.
- **[P1] Automatic ESC Timeout & Buffer Bounds in `KeyDecoder`**:
  - Isolated trailing `ESC` keys are automatically flushed and enqueued on frame tick.
  - Buffer length cap prevents runaway allocations on malformed escape sequences.
- **[P0] Complete Terminal Setup Protection**:
  - `enter_alt_screen()` and `hide_cursor()` now reside inside the guaranteed `try ... finally` block, ensuring `restore()` executes under all circumstances.

---

## [1.4.1] - 2026-10-26

### Fixed & Hardened (GPT 5.6 Audit Hardening for TUI Runtime & Widgets)
- **[P0] Guaranteed Terminal State Restoration (`try ... finally`)**:
  - `AppRunner.run_interactive()` is now wrapped in a strict `try ... finally: self.driver.restore()` block. Even in the event of an uncaught exception, runtime panic, or early loop break in application code, the terminal is guaranteed to restore normal screen buffers and reveal the cursor.
- **[P0] Input Queue & Keyboard Buffering (`InputQueue`)**:
  - Added `input_queue: list[Event]` to `AppRunner`.
  - Added `queue_event()`, `queue_key()`, and `queue_raw()` methods to enqueue synthetic or streamed input.
  - Added `process_input_queue()` which drains pending user/stream events before ticking.
- **[P1] Stateful Fragmented Key Decoder (`KeyDecoder.feed`)**:
  - `KeyDecoder` now buffers incoming partial byte streams. Fragmented escape sequences (e.g. `ESC [` followed in a later read by `A`) are retained in the buffer and completed across chunks without dropping or misinterpreting keystrokes. Added `flush()` to handle isolated trailing `ESC` keys.
- **[P1] Frame Auto-Clearing (`auto_clear = True`)**:
  - `AppRunner` now defaults to `auto_clear = True`, resetting `current_buffer` at the beginning of each frame. Diff rendering ensures only modified cells are transmitted to the terminal, eliminating ghost artifacts from previously rendered shorter text.
- **[P1] Robust Widget Boundary Clipping**:
  - `Block.render()` now strictly clamps the title width to `(area.x + area.width - 1) - t_x`, preventing oversized titles from overwriting the top-right corner character.
  - `Paragraph.render()` now clips lines to `(area.x + area.width) - col`, preventing right-aligned and centered text from spilling past the bounding box.
- **[P1] Layout Constraint Solver Bounds & Max Cap**:
  - `CONSTRAINT_MAX` now clamps to `min(available_size, c.val1)` and does not expand past its cap when excess space is distributed.
  - Spacing bounds ensure child rects never spill outside the parent `target_area`.

---

## [1.4.0] - 2026-10-26

### Added (Phase 10: Event Loop & Interactive App Runner)
- **`events.nox` — Terminal Event Engine**:
  - `KeyEvent`: Encapsulates key strokes with modifiers (`ctrl`, `alt`, `shift`), `matches(pattern)` check, and canonical `to_str()` format (`"ctrl+c"`, `"alt+x"`, `"up"`, etc.).
  - `TickEvent`: Monotonic timer event carrying `delta_ms` interval for telemetry updates and frame animation.
  - `ResizeEvent`: Terminal geometry change notifications carrying new `width` and `height`.
  - `Event`: Unified event wrapper with type discrimination (`EVENT_KEY`, `EVENT_TICK`, `EVENT_RESIZE`), convenience helpers (`is_key()`, `is_ctrl()`, `is_tick()`, `is_resize()`).
  - `KeyDecoder`: ANSI VT escape sequence parser decoding raw terminal byte streams into typed `KeyEvent`s:
    - Arrow keys: Up (`\x1b[A`), Down (`\x1b[B`), Right (`\x1b[C`), Left (`\x1b[D`).
    - Modified keys: Ctrl+Up (`\x1b[1;5A`), Shift+Down (`\x1b[1;2B`), Alt+Key (`\x1b<c>`).
    - Navigation keys: Home (`\x1b[H`), End (`\x1b[F`), PageUp (`\x1b[5~`), PageDown (`\x1b[6~`), Insert (`\x1b[2~`), Delete (`\x1b[3~`).
    - Function keys: F1..F4 (SS3 `\x1bOP`..`\x1bOS`), F5..F12 (CSI `\x1b[15~`..`\x1b[24~`).
    - Control characters: Ctrl+A..Ctrl+Z (`chr(1)`..`chr(26)`), Tab (`\t`), Enter (`\n`/`\r`), Backspace (`\x7f`).
- **`tui_runtime.nox` — Elm/Bubbletea-style Application Architecture**:
  - `Frame`: Drawing surface wrapper around `ScreenBuffer`, exposing `size() -> Rect`, `write_string()`, and `fill_rect()`.
  - `TuiApp`: Base class for interactive terminal applications defining lifecycle hooks:
    - `init()`: Called before the first frame.
    - `update(event: Event) -> bool`: State transition function. Returns `True` to keep running, `False` to exit. Handles `q`, `Ctrl+C`, `Esc` by default.
    - `view(frame: Frame) -> None`: Pure render function drawing UI to the frame.
  - `AppRunner`: Application execution engine:
    - `step(event: Event) -> bool`: Dispatches a single event, calls `update()`, renders via `view()`, and flushes diff to terminal.
    - `run_event_stream(event_list: list[Event]) -> int`: Deterministic headless event runner for automated testing, scripting, and CI environments.
    - `run_interactive(max_frames: int = -1)`: Interactive loop managing alt-screen buffer (`\x1b[?1049h`), cursor hiding (`\x1b[?25l`), `nox.time` monotonic tick scheduling, and guaranteed cleanup restoration via `driver.restore()`.
- **Test Suite**:
  - Added `tests/unit/tui_app_test.nox` (22 passing test suites total).
- **Example**:
  - Added `examples/tui_dashboard.nox` demonstrating an interactive full-screen cloud monitor with live telemetry ticks, keyboard navigation, and tab switching.

---

## [1.3.0] - 2026-10-26

### Added (Phase 9: Declarative TUI Widgets)
- **`Block` Widget (`rune.tui` / `widgets.nox`)**:
  - Container box with customizable border styles (`"rounded"`, `"square"`, `"double"`, `"ascii"`, `"none"`).
  - Title positioning with alignment support (`"left"`, `"center"`, `"right"`).
  - Automatic inner area calculation via `block.inner(area)` for nested widget rendering.
- **`Paragraph` Widget**:
  - Multiline text rendering with automatic wrapping or raw newline splitting.
  - Text alignment (`"left"`, `"center"`, `"right"`) and vertical scroll offset support.
- **`Gauge` Widget**:
  - Visual progress bar / meter (0%..100%) with fill character (`"█"`), track character (`"─"`), and centered label.
  - Boundary value clamping (negative values clamp to 0%, over-100% clamp to 100%).
- **`Tabs` Widget**:
  - Interactive tab bar header with active tab highlighting (`[Tab Name]`), inactive tab styling, and configurable separators (`"│"`).
- **`ListItem` & `ListWidget`**:
  - Selectable vertical list of items.
  - Navigation helpers: `select(index)`, `select_next()`, `select_prev()` with cyclic wrap-around.
  - Active selection indicator pointer (`"► "`) and unselected item indentation alignment.
- **`TableWidget`**:
  - Structured 2D table grid with customizable headers, column widths, separator rule, and data rows.
  - Selected row highlighting and vertical scroll offset.
- **`HelpBar` Widget**:
  - Footer keybinding shortcuts bar (`key` + `description`), with customizable separators and styling.
- **`ScreenBuffer.to_text()`**:
  - Utility method converting a 2D screen buffer into a newline-separated plain text string for snapshot testing, logging, and inspection.
- **Test Suite**:
  - Added `tests/unit/tui_widgets_test.nox` (21 passing test suites total).
- **Example**:
  - Added `examples/tui_widgets_demo.nox` featuring a comprehensive 80x24 multi-pane terminal dashboard combining `Layout`, `Block`, `Tabs`, `Gauge`, `ListWidget`, `TableWidget`, and `HelpBar`.

---

## [1.2.0] - 2026-10-26

### Added (Phase 8: Geometry & Flex Layout Engine)
- **`Rect` Class (`rune.tui`)**:
  - 2D rectangular bounding box representing terminal UI coordinates: `Rect(x, y, width, height)`.
  - Methods: `area()`, `is_empty()`, `contains(px, py)`.
  - Margin helpers: `inner(margin)` for uniform margin inset and `inner_xy(margin_x, margin_y)` for differential margin insets.
  - Set geometry: `intersection(other)` for finding overlapping areas and `union(other)` for computing bounding boxes.
- **`Constraint` Class & Helpers**:
  - `length_constraint(n)`: fixed cell dimension (e.g. 3 rows for a header).
  - `percentage_constraint(p)`: percentage of parent available size (0%..100%).
  - `ratio_constraint(num, den)`: exact fractional width/height allocation.
  - `min_constraint(n)`: flexible dimension with minimum size guarantee.
  - `max_constraint(n)`: flexible dimension with maximum size cap.
- **`Layout` Class**:
  - Flex constraint solver dividing parent `Rect` into child chunks along `DIRECTION_VERTICAL` or `DIRECTION_HORIZONTAL`.
  - Automatic margin inset and inter-chunk spacing (`set_margin()`, `set_spacing()`).
  - Deterministic remainder distribution: eliminates 1-cell rounding errors and pixel/cell gaps.
  - Handles nested layouts (grids, matrices, split sidebars, multi-column dashboards).
- **Test Suite**:
  - Added `tests/unit/tui_layout_test.nox` (20 passing test suites total).
- **Example**:
  - Added `examples/tui_layout_demo.nox` demonstrating nested multi-pane grid layout.

---

## [1.1.0] - 2026-10-26

### Added (Phase 7: Terminal Screen Driver & Virtual Buffer)
- **`rune.tui` Module**:
  - Introduced the dedicated Full-Screen Terminal User Interface module.
- **`Cell` Class**:
  - Models an individual terminal display cell with character glyph, visible cell width (1 or 2), and associated `Style`.
  - Includes `is_same()`, `clone()`, and `reset()`.
- **`ScreenBuffer` Class**:
  - 2D character cell matrix for off-screen rendering.
  - Bounds-checked `get()`, `set()`, `fill_rect()`, `clear()`, `clone()`, and `resize()`.
  - `write_string()` supporting automatic wide-character (CJK/Emoji) 2-cell allocation and placeholder positioning.
- **`TerminalDriver` & `MockTerminalDriver`**:
  - Alternate screen buffer management: `enter_alt_screen()` (`\x1b[?1049h`), `leave_alt_screen()` (`\x1b[?1049l`).
  - Cursor control: `hide_cursor()` (`\x1b[?25l`), `show_cursor()` (`\x1b[?25h`), `move_cursor(row, col)` (`\x1b[{r};{c}H`).
  - Screen operations: `clear_screen()` (`\x1b[2J\x1b[H`), `clear_line()` (`\x1b[2K`).
  - Double-buffering & Diff-based painter: `render_diff()` and `flush()`. Compares `current_buffer` and `previous_buffer`, emitting minimal ANSI commands and localized cursor jumps to eliminate screen flicker.
  - Restoration guarantee: `restore()` ensures the cursor is restored and alternate screen exited safely on errors or exit.
  - `MockTerminalDriver` for headless testing and ANSI escape sequence stream capture.
- **Test Suite**:
  - Added `tests/unit/tui_buffer_test.nox` (19 passing test suites total).
- **Example**:
  - Added `examples/tui_buffer_demo.nox` demonstrating double-buffered diff rendering.

---

## [1.0.1] - 2026-10-26

### Security & Hardening Fixes
- **ANSI Sanitization & Line Spoofing Protection (P0)**:
  - `sanitize_ansi()` now drops standalone `\r` (carriage return) characters to prevent terminal line overwrite and status spoofing; `\r\n` sequences are safely normalized to `\n`.
  - Added stripping of C1 control codes (code points 128 through 159) to prevent 8-bit terminal control injections.
  - Refactored `strip_ansi()` and `sanitize_ansi()` from byte indexing to Unicode scalar `ord(s[i])` indexing.
- **Unicode Terminal Display Cell Width (P1)**:
  - `char_width()` and `cell_width()` now accurately compute display cell counts matching standard terminal behaviour:
    - CJK Unified Ideographs, Hangul, Fullwidth punctuation, and Katakana/Hiragana return width 2 (`界`, `你`, `好`).
    - Standard emojis, pictographs, and symbols return width 2 (`🙂`, `🦀`, `☀️`).
    - Combining diacritical marks (U+0300..U+036F), zero-width joiners (ZWJ), non-joiners (ZWNJ), and variation selectors return width 0.
    - Standard Latin, numbers, and basic symbols return width 1 (`A`, `é`).
  - `truncate()` and `wrap_text()` updated to respect display cell widths instead of naive code point counts.
- **Consistent Non-Interactive Prompt Behavior (P1)**:
  - `prompt_int()` now strictly raises `NonInteractiveError` when invoked in non-interactive/CI environments without an explicit default value (preventing silent 0 or min_val fallback).
  - `select_choice()` and `select_multiple()` now strictly raise `NonInteractiveError` in non-interactive/CI environments when required selections lack explicit defaults.
- **Color Validation & SGR Safety (P1)**:
  - `color_rgb()`, `bg_rgb()`, `color_256()`, and `bg_256()` clamp integer components to `0..255`.
  - `hex_to_rgb()` and `hex_to_bg_rgb()` strictly validate all 6 hex digits, rejecting invalid strings instead of falling back to 0.
  - Semicolon-separated custom SGR parameters in `resolve_color_code()` are checked against `_is_safe_sgr_params()` to prevent escape sequence parameter injection.
- **Progress Task Boundary Enforcement (P1)**:
  - `Task.update()` and `Task.advance()` clamp completed values to `>= 0`.
  - `Task.percentage()` clamps percentage between 0% and 100%.
- **Documentation Parity (P2)**:
  - Synchronized the executive dashboard table at the top of `ROADMAP.md` to reflect all completed milestones.
- **Test Suites**:
  - Added `tests/unit/security_hardening_test.nox` (18 passing test suites total).

---

## [1.0.0] - 2026-10-26

### Production Release — 1.0.0 General Availability
- **API Stability Freeze**: All public signatures across `rune.core`, `rune.render`, `rune.progress`, `rune.prompt`, and `rune.cli` are stable and contractual.
- **Rich + Click + Typer Unified Framework**: Full feature set delivered with zero native C dependencies, 100% pure statically typed Nox.
- **Verification**: 17 comprehensive test suites passing green across unit, integration, benchmark, and snapshot testing.
- **Documentation**: Complete API documentation in `docs/api.md`, architecture guide in `docs/architecture.md`, roadmap tracking in `ROADMAP.md`, and 5 compiling examples in `examples/`.

---

## [0.9.0] - 2026-10-26

### Added
- **Cross-Platform & Windows VT Mode Detection (`rune.core`)**:
  - `detect_console_options()` audited for Windows console hosts.
  - Automatically identifies modern Windows Terminal (`WT_SESSION`), ConEmu (`ConEmuANSI`), and `ANSICON` environments.
  - Graceful ASCII & plain-text fallback on legacy Windows Command Prompt hosts lacking VT support.
- **Deterministic Golden Snapshot Testing (`tests/snapshot/`)**:
  - `tests/snapshot/snapshot_test.nox` covering byte-exact golden snapshots for Unicode & ASCII tables, rounded panels, centered rules, CLI `--help` pages, and choice error diagnostics.
- **Monochrome & Screen Reader Accessibility**:
  - Verified structure, labels, and borders ensure complete clarity without requiring color vision.

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
