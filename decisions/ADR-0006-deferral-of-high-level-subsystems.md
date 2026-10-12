# ADR-0006: Staged Development and Deferral of High-Level Subsystems

- **Status:** Accepted
- **Date:** Architectural Baseline Phase
- **Deciders:** ARK-OS Project Maintainers

---

## Context and Problem Statement

Building an operating system for daily use requires dozens of distinct subsystems, including graphical display servers, window compositors, audio daemons, complex inter-process communication frameworks, application sandboxes, and software packaging managers.

Attempting to design or specify all of these high-level subsystems before establishing a functional kernel, boot handoff, init runtime, and interactive shell leads to architectural bloat, speculative designs, and unmaintainable complexity.

---

## Decision

ARK-OS will strictly defer the design and implementation of high-level subsystems until foundational milestones are reached:

1. **Desktop & Graphics Compositor (Deferred):**
   - No compositor or display protocol (e.g., Wayland or DRM-KMS framebuffer) will be implemented until early userspace (`arkrt` as PID 1) and interactive shell access (`sash`) are functional on the serial and virtual console.
2. **Advanced IPC Protocols (Deferred):**
   - Advanced binary protocols or desktop message buses are deferred. Early system communication will rely strictly on standard Unix domain sockets with simple, versioned line-delimited or framed JSON payloads as needed by initial services.
3. **Application Sandboxing and Packaging Bundles (Deferred):**
   - Application containerization (namespaces, cgroups, Landlock, seccomp filters) and `.app`-style bundle directory formats are deferred. Initial applications will be standard Linux executables using established POSIX file permissions.

---

## Consequences

### Positive
- Keeps early development focused entirely on core correctness: kernel boot, PID 1 execution, and shell interaction.
- Avoids premature commitments to graphics or sandboxing models before actual system performance and interface characteristics are understood.
- Adheres strictly to [AGENTS.md](file:///home/aarav/repo/arkos/AGENTS.md) Section 1.2 (preventing premature abstractions and bloat).

### Negative / Trade-Offs
- The system will remain text-mode and CLI-only during initial bring-up milestones.
- Application isolation will rely entirely on traditional Unix user/group separation until sandbox policies are formally introduced.
