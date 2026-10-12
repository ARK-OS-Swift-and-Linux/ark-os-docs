# ADR-0005: Host Tooling and Build Orchestration Strategy

- **Status:** Accepted
- **Date:** Architectural Baseline Phase
- **Deciders:** ARK-OS Project Maintainers

---

## Context and Problem Statement

Building ARK-OS requires compiling first-party Swift and C components, configuring and building the upstream Linux kernel, and packing the resulting binaries and configuration into a bootable initramfs archive.

The repository designates `host-tools/aake/` as the build orchestrator. However, attempting to implement `aake` as a complex, monolithic tool in Swift on day one introduces a circular bootstrapping requirement: a developer would need an advanced build system just to build the build system that compiles the OS components.

---

## Considered Alternatives

1. **Monolithic Swift Build Tool on Day One:**
   - *Mechanism:* Implement `aake` as a full-featured Swift package parsing TOML manifests, orchestrating compilers, and packaging images.
   - *Advantages:* Dogfoods Swift across the entire project repository.
   - *Disadvantages:* Creates a host bootstrapping barrier; requires substantial upfront implementation before a single operating system binary can be built.
2. **Custom Compiler / Build Engine from Scratch:**
   - *Mechanism:* Re-implement compiler wrappers and linker drivers inside `aake`.
   - *Advantages:* Complete internal control.
   - *Disadvantages:* Severe reinvention of existing build engines (Ninja, SwiftPM, CMake); explicitly violates [AGENTS.md](file:///home/aarav/repo/arkos/AGENTS.md) Section 1.2 and Section 3.2.
3. **Staged Orchestration: Lightweight Scripts First, Tool Later (Selected):**
   - *Mechanism:* Clearly separate build *responsibilities* from tool *implementation language*. For initial bring-up, use lightweight, auditable POSIX host scripts (or a minimal Python script) that directly invoke standard upstream toolchains (`swift build`, `make`, `cpio`). Evolve `aake` into a formal tool once the build pipeline is proven.
   - *Advantages:* Zero bootstrap friction; immediately enables building the minimal kernel and initramfs; keeps host tooling simple and reproducible.
   - *Disadvantages:* Early build invocation relies on shell scripts rather than a single unified binary CLI.

---

## Decision

ARK-OS will adopt a **staged build orchestration strategy**:

1. **Responsibility Separation:** Build orchestration is strictly defined as invoking upstream compilers and packing artifacts. `aake` will not replace or duplicate SwiftPM, Clang, or the Linux kernel build system (`kbuild`).
2. **Early Bring-Up Phase:** Host orchestration will be driven by minimal, auditable host scripts located in `devtools/` or `host-tools/` that:
   - Invoke `kbuild` to compile the minimal Linux kernel with the in-tree `defconfig`.
   - Invoke standard Swift toolchains (SwiftPM with static Linux SDK) to compile `arkrt` and `sash`.
   - Assemble target binaries into an initramfs CPIO archive using standard archive utilities (`find . | cpio -o -H newc | gzip`).
3. **Evolution to `aake`:** A dedicated `aake` binary will only be implemented after the build steps and manifest schemas are validated by working bring-up scripts.

---

## Consequences

### Positive
- Removes premature complexity from the early bring-up workflow.
- Avoids circular dependency issues when bootstrapping the project on new developer machines.
- Leverages mature, standard build tools rather than reinventing compiler drivers.

### Negative / Trade-Offs
- Early build commands will be script-driven rather than unified under a single `aake` executable.
