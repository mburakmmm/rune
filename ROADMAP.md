# Rune — Comprehensive Development Roadmap

> **North Star:** Deliver a robust, dependency-free, statically typed terminal presentation (Rich) and CLI application (Click + Typer) framework for the **Nox programming language**, strictly adhering to the invariants and quality gates defined in `AGENTS.md`.

---

## 1. Executive Summary & Progress Dashboard

| Phase / Milestone | Target Version | Scope | Status | Test Suites |
| :--- | :--- | :--- | :--- | :--- |
| **Phase 0: Discovery & Audit** | `v0.1.0` | Capability matrix, architecture docs, stdlib audit | **Completed** | 100% verified |
| **Phase 1: Core & Render MVP** | `v0.1.0` | Styles, ANSI sanitization, Rules, Panels, Tables, Trees | **Completed** | `core_test`, `render_test` |
| **Phase 2: Progress Indicators** | `v0.1.0` | Task tracking, Progress bars, Non-TTY spinners | **Completed** | `progress_test` |
| **Phase 3: CLI Framework MVP** | `v0.1.0` | Declarative args, options, subcommands, help, exit codes | **Completed** | `cli_test` |
| **Phase 4: Prompts MVP** | `v0.1.0` | Text prompt, Confirm, Choice selection, CI safety | **Completed** | `prompt_test` |
| **Integration & Release 0.1** | `v0.1.0` | Facade, showcase, examples, docs, publish to index | **Completed** | `integration_test` (6/6 passing) |
| **Phase 1.5: Advanced Render** | `v0.2.0` | Multiline table wrapping, 256/RGB colors, `Text`/`Span` | **Completed** | `text_test`, `core_color_wrap_test`, `table_multiline_test` |
| **Phase 3.5: Advanced CLI** | `v0.3.0` | Choice/enum options, env fallbacks, repeated options, aliases | **Completed** | `cli_advanced_test` |
| **Phase 2.5: Live Progress** | `v0.4.0` | Rate throttling, multi-task live layout, ETA/speed math | **Completed** | `progress_live_test` |
| **Phase 4.5: Advanced Prompts** | `v0.5.0` | Multi-select, input validators, retry limits | **Completed** | `prompt_advanced_test` |
| **Phase 5: Terminal Markdown & DX** | `v0.6.0` | Basic terminal Markdown parser, syntax highlighter | **Completed** | `markdown_test`, `syntax_test` |
| **Phase 5.5: Shell Completion** | `v0.7.0` | Bash, Zsh, Fish autocompletion script generation | **Completed** | `completion_test` |
| **Phase 5.8: Benchmarks & Perf** | `v0.8.0` | Allocation audits, 1000-row table & 1000-arg CLI benchmarks | **Completed** | `benchmark_test`, `run_benchmarks` |
| **Phase 5.9: Cross-Platform** | `v0.9.0` | Windows VT console compatibility & snapshot tests | **Completed** | `snapshot_test` |
| **Phase 6: Rune 1.0 Release** | `v1.0.0` | Public API freeze, complete documentation, release gates | **Completed** | Full Suite (17 suites) |
| **Phase 6.1: Security & Unicode** | `v1.0.1` | \r line spoofing fix, C1 stripping, cell_width wcwidth, CI prompts | **Completed** | `security_hardening_test` (18 suites) |
| **Phase 7: TUI Screen Driver** | `v1.1.0` | Alt-screen buffers, diff-based cell grid painter, cursor control | **Completed** | `tui_buffer_test` (19 suites) |
| **Phase 8: Geometry & Flex Layout** | `v1.2.0` | `Rect` geometry, `Constraint` types, flex layout solver | **Completed** | `tui_layout_test` (20 suites) |
| **Phase 9: Declarative Widgets** | `v1.3.0` | `Block`, `Paragraph`, `Gauge`, `Tabs`, `ListWidget`, `TableWidget` | **Completed** | `tui_widgets_test` (21 suites) |
| **Phase 10: Event Loop & TUI App** | `v1.4.0` | ANSI key parser, `TuiApp` runtime, frame cycle, showcase dashboard | **Completed** | `tui_app_test` (22 suites) |

---

## 2. Completed Work (v0.1.0 — Released)

All deliverables from the initial autonomous implementation task (`AGENTS.md §23`) have been implemented, formatted in-place (`noxc fmt`), tested with `noxc test` (6 passed, 0 failed), published via `noxc publish`, and pushed to GitHub:

### Architecture & Documentation
- [x] Comprehensive capability audit (`docs/capabilities.md`).
- [x] Architecture specification and module boundaries (`docs/architecture.md`).
- [x] Package manifest (`nox.json`), MIT license (`LICENSE`), and `README.md`.
- [x] API reference manual (`docs/api.md`).
- [x] Contribution standards (`CONTRIBUTING.md`) and changelog (`CHANGELOG.md`).

### `rune.core`
- [x] ANSI 16-color palette (foreground & background) with SGR reset sequences.
- [x] `Style` model supporting `bold`, `dim`, `italic`, `underline`, and `reverse`.
- [x] Security sanitization: `strip_ansi` and `sanitize_ansi` to prevent terminal injection.
- [x] Visible cell width calculation (`cell_width`).
- [x] `Console` and `ConsoleOptions` supporting direct terminal printing and in-memory capture.
- [x] Automatic environment detection (`NO_COLOR`, `CLICOLOR_FORCE`, `TERM=dumb`, `CI`).

### `rune.render`
- [x] `Rule`: Horizontal divider with title alignments (left, center, right) and ASCII fallback.
- [x] `Panel`: Box container with rounded, square, double, and ASCII borders, title, subtitle, and padding.
- [x] `Table`: Tabular layout with column alignments, auto-width computation, and border styles.
- [x] `Tree`: Hierarchical tree rendering using an iterative stack algorithm (immune to recursion limits).

### `rune.progress`
- [x] `Task`: Progress tracking with percentage and completion states.
- [x] `ProgressBar`: Deterministic progress bar with unicode block characters and ASCII fallback.
- [x] `Spinner`: Unicode dots animation with non-TTY linear `[STATUS]` fallback for CI.

### `rune.prompt`
- [x] `prompt_text`: Line input with default value support.
- [x] `confirm`: Boolean confirmation prompt (`[Y/n]`, `[y/N]`).
- [x] `select_choice`: Numbered or keyword-based choice selection.
- [x] `NonInteractiveError`: Prevents hanging in CI environments when required input lacks a default.
- [x] `prompt_password`: Explicitly raises `PromptError` acknowledging current Nox runtime limitation.

### `rune.cli`
- [x] `Argument`: Positional arguments with typed conversions (`str`, `int`, `float`, `bool`) and variadic support.
- [x] `Option`: Long options (`--opt val`, `--opt=val`), short options (`-o val`), and boolean flags.
- [x] Negative number handling: Prevents `-42` from being misclassified as a flag.
- [x] End of options delimiter: `--` stops option processing.
- [x] `Command`: Nested subcommand routing (`Command.add_subcommand`).
- [x] Automated `--help` synopsis with formatted descriptions and options.
- [x] Standard exit codes: `0` (success/help/version), `2` (usage error), `1` (general failure).
- [x] `nox.os.exit()` safety: Never called from parser core; exclusively managed by the top-level runner.

---

## 3. Detailed Version Roadmap

### 📦 v0.2.0 — Advanced Rendering & Rich Layout (Completed)
**Goal:** Enhance table layout and inline styling ergonomics to match advanced Rich formatting.

- [x] **Multiline Cell Wrapping in `Table`**:
  - Automatically wrap cell text exceeding column width across multiple lines.
  - Maintain border continuity across multiline rows.
- [x] **`Text` & `Span` Model**:
  - Support rich text with inline styled segments (`Text.append("Error: ", Style("red")).append("file not found")`).
  - Calculate combined visible width accurately.
- [x] **256-Color & RGB Truecolor Support**:
  - Add ANSI 256-color codes (`38;5;n` / `48;5;n`).
  - Add 24-bit RGB Truecolor SGR formatting (`38;2;r;g;b`).
  - Hex color parsing (`#RRGGBB` via `hex_to_rgb`).
- [x] **Terminal Width Discovery**:
  - Probe environment variables `COLUMNS` and `LINES` before falling back to 80x24.
- [x] **Text Truncation & Overflow Helpers**:
  - Implement `truncate(s: str, max_width: int, ellipsis: str = "...")`.
  - Implement `wrap_text(s: str, max_width: int) -> list[str]`.

---

### 📦 v0.3.0 — Advanced CLI Engine & Developer Ergonomics (Completed)
**Goal:** Expand CLI features to match Python Click/Typer versatility.

- [x] **Choice / Enum Option Validation**:
  - Support `choices: list[str]` on options and positional arguments.
  - Automatically validate and generate readable errors: `Error: Invalid value for '--mode': choose from 'fast', 'slow'`.
  - List allowed choices in `--help` output.
- [x] **Environment Variable Sourced Defaults**:
  - Support `Option.env_var: str` (e.g. `--token` defaults to `API_TOKEN` env var).
  - Precedence hierarchy: `CLI argument > Environment variable > Declared default`.
  - Safe exception handling with `get_env_safe` (handling `OsError`).
- [x] **Grouped Short Flags**:
  - Support clustered boolean flags: `-xvf` expands to `-x`, `-v`, `-f`.
- [x] **Repeated / Multiple Options**:
  - Support accumulating repeated options (e.g. `--include dir1 --include dir2` -> `list[str]`).
  - Retrieve using `CliContext.get_list(name)`.
- [x] **Command & Option Aliases**:
  - Support command aliases (e.g. `ci` for `commit`, `i` for `install`).
- [x] **Secret Redaction in Diagnostics**:
  - Automatically mask options flagged with `is_secret = True` in help defaults as `[default: ******]`.

---

### 📦 v0.4.0 — Live Display & Progress Hardening (Completed)
**Goal:** Production-grade progress display with rate limiting and duration estimation.

- [x] **Update Rate Limiter**:
  - Throttle terminal redraws to a configurable interval (`min_interval_ms`, default 50ms) to avoid stdout flooding.
- [x] **Monotonic ETA & Transfer Speed Math**:
  - Calculate transfer rates (`speed() -> float`) using monotonic clock.
  - Formatted elapsed duration and remaining time estimation (`format_duration`, `format_eta`).
  - Format byte representations (`format_bytes`).
  - Division-by-zero guarded against zero rate/completion.
- [x] **Multi-Task Live Progress Display**:
  - Render multiple simultaneous concurrent tasks in a consolidated live block (`render_all()`).
- [x] **Completion Summary**:
  - Display clean completion line upon task finish: `[Done in 00:02]`.

---

### 📦 v0.5.0 — Advanced Interactive Prompts (Completed)
**Goal:** Rich, validated user interaction for interactive CLI wizards.

- [x] **Validated Integer & Numeric Input (`prompt_int`)**:
  - Validates integer formatting with automatic whitespace trimming.
  - Bounds enforcement: `min_val` and `max_val` constraints with descriptive errors.
  - Configurable retry loop (`max_retries`) raising `PromptError` when exhausted.
- [x] **Multi-Select Prompt (`select_multiple`)**:
  - Allow selecting multiple options via comma-separated numbers (e.g. `1, 3`) or direct names.
  - Supports keyword `all` to select all available choices.
  - Configurable `default_selected` fallback on empty input.
- [x] **Runtime-Gated Password Prompt**:
  - Explicit `PromptError` detailing terminal raw mode / echo suppression prerequisite.

---

### 📦 v0.6.0 — Terminal Markdown & Syntax Highlighting (Completed)
**Goal:** Rich readable documentation and code snippets inside the terminal.

- [x] **Terminal Markdown Subset**:
  - Headings (`# H1`, `## H2`, `### H3`) rendered with distinctive styles and underlines (`═` / `─` / ASCII fallback).
  - Bulleted (`*`, `-`) and numbered (`1.`) lists with indentation.
  - Blockquotes (`> text`) with left border guides (`│ ` / `| `).
  - Inline code (`` `code` ``), bold (`**bold**`), italic (`*italic*`).
  - Fenced code blocks (```` ```nox ````) framed in clean panels with syntax highlighting.
- [x] **Lightweight Nox Code Highlighter**:
  - Highlights keywords (`def`, `class`, `import`, `return`, `if`, `else`, etc.).
  - Highlights builtin types (`int`, `str`, `float`, `bool`, `list`, etc.).
  - Highlights string literals with escape sequence handling.
  - Highlights numeric literals (integers, floats, hex `0x...`).
  - Highlights comments (`# ...`) with dim styling.
  - Configurable line numbers with padded gutter (`│` / `|`).
  - Pure Nox token-scanning implementation without heavy compiler dependencies.

---

### 📦 v0.7.0 — Shell Autocompletion (Completed)
**Goal:** Fast, native shell tab-completion for Rune CLIs.

- [x] **Completion Script Generator**:
  - Generate completion scripts for **Bash**, **Zsh**, and **Fish**.
  - Extract subcommands, subcommand aliases, options, short flags, and choices statically from `Command` tree.
- [x] **Safe Shell Escaping**:
  - `sanitize_shell_str()` eliminates shell injection vulnerabilities (semicolons, backticks, subshells, quotes, pipes).
- [x] **Shell Dispatcher**:
  - `generate_completion(cmd, shell)` with automatic shell routing and error handling.
  - Re-exported under `rune` and `rune.cli`.

---

### 📦 v0.8.0 — Performance Hardening & Benchmarking (Completed)
**Goal:** Quantifiable performance benchmarks and zero regression guarantees.

- [x] **Benchmark Suite (`benchmarks/`)**:
  - `benchmarks/bench_table.nox`: measures table rendering across 10, 100, and 1,000 rows (sub-3ms SLA).
  - `benchmarks/bench_cli.nox`: measures argument parsing across 10, 100, and 1,000 options (sub-2ms SLA).
  - `benchmarks/bench_render.nox`: measures 800-line syntax highlighting, 700-line markdown, and 1,000-node trees.
  - `benchmarks/run_benchmarks.nox`: unified benchmark runner reporting metrics in a Rune `Table`.
- [x] **String Concatenation Optimization**:
  - Verified `nox.strings.join` and `nox.strings.repeat` across all render loops to eliminate intermediate string allocations.
- [x] **Allocation Bounds Enforcement**:
  - Table column count capped at 100 to prevent denial-of-service via huge dimensions.
  - Table padding clamped between 0 and 20.
  - Tree rendering capped at 10,000 visited nodes to prevent OOM / cyclic hangs.
  - `tests/unit/benchmark_test.nox` testing bounds and limits.

---

### 📦 v0.9.0 — Cross-Platform & Snapshot Parity (Completed)
**Goal:** Verified behavior across macOS, Linux, and Windows.

- [x] **Windows VT Mode Audit**:
  - Implemented environment detection for Windows Terminal (`WT_SESSION`), ConEmu (`ConEmuANSI`), and `ANSICON` in `detect_console_options()`.
  - Added clean ASCII & monochrome fallback for legacy Windows Command Prompt hosts without VT mode.
- [x] **Snapshot Testing Harness (`tests/snapshot/`)**:
  - `tests/snapshot/snapshot_test.nox` verifying golden deterministic string parity.
  - Golden snapshot coverage for: Unicode rounded table, ASCII table, rounded panel, centered rule, CLI `--help` synopsis, and choice error diagnostics.
- [x] **Monochrome & Screen Reader Review**:
  - Verified that all components convey complete meaning with structure, borders, and labels without relying on ANSI color alone.

---

### 🚀 v1.0.0 — Production Release Gate (Completed)
**Goal:** Stable, hardened 1.0 release ready for ecosystem-wide adoption.

- [x] **API Stability Freeze**:
  - Finalized all public contracts across `rune.core`, `rune.render`, `rune.progress`, `rune.prompt`, `rune.cli`.
  - Full backward-compatibility and migration contracts documented.
- [x] **Definition of Done Verification (`AGENTS.md §22`)**:
  - [x] Valid statically typed Nox with zero invented APIs.
  - [x] 100% green test suite (17 passed, 0 failed) with zero skipped tests.
  - [x] ANSI injection sanitization proven across all untrusted inputs.
  - [x] Deterministic exit codes (0, 1, 2) and cleanup safety.
  - [x] Sub-millisecond to low single-digit millisecond performance benchmark SLA.
  - [x] Comprehensive documentation, feature matrix, API reference, and compiling examples.

---

### 🛡️ v1.0.1 — Security, Unicode & Prompt Hardening (Completed)
**Goal:** Address audit findings regarding ANSI sanitization, Unicode cell widths, and non-interactive consistency.

- [x] **P0 — ANSI Sanitization Security**:
  - Strip standalone `\r` (carriage return) to prevent terminal line overwrite spoofing attacks; normalize `\r\n` to `\n`.
  - Strip C1 control codes (code points 128 to 159) to prevent 8-bit terminal control injections.
  - Migrate parser from byte indexing to Unicode `ord(s[i])` scalars.
- [x] **P1 — Unicode Display Cell Width (wcwidth)**:
  - Accurate width calculation for CJK wide characters (`界` = 2, `你` = 2) and emojis (`🙂` = 2, `🦀` = 2).
  - Combining marks (U+0300..U+036F), zero-width spaces, and variation selectors return 0.
  - `truncate()` and `wrap_text()` updated to operate strictly on display cell widths.
- [x] **P1 — Non-Interactive Prompt Consistency**:
  - `prompt_int()`, `select_choice()`, and `select_multiple()` consistently raise `NonInteractiveError` when required inputs lack defaults in CI / non-TTY mode.
- [x] **P1 — Color Code Validation**:
  - `color_rgb()`, `bg_rgb()`, `color_256()`, `bg_256()` clamp values to `0..255`.
  - `hex_to_rgb()` strictly validates all 6 hexadecimal characters; rejects invalid strings.
  - `resolve_color_code()` validates semicolon SGR parameters to prevent parameter injection.
- [x] **P1 — Progress Boundaries**:
  - `Task.update()` and `Task.advance()` clamp completed values to `>= 0`.
  - `Task.percentage()` clamped between 0 and 100.
- [x] **P2 — Documentation Consistency**:
  - Fully synchronized status dashboard at the top of `ROADMAP.md`.
  - `tests/unit/security_hardening_test.nox` added (18/18 test suites passing).

---

### 🔮 Post-1.0: Full-Screen TUI Framework (`rune.tui`)
**Status:** Approved for Post-1.0 Development.
**Architectural Decision:** Implemented as first-class module `rune.tui` (depending strictly on `rune.core` and `rune.render`).

---

### 🖥️ v1.1.0 — Phase 7: Terminal Screen Driver & Virtual Buffer (Completed)
**Goal:** Deliver a flicker-free, double-buffered screen rendering engine and alternate screen manager in pure Nox.

- [x] **Virtual Screen Buffer (`ScreenBuffer` / `CellGrid`)**:
  - 2D grid of character cells: `Cell(char: str, style: Style, width: int)`.
  - Double buffering: maintain `current_buffer` and `previous_buffer` to eliminate redraw flicker.
  - Diff-based painter: compute only mutated cells and emit localized cursor jump sequences (`\x1b[{r};{c}H`), minimizing ANSI bandwidth.
- [x] **Alternate Screen Buffer & Cursor Control (`TerminalDriver`)**:
  - `enter_alt_screen()`: switch to alternate screen buffer (`\x1b[?1049h`).
  - `leave_alt_screen()`: restore user's original shell buffer (`\x1b[?1049l`).
  - `hide_cursor()` (`\x1b[?25l`) and `show_cursor()` (`\x1b[?25h`).
  - `clear_screen()` (`\x1b[2J\x1b[H`) and `clear_line()` (`\x1b[2K`).
- [x] **Terminal Restoration Safety**:
  - Guaranteed restoration cleanup: ensure `leave_alt_screen()` and `show_cursor()` execute reliably on unhandled exceptions or normal exit.
- [x] **Unit & Mock Tests**:
  - In-memory `MockTerminalDriver` capturing escape sequence streams.
  - Diff-calculation unit tests verifying minimal ANSI emission (`tests/unit/tui_buffer_test.nox`).
  - Example in `examples/tui_buffer_demo.nox`.

---

### 📐 v1.2.0 — Phase 8: Geometry & Flex Layout Engine (Completed)
**Goal:** Provide a flexible bounding-box and constraint-based layout engine inspired by Ratatui / CSS Flexbox.

- [x] **Rectangular Geometry (`Rect`)**:
  - `Rect(x: int, y: int, width: int, height: int)`.
  - Intersection, containment, padding, and inner margin helpers (`rect.inner(margin)`, `rect.inner_xy(mx, my)`).
  - Area, empty checks, and bounding union calculations.
- [x] **Layout Constraints (`Constraint`)**:
  - `length_constraint(n)`: fixed cell dimension.
  - `percentage_constraint(p)`: percentage of parent container (0..100%).
  - `ratio_constraint(num, den)`: exact fractional width/height.
  - `min_constraint(n)` / `max_constraint(n)`: bounded dynamic dimensions.
- [x] **Flex Layout Solver (`Layout`)**:
  - Direction support: `DIRECTION_HORIZONTAL` and `DIRECTION_VERTICAL`.
  - Margin & spacing distribution between split chunks (`set_margin()`, `set_spacing()`).
  - Deterministic integer coordinate rounding without pixel/cell gaps.
  - Comprehensive unit tests covering 2-column, 3-row, and nested grid layouts (`tests/unit/tui_layout_test.nox`).
  - Example in `examples/tui_layout_demo.nox`.

---

### 🧩 v1.3.0 — Phase 9: Declarative TUI Widgets (Completed)
**Goal:** Introduce a composable widget library for building dashboards, monitors, and navigation panels.

- [x] **Container Widgets (`Block`)**:
  - Frame container with borders (`rounded`, `square`, `double`, `ascii`, `none`).
  - Configurable title, title alignment (`left`, `center`, `right`), and border styling.
  - Automatic inner area calculation via `block.inner(area)` for nested child rendering.
- [x] **Text & Content Widgets**:
  - `Paragraph`: multiline styled text with word wrapping, alignment (`left`, `center`, `right`), and vertical scroll offset (`scroll: int`).
  - `Gauge`: visual percentage bar with fill characters (`"█"`), track character (`"─"`), and centered label.
  - `Tabs`: top navigation bar with active tab highlight (`[Tab]`) and configurable dividers (`"│"`).
- [x] **Data & Selection Widgets**:
  - `ListWidget`: vertical item list with selectable active row highlight (`selected_index: int`, `select_next()`, `select_prev()`).
  - `TableWidget`: fixed column headers, scrollable rows, and selected row indicator.
  - `HelpBar`: bottom footer displaying active keybindings (e.g. `[q] Quit  [Tab] Switch  [↑/↓] Navigate`).
- [x] **Buffer Plain-Text Inspection (`ScreenBuffer.to_text`)**:
  - Convert 2D buffer into newline-delimited plain text string for snapshot tests and terminal fallbacks.
- [x] **Comprehensive Test Suite**:
  - Added `tests/unit/tui_widgets_test.nox` (21 suites total).
- [x] **Comprehensive Example**:
  - Added `examples/tui_widgets_demo.nox` demonstrating an 80x24 multi-pane terminal dashboard.

---

### 🎮 v1.4.0 — Phase 10: Event Loop & Interactive App Runner (Completed)
**Goal:** Deliver an event-driven application runtime (Elm Architecture / Bubbletea style) for full-screen Nox applications.

- [x] **ANSI Key & Escape Sequence Parser (`KeyDecoder`)**:
  - Parse VT escape sequences: Arrow keys (Up, Down, Left, Right), Home, End, PageUp, PageDown, Delete, Backspace, Enter, Tab, Esc.
  - Detect Ctrl combinations (`Ctrl+C`, `Ctrl+D`, `Ctrl+A`..`Ctrl+Z`), Alt combinations (`Alt+Key`), Shift modifications (`Shift+Tab`, `Shift+Up`).
- [x] **Event Model (`Event`)**:
  - `KeyEvent(key: str, ctrl: bool, alt: bool, shift: bool)`.
  - `TickEvent(delta_ms: int)` for periodic animations and telemetry refresh.
  - `ResizeEvent(width: int, height: int)` for terminal size adjustments.
  - Helper functions: `make_key_event()`, `make_tick_event()`, `make_resize_event()`.
- [x] **TUI Application Runtime (`TuiApp`, `Frame`, `AppRunner`)**:
  - Lifecycle methods: `init()`, `update(event) -> bool`, `view(frame: Frame)`.
  - Elm/Bubbletea state transition architecture.
  - `run_event_stream()` for deterministic testing and automated headless execution.
  - `run_interactive()` with alt-screen management and guaranteed `restore()` cleanup.
- [x] **Complete TUI Showcase App**:
  - Interactive system monitor / process dashboard example in `examples/tui_dashboard.nox`.
- [x] **Comprehensive Test Suite**:
  - Added `tests/unit/tui_app_test.nox` (22 suites total).

---

## 4. Release Protocol & Quality Gates

For every minor or patch release:
1. **Develop Vertical Slice:** Implement feature strictly within its designated module.
2. **Add Unit & Integration Tests:** Never commit code without accompanying `*_test.nox` coverage.
3. **Run Verification Tooling:**
   ```bash
   noxc fmt <file.nox>
   noxc check <file.nox>
   noxc test
   ```
4. **Update Documentation:** Sync `README.md`, `docs/api.md`, `CHANGELOG.md`, and this `ROADMAP.md`.
5. **Publish:** Push to GitHub and submit metadata using `noxc publish https://github.com/mburakmmm/rune`.
