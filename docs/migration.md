# Rune — Migration & Stability Contract

> **Current Version:** `2.0.0`

Rune follows strict Semantic Versioning (`MAJOR.MINOR.PATCH`).

---

## 1. Compatibility Policy (`1.x` -> `2.0.0`)

1. All `v1.0.x` – `v1.5.0` releases preserved strict backwards compatibility within the `1.x` line.
2. `v2.0.0` targets **Nox `>= 2.1.186`** and adopts strongly-typed enums, `@dataclass` value types, `@property` getters, variadic `*args`, RAII `with` / `__del__` resource blocks, and lazy `yield` iterators.
3. Compatibility constant aliases (`TYPE_STR..TYPE_BOOL`, `EVENT_KEY..EVENT_RESIZE`, `DIRECTION_VERTICAL`, `DIRECTION_HORIZONTAL`) are retained in `2.0.0` alongside the new enums, whereas integer constraint constants `CONSTRAINT_LENGTH..CONSTRAINT_MAX` are replaced by `ConstraintKind` ADTs and constructor helpers.

---

## 2. Migrating from `1.x` to `2.0.0`

### Minimum Compiler Version
- Upgrade `noxc` to **`>= 2.1.186`**.
- Package manifests (`nox.json`) specify `"source": "nox"`.

### Strongly-Typed Enums (`ConstraintKind`, `Direction`, `ArgType`, `EventKind`)
- **`layout.nox` Constraints (Breaking):**
  - Replace raw integer tags `CONSTRAINT_LENGTH` / `CONSTRAINT_PERCENTAGE` / `CONSTRAINT_RATIO` / `CONSTRAINT_MIN` / `CONSTRAINT_MAX` with `ConstraintKind` variants or keep using the helper constructors `length_constraint(n)`, `percentage_constraint(p)`, `ratio_constraint(num, den)`, `min_constraint(n)`, `max_constraint(n)`:
    ```nox
    # 1.x
    if c.kind == CONSTRAINT_LENGTH: ...
    # 2.0
    match c.spec:
        case ConstraintKind.Length(n):
            ...
        case ConstraintKind.Percentage(p):
            ...
    ```
  - Use `Direction.Vertical` / `Direction.Horizontal` (aliases `DIRECTION_VERTICAL`, `DIRECTION_HORIZONTAL` still work).
- **`arguments.nox` / `cli.nox` Types:**
  - Prefer `ArgType.Str`, `ArgType.Int`, `ArgType.Float`, `ArgType.Bool` (aliases `TYPE_STR..TYPE_BOOL` remain available).
  - Format readable type names with `arg_type_name(t: ArgType)`.
- **`events.nox` Event Types:**
  - Prefer `EventKind.Key`, `EventKind.Tick`, `EventKind.Resize` (aliases `EVENT_KEY..EVENT_RESIZE` remain available).

### `@dataclass` Value Types & `@property` Accessors
- `Size(width, height)` (`@dataclass @derive("copy", "order")`) and `Padding(top, right, bottom, left)` (`@dataclass @derive("copy")`) join `layout.nox`; use `Rect.inner_padding(pad)`.
- `TreeItem(label, depth, is_last, branch_prefix)` in `tree.nox` is a `@dataclass @derive("copy")`.
- New `@property` getters:
  - `ConsoleOptions.is_plain`
  - `Task.done`, `Task.ratio`
  - `Rect.right`, `Rect.bottom`, `Rect.bounds`, `Rect.empty`
  - `Constraint.is_fixed`
  - `Frame.bounds`, `Frame.is_clipped`
  - `TerminalSession.is_open`, `TerminalSession.is_raw`

### Context Managers (`with` / `__del__`)
- `TerminalSession` and `Live` implement `__enter__`, `__exit__`, and `__del__` safety cleanup:
  ```nox
  with TerminalSession(driver) as sess:
      ...

  with Live(console, True) as live:
      live.update("Working...")
  ```

### Variadic `*args` & `yield` Iterators
- `Console.print_items(*items: str)`, `Table.add_cells(*cells: str)`, and `Layout.split_constraints(area, *items: Constraint)`.
- Lazy generators:
  - `track(items: list[str], description, console, show_eta) -> Iterator[str]`
  - `track_range(total: int, description, console, show_eta) -> Iterator[int]`
  - `Tree.walk() -> Iterator[TreeNode]`
  - `ScreenBuffer.iter_cells() -> Iterator[Cell]`

### Runtime Changes Since `1.4.x` (`1.5.0` / `2.0.0`)
- **`Console.write()` & `TerminalDriver.write_raw()`**: Now backed by `nox.terminal.write_raw()` and `nox.terminal.flush()`. Set `capture=True` in tests to keep output in memory.
- **`prompt_password()`**: Uses real echo suppression via `nox.terminal.set_echo(False)` in `try ... finally` when stdin is a TTY; raises `PromptError` / `NonInteractiveError` cleanly on non-TTY or unsupported Windows consoles.
- **`detect_console_options()`**: Queries live `nox.terminal.get_capabilities()`, `get_size()`, and `is_tty(1)` while honoring `NO_COLOR`, `CLICOLOR_FORCE`, `TERM=dumb`, and `CI`.

---

## 3. Migrating from `0.x` to `1.0.x`

### Unified Namespace
- Replace legacy references to individual packages with the single unified `rune` import:
  ```nox
  import rune
  from rune import Console, Style, Table, Panel, Command, CliApp
  ```

### Non-Interactive Prompt Behavior (1.0.1+)
- When calling `prompt_int()`, `select_choice()`, or `select_multiple()` in automated CI environments, supply a valid default value or handle `NonInteractiveError`.

### Color Validation (1.0.1+)
- RGB color channels and 256-color indices are clamped to `0..255`.
- Hex strings passed to `hex_to_rgb()` must contain exactly 6 valid hex characters (`#RRGGBB`).
