# Contributing to Rune

We welcome contributions to Rune! To ensure that Rune remains rock-solid, predictable, and fully compliant with the Nox language, please follow these guidelines.

---

## 1. Compliance with Architecture & AGENTS.md

All contributions must strictly follow the architectural boundaries, dependency directions, and safety invariants defined in `AGENTS.md`:
- **Pure Statically-Typed Nox**: Never introduce compiler forks, unsafe native C shims, or unverified syntax.
- **Strict Dependency Hierarchy**:
  - `core` does not depend on anything.
  - `render`, `progress`, and `prompt` depend only on `core`.
  - `cli` and `tui` may consume lower-level modules.
  - No circular imports across modules.
- **Process Cleanup**: Never call `nox.os.exit()` inside rendering, parsing, or widget layers; process exit is reserved exclusively for the top-level application runner.
- **Graceful Degradation**: Plain-text fallback and non-interactive environments (CI, redirected stdout) must remain 100% supported and tested.

---

## 2. Local Development & Verification

Before submitting changes, ensure the entire toolchain passes cleanly:

```bash
# 1. Type check core packages
noxc check rune.nox
noxc check main.nox

# 2. Run the full test suite (22+ suites across unit, integration, snapshot)
noxc test

# 3. Run performance benchmarks
noxc run benchmarks/run_benchmarks.nox

# 4. Optional: Format code in-place
noxc fmt <file.nox>
```

---

## 3. Pull Request Guidelines

1. **Test Coverage**: Every bug fix or new feature must include automated tests under `tests/unit/`, `tests/integration/`, or `tests/snapshot/`.
2. **Documentation**: Update `README.md`, `ROADMAP.md`, or relevant guides under `docs/` for any user-facing API changes.
3. **Changelog**: Add an entry under `## [Unreleased]` in `CHANGELOG.md` following [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) standards.
4. **CI Verification**: Ensure all matrix builds pass green across Linux (`ubuntu-latest`), macOS (`macos-latest`), and Windows (`windows-latest`).
