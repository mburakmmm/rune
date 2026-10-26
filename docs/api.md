# Rune API Reference

## 1. `rune.core`

### `ConsoleOptions`
Represents terminal capabilities and layout preferences.
```nox
class ConsoleOptions:
    supports_color: bool
    supports_unicode: bool
    width: int
    height: int
    is_tty: bool
    capture: bool
```

### `Console`
Output sink managing terminal rendering and capturing.
```nox
class Console:
    options: ConsoleOptions
    buffer: str

    def print(self: Console, text: str = "") -> None: ...
    def print_styled(self: Console, text: str, style: Style) -> None: ...
    def write(self: Console, text: str) -> None: ...
    def get_captured(self: Console) -> str: ...
    def clear_captured(self: Console) -> None: ...
```

### `Style`
Text styling model.
```nox
class Style:
    fg: str
    bg: str
    bold: bool
    dim: bool
    italic: bool
    underline: bool
    reverse: bool

    def is_plain(self: Style) -> bool: ...
    def to_ansi(self: Style) -> str: ...
    def apply(self: Style, text: str) -> str: ...
```

### Utilities
```nox
def strip_ansi(s: str) -> str: ...
def sanitize_ansi(s: str) -> str: ...
def cell_width(s: str) -> int: ...
def truncate(s: str, max_width: int, ellipsis: str = "...") -> str: ...
def wrap_text(s: str, max_width: int) -> list[str]: ...
def color_256(n: int) -> str: ...
def bg_256(n: int) -> str: ...
def color_rgb(r: int, g: int, b: int) -> str: ...
def bg_rgb(r: int, g: int, b: int) -> str: ...
def hex_to_rgb(hex_str: str) -> str: ...
def hex_to_bg_rgb(hex_str: str) -> str: ...
```

---

## 2. `rune.text`

### `Span`
```nox
class Span:
    content: str
    style: Style
```

### `Text`
Fluent inline styled string container.
```nox
class Text:
    spans: list[Span]

    def append(self: Text, content: str, style: Style | None = None) -> Text: ...
    def plain_text(self: Text) -> str: ...
    def cell_width(self: Text) -> int: ...
    def render(self: Text, options: ConsoleOptions) -> str: ...
```

---

## 2. `rune.render`

### `Rule`
```nox
class Rule:
    title: str
    char: str
    align: str
    style: Style

    def render(self: Rule, options: ConsoleOptions) -> str: ...
```

### `Panel`
```nox
class Panel:
    content: str
    title: str
    subtitle: str
    border_style: str
    padding: int
    width: int
    style: Style

    def render(self: Panel, options: ConsoleOptions) -> str: ...
```

### `Table`
```nox
class Table:
    title: str
    columns: list[Column]
    rows: list[Row]
    border_style: str
    header_style: Style
    padding: int

    def add_column(self: Table, title: str, align: str = "left", style: Style | None = None) -> None: ...
    def add_row(self: Table, cells: list[str]) -> None: ...
    def render(self: Table, options: ConsoleOptions) -> str: ...
```

### `Tree`
```nox
class Tree:
    root: TreeNode
    guide_style: Style

    def add(self: Tree, label: str, style: Style | None = None) -> TreeNode: ...
    def render(self: Tree, options: ConsoleOptions) -> str: ...
```

---

## 3. `rune.progress`

### `Task` & `ProgressBar`
```nox
def format_duration(seconds: int) -> str: ...
def format_bytes(b: int) -> str: ...

class Task:
    name: str
    total: int
    completed: int
    finished: bool
    unit: str
    start_time_ms: int

    def advance(self: Task, step: int = 1) -> None: ...
    def update(self: Task, completed: int) -> None: ...
    def percentage(self: Task) -> int: ...
    def is_finished(self: Task) -> bool: ...
    def elapsed_ms(self: Task) -> int: ...
    def elapsed_seconds(self: Task) -> int: ...
    def speed(self: Task) -> float: ...
    def eta_seconds(self: Task) -> int: ...
    def format_eta(self: Task) -> str: ...

class ProgressBar:
    width: int
    bar_char: str
    empty_char: str
    min_interval_ms: int

    def add_task(self: ProgressBar, name: str, total: int = 100, unit: str = "items") -> Task: ...
    def should_render(self: ProgressBar) -> bool: ...
    def render_task(self: ProgressBar, task: Task, options: ConsoleOptions, show_eta: bool = True) -> str: ...
    def render_all(self: ProgressBar, options: ConsoleOptions, show_eta: bool = True) -> str: ...
```

### `Spinner`
```nox
class Spinner:
    message: str

    def next_frame(self: Spinner, supports_unicode: bool = True) -> str: ...
    def render(self: Spinner, options: ConsoleOptions) -> str: ...
```

---

## 4. `rune.prompt`

```nox
def prompt_text(question: str, default_val: str = "", console: Console | None = None) -> str: ...
def confirm(question: str, default_val: bool = True, console: Console | None = None) -> bool: ...
def select_choice(question: str, choices: list[str], default_val: str = "", console: Console | None = None) -> str: ...
def prompt_password(question: str) -> str: ... # Raises PromptError
```

---

## 6. `rune.cli`

```nox
class Argument:
    name: str
    arg_type: int
    required: bool
    default_val: str
    help: str
    variadic: bool
    choices: list[str]

class Option:
    name: str
    short_name: str
    opt_type: int
    is_flag: bool
    required: bool
    default_val: str
    help: str
    choices: list[str]
    env_var: str
    multiple: bool
    is_secret: bool

class Command:
    name: str
    description: str
    aliases: list[str]

    def add_argument(self: Command, name: str, arg_type: int = 1, required: bool = True, default_val: str = "", help: str = "", variadic: bool = False, choices: list[str] | None = None) -> Argument: ...
    def add_option(self: Command, name: str, short_name: str = "", opt_type: int = 1, is_flag: bool = False, required: bool = False, default_val: str = "", help: str = "", choices: list[str] | None = None, env_var: str = "", multiple: bool = False, is_secret: bool = False) -> Option: ...
    def add_alias(self: Command, alias: str) -> None: ...
    def add_subcommand(self: Command, sub: Command) -> None: ...
    def format_help(self: Command, console: Console) -> str: ...

class CliContext:
    def get_str(self: CliContext, key: str, fallback: str = "") -> str: ...
    def get_int(self: CliContext, key: str, fallback: int = 0) -> int: ...
    def get_float(self: CliContext, key: str, fallback: float = 0.0) -> float: ...
    def get_bool(self: CliContext, key: str, fallback: bool = False) -> bool: ...
    def get_list(self: CliContext, key: str) -> list[str]: ...

class CliApp:
    root: Command
    version: str
    console: Console

    def parse(self: CliApp, raw_args: list[str]) -> CliResult: ...
    def run(self: CliApp, argv: list[str] | None = None) -> int: ...
```
