# Rune — Interactive Prompts Guide

> **Module:** `rune.prompt`

Rune provides interactive command-line wizards with strict non-interactive CI safety.

---

## 1. Available Prompt Types

### Text Input (`prompt_text`)
Prompts the user for a single line of text with an optional default value:
```nox
from rune import prompt_text

name: str = prompt_text("Project name", "my_app")
```

### Confirmation (`confirm`)
Prompts for a boolean confirmation (`[Y/n]` or `[y/N]`):
```nox
from rune import confirm

proceed: bool = confirm("Continue with database migration?", False)
```

### Validated Integer Input (`prompt_int`)
Prompts for an integer within specific bounds with retry loops:
```nox
from rune import prompt_int

port: int = prompt_int("Worker threads", 4, 1, 64, 3) # default: 4, min: 1, max: 64, max_retries: 3
```

### Single Choice Selection (`select_choice`)
Presents a numbered list of choices:
```nox
from rune import select_choice

env: str = select_choice("Target environment", ["dev", "staging", "prod"], "dev")
```

### Multi-Selection (`select_multiple`)
Allows selecting several items using numbers (e.g., `1, 3`), names, or `all`:
```nox
from rune import select_multiple

addons: list[str] = select_multiple(
    "Choose plugins to install",
    ["metrics", "tracing", "profiler", "alerting"],
    ["metrics"]
)
```

---

## 2. Non-Interactive CI Safety

In automated environments (CI/CD pipelines, piped inputs, cron jobs), interactive prompts must not hang indefinitely waiting for stdin.

Rune enforces the following invariants:
1. If `is_tty` is `False` and a `default_val` is supplied, the prompt **returns the default value immediately**.
2. If `is_tty` is `False` and **no default value exists**, Rune raises `NonInteractiveError`.
3. Rune never silently fabricates an arbitrary value (such as 0 or empty string) for required fields.

---

## 3. Password Input Status

In accordance with `AGENTS.md §11`, `prompt_password()` explicitly raises `PromptError` acknowledging the lack of portable terminal echo-suppression in the current Nox runtime. Rune will never claim masked password entry without native no-echo support.
