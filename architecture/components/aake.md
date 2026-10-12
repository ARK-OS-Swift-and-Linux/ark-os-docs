# Component Specification: aake (Build Orchestrator)

## Overview

- **Component Name:** `aake`
- **Source Location:** `host-tools/aake/`
- **Role:** Host-Side Build and System Image Orchestration Tool
- **Execution Target:** Host Development Machine (Linux)
- **Implementation Status:** Planned (Not implemented)

`aake` is the build orchestration tool designed for ARK-OS development. It coordinates the compilation of first-party Swift and C components, the preparation of kernel assets, and the assembly of root filesystems and disk images.

---

## Core Responsibilities

1. **Manifest Parsing:**
   - Reading component declarations from `manifests/components.toml`.
   - Resolving target architecture profiles from `manifests/targets/` (e.g., `x86_64.toml`, `arm64.toml`).
   - Verifying external source hashes against `manifests/sources.lock.toml`.
2. **Build Toolchain Coordination:**
   - Invoking upstream build tools (Swift Package Manager, Clang, CMake, Ninja) rather than re-implementing compiler or linker internals.
   - Enforcing reproducible build flags, target SDK paths, and sysroot configurations.
3. **Artifact Assembly:**
   - Assembling compiled binaries into the root filesystem staging area according to `packaging/recipes/` and `packaging/rootfs/`.
   - Generating bootable disk images (e.g., raw GPT disk images, ISOs, or QEMU qcow2 images).
4. **Diagnostic Reporting:**
   - Emitting clear, deterministic build logs.
   - Halting immediately on subprocess failure with non-zero exit codes.

---

## Design Invariants and Constraints

- **Host-Only Tool:** `aake` runs strictly on the developer host. It is never installed into or bundled inside the target operating system image.
- **Out-of-Tree Output:** All intermediate object files, staging directories, and generated images must be placed in a dedicated build directory (e.g., `build/` or `.aake/`), never scattered across tracked source directories.
- **No Hidden Modifications:** A build execution must never modify tracked source files, rewrite manifests, or alter host system configuration as a side effect.
- **Deterministic and Actionable Failures:** Do not swallow errors or continue building when a prerequisite tool is missing or a compilation unit fails.
