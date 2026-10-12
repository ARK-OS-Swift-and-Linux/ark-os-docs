# ADR-0001: Adoption of the Linux Kernel as Operating System Foundation

- **Status:** Accepted
- **Date:** Architectural Baseline Phase
- **Deciders:** ARK-OS Project Maintainers

---

## Context and Problem Statement

An operating system requires a kernel to manage processor execution, virtual memory paging, process scheduling, inter-process communication, and hardware device drivers (storage controllers, networking chipsets, display outputs, and input devices).

Developing a custom microkernel or monolithic kernel from scratch requires years of specialized engineering and produces severe driver compatibility deficits, limiting the OS to a narrow set of virtualized devices. The project objective is to create an understandable, customizable, and secure operating system with a Swift-first userspace for daily use, not to reinvent low-level hardware drivers.

---

## Considered Alternatives

1. **Custom Microkernel (from scratch in Rust, Zig, or C):**
   - *Advantages:* Minimal trusted computing base, formal isolation boundaries.
   - *Disadvantages:* Extreme engineering overhead, no production-grade device drivers, complex userspace driver IPC, severe delay in delivering a daily-use operating system.
2. **BSD Kernel (FreeBSD or NetBSD kernel foundation):**
   - *Advantages:* Clean architectural organization, permissive licensing.
   - *Disadvantages:* Smaller hardware support matrix than Linux, less mature Swift toolchain and runtime integration.
3. **Linux Kernel (Monolithic Upstream Kernel):**
   - *Advantages:* Broadest hardware and architecture support (x86_64, ARM64, RISC-V), mature virtual memory and scheduler implementations, extensive virtualization support (KVM), full support for the modern Swift static Linux SDK.
   - *Disadvantages:* Large monolithic codebase, GPL-2.0 copyleft obligations for kernel modifications.

---

## Decision

Adopt the standard upstream Linux kernel as the operating-system foundation for ARK-OS.

ARK-OS will not maintain a custom microkernel or proprietary kernel fork. All first-party engineering effort will focus on early userspace (`arkrt`), core libraries (`libark`), system daemons (`components/services`), the shell (`sash`), and user applications (`components/apps`).

---

## Consequences

### Positive
- Instant hardware support across standard x86_64 PCs and QEMU virtual machines.
- Reliable, mature process scheduling and memory management.
- Ability to utilize the official Swift static Linux runtime SDK.
- Clear separation between kernel-space driver operations and first-party userspace architecture.

### Negative / Trade-Offs
- The kernel is monolithic, requiring careful userspace sandboxing to enforce security boundaries.
- Kernel binary distributions must comply with the GNU General Public License v2 (GPL-2.0).
- Userspace components must interface via standard Linux system call ABIs.
