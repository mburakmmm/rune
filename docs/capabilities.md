# Terminal & Runtime Capabilities Audit for Rune

This document audits the capabilities of the Nox runtime (`noxc 2.0.27`, `noxrt.o`, and `stdlib/nox/*`) as of October 2026, categorizing primitives as **Verified**, **Missing**, or **Experimental/Deferred**.

---

## 1. Capability Matrix

| Primitive | Status | Source / Mechanism | Notes / Limitations |
| :--- | :--- | :--- | :--- |
| **Command Line Arguments** | **Verified** | `nox.os.arg_count()`, `nox.os.arg(i)` | Index 0 is program name. Slices beyond `argc` return `""`. |
| **Environment Variables** | **Verified** | `nox.os.getenv(name)` | Raises `OsError` if unset. No default parameter overload; Rune wraps with safe `get_env_opt`. |
| **Process Termination** | **Verified** | `nox.os.exit(code)` | Exits immediately, bypassing cleanup/defer. Only top-level CLI runner may call this; never called in parser/renderer core. |
| **Monotonic Clock** | **Verified** | `nox.time.instant_now()`, `.elapsed_ms()` | Uses OS monotonic clock (`CLOCK_MONOTONIC`). Safe for progress bars & ETA calculations. |
| **Wall Clock** | **Verified** | `nox.time.now()`, `nox.time.now_ms()` | System wall clock; subject to NTP adjustments. |
| **Standard Input (Lines)** | **Verified** | `input() -> str` in core | Non-blocking/raw mode unavailable; line-buffered via libc `read()`. Returns `""` on EOF. |
| **ANSI Escape Sequences** | **Verified** | SGR via `nox.strings.char_from_byte(27)` | Fully supported by modern terminal emulators. Byte 27 is ESC. |
| **String Slices & Operations** | **Verified** | `s[a:b]`, `len(s)`, `nox.strings.*` | `len(s)` is codepoint count; `nox.strings.byte_len(s)` is byte count. |
| **Test Framework** | **Verified** | `nox.test.assert_eq_*`, `TestSuite` | Executed via `noxc test`. |
| **TTY Detection (`isatty`)**| **Missing** | Not exposed in `nox.os` or `noxrt.o` | Cannot query file descriptor `isatty` without native shims. Controlled via capability injection & environment variables (`TERM`, `CI`, `NO_COLOR`). |
| **Terminal Size (cols/rows)** | **Missing** | `TIOCGWINSZ` not exposed in stdlib | Defaults to 80x24 conservative fallback; injectable in `Console`. |
| **Raw Keyboard Mode** | **Missing** | `termios` not exposed | Keypress-by-keypress navigation and interactive line-editors are out of scope for 1.0. |
| **Echo Suppression (Passwords)**| **Missing** | `termios.c_lflag &= ~ECHO` not exposed | Masked/hidden password prompt is flagged as unsupported/deferred until runtime support arrives. |
| **Unicode Display Width (wcwidth)**| **Experimental** | Pure Nox ASCII/Latin cell accounting | Emoji, CJK full-width, and complex zero-width sequences documented with limitation. SGR ANSI escapes consume 0 cells. |

---

## 2. Terminal Detection & Fallback Policy

Since direct file descriptor inspection (`isatty`) is not currently exposed by Nox stdlib:
1. Rune provides explicit, deterministic capability configuration through `ConsoleOptions`.
2. Environment conventions are respected in order of strict precedence:
   - If `NO_COLOR` is present in environment, color output is disabled (`supports_color = False`).
   - If `CLICOLOR_FORCE=1` is set, color is explicitly forced (`supports_color = True`).
   - If `TERM=dumb`, color and advanced styling are disabled.
   - If `CI=true` or non-interactive environment is detected, animated spinners and live cursor movements are replaced with plain linear text updates.
3. Every component (`Text`, `Table`, `Panel`, `ProgressBar`, `Command`) produces pure, accessible plain text without ANSI escape sequences when `supports_color = False`.
