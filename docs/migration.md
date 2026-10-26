# Rune — Migration & Stability Contract

> **Current Version:** `1.0.1`

Rune follows strict Semantic Versioning (`MAJOR.MINOR.PATCH`).

---

## 1. 1.0 Stability Guarantees

As of `v1.0.0`, all public APIs across `rune`, `rune.core`, `rune.render`, `rune.progress`, `rune.prompt`, and `rune.cli` are **frozen and backward-compatible**:

1. No public function, class, or parameter will be removed or altered incompatibly in any `1.x` release.
2. New features in `1.x` will be strictly additive.
3. Breaking changes will occur only in major version increments (`2.0.0`), preceded by documented deprecation warnings.

---

## 2. Migrating from 0.x to 1.0.x

### Unified Namespace
- Replace legacy references to individual packages with the single unified `rune` import:
  ```nox
  import rune
  from rune import Console, Style, Table, Panel, Command, CliApp
  ```

### Non-Interactive Prompt Behavior (1.0.1 Update)
- If calling `prompt_int()`, `select_choice()`, or `select_multiple()` in automated CI environments, always supply a valid default value. If no default is provided in non-TTY mode, `NonInteractiveError` will be raised to prevent silent fallback or hangs.

### Color Validation (1.0.1 Update)
- RGB color channels and 256-color indices are now strictly clamped to `0..255`.
- Hex strings passed to `hex_to_rgb()` must contain exactly 6 valid hex characters (`#RRGGBB`).
