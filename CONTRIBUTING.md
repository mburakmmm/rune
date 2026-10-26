# Contributing to Rune

We welcome contributions to Rune! To ensure that Rune remains rock-solid, predictable, and fully compliant with the Nox language, please follow these guidelines.

---

## 1. Compliance with AGENTS.md

All contributions must strictly follow the architectural boundaries, dependency directions, and safety invariants defined in `AGENTS.md`:
- Pure statically typed Nox code only.
- Strict dependency hierarchy: `core` -> `render` / `progress` / `prompt` -> `cli`.
- Never introduce circular dependencies.
- Never call `nox.os.exit()` inside rendering or parsing core libraries.
- Support plain-text and non-TTY execution everywhere.

---

## 2. Testing & Verification

Every bug fix or new feature must include corresponding automated tests under `tests/unit/` or `tests/integration/`:
```bash
# Type check all files
noxc check <file.nox>

# Run full test suite
noxc test

# Format code in-place
noxc fmt <file.nox>
```

---

## 3. Pull Request Guidelines

1. Ensure `noxc test` passes completely (0 failures).
2. Document new public APIs in `docs/api.md` and `README.md`.
3. Add a summary entry to `CHANGELOG.md`.
