# Component Specification: sash (ARK Shell)

## Overview

- **Component Name:** `sash`
- **Source Location:** `components/shell/sash/`
- **Role:** Interactive Command-Line Shell and Script Runner
- **Primary Language:** Swift
- **Implementation Status:** Planned (Not implemented)

`sash` is the first-party command-line shell designed for ARK-OS. It provides an intuitive, responsive command line for interactive system exploration, administrative tasks, and script execution.

---

## Core Responsibilities

1. **Interactive Command Line:**
   - Command input reading, line editing, and history management.
   - Lexical tokenization, quotation parsing, and argument segmentation.
2. **Command Execution:**
   - Executable path lookup across the system `PATH`.
   - Process spawning, standard stream redirection (`<`, `>`, `>>`), and pipeline management (`|`).
   - Signal propagation (`Ctrl+C` / `SIGINT` to foreground process groups).
3. **Built-in Commands:**
   - Essential shell built-ins: `cd`, `pwd`, `exit`, `help`, `export`, `unset`, `history`.
   - Service control built-ins communicating with `arkrt` via `libark` (e.g., `service start <name>`, `service status`).
4. **Script Execution:**
   - Deterministic execution of shell script files.

---

## Design Invariants and Constraints

- **Compatibility Stance:** `sash` is an independent shell designed for clarity and safety. It does **not** claim full POSIX.2 or Bash compatibility unless specific features are implemented, verified, and documented. Scripts must not rely on undocumented Bash-specific syntax.
- **Reuse `libark`:** Process execution, path validation, and system calls must reuse `libark` abstractions rather than duplicating low-level C shims inside the shell.
- **Predictable Error Handling:** Malformed user input, missing commands, or permission errors must produce clear, readable error messages without terminating an interactive session.
- **No Service Bloat:** `sash` is a user interface and script executor. It must not implement system daemons, hardware device monitoring, or networking services inside the shell binary.
