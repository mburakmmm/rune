# Rune — Rendering Guide

> **Module:** `rune.render` & `rune.core`

This guide explains how to use Rune's terminal presentation components, combining Rich-like styling, layout, tables, trees, panels, terminal markdown, and syntax highlighting.

---

## 1. Terminal Styles & Colors

Rune provides a unified `Style` model supporting standard ANSI 16 colors, 256-color palettes, and 24-bit Truecolor RGB.

### Standard 16-Color Palette
Supported color names: `black`, `red`, `green`, `yellow`, `blue`, `magenta`, `cyan`, `white`, `bright_black`, `bright_red`, `bright_green`, `bright_yellow`, `bright_blue`, `bright_magenta`, `bright_cyan`, `bright_white`.

```nox
from rune import Style

err_style: Style = Style("red", "", True)      # Bold red
info_style: Style = Style("cyan", "", False, False, False, True) # Underline cyan
```

### 256-Color & 24-Bit RGB
```nox
from rune import Style, color_256, bg_256, color_rgb, bg_rgb, hex_to_rgb

# ANSI 256 color
s256: Style = Style(color_256(208)) # Orange

# 24-bit Truecolor RGB
s_rgb: Style = Style(color_rgb(255, 128, 0))

# Hexadecimal helper (#RRGGBB)
s_hex: Style = Style(hex_to_rgb("#4EC9B0"))
```

---

## 2. Inline Rich Text (`Text` & `Span`)

The `Text` model allows chaining styled segments within a single line or paragraph:

```nox
from rune import Text, Style

txt: Text = Text("Status: ")
txt.append("CONNECTED", Style("green", "", True))
txt.append(" (latency: 12ms)", Style("bright_black"))

rendered: str = txt.render(console.options)
```

---

## 3. Horizontal Dividers (`Rule`)

`Rule` creates clean horizontal dividers with optional titles:

```nox
from rune import Rule, Style

rule: Rule = Rule("Execution Summary", "─", "center", Style("bold"))
console.print(rule.render(console.options))
```

Supported alignments: `"left"`, `"center"`, `"right"`. In ASCII/non-Unicode mode, rules automatically fall back to `"-"`.

---

## 4. Bounded Containers (`Panel`)

`Panel` frames multiline text inside styled borders:

```nox
from rune import Panel, Style

p: Panel = Panel(
    "Application deployed successfully to production.\nReady for traffic.",
    "Deployment",
    "v1.0.0",
    "rounded",
    1,
    0,
    Style("green")
)
console.print(p.render(console.options))
```

Supported border styles: `"rounded"`, `"square"`, `"double"`, `"ascii"`.

---

## 5. Tabular Data (`Table`)

`Table` formats structured data with automatic width calculation, alignment, and multiline word wrapping:

```nox
from rune import Table, Style

tbl: Table = Table("Services Overview", "rounded", Style("bright_blue", "", True))
tbl.add_column("Service", "left")
tbl.add_column("Instances", "right")
tbl.add_column("Health", "center")

tbl.add_row(["auth-service", "3", "Healthy"])
tbl.add_row(["gateway", "5", "Healthy"])
tbl.add_row(["worker-queue", "2", "Degraded"])

console.print(tbl.render(console.options))
```

### Column Constraints
- Max columns: 100 (hard allocation bound).
- Padding: clamped to `0..20`.
- Cell auto-wrapping: long text wraps across multiple terminal lines while preserving border alignment.

---

## 6. Hierarchical Trees (`Tree`)

`Tree` visualizes parent-child hierarchies using an iterative stack layout that cannot blow the call stack:

```nox
from rune import Tree, TreeNode, Style

root: Tree = Tree("project/", Style("bold"))
src: TreeNode = root.add("src/")
src.add("main.nox")
src.add("utils.nox")
root.add("nox.json")

console.print(root.render(console.options))
```

Max node traversal is capped at 10,000 nodes to prevent denial-of-service on cyclic graphs.

---

## 7. Terminal Markdown (`Markdown`)

`Markdown` renders standard CommonMark-compatible subsets to the terminal:

- Headings (`#`, `##`, `###`) with distinct ANSI styling and underlines.
- Bulleted lists (`*`, `-`) and numbered lists (`1.`).
- Blockquotes (`> text`) with left border guides (`│` / `|`).
- Fenced code blocks framed in panels.

```nox
from rune import Markdown

md: Markdown = Markdown("# Quick Start\n> Simple terminal markdown.\n- Step 1\n- Step 2")
console.print(md.render(console.options))
```

---

## 8. Pure Nox Syntax Highlighter (`Syntax`)

`Syntax` tokenizes and highlights Nox code without external dependencies:

```nox
from rune import Syntax

code: str = "def add(a: int, b: int) -> int:\n    return a + b\n"
syn: Syntax = Syntax(code, "nox", True) # line numbers enabled
console.print(syn.render(console.options))
```
