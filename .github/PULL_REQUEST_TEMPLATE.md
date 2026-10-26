## Summary
<!-- Briefly explain the motivation and context for this change. -->

## Changes
<!-- List the specific architectural or implementation changes introduced. -->
- 

## Verification & Testing
<!-- Describe the tests you ran to verify your changes. Include commands used. -->
- [ ] `noxc check rune.nox` (no type errors)
- [ ] `noxc test` (all 22+ suites passing)
- [ ] `noxc run benchmarks/run_benchmarks.nox` (benchmarks meet SLA)
- [ ] Added unit or integration tests for new functionality

## Checklist
- [ ] Adheres to `AGENTS.md` guidelines and module dependency directions (`core` -> `render` / `progress` / `prompt` -> `cli` / `tui`).
- [ ] No compiler forks or unsafe native shims introduced.
- [ ] Non-interactive and plain-text fallback preserved.
- [ ] ANSI sequences sanitized in untrusted text paths.
- [ ] Updated `CHANGELOG.md` following Keep a Changelog standards.
