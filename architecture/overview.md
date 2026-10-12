# ARK-OS Architecture Overview

## 1. System Vision and Overview

ARK-OS is an independent operating system designed to combine the accessibility, coherence, and safety of modern mobile and desktop platforms with the open, modular, and customizable nature of Unix-like systems.

The operating system is architected around two primary structural decisions:
1. **The Linux Kernel as System Foundation:** Leveraging a proven, production-grade monolithic kernel for device driver support, hardware enablement, memory paging, and process scheduling.
2. **A Swift-First Userspace:** Implementing first-party runtime, system library, shell, and service components in Swift on Linux, taking advantage of modern type safety, structured concurrency, memory safety by default, and expressive systems programming features.

---

## 2. System Layering and Component Boundaries

ARK-OS is structured into strict horizontal layers with defined communication boundaries:

```
+-----------------------------------------------------------------+
| Applications (components/apps)                                  |
| First-party and third-party user applications (sandboxed)       |
+-----------------------------------------------------------------+
| Shell & User Environment (components/shell/sash)                |
| Command-line interface, interactive built-ins, scripting        |
+-----------------------------------------------------------------+
| System Services (components/services)                           |
| Modular background daemons running under dedicated UIDs         |
+-----------------------------------------------------------------+
| Core System Library (components/libraries/libark)               |
| Swift & C shared library, system calls, IPC clients, models     |
+-----------------------------------------------------------------+
| System Runtime & Init (components/runtime/arkrt) [PID 1]        |
| Early userspace mount, service supervisor, lifecycle manager    |
+-----------------------------------------------------------------+
| Linux Kernel Foundation (Upstream Kernel)                       |
| Memory management, scheduler, hardware drivers, syscall ABI     |
+-----------------------------------------------------------------+
| Target Hardware / Virtual Platform                              |
| x86_64 PC (QEMU/KVM virtual machine), future ARM64 platform     |
+-----------------------------------------------------------------+
```

### Component Boundaries and Contracts

- **Kernel to Userspace Boundary:** Strictly mediated by the Linux System Call Application Binary Interface (`libc` or direct system calls). ARK-OS components do not invent kernel mechanisms or rely on out-of-tree kernel modifications unless established by an accepted decision.
- **Runtime (`arkrt`) Boundary:** Executes as PID 1. It owns early mount setup (`/proc`, `/sys`, `/dev`), process reaping, signal forwarding, and service supervision. It does not implement application logic, graphical compositing, or complex shell parsing.
- **System Library (`libark`) Boundary:** Shared by first-party userspace programs. It exposes high-level, safe Swift interfaces over low-level Linux operations and system IPC, maintaining ABI stability.
- **Service Boundary:** Services are unprivileged processes communicating over defined inter-process communication (IPC) channels. Services must validate all incoming requests at process boundaries.
- **Host Tools vs. Target Software:** Build orchestration (`host-tools/aake`) and image assembly tools run strictly on the developer host machine and are not packaged into the target runtime image.

---

## 3. Intended Boot-to-Userspace Flow

The boot sequence follows a standard Linux-based handoff:

1. **Firmware Initialization:** UEFI or BIOS firmware initializes physical hardware or virtual machine devices and loads the bootloader.
2. **Bootloader Phase (`components/boot`):** The bootloader loads the Linux kernel binary and an initial ramdisk (initramfs) or direct root filesystem into system memory, passing command-line parameters (e.g., `console=ttyS0 init=/sbin/arkrt`).
3. **Kernel Initialization:** The Linux kernel initializes CPU cores, memory paging, device drivers, and mounts the root filesystem.
4. **Init Execution (`arkrt` as PID 1):** The kernel executes `/sbin/arkrt` as the initial user-mode process:
   - Sets up process group and handles signals (`SIGCHLD`, `SIGTERM`, `SIGINT`).
   - Mounts essential virtual filesystems: `/proc` (procfs), `/sys` (sysfs), `/dev` (devtmpfs), and `/run` (tmpfs).
   - Configures system hostname, logging pipes, and basic device nodes.
   - Parses system service manifests from `/etc/ark/services` or compiled configuration.
5. **System Service Launch:** `arkrt` spawns core system daemons in dependency order with appropriate privilege drops.
6. **Shell and User Handoff:** Once basic system services are ready, `arkrt` spawns `sash` on the primary console (or graphical environment when implemented) for user interaction.

---

## 4. Confirmed vs. Unresolved Architectural Decisions

To preserve engineering accuracy, the following table catalogs what has been formally decided versus what remains open:

| Architectural Topic | Status | Decision Summary / Reference |
| :--- | :--- | :--- |
| Kernel Foundation | **Confirmed** | Use upstream Linux kernel. (See [ADR-0001](file:///home/aarav/repo/arkos/docs/decisions/ADR-0001-linux-kernel-foundation.md)). |
| Primary Userspace Language | **Confirmed** | Swift is the primary language for userspace components on Linux. (See [ADR-0002](file:///home/aarav/repo/arkos/docs/decisions/ADR-0002-swift-first-userspace.md)). |
| Kernel Build & Config Strategy | **Confirmed** | Upstream Linux sources with in-tree minimal `defconfig` targeting QEMU/KVM virtio devices. (See [ADR-0003](file:///home/aarav/repo/arkos/docs/decisions/ADR-0003-kernel-configuration-and-build-strategy.md)). |
| Initial Boot Architecture | **Confirmed** | Standalone minimal initramfs in QEMU for early bring-up; multi-partition disk boot deferred. (See [ADR-0004](file:///home/aarav/repo/arkos/docs/decisions/ADR-0004-initial-boot-process-initramfs.md)). |
| Host Tooling Strategy | **Confirmed** | Responsibilities separated from language; lightweight host scripts first, unified `aake` later. (See [ADR-0005](file:///home/aarav/repo/arkos/docs/decisions/ADR-0005-host-tooling-and-build-orchestration.md)). |
| High-Level Subsystems | **Confirmed** | Explicit deferral of desktop/compositor, complex IPC framing, and app sandboxing. (See [ADR-0006](file:///home/aarav/repo/arkos/docs/decisions/ADR-0006-deferral-of-high-level-subsystems.md)). |
| Initial Target Hardware | **Confirmed** | x86_64 under QEMU/KVM virtual machines. |
| Secondary Target Hardware | **Planned** | ARM64 (aarch64); deferred until x86_64 baseline is functional. |
| Service IPC Schemas | **Deferred** | Unix domain sockets with simple versioned JSON framing when first service is built. |
| Contributor Governance | **Open** | DCO vs CLA adoption and automated source-release workflow before binary distribution. |

Detailed context for unresolved and deferred items is documented in [docs/decisions/unresolved-decisions.md](file:///home/aarav/repo/arkos/docs/decisions/unresolved-decisions.md).

---

## 5. Implementation Status

Currently, all layers described above are architectural specifications. No functional code, binaries, or active service daemons exist in the repository. Implementation will proceed incrementally starting with `libark` and `arkrt` early startup.
