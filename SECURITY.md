# Security Policy

## Supported Versions

Only the latest minor release branch of Rune receives security patches and bug fixes.

| Version | Supported          | Status |
| :------ | :----------------- | :----- |
| 1.4.x   | :white_check_mark: | Active production release |
| 1.3.x   | :x:                | Replaced by 1.4.x |
| 1.2.x   | :x:                | Replaced by 1.3.x |
| 1.1.x   | :x:                | Replaced by 1.2.x |
| 1.0.x   | :x:                | Replaced by 1.1.x |
| < 1.0   | :x:                | Pre-release (unsupported) |

---

## Reporting a Vulnerability

The Rune project takes terminal security and user data protection seriously. If you discover a security vulnerability (such as ANSI escape code injection, terminal spoofing, unbounded memory allocation, or credential leakage), please report it responsibly:

1. **Private Disclosure**: Do **NOT** open a public GitHub issue for security vulnerabilities.
2. **Contact**: Open a draft Security Advisory directly on the GitHub repository via [Security Advisories](https://github.com/mburakmmm/rune/security/advisories/new) or contact the maintainer directly.
3. **Information to Include**:
   - Detailed description of the vulnerability.
   - Minimal proof of concept reproducing the injection or failure.
   - Operating system and terminal emulator (e.g. Windows Terminal, iTerm2, Alacritty, GNOME Terminal).
   - Any potential attack vectors (e.g. malicious CLI input, log injection, OSC escape triggers).

---

## Response Timeline

- **Initial Acknowledgment**: Within 48 hours.
- **Assessment & Triage**: Within 5 business days.
- **Fix & Advisory Release**: Coordinated disclosure with a patch release and CVE identifier if applicable.

---

## Core Security Invariants

Rune enforces the following architectural security rules:

1. **ANSI Escape Sanitization**: Untrusted terminal strings must pass through `sanitize_ansi()`, which strips dangerous terminal operating system commands (OSC), CSI cursor sequences, C1 8-bit controls, and standalone carriage returns (`\r`) to prevent screen spoofing and line overwriting.
2. **Deterministic Bounded Allocations**: Tables, trees, and layouts enforce hard limits on dimensions (e.g., max 100 columns, max 10,000 tree nodes) to prevent Denial of Service through adversarial inputs.
3. **No Process Abort in Libraries**: Low-level parser and rendering functions never invoke `nox.os.exit()`. Termination is owned strictly by the top-level CLI runner.
4. **Secret Redaction**: CLI options flagged as `is_secret` mask default values and help text to prevent accidental exposure of tokens or keys in command logs.
