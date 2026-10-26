# Rune — Compatibility & Portability Matrix

> **Operating Systems:** Linux, macOS, Windows (Windows Terminal, ConEmu, legacy Command Prompt).

Rune is designed to operate predictably across all environments, gracefully degrading from full 24-bit Truecolor terminal rendering down to strict ASCII monochrome plain text when redirected to files or run in CI.

---

## 1. Terminal Detection Hierarchy

Console options (`ConsoleOptions`) are resolved in the following strict order of precedence:

1. **`NO_COLOR`**: If set to any value (and non-empty), ANSI color is completely disabled.
2. **`CLICOLOR_FORCE`**: If set to `"1"`, ANSI colors are forced regardless of TTY status.
3. **`TERM=dumb`**: Disables ANSI color and forces ASCII box-drawing characters.
4. **`CI`**: If set (e.g. GitHub Actions, GitLab CI), marks `is_tty = False` to prevent interactive prompt blocking.
5. **Dimensions**: Probes environment variables `COLUMNS` and `LINES`; falls back to conservative 80x24 if undefined.
6. **Windows VT Support**: Inspects `WT_SESSION` (Windows Terminal), `ConEmuANSI`, or `ANSICON`. When absent on Windows hosts, defaults to ASCII borders and monochrome output.

---

## 2. Platform Support Status

| Platform / Host | ANSI Color | Unicode Glyphs | Auto Wrap | Notes |
| :--- | :--- | :--- | :--- | :--- |
| **Linux (TTY)** | Full (16 / 256 / Truecolor) | Full (`┌─┐`) | Yes | Standard VT100 / xterm |
| **macOS (Terminal.app / iTerm2)** | Full | Full | Yes | Standard VT100 / xterm |
| **Windows Terminal (wt.exe)** | Full | Full | Yes | `WT_SESSION` detected |
| **ConEmu / cmder** | Full | Full | Yes | `ConEmuANSI` detected |
| **Legacy Windows Command Prompt** | Monochrome | ASCII (`+-+`) | Conservative | Safe plain fallback |
| **Piped Output / Redirect (`> file.txt`)** | None | Preserved | Conservative | No ANSI escape codes |
| **CI / Non-Interactive** | None | Clean ASCII/Plain | Strict | Prompts require defaults |

---

## 3. Unicode Display-Cell Widths (`wcwidth`)

Terminal columns do not map 1:1 to string byte lengths or code point counts:
- Standard ASCII characters: **1 cell**.
- CJK Wide Ideographs (`界`, `你`, `中`): **2 cells**.
- Emojis (`🙂`, `🦀`, `🚀`): **2 cells**.
- Combining characters (accents, umlauts): **0 cells**.
- Control characters: **0 cells**.

Rune's `cell_width()`, `truncate()`, and `wrap_text()` calculate widths using these display-cell rules to prevent broken table borders and Misaligned layout.
