<p align="center">
  <h1 align="center">Rune</h1>
  <p align="center">
    <strong>The modern terminal toolkit for the Nox programming language</strong>
  </p>
  <p align="center">
    Rich-like terminal styling, 2D tables, flex layouts, Click/Typer-like CLI parsing, and full-screen declarative TUI applications.
  </p>
  <p align="center">
    <a href="https://github.com/mburakmmm/rune/actions/workflows/ci.yml"><img src="https://github.com/mburakmmm/rune/actions/workflows/ci.yml/badge.svg" alt="CI"></a>
    <a href="https://github.com/mburakmmm/rune/releases"><img src="https://img.shields.io/github/v/release/mburakmmm/rune?color=6f42c1&logo=github" alt="GitHub Release"></a>
    <a href="https://noxlang.com"><img src="https://img.shields.io/badge/noxc-%3E%3D%202.0.27-0366d6?logo=terminal" alt="Nox Version"></a>
    <a href="https://github.com/mburakmmm/rune/blob/main/LICENSE"><img src="https://img.shields.io/github/license/mburakmmm/rune?color=2ea44f" alt="License"></a>
    <a href="https://github.com/mburakmmm/rune#testing--verification"><img src="https://img.shields.io/badge/tests-22%20passed-success" alt="Tests"></a>
    <a href="https://github.com/mburakmmm/rune#security-model"><img src="https://img.shields.io/badge/dependencies-0%20native-blue" alt="Zero Dependencies"></a>
  </p>
</p>

---

```text
╭─────────────────────────── Rune System Monitor v1.4.5 ───────────────────────────╮
│  [Cluster: prod-us-east]     CPU: [████████████████████░░░░░░] 74%    Uptime: 14d │
├──────────────────────────────────────┬───────────────────────────────────────────┤
│ Active Microservices                 │ Real-Time Health & Storage Metrics        │
│ ╭──────────────┬──────┬────────────╮ │ ╭─ Memory Allocation (16 GB) ────────────╮│
│ │ Service      │ Port │ Status     │ │ │ Heap:   [██████████░░░░░░░░░░] 52%     ││
│ ├──────────────┼──────┼────────────┤ │ │ Cache:  [███████████████░░░░░] 78%     ││
│ │ api-gateway  │ 8080 │ ● RUNNING  │ │ ╰────────────────────────────────────────╯│
│ │ auth-service │ 8081 │ ● HEALTHY  │ │ ╭─ Event Stream ─────────────────────────╮│
│ │ worker-queue │ 5432 │ ○ IDLE     │ │ │ 12:04:18 [INFO] Cluster sync complete  ││
│ │ ingest-node  │ 9092 │ ● ACTIVE   │ │ │ 12:04:19 [WARN] Disk I/O spike: sda1   ││
│ ╰──────────────┴──────┴────────────╯ │ ╰────────────────────────────────────────╯│
╰─────────────────────────────────────────────────────── [q] Quit  [r] Refresh ───╯
```

---

## Overview

**Rune** unites the capabilities of Python's **Rich**, **Click**, and Rust's **Ratatui** into a unified, statically-typed, zero-dependency library engineered specifically for the [Nox programming language](https://noxlang.com).

Whether you are writing a simple CLI utility with nested subcommands, rendering structured diagnostic tables, or building full-screen interactive dashboards, Rune provides predictable defaults, rock-solid Unicode handling, and graceful plain-text fallback.

---

## Key Highlights

- **100% Pure Nox**: Zero external C dependencies, no compiler forks, and no unsafe memory shims.
- **Terminal Presentation (Rich-like)**:
  - ANSI 16, 256-color, and 24-bit Truecolor (`#RRGGBB`) styling engine.
  - Automatic environment detection (`NO_COLOR`, `CLICOLOR_FORCE`, `TERM=dumb`, CI, Windows VT mode).
  - Robust ANSI sanitization protecting against OSC/CSI escape injections and screen spoofing.
  - Formatted 2D **Tables** with dynamic column width solvers and multiline cell wrapping.
  - Rounded, square, double, and ASCII **Panels** and unbounded **Trees**.
  - Multi-task **ProgressBar** and non-TTY compatible **Spinners**.
- **CLI Framework (Click/Typer-like)**:
  - Declarative commands, subcommands, and options with typed conversions (`str`, `int`, `float`, `bool`).
  - Short and long flags (`-v`, `-xvf`, `--output=file.json`), choices/enum validation, and secret masking.
  - Environment variable fallback (`CLI > Env > Default`).
  - Native autocompletion generators for **Bash**, **Zsh**, and **Fish**.
  - Automatic, beautifully wrapped `--help` synopsis with standard exit codes (0 for success, 2 for usage errors).
- **Interactive Prompts**: Line inputs with defaults, boolean confirmations (`[Y/n]`), and single/multiple selections with non-interactive CI safety.
- **Full-Screen TUI Engine (Ratatui-like)**:
  - 2D character matrix **ScreenBuffer** with mathematical Unicode display cell width (`wcwidth`) and CJK/Emoji continuation handling.
  - Double-buffered diff painter emitting minimal ANSI sequences for flicker-free rendering.
  - Flex-inspired **Layout** solver supporting Length, Percentage, Ratio, Min, and Max constraints.
  - Declarative widget suite: `Block`, `Paragraph`, `Gauge`, `Tabs`, `ListWidget`, `TableWidget`, and `HelpBar`.
  - Elm/Bubbletea-style interactive application runtime (`TuiApp`, `AppRunner`, `KeyEvent`, `TickEvent`).
  - Atomic cell transaction model guaranteeing zero border leaks and strict invariant validation.

---

## Minimum Compiler Requirement

- **Nox Compiler**: `noxc >= 2.0.27`
- **Supported Operating Systems**: Linux (`x86_64`, `arm64`), macOS (`Apple Silicon`, `Intel`), Windows 10/11 (`x64`)

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

Or install via the Nox package registry:

```bash
noxc add rune
```

---

## Quick Start

### 1. Formatted Tables & Terminal Styling

```nox
import rune
from rune import Console, Table, Style

console: Console = Console()

# Create a styled, rounded table
tbl: Table = Table("Production Services", "rounded")
tbl.add_column("Service Name", "left")
tbl.add_column("Port", "right")
tbl.add_column("Health Status", "center")

tbl.add_row(["api-gateway", "8080", "HEALTHY"])
tbl.add_row(["auth-service", "8081", "HEALTHY"])
tbl.add_row(["cache-redis", "6379", "IDLE"])

console.print(tbl.render(console.options))
```

### 2. Declarative CLI with Subcommands & Help

```nox
import rune
from rune import Command, CliApp, Context, TYPE_STR, TYPE_INT, TYPE_BOOL

def handle_deploy(ctx: Context) -> int:
    env: str = ctx.get_str("env")
    replicas: int = ctx.get_int("replicas")
    print("Deploying to " + env + " with " + str(replicas) + " replicas...")
    return 0

deploy: Command = Command("deploy", "Deploy application to target environment")
deploy.add_option("env", "e", TYPE_STR, False, False, "staging", "Target environment")
deploy.add_option("replicas", "r", TYPE_INT, False, False, "3", "Number of replicas")
deploy.set_handler(handle_deploy)

root: Command = Command("app", "Cloud application manager")
root.add_subcommand(deploy)

app: CliApp = CliApp(root, "1.4.5")
app.run()
```

Output with `--help`:

```text
Usage: app deploy [OPTIONS]

Deploy application to target environment

Options:
  -e, --env <value>           Target environment [default: staging]
  -r, --replicas <int>        Number of replicas [default: 3]
  -h, --help                  Show this help message and exit
```

### 3. Interactive Prompts & Progress Indicators

```nox
import rune
from rune import Console, ProgressBar, prompt_str, confirm

console: Console = Console()

# Interactive prompt with default value
username: str = prompt_str("Enter your username", "admin")

# Boolean confirmation
if confirm("Proceed with database migration?", True):
    bar: ProgressBar = ProgressBar("Migrating records")
    task_id: int = bar.add_task("Processing tables", 100)
    
    # Advance progress
    bar.advance(task_id, 50)
    bar.update(task_id, 100)
```

### 4. Full-Screen Declarative TUI Application

```nox
import rune
from rune import (
    TuiApp, AppRunner, Frame, Event, Block, Paragraph, Gauge,
    Layout, Rect, DIRECTION_VERTICAL, length_constraint, min_constraint,
    Style
)

class MonitorApp(TuiApp):
    cpu_percent: int

    def init(self) -> None:
        self.cpu_percent = 75

    def update(self, event: Event) -> bool:
        if event.is_key():
            if event.key.to_str() == "q" or event.key.to_str() == "ctrl+c":
                return False
        return True

    def view(self, frame: Frame) -> None:
        area: Rect = frame.size()
        chunks: list[Rect] = Layout(DIRECTION_VERTICAL, [
            length_constraint(3),
            min_constraint(5)
        ]).split(area)

        # Header Block
        header: Block = Block("rounded", " System Dashboard ", "left")
        header.render(frame.buffer, chunks[0])

        # CPU Usage Meter
        gauge: Gauge = Gauge(self.cpu_percent, "CPU Load: " + str(self.cpu_percent) + "%")
        gauge.render(frame.buffer, chunks[1].inner(1))

# Run interactive session
runner: AppRunner = AppRunner()
runner.run_interactive(MonitorApp())
```

---

## Architectural Modules

```text
rune
│
├── rune.core        # Terminal capabilities, styles, ANSI engine, display cell widths
│
├── rune.render      # Tables, panels, trees, rules, syntax highlighting, markdown
│   └── depends on rune.core only
│
├── rune.progress    # Multi-task progress bars, ETA/speed calculators, spinners
│   └── depends on rune.core
│
├── rune.prompt      # Validated user inputs, confirmations, single/multiple selections
│   └── depends on rune.core
│
├── rune.cli         # Arguments, options, subcommands, help formatting, completion
│   └── depends on rune.core + optional rune.render/prompt
│
└── rune.tui         # ScreenBuffer, diff painter, flex layout, widgets, event runner
    └── depends on rune.core
```

---

## Feature Support Matrix

| Category | Feature | Status | Notes |
| :--- | :--- | :--- | :--- |
| **Styling** | ANSI 16 Colors (fg/bg) | **Supported** | Automatic style reset prevention |
| **Styling** | Truecolor 24-bit RGB & ANSI 256 | **Supported** | Hex `#RRGGBB`, RGB tuples, 256 palette |
| **Styling** | Bold, Dim, Italic, Underline | **Supported** | Standards-compliant SGR escape codes |
| **Styling** | `NO_COLOR` / `CLICOLOR_FORCE` | **Supported** | Strictly respects CLI ecosystem standards |
| **Styling** | Windows VT Mode / Detection | **Supported** | `WT_SESSION`, `ANSICON`, `ConEmuANSI` |
| **Styling** | ANSI Escape Sanitization | **Supported** | Strips dangerous OSC, CSI, and C1 sequences |
| **Render** | Rule Dividers | **Supported** | Centered, Left, or Right aligned titles |
| **Render** | Box Panels | **Supported** | Rounded, Square, Double, and ASCII borders |
| **Render** | Structured Tables | **Supported** | Dynamic width calculation, multiline cell wrapping |
| **Render** | Hierarchical Trees | **Supported** | Iterative stack rendering (no recursion limits) |
| **Render** | Terminal Markdown | **Supported** | Headings, lists, blockquotes, code blocks |
| **Render** | Syntax Highlighting | **Supported** | Pure Nox token lexer with line gutters |
| **Progress** | Multi-Task Live Progress | **Supported** | Concurrent tasks layout with rate formatting |
| **Progress** | Monotonic ETA & Speed | **Supported** | Monotonic clock rate, `format_eta`, `format_bytes` |
| **Progress** | Spinners | **Supported** | Deterministic plain-text status in non-TTY/CI |
| **Prompts** | Line Input with Defaults | **Supported** | Standard line-buffered input |
| **Prompts** | Validated Integer Input | **Supported** | `prompt_int` with min/max bounds and retries |
| **Prompts** | Multi-Select Choices | **Supported** | `select_multiple` with numeric/token selection |
| **Prompts** | Confirm [Y/n] | **Supported** | Safe boolean parser with default overrides |
| **CLI** | Positional Arguments | **Supported** | Required, optional, typed conversions |
| **CLI** | Long & Short Options | **Supported** | `--key val`, `--key=val`, `-k val`, flags |
| **CLI** | Clustered Short Flags | **Supported** | `-xvf` automatically expands to `-x`, `-v`, `-f` |
| **CLI** | Repeated / Multiple Options | **Supported** | Accumulate values via `ctx.get_list()` |
| **CLI** | Environment Variable Fallback | **Supported** | `CLI arg > Env var > Declared default` |
| **CLI** | Secret Redaction | **Supported** | `is_secret` masks default values in help/logs |
| **CLI** | Shell Autocompletion | **Supported** | Native generator for Bash, Zsh, and Fish |
| **CLI** | Standard Exit Codes | **Supported** | 0 (success/help), 2 (usage error), 1 (failure) |
| **TUI** | Screen Buffer & Cells | **Supported** | 2D character matrix, wide char awareness, 2D clip stack |
| **TUI** | Double-Buffering Diff | **Supported** | Flicker-free minimal ANSI diff painter |
| **TUI** | Alternate Screen Buffer | **Supported** | `\x1b[?1049h` / `\x1b[?1049l` with guaranteed restore |
| **TUI** | Flex Layout Solver | **Supported** | Length, Percentage, Ratio, Min, Max constraints |
| **TUI** | Declarative Widgets | **Supported** | Block, Paragraph, Gauge, Tabs, ListWidget, TableWidget, HelpBar |
| **TUI** | Event Model & Key Decoder | **Supported** | `KeyEvent`, `TickEvent`, `ResizeEvent`, stateful VT decoder |
| **TUI** | Elm/Bubbletea Runtime | **Supported** | `TuiApp` lifecycle, `AppRunner` with event batching & escape delays |
| **TUI** | Atomic Cell Transactions | **Supported** | Mathematical wide-character integrity & invariant verification |

---

## Testing & Verification

Rune maintains a 100% green automated test suite across unit, integration, benchmark, and visual snapshot tests:

```bash
# Run all 22 test suites
noxc test

# Run performance benchmarks
noxc run benchmarks/run_benchmarks.nox
```

### Performance SLA

| Benchmark Component | Workload | Latency |
| :--- | :--- | :--- |
| **Table Layout & Render** | 1,000 rows × 3 columns | **4 ms** |
| **CLI Argument Parsing** | 1,000 arguments | **3 ms** |
| **Syntax Highlighting** | 800 lines with line numbers | **3 ms** |
| **Markdown Parsing** | 700 lines with code blocks | **1 ms** |
| **Tree Traversal** | 1,000 nodes | **< 1 ms** |

---

## Security Model

- **ANSI Sanitization**: All untrusted terminal output can be sanitized using `sanitize_ansi()`, stripping terminal titles (OSC), cursor positioning (CSI), and clipboard manipulation sequences.
- **No Uncontrolled Exits**: `nox.os.exit()` is never invoked inside parsing or rendering layers; only the top-level application runner terminates the process.
- **Redaction Safety**: Prompts and error messages do not leak internal state or credentials.
- **Vulnerability Reporting**: See [SECURITY.md](SECURITY.md) for details on our coordinated disclosure policy.

---

## Nox Compiler & Runtime Integration

For a comprehensive systems-level specification and Zig runtime implementation plan to bridge Rune with live terminal raw mode, non-blocking input, and window resize events in the Nox compiler runtime, see [NOX_LIMITATIONS.md](NOX_LIMITATIONS.md).

---

## Contributing

We welcome community contributions! Please review our [Contributing Guide](CONTRIBUTING.md) and architectural boundaries in [AGENTS.md](AGENTS.md) before submitting pull requests.

---

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.
