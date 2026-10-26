# Rune — Security Model & Invariants

> **Audience:** Security reviewers, CLI authors, and package maintainers.

Terminal presentation is a security boundary whenever output contains untrusted inputs (such as HTTP request logs, external file names, user input, or remote server responses).

Rune implements multi-layer defense-in-depth across the following vectors:

---

## 1. ANSI & Control Sequence Injection Defense

### CSI & OSC Escape Sanitization
Untrusted text must never be permitted to execute arbitrary escape sequences (e.g. changing window titles, hijacking clipboards, or resetting terminal palettes).
- `sanitize_ansi(s: str)` strips all standard ESC-based CSI (`\x1b[...]`) and OSC (`\x1b]...`) sequences.
- 8-bit C1 control codes (character codes 128 to 159) are stripped to prevent bypasses on legacy or 8-bit terminal emulators.

### Line Overwrite & Spoofing Prevention (`\r`)
Carriage returns (`\r`) without newlines allow attackers to move the terminal cursor to the beginning of the line and overwrite previous output (e.g. replacing `[FAIL] Error occurred` with `[OK] Success`).
- Rune normalizes valid `\r\n` line endings to `\n`.
- Any standalone `\r` character is automatically stripped from sanitized text.

---

## 2. Shell Injection Defense in Autocompletion

Generating shell completion scripts for Bash, Zsh, or Fish involves interpolating command and option metadata into shell scripts.
- `sanitize_shell_str(s: str)` strictly neutralizes shell metacharacters: semicolons (`;`), backticks (`` ` ``), subshell expansions (`$()`, `${}`), quotes (`"`, `'`), pipes (`|`), ampersands (`&`), and redirects (`>`, `<`).
- Variable and function names are sanitized into alphanumeric and underscore identifiers only.

---

## 3. Secret & Credential Redaction

CLI help outputs and error traces must not leak sensitive information.
- Options flagged with `is_secret = True` automatically redact declared default values in `--help` output, displaying `[default: ******]` instead of the plaintext credential.
- Password prompts explicitly raise `PromptError` rather than echoing secrets in plaintext.

---

## 4. Resource Allocation & Denial-of-Service Bounds

To protect CLI applications against memory exhaustion (OOM) or catastrophic backtracking from maliciously crafted inputs:
- **Table Columns:** Capped at a hard maximum of 100 columns.
- **Table Padding:** Clamped between 0 and 20 characters.
- **Tree Traversal:** Iterative stack-based algorithm with a hard limit of 10,000 visited nodes, preventing stack overflows and circular graph hangs.
- **Progress completed values:** Clamped to non-negative values (`completed >= 0`) and percentages clamped between 0% and 100%.
