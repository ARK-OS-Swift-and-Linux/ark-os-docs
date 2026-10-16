# ARK-OS Documentation

## Overview

The `docs/` repository contains the technical documentation baseline, architectural specifications, design decision records, and development manuals for ARK-OS.

Documentation in this repository acts as the single source of truth for architectural invariants, component boundaries, and implementation requirements across the project.

---

## Core Documentation

- **[System State, Architecture, and Verification Guide](file:///home/aarav/repo/arkos/docs/architecture/ARK-OS-STATE-AND-ARCHITECTURE.md):** Canonical, evidence-based status of the entire ARK-OS ecosystem, verification matrices, component implementation inventory, and subsystem architecture.

---

## Directory Organization

- `architecture/` - High-level system architecture, component specifications (`components/`), subsystem designs, and IPC models. See [System State and Architecture](file:///home/aarav/repo/arkos/docs/architecture/ARK-OS-STATE-AND-ARCHITECTURE.md).
- `decisions/` - Architecture Decision Records (ADRs 0001 through 0006) recording accepted technical decisions, considered alternatives, and the unresolved-decision catalog.
- `bringup/` - Step-by-step procedures, emulator parameters, and milestone criteria for bringing up ARK-OS on target platforms.
- `development/` - Developer guides, toolchain setup requirements, coding standards, and debugging workflows.
- `licensing/` - Licensing policies, dual-licensing boundaries (GPL kernel vs. Apache-2.0 userspace), and third-party compliance standards.

---

## Documentation Policies

- Maintain factual, evidence-based descriptions. Speculative features must be labeled as proposed or deferred.
- Avoid promotional language, emojis, and unverified claims.
- Keep links and cross-references synchronized across the documentation tree.

---

## License

This documentation is licensed under the Apache License, Version 2.0. See [LICENSE](file:///home/aarav/repo/arkos/docs/LICENSE) for the full license text.
