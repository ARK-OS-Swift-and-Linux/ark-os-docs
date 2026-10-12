# ADR-0002: Adoption of Swift as Primary Userspace Language

- **Status:** Accepted
- **Date:** Architectural Baseline Phase
- **Deciders:** ARK-OS Project Maintainers

---

## Context and Problem Statement

Traditional Unix-like operating-system userspaces (such as standard GNU/Linux distributions) are predominantly implemented in C. While C provides unmatched low-level control, it lacks memory safety by default, has weak type encapsulation, lacks structured error handling, and relies on manual memory management that frequently results in buffer overflows, use-after-free defects, and privilege escalation vulnerabilities.

ARK-OS seeks to build a modern, robust, and maintainable userspace that combines high performance with memory safety, strong typing, and modern language semantics.

---

## Considered Alternatives

1. **C / C++:**
   - *Advantages:* Universal tooling, zero runtime overhead, direct POSIX integration.
   - *Disadvantages:* Inherently prone to memory-safety bugs, lack of modern concurrency safety, higher maintenance burden.
2. **Rust:**
   - *Advantages:* Industry-leading borrow checker, zero-cost abstractions, strong systems community.
   - *Disadvantages:* Steep learning curve, verbose syntax for rapid userspace experimentation, complex C/C++ cross-interoperability compared to Swift's native bidirectional C bridging.
3. **Swift (on Linux):**
   - *Advantages:* Memory-safe by default, expressive syntax, powerful type system, structured concurrency (`async`/`await`, actors, `Sendable`), automatic reference counting (ARC), excellent bidirectional C interop, and first-class static Linux SDK support in Swift 5.10+.
   - *Disadvantages:* Larger runtime footprint than pure C, dependency on the Swift runtime library and libdispatch (mitigated by static Linux linking), historically smaller server/Linux systems ecosystem than C or Rust.

---

## Decision

Adopt Swift as the primary programming language for first-party ARK-OS userspace components:
- System Runtime & Init (`components/runtime/arkrt`)
- Core System Library (`components/libraries/libark`)
- Interactive Shell (`components/shell/sash`)
- System Daemons (`components/services/`)
- User Applications (`components/apps/`)

C and C++ are permitted for low-level system call shims, kernel-facing data structures, and architecture-specific assembly wrappers where Swift cannot directly express the required semantics.

---

## Consequences

### Positive
- Memory safety across core system daemons, reducing vulnerability classes (buffer overflows, dangling pointers).
- High-level, ergonomic abstractions for system calls and IPC.
- Structured concurrency and task cancellation natively supported in service daemons.
- Direct C interoperability allows seamless integration with Linux kernel headers and standard C libraries.

### Negative / Trade-Offs
- The Swift runtime must be present in early userspace, requiring static linking or bundling the Swift core runtime into the root filesystem.
- Developers contributing to first-party components must be proficient in modern Swift and systems programming concepts on Linux.
- Must avoid Apple-specific frameworks (such as macOS Foundation APIs or SwiftUI) and restrict code strictly to cross-platform Swift on Linux.
