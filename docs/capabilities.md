# Terminal & Runtime Capabilities Audit for Rune

This document audits the capabilities of the Nox runtime (`noxc 2.1.186`, `noxrt.o`, and `stdlib/nox/*`), categorizing primitives as **Verified**, **Platform-Bounded**, or **Experimental/Deferred**.

---

## 1. Capability Matrix

| Primitive | Status | Source / Mechanism | Notes / Limitations |
| :--- | :--- | :--- | :--- |
| **Command Line Arguments** | **Verified** | `nox.os.arg_count()`, `nox.os.arg(i)` | Index 0 is program name. `noxc run file.nox args...` works directly without `--`. |
| **Environment Variables** | **Verified** | `nox.os.getenv(name)` | Raises `OsError` if unset; Rune wraps safely via `get_env_safe`. |
| **Process Termination** | **Verified** | `nox.os.exit(code)` | Bypasses pending `finally`/`defer` cleanup; restricted to top-level CLI runner. |
| **Monotonic Clock** | **Verified** | `nox.time.monotonic_ns()`, `Instant`, `instant_now()`, `.elapsed_ms()` | Nanosecond/ms OS monotonic clock. Used across `Task`, `ProgressBar`, and `AppRunner`. |
| **Wall Clock** | **Verified** | `nox.time.now()`, `nox.time.now_ms()`, `nox.time.now_ns()` | System wall clock (`DateTime`). |
| **Cooperative Sleep** | **Verified** | `nox.time.sleep_ms(ms)` | Fiber-cooperative on POSIX; blocks thread on Windows / outside fibers. |
| **TTY Detection (`isatty`)** | **Verified** | `nox.terminal.is_tty(fd=1)` | Real POSIX TTY check on stdin/stdout (`0`/`1`). Returns `False` on Windows (`TerminalCapabilities` falls back cleanly). |
| **Terminal Capabilities** | **Verified** | `nox.terminal.get_capabilities()` | Exposes `is_tty`, `supports_ansi`, `supports_truecolor`, `color_depth` (`0`/`16`/`256`/`16777216`), respecting `NO_COLOR` & `TERM=dumb`. |
| **Terminal Size (cols/rows)** | **Verified** | `nox.terminal.get_size()` | Returns `TerminalSize(cols, rows)`; falls back to `COLUMNS`/`LINES` then `80x24`. |
| **Unbuffered Raw Output** | **Verified** | `nox.terminal.write_raw(content)`, `nox.terminal.flush()` | Flushes buffered output and writes directly to stdout without trailing `\n`. Used by `Console.write()` and `TerminalDriver.write_raw()`. |
| **Non-Blocking Raw Input** | **Verified** | `nox.terminal.poll_input(timeout_ms)`, `read_raw(max_bytes=256)` | Fiber-cooperative polling + byte-safe `str` preserving `\x00` and partial UTF-8 bytes; Rune's `KeyDecoder` reassembles split multi-byte UTF-8 via `nox.strings.byte_at` / `nox.bytes.str_from_list`. |
| **Raw Mode & Terminal Reset** | **Verified (POSIX)** | `nox.terminal.enable_raw_mode()`, `disable_raw_mode()`, `restore_state()` | Ref-counted raw mode + emergency `restore_state()` wrapped in Rune `TerminalSession`. Raises `TerminalError` on non-TTY / Windows; Rune guards and degrades cleanly. |
| **Echo Suppression (Passwords)** | **Verified (POSIX)** | `nox.terminal.set_echo(enabled)` | Used in `prompt_password()` inside `try ... finally`. Raises `PromptError`/`NonInteractiveError` when non-TTY or Windows throws `TerminalError`. |
| **POSIX Signals** | **Verified (POSIX)** | `nox.signal` (`sigint`, `sigterm`, `install`, `take`, `wait`) | Used by `AppRunner.run_interactive()` for clean Ctrl+C/SIGTERM shutdown & restoration. Returns `False` on Windows; `SIGWINCH` is not exposed (Rune polls `get_size()`). |
| **Unicode NFC & Turkish Case** | **Verified** | `nox.unicode.normalize(s, "NFC")`, `upper_tr`, `lower_tr` | Used by `cell_width` / `normalize_text` so decomposed diacritics normalize accurately. |
| **Unicode Display Width (`wcwidth`)** | **Verified / Evolving** | Pure Nox cell-width tables + `nox.unicode.normalize("NFC")` | Covers ASCII, combining marks, CJK wide, and common emoji ranges; grapheme-cluster ZWJ sequences remain bounded. |
| **Regex & Property Fuzzing** | **Verified** | `nox.regex` (pure Zig VM), `nox.property` (`Rng`, `check`, `replay`) | Pure-Zig regex needs no C libs; `nox.property` drives fuzzing tests with automatic shrinking. |
| **Test Framework** | **Verified** | `nox.test`, `nox.property`, `noxc test` | Executed via `noxc test` across all unit, integration, snapshot, and fuzz suites. |

---

## 2. Terminal Detection & Fallback Policy

1. `detect_console_options()` queries `nox.terminal.get_capabilities()` and `nox.terminal.get_size()` rather than relying solely on environment guesses.
2. Strict precedence rules apply across platforms:
   - `NO_COLOR` (non-empty) disables color (`supports_color = False`, `color_depth = 0`).
   - `CLICOLOR_FORCE=1` forces color (`supports_color = True`, `supports_ansi = True`).
   - `TERM=dumb` disables ANSI and color.
   - `CI` non-empty forces `is_tty = False`.
   - On Windows without modern VT env (`WT_SESSION` / `ANSICON` / `ConEmuANSI`), plain ASCII fallback is selected safely, and raw-mode / echo operations never crash because `TerminalError` and `is_tty(0)` are explicitly guarded.
3. Every component (`Text`, `Table`, `Panel`, `ProgressBar`, `Command`, `TuiApp`) degrades cleanly to deterministic plain text when ANSI/TTY support is absent.
