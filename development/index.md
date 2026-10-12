# Development Environment and Workflow Guide

## Overview

This guide explains how to set up a developer environment, navigate the repository structure, execute available inspection checks, and adhere to development workflows for ARK-OS.

ARK-OS is currently at the architectural baseline stage. No build commands (such as `swift build` or `make`) are operational yet, and running unverified build commands is prohibited under [AGENTS.md](file:///home/aarav/repo/arkos/AGENTS.md) Section 6.

---

## 1. Development Prerequisites

To prepare for future implementation work, a Linux development host with the following packages is recommended:

- **Operating System:** Linux x86_64 (Debian 12+, Ubuntu 22.04 LTS+, or compatible)
- **Swift Toolchain:** Swift 5.10 or newer (downloadable from https://swift.org)
- **C/C++ Compiler & Tools:** `clang` (version 16+), `gcc`, `cmake`, `ninja-build`
- **Virtualization Tools:** `qemu-system-x86`, `qemu-utils`
- **Inspection Tools:** `git`, standard POSIX utilities

---

## 2. Repository Layout and Component Ownership

When making changes, place files strictly in their owning directory as specified in [AGENTS.md](file:///home/aarav/repo/arkos/AGENTS.md) Section 3:

- `components/runtime/arkrt/`: Runtime and PID 1 init code.
- `components/libraries/libark/`: Shared system library code.
- `components/shell/sash/`: First-party command-line shell code.
- `components/services/`: First-party system daemons.
- `components/apps/`: First-party user applications.
- `host-tools/aake/`: Build and packaging orchestration tool.
- `host-tools/signing/`: Host-side cryptographic signing utilities.
- `manifests/`: Declarative configuration files (`components.toml`, `sources.lock.toml`, `targets/`).
- `docs/`: Canonical architecture, bring-up, decision, development, and licensing documentation.

Do not introduce arbitrary root-level files or directories.

---

## 3. Available Checks and Validation

Currently, validation in this repository consists of non-destructive static analysis and documentation inspection.

### Permitted Inspection Workflows

1. **Repository Structure Verification:**
   Ensure directory layouts comply with [AGENTS.md](file:///home/aarav/repo/arkos/AGENTS.md) Section 3.
2. **Documentation Review:**
   Verify that relative links point to existing files, terminology is consistent, and no promotional hype or emojis are present.
3. **Status Accuracy Checks:**
   Verify that unimplemented components are explicitly marked as **Planned** or **Not implemented**, not as working features.

### Planned Automated Checks (`devtools/check/`)

In future milestones, `devtools/check/` will host deterministic, read-only verification scripts:
- `check_structure.sh`: Validates that files adhere to directory boundaries.
- `check_docs.sh`: Checks markdown links and verifies the absence of emojis.
- `check_manifests.sh`: Validates schema conformance of TOML manifests.

---

## 4. Build and Command Execution Restrictions

In accordance with [AGENTS.md](file:///home/aarav/repo/arkos/AGENTS.md) Section 6:

- **Never execute build commands** (`swift build`, `make`, `ninja`, or custom compiler commands) without explicit authorization from the user.
- **Never execute kernel or firmware compilation** or system image flashing.
- When working on documentation or planning code, rely on static inspection. Clearly report what was inspected and what remains unverified.

---

## 5. Troubleshooting and FAQ

- **Q: Why are there no binaries or build scripts in `components/`?**
  **A:** ARK-OS is at the architectural baseline phase. Components are currently specified in documentation and manifests; implementation has not yet begun.
- **Q: Can I run `swift build` in `components/runtime/arkrt`?**
  **A:** No source code or `Package.swift` currently exists in that directory. Furthermore, unauthorized build commands are strictly restricted.
- **Q: Where should architectural questions and design proposals be documented?**
  **A:** Proposals should be documented as Architecture Decision Records in `docs/decisions/`.
