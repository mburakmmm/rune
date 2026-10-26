# AGENTS.md — RUNE

> Binding instructions for AI coding agents developing **Rune**, the unified terminal presentation + command-line application toolkit for the **Nox programming language**.

## 0. Project identity

- **Canonical name:** Rune.
- **Package/import namespace:** `rune` (subject to actual Nox package layout).
- **Merged concepts:** Onyx (Rich-like terminal rendering) + original Rune (Click/Typer-like CLI framework).
- **Do not** create separate Onyx or legacy Rune packages. No `onyx.*` public imports or mixed branding.
- **Language:** Nox; do not write Python code disguised as Nox.
- **Nox source:** https://github.com/mburakmmm/nox-lang
- **Documentation:** https://noxlang.com
- **Primary audience:** authors of Nox CLIs, build tools, package managers, developer utilities, administrative tools and scripts.
- **Purpose:** provide coherent styling, tables, panels, progress indicators, interactive prompts, command parsing, help output, and eventually advanced terminal capabilities through a modular Nox-native library.
- **Non-goals for 1.0:** full-screen TUI framework, shell replacement, terminal emulator, logging backend, web UI, or Python Click/Rich compatibility.
- **North star:** small useful components, excellent defaults, predictable behavior, graceful plain-text fallback, minimal dependencies.

### Binding precedence

1. User's explicit requirements and security constraints.
2. This AGENTS.md for Rune.
3. Current Nox compiler/runtime invariants and verified documentation.
4. Existing repository architecture and accepted decisions.
5. Agent implementation preferences.

If a requirement conflicts with a verified Nox limitation, do not fabricate an implementation: explain the limitation and propose an honest fallback.

---

## 1. Mandatory Nox capability audit before coding

Before writing or modifying any Rune feature:

1. Read this file and inspect the current Rune repository and tests.
2. Read the Nox repository `README.md`, `AGENTS.md`, relevant `docs/language/*`, `docs/stdlib/*`, and `docs/tools/*`.
3. Run `noxc version`; note compiler version, OS, backends, and package layout.
4. Inspect `nox.console`, `nox.os`, `nox.strings`, `nox.time`, `nox.test`, `nox.fs`, `nox.process`, and any verified terminal/TTY, stdin, stdout/stderr, environment, Unicode, and keyboard APIs.
5. Inspect actual stdlib source when docs are insufficient.
6. Search existing Rune code to avoid duplicating functionality.
7. Create/update `docs/capabilities.md` with **verified / missing / experimental** status for each primitive.
8. Compile a minimal proof of concept before introducing any unverified language syntax or runtime capability.
9. Document the real commands used to build and test.

### Verified facts from Nox documentation at drafting time

These are documented by `mburakmmm/nox-lang`; verify against the actual installed compiler:

- `nox.os.arg_count()` counts the program name at index 0; `nox.os.arg(i)` returns an argument string. With `noxc run file.nox -- a b`, `a` is `arg(1)`.
- `nox.os.getenv(name)` raises `OsError` for missing variables; no documented default-value overload.
- `nox.os.exit(code)` terminates immediately, bypassing pending `finally`/`defer` cleanup.
- `nox.console` offers `format(level,message)`, `debug`, `info`, `warn`, `error`; it is a **simple levelled output facility**, not a documented ANSI styling/TTY API.
- `nox.strings.byte_len(s)` counts UTF-8 bytes; `len(s)` counts Unicode code points. Neither automatically provides terminal display-cell width.
- `nox.strings.upper/lower` are ASCII-oriented; do not assume full Unicode case mapping.
- `nox.test` offers typed assertions and `TestSuite`; `noxc test` runs `*_test.nox` files.
- `noxc check`, `noxc run`, `noxc build`, `noxc test`, `noxc fmt` are documented compiler commands.
- Nox packages use `nox.json` and optionally `nox.lock`, with `noxc fetch` and a Git-based package registry.

**Not yet established by those documents:** raw terminal mode, nonblocking keypresses, cursor positioning APIs, stdout/stderr file descriptor control, `isatty`, terminal width/height, ANSI support detection, Windows VT mode, shell completion hooks, portable password input, SIGWINCH, interactive screen refresh. Treat these as unverified and feature-gate them.

---

## 2. Non-negotiable invariants

1. **Nox-native:** use valid statically typed Nox, not dynamic Python conventions.
2. **No compiler fork:** never modify Nox compiler/runtime or use unsafe native shims without explicit user approval.
3. **No ownership exposure:** preserve Nox automatic memory management and normal type guarantees.
4. **No invented APIs:** never assume terminal I/O, reflection, decorators, arbitrary callbacks, or generic types exist without compiling a proof.
5. **One brand:** Rune owns rendering and CLI modules; do not introduce an Onyx dependency.
6. **Strict module boundaries:** `rune.cli` may use `rune.render`; `rune.render` must never depend on `rune.cli`.
7. **No global mutable terminal state:** explicit console/context instances; any process-wide coordination must be justified and documented.
8. **Plain text first:** useful output even when ANSI/TTY support is absent.
9. **Never emit escape codes to non-TTY output by default** unless an explicit force-color mode is requested.
10. **No ANSI injection:** untrusted strings must be escaped/sanitized before styling and cursor control.
11. **No silent CLI ambiguity:** parse failures must be explicit and predictable.
12. **Never expose secrets** through prompts, errors, help text, logs, or process arguments where avoidable.
13. **Deterministic formatting:** same input, width, style, and options yield the same output.
14. **Graceful degradation:** unsupported interactivity must be reported or replaced with documented noninteractive behavior, never fake input.
15. **No feature inflation:** full-screen TUI is outside the initial scope.
16. **Every public feature needs real compiling examples and tests.**
17. **Minimal dependencies and resource usage:** avoid heavyweight terminal engines and framework coupling.
18. **No destructive CLI side effects by default:** confirmation and dry-run patterns for dangerous operations.
19. **Accessibility matters:** color cannot be the sole carrier of meaning.
20. **Compatibility is contractual:** document breaking API changes and migration.

---

## 3. Product architecture

```text
Rune public API
│
├── rune.core        # terminal capabilities, output sink, width, styles, text model
│
├── rune.render      # formatted text, panels, tables, trees, rule/dividers
│   └── depends on rune.core only
│
├── rune.progress    # progress bars, spinners, task/status display
│   └── depends on rune.core + optional rune.render
│
├── rune.prompt      # input, confirmation, choice, password when supported
│   └── depends on rune.core + optional rune.render
│
├── rune.cli         # arguments, options, commands, help, errors, completion
│   └── depends on rune.core + optional rune.render/prompt
│
└── rune.tui         # OPTIONAL POST-1.0 / separate future package decision
```

### Dependency direction

- `core` cannot import `render`, `progress`, `prompt`, or `cli`.
- `render` cannot import `cli` or `prompt`.
- `progress` cannot import `cli`.
- `prompt` cannot import `cli`.
- `cli` may consume the other public modules but must remain usable in plain-text mode.
- No cyclic imports.
- Optional interactive/TTY modules must not be mandatory for basic formatting or CLI parsing.
- Terminal detection belongs in core and should be injectable/mocked in tests.
- Output sinks and clocks should be injectable where feasible for deterministic tests.

### Suggested repository layout (adapt to verified Nox imports)

```text
rune/
  AGENTS.md
  README.md
  LICENSE
  CHANGELOG.md
  CONTRIBUTING.md
  nox.json
  nox.lock                  # only when dependencies exist
  docs/
    architecture.md
    capabilities.md
    api.md
    rendering.md
    cli.md
    prompts.md
    security.md
    compatibility.md
    roadmap.md
    migration.md
  rune/                     # verify module path in Nox before creating
    core.nox
    style.nox
    text.nox
    render.nox
    table.nox
    panel.nox
    tree.nox
    progress.nox
    prompt.nox
    cli.nox
    arguments.nox
    options.nox
    help.nox
    errors.nox
    completion.nox
  tests/
    core_test.nox
    render_test.nox
    cli_test.nox
    prompt_test.nox
    regression_test.nox
  examples/
    hello.nox
    table.nox
    progress.nox
    command.nox
    subcommands.nox
    prompt.nox
  benchmarks/
  .github/workflows/
```

Do not generate empty modules merely to satisfy a suggested tree. Real layout must compile.

---

## 4. Public API design

API names below are **design candidates, not verified Nox syntax**:

- `Console`: output sink, terminal capability and style configuration.
- `Style`: foreground/background color, bold, dim, underline, etc.
- `Text` / `Span`: plain text plus styled ranges, if efficiently representable.
- `Table`, `Column`, `Row`: structured tabular output.
- `Panel`, `Tree`: optional structured rendering.
- `Progress`, `Task`, `Spinner`: status indicators.
- `Prompt`, `Confirm`, `Select`: interactive input.
- `Command`, `Argument`, `Option`, `CliApp`: CLI declarations.
- `CliError`, `UsageError`, `RenderError`, `TerminalError`.

Use idiomatic Nox constructors and explicit types. Do not rely on Python decorators, `*args/**kwargs`, runtime-generated functions, monkeypatching, dynamic attributes, or reflection without verified compiler support.

### Core ergonomics

A user should be able to:
1. Print styled text.
2. Print a table with sensible alignment.
3. Define a command with typed options and positional arguments.
4. Obtain a predictable `--help` page.
5. Receive structured parse errors and proper exit codes.
6. Run the same command in a noninteractive CI environment without escape-sequence garbage.

The default quick start must work without native dependencies.

---

## 5. Terminal capabilities and output abstraction

Design an explicit terminal capability model:

- `is_tty` (only if detectible)
- `supports_ansi`
- `color_depth`: none / ANSI 16 / ANSI 256 / truecolor
- `supports_unicode`
- `width`, `height` (optional/unknown permitted)
- `interactive_input_available`
- `supports_cursor_control`
- `supports_raw_mode`
- `supports_alt_screen`

### Detection policy

- Inspect actual OS/runtime facilities first.
- Respect `NO_COLOR` convention: color disabled when present unless a documented explicit force override applies.
- Consider `CLICOLOR`, `CLICOLOR_FORCE`, `TERM=dumb`, and CI environment only with clearly defined precedence.
- Do not treat the mere presence of `TERM` as proof of TTY support.
- Do not claim stdout/stderr separation if Nox's output API cannot express it.
- Terminal width defaults to a documented conservative value if unknown, but must not be misrepresented as measured.
- Avoid emitting cursor-control sequences unless capability is proven or explicitly forced.
- Windows support must be tested; do not assume ANSI escapes work in every console host.

### Output model

- Separate **render to text** from **write to output sink**.
- Pure rendering functions should accept explicit width/style/capability parameters.
- Inject an output sink for tests where Nox allows it.
- Flush only when supported and required; do not invent flush APIs.
- Do not silently swallow write errors.
- Maintain explicit newline policy and prevent double newlines.
- Make output capture and snapshot testing practical.

---

## 6. Unicode, ANSI and display-width correctness

**Critical:** byte length, Unicode code-point count, and terminal display-cell width are different.

- `nox.strings.byte_len("é")` is not the displayed width.
- `len("é")` counts code points, not necessarily terminal cells.
- Emoji, CJK fullwidth characters, combining marks, ZWJ sequences, flags, and variation selectors require explicit handling.
- If a standards-quality `wcwidth`/grapheme implementation is not feasible in MVP, support ASCII-width rendering with documented Unicode limitations rather than lying.
- Never split a UTF-8 sequence in the middle.
- Never cut an ANSI escape sequence in the middle.
- ANSI SGR sequences consume zero display cells.
- Truncation, wrapping, and padding must operate on display width where supported.
- Preserve grapheme clusters where a verified grapheme segmentation implementation exists.
- Tab width, CR, LF, control characters, and embedded ANSI must follow explicit policies.
- Sanitize untrusted terminal escape codes (OSC, CSI, DCS, etc.) by default.
- No terminal hyperlinks (OSC 8) until escaping and capability detection are robust.

### Color/style policy

- Explicit style model, not arbitrary concatenation of escape sequences.
- Reset styles correctly even for nested styles and early exits.
- Avoid color bleed into following output.
- Provide plain-text fallback with no control sequences.
- No default animations or cursor movement when output is piped.

---

## 7. Rendering components

### Styled text
- Foreground/background colors, bold, dim, underline, italic only when supported.
- Compose styles predictably; define precedence for nested styles.
- Escape or neutralize control sequences in untrusted text.
- Keep raw text separately available for plain output.

### Tables
- Column titles, row values, alignment, padding and borders.
- Explicit width allocation; deterministic layout.
- Handle empty table, zero rows, long words, multiline cells, narrow terminals.
- Horizontal clipping/truncation/wrapping strategy documented.
- Do not use bytes or code-point count as a universal width metric.
- Avoid quadratic concatenation on large tables where practical.
- No hidden I/O in table formatting.

### Panels
- Title, padding, border styles, multiline content.
- Clip or wrap based on available width.
- Correct handling of zero/negative/unknown width.
- Plain fallback remains readable.

### Trees
- Parent/child indentation and connector characters.
- Bound recursion depth.
- Avoid infinite loops on cyclic references.
- ASCII fallback for terminals without reliable Unicode support.

### Markdown and syntax highlighting (later)
- Only implement documented subsets; do not claim full CommonMark compliance without a compliant parser.
- Escape terminal control sequences inside code and text.
- Avoid importing an entire compiler/parser for syntax highlighting.
- Support plain-text fallback.

---

## 8. Progress indicators and spinners

Progress must be **correct and optional**, not decorative at the cost of reliability.

- Deterministic progress model: total, completed, label, state.
- Define behavior for total=0, unknown total, over-completion, cancellation, and failures.
- Bound update rate; avoid excessive terminal writes.
- Use monotonic time for elapsed time and ETA if supported.
- ETA is an estimate, not a promise; avoid division by zero.
- Spinner animation requires a real timing and terminal redraw mechanism; otherwise provide a static status indicator.
- In non-TTY mode, emit bounded periodic textual updates or a final summary; never print endless animation frames.
- Stop animation and restore cursor state on errors and normal exit.
- Do not spawn unmanaged background tasks.
- Concurrent progress updates require synchronization only when verified supported; otherwise document single-thread use.
- Allow caller to disable progress entirely.

---

## 9. CLI parsing: grammar and contracts

Rune CLI is a **real argument parser**, not merely pretty help text.

### Input model

- Parse arguments from an explicit `list[str]` (or verified equivalent).
- Provide a separate convenience adapter to `nox.os.arg_count/arg`.
- Do not read process-global arguments inside the parser core.
- Program name at index 0 is excluded from application arguments.

### Syntax and parsing

- Long options: `--output file`, `--output=file`.
- Short options: `-o file`; grouped flags (`-abc`) only if explicitly supported and tested.
- `--` ends option parsing; subsequent tokens are positional.
- `-` by itself is positional unless explicitly documented otherwise.
- Negative numbers must not be misclassified as flags when a numeric value is expected.
- Preserve argument order and support repeated options only according to explicit policy.
- Reject unknown options and ambiguous abbreviations.
- Do not silently accept malformed option names or invalid values.
- Define duplicate-option policy: error / last wins / append; use type-appropriate documented defaults.
- Explicitly define optional-value options vs required-value options.
- Support subcommands without ambiguous dispatch.
- Define aliases and conflicts with `--help` and `--version`.

### Positional arguments

- Required and optional positionals.
- Ordered matching and arity checks.
- Variadic positionals only when unambiguous.
- Missing/extra argument errors with helpful usage.
- Typed conversions with overflow and range checking.
- Validate defaults as strictly as supplied values.

### Option types

Prioritize:
- `str`, `int`, `float`, `bool` flags.
- Enum/choice sets.
- Paths as strings initially; do not claim filesystem validation unless performed.
- Repeated options / lists when verified.
- Environment-sourced defaults in a later phase.

### Value precedence

When supported, define explicitly:
`CLI arguments > environment variables > application configuration > declared defaults`.

Do not read `.env` files implicitly. Environment variable absence must respect `nox.os.getenv` raising `OsError`.

### Command callbacks

- Callback signatures must compile in current Nox.
- No reflection-based automatic function registration unless verified.
- Do not invent decorators like `@command` unless compiler support and behavior are proven.
- Prefer explicit command registration with statically typed handlers.
- Return structured parse results or errors rather than immediately exiting from parser core.
- Make error handling testable without process termination.

---

## 10. Help output and command UX

A CLI framework is judged by its help screens and diagnostics.

- Built-in `--help` and optional `-h` support.
- Optional `--version` only if version metadata is available.
- Usage synopsis, command description, positional args, options, defaults, choices, examples, subcommands.
- Deterministic ordering: declared order or documented sorted order.
- Automatic width-aware wrapping when display-width support is correct.
- Clear error: invalid option, missing value, unknown command, invalid type, out of range.
- Error messages must not expose passwords/tokens.
- Color is optional; plain help must be complete and accessible.
- Help should be readable at narrow widths and when piped.
- Support Unicode text subject to documented width limitations.
- No excessive ASCII art or animation in default help.
- Shell completion scripts are later-phase features and require real shell tests.

### Exit codes

Recommended policy:
- `0`: successful command/help/version.
- `2`: usage or argument parsing error.
- `1`: general application failure.
- `130`: user interrupt only when correctly observable and safely handled.

Define any additional codes clearly. **Never call `nox.os.exit` from low-level parser or renderer**. The top-level CLI runner owns process termination and must finish cleanup first; `nox.os.exit` bypasses pending cleanup.

---

## 11. Prompts and user interaction

Prompt support depends on verified stdin and terminal capabilities.

### Planned prompt types

- Text input with default.
- Yes/no confirmation.
- Single selection from choices.
- Multi-selection (later).
- Secret/password entry **only when echo suppression is truly supported**.
- Validation and retry of user input.
- Noninteractive fallback or explicit failure.

### Rules

- Never claim masked/hidden password input without verified no-echo terminal support.
- Do not print secret values in errors, prompts, or logs.
- In CI/non-TTY, do not hang indefinitely waiting for input.
- Support default values only when safe and explicitly requested.
- Bound retries; handle EOF and cancellation distinctly.
- A prompt cannot assume arrow-key navigation or raw mode.
- For initial release, simple line-based input is acceptable if available; otherwise keep prompts deferred.
- No hidden interactive questions during CLI argument parsing.
- Destructive operations require explicit confirmation in application code; Rune should provide helpers, not override application policy.

---

## 12. Shell completion (later phase)

- Implement only after command grammar and parser are stable.
- Bash, Zsh, Fish, PowerShell are separate targets with separate tests.
- Generated completion must quote/escape shell metacharacters safely.
- Never execute arbitrary untrusted command strings to generate completions.
- Dynamic completion callbacks must be constrained and documented.
- Completion scripts are generated from actual command metadata.
- Shell-specific support is explicitly versioned and tested.

---

## 13. Full-screen TUI: deliberate non-goal for initial release

Do not let Rune become a giant terminal framework before CLI + rendering are stable.

**Not in the initial stable scope:**
- Alternate screen buffers.
- Full-screen layout engine.
- Focus/keyboard/mouse event loop.
- Interactive widgets, scroll views, live dashboards.
- Raw terminal mode and resize-event handling.

After a stable Rune core, evaluate either:
1. `rune.tui` as an optional module, or
2. a separate future package that depends on Rune core.

A full-screen TUI implementation requires an independent design review, portable input/output primitives, terminal restoration guarantees, and explicit user approval.

---

## 14. Security model

Terminal output is a security boundary when it contains untrusted text.

- Sanitize or escape ANSI CSI/OSC/DCS, control characters, terminal hyperlinks and clipboard-setting sequences in untrusted strings.
- Never treat raw user strings as markup without opt-in parsing and escaping.
- Separate trusted style instructions from untrusted content.
- Avoid terminal title manipulation, clipboard access, or hyperlinks by default.
- Prevent shell injection in generated completion or command suggestions.
- Never invoke a shell to parse CLI arguments.
- Do not execute arbitrary external commands from a renderer or parser.
- Redact passwords, API tokens, auth headers and secret environment values.
- Prevent huge allocations from maliciously large width, depth, table dimensions or argument counts.
- Limit recursion in trees and nested renderables.
- Avoid leaking filesystem paths or environment contents in default error messages.
- Native/FFI terminal extensions need explicit threat review and approval.
- Document that ANSI sanitization and shell escaping are separate problems.

---

## 15. Accessibility and portability

- Color-blind-friendly: pair colors with labels/symbols.
- Honor no-color and plain-output modes.
- Unicode and ASCII fallback glyph sets.
- Do not rely on emoji for essential status.
- Avoid flicker and excessive animation; offer reduced-motion/no-animation mode.
- Never animate by default when stdout is redirected.
- Ensure errors remain understandable in monochrome.
- Test Windows, macOS, Linux separately where support is claimed.
- Do not assume UTF-8 locale or VT mode on all Windows hosts.
- Handle unknown terminal size and small terminals.
- Screen reader and noninteractive usage favor clear linear plain text.
- Do not claim platform support without CI or documented manual verification.

---

## 16. Testing strategy

### Verified tooling

- `noxc check <file.nox>`: syntax/typecheck.
- `noxc run <file.nox>`: compile and execute.
- `noxc build <file.nox>`: compile.
- `noxc test`: execute `*_test.nox` files.
- `noxc fmt <file.nox>`: format **in place**.
- `nox.test` assertions and `TestSuite` are documented.

Confirm tool versions and commands locally; do not invent flags.

### Core tests

- Style application and reset; nested styles.
- No-color/forced-color precedence.
- Plain-text output contains no ANSI sequences.
- Untrusted escape-sequence sanitization.
- Unicode widths, combining marks, CJK, emoji, tabs, multiline text (as supported).
- Width 0/1, narrow/wide terminals, unknown width.
- Empty and multiline tables, alignment, borders, wrapping/truncation.
- Panel and tree rendering, depth limits.
- Progress totals 0/unknown, completion and error state.
- CLI no args, valid args, missing/extra args, unknown options.
- `--help`, `--version`, `--`, `--key=value`, repeated options.
- Negative numbers, invalid int/float, integer overflow.
- Subcommands, alias collisions, conflicting options.
- Defaults, env precedence, absence of env var.
- Secret redaction in errors/help.
- Prompt EOF, noninteractive mode, cancellation and validation.
- Exit-code contract and cleanup behavior.

### Testing approach

1. Pure rendering tests with exact snapshots for supported ASCII cases.
2. Property/invariant tests where tooling allows: width never negative, no control sequence leak in plain mode, parse round-trip invariants.
3. CLI integration tests via real `noxc run` programs and captured output/exit status where harness permits.
4. Pseudo-terminal tests only when a reliable PTY facility exists; otherwise mark TTY-dependent features unverified.
5. Cross-platform tests on supported operating systems.
6. LLVM/QBE parity tests when both are available and relevant.
7. Every bug fix adds a regression test.
8. Never skip, delete or weaken tests to obtain green CI.
9. Never report test success without executing tests.

### Golden output rules

- Normalize only truly nondeterministic elements (time, terminal width) through injected test values.
- Do not normalize away actual rendering defects.
- Keep ANSI snapshots and plain snapshots separate.
- Use controlled terminal capabilities rather than depending on developer machine environment.

---

## 17. Performance and memory

- Benchmark pure text rendering, tables (10/100/1000 rows), CLI parsing (10/100/1000 args), and progress updates.
- Record compiler version, backend, OS, optimization, sample sizes and measurement method.
- Avoid quadratic string concatenation for large output where feasible.
- Cache immutable style/width computations only with proven correctness and explicit ownership/lifetime.
- Avoid mutable global caches.
- Bound table dimensions, recursion depth, output size and update frequency.
- Measure import/compile impact of modules; keep basic `rune.core` lean.
- Do not claim Rich/Click performance parity without comparable measurements.
- Performance never justifies broken Unicode, unsafe ANSI handling or incorrect parsing.

---

## 18. Documentation standards

Documentation and public API examples should be in English; Turkish supplementary docs are welcome.

`README.md` must contain:
- One-sentence purpose and scope.
- Nox compiler minimum version (verified).
- Package installation instructions using real `nox.json` conventions.
- Compiling colored-text quick start.
- Compiling table quick start.
- Compiling CLI command with help quick start.
- Feature support matrix: implemented / planned / unsupported.
- Plain/CI mode and accessibility notes.
- Platform support and Unicode-width limitations.
- Security model and license.

Docs must distinguish:
- **Supported**: implemented and tested.
- **Experimental**: works under documented limitations.
- **Planned**: not yet implemented.
- **Unsupported**: unavailable due to Nox/runtime limitations.

Never publish pseudocode as executable Nox. Label proposals `PSEUDOCODE — NOT IMPLEMENTED`.

---

## 19. Package management, CI and release

- Package is named `rune`, not `onyx`.
- Use actual Nox `nox.json` manifest conventions.
- Use `nox.lock` for resolved dependencies and commit it when applicable.
- No mandatory Nyx/Aether/Aura dependencies.
- Prefer Nox stdlib and pure Nox implementation.
- Run formatting, typecheck, tests, example compilation, and supported platform/backend CI.
- Keep changelog and migration notes for public behavior changes.
- Use semantic versioning; even pre-1.0 breaking changes need notes.
- Never publish a package, tag a release, or push to a repository without explicit permission.
- No destructive Git operations or unrelated code changes.
- Never commit credentials or generated secrets.

---

## 20. Roadmap with release gates

### Phase 0 — Discovery / architecture
- Audit Nox terminal I/O, CLI arguments, strings, Unicode, tests, and package conventions.
- Compile proof of concept for output and argument parsing.
- Document missing capabilities and implementation decisions.
**Exit:** factual capability matrix and a working minimal example.

### Phase 1 — Rune Core / Render MVP
- Plain/ANSI styled text with safe fallback.
- Terminal capability configuration.
- Basic tables and panels.
- ANSI sanitization.
- Unit tests, examples, README.
**Exit:** all documented rendering APIs compile and pass tests.

### Phase 2 — Progress and structured output
- Trees, progress bars, status indicators.
- Optional spinner if timer/cursor capabilities are verified.
- Width, accessibility and output safety improvements.
**Exit:** deterministic tests and non-TTY behavior.

### Phase 3 — CLI framework MVP
- Positional arguments, long/short options, typed conversion.
- Command registration, subcommands.
- Auto-help, usage errors, exit-code contract.
- Integration tests.
**Exit:** complete sample CLI works with no rendering dependency cycle.

### Phase 4 — Interactive UX
- Text/confirm/select prompts only with verified stdin.
- Secret prompts only with verified echo suppression.
- Environment defaults, help refinements.
**Exit:** no hanging CI and no secret leaks.

### Phase 5 — Advanced developer experience
- Markdown subset, syntax highlighting, completion (where tested).
- Rendering/CLI performance hardening.
- Windows/macOS/Linux verification.
**Exit:** support matrix and security review complete.

### Phase 6 — Rune 1.0
- Freeze public API, full docs, regression suite.
- Stable package installation, release notes, migration guide.
- User authorization before publishing.
**Exit:** no misleading support claims and all release gates green.

**TUI decision:** after 1.0 or a separately approved architecture milestone; not a prerequisite for 1.0.

---

## 21. Agent execution protocol

For every task:

1. Restate task and acceptance criteria.
2. Read relevant existing code/tests/docs.
3. Verify Nox language/stdlib support.
4. Describe proposed module boundary and any new public API.
5. Implement the smallest coherent vertical slice.
6. Add tests alongside the code.
7. Run formatting, `noxc check`, `noxc test`, examples and relevant CI.
8. Review security (ANSI injection, secrets, shell completion), accessibility, and plain fallback.
9. Update docs, capability matrix and changelog.
10. Report changed files, exact executed commands, actual pass/fail results and limitations.

### If blocked

- Do not fabricate terminal APIs or output.
- Produce a minimal reproduction of the missing capability.
- Document the exact limitation and the source inspected.
- Offer a supported reduced-scope implementation.
- Ask permission before changing Nox compiler/runtime or adding native dependencies.
- Never use subprocesses or external shell utilities as a secret mandatory runtime dependency.

---

## 22. Definition of Done

A feature is complete only when all applicable items hold:

- [ ] Uses verified Nox syntax and stable public APIs.
- [ ] Builds and runs on claimed toolchains.
- [ ] Tests cover valid/invalid/boundary cases.
- [ ] ANSI/control injection is addressed.
- [ ] Plain-text/non-TTY behavior is correct.
- [ ] Unicode width limitations are documented or tested.
- [ ] CLI parser behavior and exit codes are deterministic.
- [ ] No accidental Onyx naming or dependency.
- [ ] No circular imports or framework dependencies.
- [ ] No invented interactive/TTY capabilities.
- [ ] Examples compile and documentation is current.
- [ ] No tests hidden, weakened, or skipped to claim success.
- [ ] Final report states actual verification and remaining gaps.

---

## 23. First autonomous implementation task

**Build a working Rune core and CLI foundation, not a giant imaginary Rich/Textual clone.**

1. Audit the current Nox runtime and stdlib for ANSI output, stdin, TTY detection, terminal width, argument access and testing.
2. Create `docs/capabilities.md` and `docs/architecture.md`.
3. Scaffold a valid standalone `rune` package.
4. Implement a small plain/ANSI text renderer with explicit capabilities and safe sanitization.
5. Implement a simple ASCII-correct table renderer and tests; clearly document Unicode-width limitations.
6. Implement a pure CLI argument parser for positional arguments and `--long` options using explicit argument lists.
7. Add auto-help and usage errors using the shared render layer.
8. Compile example programs and execute tests on the installed compiler.
9. Record unsupported interactive features, terminal capabilities and platform limitations.
10. Propose the next incremental milestones.

**Quality bar:** an actual, tested, Nox-native terminal + CLI toolkit beats an impressive-looking but non-compiling framework.

---

## 24. Final rules for agents

- Never infer Nox capabilities from Python syntax alone.
- Never implement an entire framework in one untested step.
- Never use a visual feature that breaks redirected output.
- Never treat terminal escape sequences as harmless text.
- Never couple rendering to CLI command parsing.
- Never claim password masking, raw mode, cursor control or terminal width detection without verifying the underlying API.
- Never let the legacy Onyx name leak into public interfaces.
- Never call the project complete while documented examples fail.
- Prefer correctness, small modules, compatibility and testability over feature count.
