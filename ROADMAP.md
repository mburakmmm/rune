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
| **Phase 3.5: Advanced CLI** | `v0.3.0` | Choice/enum options, env fallbacks, repeated options, aliases | **Next Up** | Pending |
| **Phase 2.5: Live Progress** | `v0.4.0` | Rate throttling, multi-task live layout, ETA/speed math | **Planned** | Pending |
| **Phase 4.5: Advanced Prompts** | `v0.5.0` | Multi-select, input validators, retry limits | **Planned** | Pending |
| **Phase 5: Terminal Markdown & DX** | `v0.6.0` | Basic terminal Markdown parser, syntax highlighter | **Planned** | Pending |
| **Phase 5.5: Shell Completion** | `v0.7.0` | Bash, Zsh, Fish autocompletion script generation | **Planned** | Pending |
| **Phase 5.8: Benchmarks & Perf** | `v0.8.0` | Allocation audits, 1000-row table & 1000-arg CLI benchmarks | **Planned** | Pending |
| **Phase 5.9: Cross-Platform** | `v0.9.0` | Windows VT console compatibility & snapshot tests | **Planned** | Pending |
| **Phase 6: Rune 1.0 Release** | `v1.0.0` | Public API freeze, complete documentation, release gates | **Target** | Full Suite |
| **Post-1.0: TUI Evaluation** | `v1.1+` / Sub | Alternate screen buffers, raw keyboard mode, event loop | **Deferred** | Design Review |

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

### 📦 v0.3.0 — Advanced CLI Engine & Developer Ergonomics
**Goal:** Expand CLI features to match Python Click/Typer versatility.

- [ ] **Choice / Enum Option Validation**:
  - Support `choices: list[str]` on options and positional arguments.
  - Automatically validate and generate readable errors: `Error: Invalid value for '--mode': choose from 'fast', 'slow'`.
  - List allowed choices in `--help` output.
- [ ] **Environment Variable Sourced Defaults**:
  - Support `Option.env_var: str` (e.g. `--token` defaults to `API_TOKEN` env var).
  - Precedence hierarchy: `CLI argument > Environment variable > Declared default`.
  - Safe exception handling with `get_env_safe` (handling `OsError`).
- [ ] **Grouped Short Flags**:
  - Support clustered boolean flags: `-xvf` expands to `-x`, `-v`, `-f`.
- [ ] **Repeated / Multiple Options**:
  - Support accumulating repeated options (e.g. `--include dir1 --include dir2` -> `list[str]`).
- [ ] **Command & Option Aliases**:
  - Support command aliases (e.g. `co` for `commit`, `i` for `install`).
- [ ] **Secret Redaction in Diagnostics**:
  - Automatically mask options flagged with `is_secret = True` in help and error outputs.

---

### 📦 v0.4.0 — Live Display & Progress Hardening
**Goal:** Production-grade progress display with rate limiting and duration estimation.

- [ ] **Update Rate Limiter**:
  - Throttle terminal redraws to a configurable interval (e.g. max 10 updates/second) to avoid terminal I/O flooding.
- [ ] **Monotonic ETA & Transfer Speed Math**:
  - Calculate transfer rates (e.g. `MB/s`, `items/s`) using `nox.time.instant_now()`.
  - Formatted elapsed duration and estimated time remaining (`00:04:12`).
  - Handle `total = 0` or indeterminate tasks safely with moving-average rate calculation.
- [ ] **Multi-Task Live Progress Display**:
  - Render multiple simultaneous concurrent tasks in a consolidated live block.
- [ ] **Transient vs Persistent Logs**:
  - Allow completed tasks to print a clean final summary line while erasing animation frames.

---

### 📦 v0.5.0 — Advanced Interactive Prompts
**Goal:** Rich, validated user interaction for interactive CLI wizards.

- [ ] **Custom Validators & Retry Loops**:
  - Support validator functions `validate(input: str) -> bool`.
  - Configurable maximum retries before raising `PromptError`.
- [ ] **Multi-Select Prompt (`select_multiple`)**:
  - Allow selecting zero, one, or multiple items from a list using comma-separated numbers or keys.
- [ ] **Runtime-Gated Password Prompt**:
  - When Nox stdlib introduces terminal echo suppression, integrate secure password entry without breaking backward compatibility.

---

### 📦 v0.6.0 — Terminal Markdown & Syntax Highlighting
**Goal:** Rich readable documentation and code snippets inside the terminal.

- [ ] **Terminal Markdown Subset**:
  - Headings (`# H1`, `## H2`, `### H3`) rendered with distinctive styles and underlines.
  - Bulleted (`*`, `-`) and numbered (`1.`) lists with indentation.
  - Blockquotes (`> text`) with left border guides.
  - Inline code (`` `code` ``) and code blocks (```` ```nox ````).
- [ ] **Lightweight Nox Code Highlighter**:
  - Highlight keywords (`def`, `class`, `import`, `return`, `if`, `else`).
  - Highlight string literals, numeric literals, and comments (`#`).
  - Pure Nox token-scanning implementation without heavy compiler dependencies.

---

### 📦 v0.7.0 — Shell Autocompletion
**Goal:** Fast, native shell tab-completion for Rune CLIs.

- [ ] **Completion Script Generator**:
  - Generate completion scripts for **Bash**, **Zsh**, and **Fish**.
  - Extract subcommands, options, and choices statically from `CliApp` definition.
- [ ] **Safe Shell Escaping**:
  - Escape all generated completions to eliminate shell injection vulnerabilities.
- [ ] **CLI Integration**:
  - Add built-in command `--install-completion [bash|zsh|fish]`.

---

### 📦 v0.8.0 — Performance Hardening & Benchmarking
**Goal:** Quantifiable performance benchmarks and zero regression guarantees.

- [ ] **Benchmark Suite (`benchmarks/`)**:
  - Measure pure text rendering, table layout (10, 100, 1000 rows), and CLI argument parsing (10, 100, 1000 args).
  - Track peak memory usage and compilation overhead.
- [ ] **String Concatenation Optimization**:
  - Refactor hotspot string builder loops to minimize intermediate allocations.
- [ ] **Allocation Bounds**:
  - Enforce bounds on table dimensions and recursion depth to prevent denial-of-service via huge inputs.

---

### 📦 v0.9.0 — Cross-Platform & Snapshot Parity
**Goal:** Verified behavior across macOS, Linux, and Windows.

- [ ] **Windows VT Mode Audit**:
  - Test ANSI sequence interpretation across Windows Terminal, Command Prompt, and PowerShell.
- [ ] **Snapshot Testing Harness**:
  - Separate plain-text golden snapshots from ANSI golden snapshots.
  - Golden output tests for tables, panels, and `--help` pages.
- [ ] **Monochrome & Screen Reader Review**:
  - Ensure all interfaces convey complete information without relying on color alone.

---

### 🚀 v1.0.0 — Production Release Gate
**Goal:** Stable, hardened 1.0 release ready for ecosystem-wide adoption.

- [ ] **API Stability Freeze**:
  - Finalize all signatures in `rune.core`, `rune.render`, `rune.progress`, `rune.prompt`, `rune.cli`.
  - Provide documented migration guarantees.
- [ ] **Definition of Done Verification (`AGENTS.md §22`)**:
  - [ ] Valid statically typed Nox with zero invented APIs.
  - [ ] 100% green test suite across all supported toolchains.
  - [ ] ANSI injection sanitization proven across all untrusted inputs.
  - [ ] Deterministic exit codes and cleanup.
  - [ ] Comprehensive documentation and compiling examples.
  - [ ] User authorization prior to tagging and publishing.

---

### 🔮 Post-1.0: Full-Screen TUI Framework
**Status:** Deliberate Non-Goal for 1.0 (`AGENTS.md §13`).

- [ ] Formal architecture review for interactive full-screen applications.
- [ ] Evaluate alternate screen buffers (`\x1b[?1049h`), raw keyboard input (`termios`), and resize signals (`SIGWINCH`).
- [ ] Decide between `rune.tui` module vs a dedicated standalone package (e.g. `rune-tui`).

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
