# Component Specification: Applications Architecture

## Overview

- **Component Area:** `components/apps/`
- **Role:** User-Facing CLI and Graphical Applications
- **Primary Language:** Swift
- **Implementation Status:** Planned (Not implemented)

The `components/apps/` directory is reserved for first-party applications running on ARK-OS. Applications represent top-level programs intended for user interaction, distinct from background system daemons and low-level runtime processes.

---

## Application Model

1. **Self-Contained Structure:** Each application resides in its own isolated subdirectory under `components/apps/`, containing its main entry point (`main.swift`), configuration, and assets.
2. **Library Reuse:** Applications must consume common system interfaces, data structures, and IPC clients through `libark`. They must not duplicate low-level system code.
3. **Execution Context:** Applications execute unprivileged under regular user credentials. They interact with system services via defined IPC channels rather than direct privileged system manipulation.
4. **Sandboxing (Planned):** In future iterations, applications will execute within isolated namespace boundaries (using Linux user/mount namespaces and seccomp filters) to prevent unauthorized access to host resources.

---

## Planned Initial Applications

- **System Diagnostics Utility (`ark-info`):** A command-line tool reporting operating-system version, kernel parameters, mounted filesystems, and service health.
- **Service Manager Client (`ark-service`):** A command-line client interacting with `arkrt` to list, inspect, start, and stop system services.

---

## Design Invariants and Constraints

- **No System Daemon Logic:** Background services running independent of a user session belong in `components/services/`, not `components/apps/`.
- **No Compiler/Host Logic:** Build tools and host orchestration logic belong in `host-tools/`, not in target application directories.
- **Resource Containment:** Application resources (icons, metadata, configuration schemas) must remain inside the application's source directory.
