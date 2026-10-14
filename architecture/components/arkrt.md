# Component Specification: arkrt (Runtime and Init System)

## Overview

- **Component Name:** `arkrt`
- **Source Location:** `components/runtime/arkrt/`
- **Role:** System Runtime and Initial Userspace Process (PID 1)
- **Primary Language:** Swift / C
- **Implementation Status:** Implemented (Two-stage C starter + Swift PID 1 supervisor with declarative `.serve` service definitions)

`arkrt` is the primary runtime process and initialization manager for ARK-OS. It is executed directly by the Linux kernel as PID 1 upon completing kernel boot. Its core responsibility is establishing early userspace, mounting required virtual filesystems, reaping orphaned child processes, and managing the lifecycle of system services.

---

## Core Responsibilities

1. **PID 1 Process Responsibilities:**
   - Adopting orphaned child processes and reaping zombie processes non-blockingly via `waitpid`.
   - Handling system-wide signals (`SIGTERM`, `SIGINT`, `SIGCHLD`).
   - Triggering clean system shutdown with a 15-second timeout and SIGKILL escalation.
2. **Early Userspace Initialization:**
   - Mounting essential pseudo-filesystems:
     - `/proc` (procfs)
     - `/sys` (sysfs)
     - `/dev` (devtmpfs)
     - `/run` (tmpfs)
     - `/dev/pts` (devpts)
     - `/dev/shm` (tmpfs)
   - Opening and redirecting standard input, output, and error streams to `/dev/console` (with serial fallback).
3. **Service Supervision:**
   - Discovering and parsing declarative `.serve` service definitions from `/etc/ark/services.d/`.
   - Enforcing deterministic topological dependency ordering (`after`, `requires`) and cycle detection.
   - Monitoring service health, restarting failed services according to restart policies (`never`, `always`, `on-failure`), and logging lifecycle transitions.
   - Isolating each service in a dedicated process group (`PGID == PID`) for clean signal broadcast.
4. **Interactive Handoff:**
   - Launching the primary interactive shell (`sash`) on the target console once core services are operational (Planned).

---

## Design Invariants and Constraints

- **Minimal Dependencies:** `arkrt` runs in an unpopulated early userspace. It must not depend on complex shared libraries, external package managers, network sockets, or non-static dependencies that may not be mounted at boot time.
- **Diagnostics and Failure Recovery:** Early startup errors must be emitted directly to the kernel console (`/dev/console` or `/dev/kmsg`). Failures during early mount must produce actionable diagnostics rather than panicking silently.
- **Kernel Relationship:** `arkrt` does not replace Linux kernel scheduling or memory management. It operates entirely in userspace using standard Linux system calls.
- **Safety Invariants:** Avoid unsafe memory manipulations; isolate low-level POSIX bindings behind well-typed Swift abstractions provided by `libark`.

---

## Intended Interfaces

- **Process Entry:** Standard Linux executable entry point (`main`), invoked with kernel boot parameters.
- **Control Interface (Planned):** A local Unix domain socket control channel (`/run/arkrt/control.sock`) allowing system utilities and shells to request service restarts, status queries, or system shutdown.
