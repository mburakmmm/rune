# Architecture Specification — Rune

Rune is the unified terminal presentation and command-line application toolkit for the Nox programming language. It combines Rich-like terminal rendering with Click/Typer-like structured CLI development into a single, cohesive, dependency-free library.

---

## 1. Module Hierarchy & Dependencies

```text
Rune Architecture
│
├── rune.core        # Capabilities, Console, Style, Color, ANSI sanitization, display width
│
├── rune.render      # Visual components (Rule, Panel, Table, Tree)
│   └── depends on rune.core only
│
├── rune.progress    # Progress bars, task completion, monotonic rate calculation
│   └── depends on rune.core
│
├── rune.prompt      # Line input, boolean confirmation, choice selection
│   └── depends on rune.core
│
└── rune.cli         # Arguments, Options, Commands, Subcommands, Help generation, App runner
    └── depends on rune.core + optional rune.render / rune.prompt
```

### Dependency Invariants
- `rune.core` NEVER imports `render`, `progress`, `prompt`, or `cli`.
- `rune.render` NEVER imports `cli` or `prompt`.
- `rune.progress` NEVER imports `cli`.
- `rune.prompt` NEVER imports `cli`.
- `rune.cli` may consume `render` and `prompt`, but MUST function gracefully in pure plain-text mode.
- No cyclic dependencies permitted anywhere.

---

## 2. Core Components

### 2.1 `Console` & `ConsoleOptions`
- Represents the output destination and active rendering context.
- Can write directly to stdout or capture output in an internal buffer for snapshot testing (`capture: bool`).
- Enforces width wrapping, plain-text fallback, and style emission policies.

### 2.2 `Style` & `Color`
- Pure data classes modeling foreground color, background color, and text attributes (`bold`, `dim`, `italic`, `underline`, `reverse`).
- Translates to standard ANSI Select Graphic Rendition (SGR) sequences using ESC (`char_from_byte(27)`).
- Automatically resets styles to prevent color bleeding into outer terminals or logs.

### 2.3 ANSI Sanitization & Display Width
- `sanitize_ansi(s: str) -> str`: Strips control characters, OSC terminal title/clipboard commands, and CSI movements from untrusted text.
- `strip_ansi(s: str) -> str`: Removes all SGR styling sequences, leaving pure printable text for length and cell width calculation.
- `cell_width(s: str) -> int`: Measures visible cell length, excluding zero-width ANSI escape sequences.

---

## 3. Render Components

### 3.1 `Rule`
- Produces a visual horizontal divider across the terminal width.
- Supports centered, left-aligned, or right-aligned titles, customizable characters (`─`, `-`, `=`), and border styles.

### 3.2 `Panel`
- Wraps multiline text in an aligned box with borders (rounded, square, ascii, minimal).
- Supports title, subtitle, horizontal padding, and automatic width constraints.

### 3.3 `Table`
- Computes column widths dynamically based on headers, cell values, and maximum available terminal width.
- Supports left, center, and right alignments per column.
- Provides standard border styles (`ROUNDED`, `SQUARE`, `ASCII`, `SIMPLE`, `MINIMAL`, `NONE`).
- Handles empty tables and multiline rows gracefully.

### 3.4 `Tree`
- Renders hierarchical structures with branching guides (`├──`, `└──`, `│`).
- Includes ASCII fallback (`+--`, `\--`, `|  `) for low-capability terminals.
- Limits recursion depth to prevent infinite cycles.

---

## 4. CLI Framework

### 4.1 Specification vs Parsing Separation
- Commands are defined declaratively:
  - Positional arguments (`Argument`)
  - Optional flags and parameters (`Option`)
  - Nested subcommands (`Command.add_subcommand()`)
- The parser accepts a clean `list[str]` of arguments (pure and decoupled from process argv).
- A separate adapter binds `nox.os.arg_count()` / `nox.os.arg(i)`.

### 4.2 Error Handling & Exit Codes
- CLI parsing never calls `nox.os.exit()` inside the core parser.
- Structured errors (`CliError`, `UsageError`, `UnknownOptionError`, `MissingArgumentError`, `InvalidValueError`) return clean status codes:
  - `0`: Success, `--help`, or `--version`.
  - `2`: Command line usage / argument error.
  - `1`: Application execution error.
