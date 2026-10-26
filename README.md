# Rune

> The unified terminal presentation and command-line application toolkit for the **Nox programming language**.

Rune combines the capabilities of Python's **Rich**, **Click**, and **Typer** into a cohesive, statically-typed, dependency-free library designed specifically for Nox.

---

## Highlights

- **Nox-Native & Dependency-Free**: Pure Nox code with no external C dependencies or unsafe compiler hacks.
- **Terminal Presentation (Rich)**:
  - ANSI color & style engine with automatic `NO_COLOR` / CI / dumb terminal detection.
  - Safe ANSI sanitization protecting against terminal escape injection.
  - Structured visual components: **Rule**, **Panel**, **Table**, **Tree**.
  - Deterministic **ProgressBar** and non-TTY compatible **Spinner**.
- **CLI Framework (Click + Typer)**:
  - Declarative arguments and options with typed conversions (`str`, `int`, `float`, `bool`).
  - Short and long options (`-o`, `--output`, `--key=val`).
  - Nested subcommands (`app git commit`).
  - Automatic, beautifully styled `--help` synopsis and standard exit codes (0 for success, 2 for usage errors).
- **Interactive Prompts**: Line input with defaults, `confirm` boolean prompts, and single-choice selection with non-interactive CI safety.

---

## Minimum Compiler Requirement

- **Nox Compiler**: `noxc >= 2.0.27`

---

## Installation

Add Rune to your project's `nox.json`:

```json
{
  "requires": [
    { "package": "rune" }
  ]
}
```

Or install using the Nox package manager:

```bash
noxc add rune
```

---

## Quick Start

### 1. Colored Text & Styles

```nox
import rune
from rune import Console, Style

console: Console = Console()
console.print_styled("Hello, Nox World!", Style("green", "", True))
```

### 2. Formatted Tables

```nox
import rune
from rune import Console, Table

console: Console = Console()

tbl: Table = Table("Server Status", "rounded")
tbl.add_column("Host", "left")
tbl.add_column("Port", "right")
tbl.add_column("State", "center")

tbl.add_row(["localhost", "8080", "RUNNING"])
tbl.add_row(["db.internal", "5432", "IDLE"])

console.print(tbl.render(console.options))
```

### 3. CLI Command & Automated Help

```nox
import rune
from rune import Command, CliApp, TYPE_STR, TYPE_INT, TYPE_BOOL

root: Command = Command("dock", "A lightweight container tool")
root.add_argument("image", TYPE_STR, True, "", "Image to run")
root.add_option("port", "p", TYPE_INT, False, False, "8080", "Port mapping")
root.add_option("detach", "d", TYPE_BOOL, True, False, "false", "Run in background")

app: CliApp = CliApp(root, "1.0.0")
app.run()
```

Output with `--help`:

```text
Usage: dock [OPTIONS] <image>

A lightweight container tool

Arguments:
  <image>                      Image to run (required)

Options:
  -p, --port <int>             Port mapping [default: 8080]
  -d, --detach                 Run in background
  -h, --help                   Show this help message and exit
```

### 4. Full-Screen TUI Double-Buffering

```nox
import rune
from rune import TerminalDriver, Style

driver: TerminalDriver = TerminalDriver()
driver.enter_alt_screen()
driver.hide_cursor()

# Draw to virtual buffer
driver.draw_string(2, 2, "Welcome to Rune TUI!", Style("green", "", True))
driver.fill_rect(2, 4, 20, 1, "─", Style("cyan"))

# Flush diff to screen (flicker-free!)
driver.flush()

# Cleanup on exit
driver.restore()
```
Usage: dock [OPTIONS] <image>

  A lightweight container tool

Arguments:
  image                   Image to run

Options:
  -h, --help              Show this message and exit
  -p, --port <value>      Port mapping [default: 8080]
  -d, --detach            Run in background [default: false]
```

---

## Feature Support Matrix

| Category | Feature | Status | Notes |
| :--- | :--- | :--- | :--- |
| **Styling** | ANSI 16 Colors (fg/bg) | **Supported** | Automatic reset prevention |
| **Styling** | Truecolor 24-bit RGB & ANSI 256 | **Supported** | Hex `#RRGGBB`, RGB tuples, 256 palette |
| **Styling** | Bold, Dim, Italic, Underline | **Supported** | Standards-compliant SGR codes |
| **Styling** | `NO_COLOR` / `CLICOLOR_FORCE` | **Supported** | Strictly respects CLI conventions |
| **Styling** | Windows VT Mode / Terminal Detection | **Supported** | `WT_SESSION`, `ANSICON`, `ConEmuANSI` |
| **Styling** | ANSI Escape Sanitization | **Supported** | Protects against OSC/CSI injection |
| **Render** | Rule Dividers | **Supported** | Centered / Left / Right titles |
| **Render** | Box Panels | **Supported** | Rounded, Square, Double, ASCII borders |
| **Render** | Structured Tables | **Supported** | Dynamic width calculation, multiline wrapping |
| **Render** | Hierarchical Trees | **Supported** | Iterative stack rendering (no recursion limit) |
| **Render** | Terminal Markdown | **Supported** | Headings, lists, quotes, rules, inline styling |
| **Render** | Syntax Highlighting | **Supported** | Pure Nox token lexer for code blocks & snippets |
| **Progress** | Multi-Task Live Progress | **Supported** | Concurrent task layout (`render_all`) |
| **Progress** | Monotonic ETA & Speed | **Supported** | Monotonic clock rate, `format_eta`, `format_bytes` |
| **Progress** | Spinners | **Supported** | Plain text status in CI/non-TTY |
| **Prompts** | Line Input with Defaults | **Supported** | Line-buffered via `input()` |
| **Prompts** | Validated Integer Input | **Supported** | `prompt_int` with min/max range & retry loop |
| **Prompts** | Multi-Select Choices | **Supported** | `select_multiple` with numbers, names, or `all` |
| **Prompts** | Confirm [Y/n] | **Supported** | Safe boolean parser |
| **Prompts** | Choice Selection | **Supported** | 1-indexed selection or name matching |
| **Prompts** | Password Masking | **Unsupported** | Requires runtime raw terminal/no-echo mode |
| **CLI** | Positional Arguments | **Supported** | Required, optional, typed conversions |
| **CLI** | Long & Short Options | **Supported** | `--key val`, `--key=val`, `-k val`, flags |
| **CLI** | Choices & Enum Validation | **Supported** | `choices: list[str]` with diagnostic errors |
| **CLI** | Environment Variable Fallback | **Supported** | `CLI arg > Env var > Declared default` |
| **CLI** | Clustered Short Flags | **Supported** | `-xvf` automatically expands to `-x`, `-v`, `-f` |
| **CLI** | Repeated / Multiple Options | **Supported** | Accumulate values via `ctx.get_list()` |
| **CLI** | Secret Redaction | **Supported** | `is_secret` masks default values in help/logs |
| **CLI** | Command Aliases | **Supported** | Aliases (`ci` for `commit`) displayed in help |
| **CLI** | Subcommands | **Supported** | Arbitrary nesting (`git commit`) |
| **CLI** | Shell Autocompletion | **Supported** | Native generator for Bash, Zsh, and Fish |
| **CLI** | Standard Exit Codes | **Supported** | 0 (success/help), 2 (usage error), 1 (failure) |
| **TUI** | Screen Buffer & Cells | **Supported** | 2D character matrix, wide char awareness, 2D clip stack |
| **TUI** | Double-Buffering Diff | **Supported** | Flicker-free minimal ANSI diff painter |
| **TUI** | Alternate Screen Buffer | **Supported** | `\x1b[?1049h` / `\x1b[?1049l` with cleanup |
| **TUI** | Cursor Control & Restoration | **Supported** | Hide/show, direct positioning, cleanup |
| **TUI** | Rect Geometry & Insets | **Supported** | 2D coordinates, area, inner margins, union, intersection |
| **TUI** | Flex Layout Solver | **Supported** | Length, Percentage, Ratio, Min, Max constraints |
| **TUI** | Declarative Widgets | **Supported** | Block, Paragraph, Gauge, Tabs, ListWidget, TableWidget, HelpBar |
| **TUI** | ANSI Key Sequence Decoder | **Supported** | Arrow keys, VT/xterm function keys, Ctrl/Alt/Shift modifiers |
| **TUI** | Event Model & Loop | **Supported** | `KeyEvent`, `TickEvent`, `ResizeEvent`, monotonic frame loop |
| **TUI** | Elm/Bubbletea Runtime | **Supported** | `TuiApp` lifecycle (`init`, `update`, `view`), `AppRunner` with event batching & escape delays |

---

## Nox Compiler & Runtime Integration

For a comprehensive systems-level specification and Zig runtime implementation plan to bridge Rune with live terminal raw mode, non-blocking input, and window resize events in the Nox compiler runtime, see [NOX_LIMITATIONS.md](NOX_LIMITATIONS.md).

## Security Model

- **ANSI Sanitization**: All untrusted terminal output can be sanitized using `sanitize_ansi()`, stripping terminal titles (OSC), cursor positioning (CSI), and clipboard manipulation sequences.
- **No Uncontrolled Exits**: `nox.os.exit()` is never invoked inside parsing or rendering layers; only the top-level application runner terminates the process.
- **Redaction Safety**: Prompts and error messages do not leak internal state or credentials.

---

## License

MIT License. See [LICENSE](LICENSE) for details.
