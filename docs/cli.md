# Rune — CLI Framework Guide

> **Module:** `rune.cli`

Rune's CLI engine provides declarative command routing, type-safe arguments, rich option handling, nested subcommands, automated `--help` screens, and shell autocompletion.

---

## 1. Defining Commands

Commands are declared using `Command`:

```nox
from rune import Command, Argument, Option, CliApp, CliResult, TYPE_STR, TYPE_INT, TYPE_BOOL

cmd: Command = Command("deploy", "Deploy the project to remote cluster")
```

---

## 2. Positional Arguments

Arguments are ordered, typed positional inputs:

```nox
# add_argument(name, type_str, required, default_val, description, choices)
cmd.add_argument("environment", TYPE_STR, True, "", "Target environment", ["staging", "production"])
cmd.add_argument("replicas", TYPE_INT, False, "1", "Number of instances")
```

---

## 3. Options & Flags

Options can have long names (`--port`), short aliases (`-p`), environment fallbacks, and validation choices:

```nox
# add_option(name, short_name, type_str, is_flag, required, default_val, description, choices, env_var, multiple, is_secret)
cmd.add_option("port", "p", TYPE_INT, False, False, "8080", "Server listen port")
cmd.add_option("verbose", "v", TYPE_BOOL, True, False, "false", "Enable verbose diagnostic logs")
cmd.add_option("token", "t", TYPE_STR, False, False, "", "API token", [], "AUTH_TOKEN", False, True)
cmd.add_option("include", "i", TYPE_STR, False, False, "", "Package pattern", [], "", True, False)
```

### Option Features
1. **Clustered Flags:** Short flags can be grouped (`-xvf` expands to `-x`, `-v`, `-f`).
2. **Value Precedence:** `CLI arguments > Environment variables > Application defaults`.
3. **Repeated Options:** Multiple occurrences (`-i pkg1 -i pkg2`) collected via `ctx.get_list("include")`.
4. **Secret Masking:** Options flagged with `is_secret = True` have their default masked as `[default: ******]` in `--help`.
5. **End of Options (`--`):** Everything following `--` is treated strictly as positional.

---

## 4. Subcommands & Aliases

Commands can be arbitrarily nested:

```nox
parent: Command = Command("docker", "Container orchestration CLI")
build_sub: Command = Command("build", "Build image from Dockerfile")
build_sub.add_alias("b")

parent.add_subcommand(build_sub)
```

---

## 5. Execution & Parse Results

`CliApp` wraps the root command and manages execution:

```nox
app: CliApp = CliApp(cmd, "1.0.0")

# Parse argument list (excludes argv[0])
res: CliResult = app.parse(["--port", "9000", "staging"])

if not res.success:
    app.console.print("Error: " + res.error_message)
else:
    port: int = res.context.get_int("port", 8080)
    env: str = res.context.get("environment", "staging")
```

### Exit Codes
- `0`: Successful execution, `--help`, or `--version`.
- `2`: Usage or argument parsing error.
- `1`: Application-level failure.

Rune core **never calls `nox.os.exit()`**. Process termination is exclusively owned by the caller.

---

## 6. Shell Autocompletion

Rune can generate static, injection-safe completion scripts:

```nox
from rune import generate_completion

# Generate Bash completion
bash_script: str = generate_completion(cmd, "bash")

# Generate Zsh completion
zsh_script: str = generate_completion(cmd, "zsh")

# Generate Fish completion
fish_script: str = generate_completion(cmd, "fish")
```
