# Component Specification: libark (Core System Library)

## Overview

- **Component Name:** `libark`
- **Source Location:** `components/libraries/libark/`
- **Role:** Shared First-Party System Library
- **Primary Language:** Swift (with minimal C shims for Linux syscalls)
- **Implementation Status:** Planned (Not implemented)

`libark` is the foundational shared library for ARK-OS. It provides common data structures, typed POSIX wrappers, system call abstractions, IPC client primitives, and core utilities required across first-party userspace programs (including `arkrt`, `sash`, and system daemons).

---

## Core Responsibilities

1. **System Interface Abstraction:**
   - Safe, ergonomic Swift wrappers around low-level Linux system calls (e.g., `mount`, `unmount`, `fork`, `execve`, `pipe2`, `epoll`, `signalfd`).
   - Strongly typed error handling replacing raw integer `errno` codes.
2. **File and Path Primitives:**
   - Safe path resolution and manipulation avoiding path traversal vulnerabilities.
   - Filesystem metadata inspection, permissions management, and file descriptor ownership wrappers.
3. **Inter-Process Communication (IPC) Primitives:**
   - Unix domain socket framing and serialization helpers.
   - Client wrappers for communicating with `arkrt` and core system services.
4. **Process and Environment Management:**
   - Process spawning, environment variable encapsulation, and signal mask manipulation.
   - Resource limit queries (`getrlimit`, `setrlimit`).

---

## Design Invariants and Constraints

- **Minimal Dependency Surface:** `libark` must not depend on high-level services, shells, applications, or external non-standard packages. It must link cleanly with standard Linux C libraries (`glibc` or `musl`) and the Swift core runtime.
- **No Miscellaneous Bloat:** Functionality is added only when it represents a shared operating-system abstraction needed by multiple core components. It must not become a dumping ground for application-specific helpers.
- **Memory Safety and Lifetime Invariants:** Pointers received from low-level C APIs must be converted immediately into safe Swift managed structures. Unsafe pointer scope must be minimized and documented.
- **Concurrency Safety:** Public types must be `Sendable` where appropriate, respecting Swift concurrency invariants.

---

## Intended Interfaces (Planned)

```swift
// Illustrative architectural interface outline (not implemented yet)

public struct SystemPath: Sendable, Equatable {
    public let rawPath: String
    public init(validating path: String) throws
}

public enum SystemError: Error, Sendable {
    case permissionDenied
    case fileNotFound
    case invalidArgument
    case ioError(errno: Int32)
}

public struct ProcessSpawnConfig: Sendable {
    public var executable: SystemPath
    public var arguments: [String]
    public var environment: [String: String]
    public var standardInput: FileDescriptorOwnership
    public var standardOutput: FileDescriptorOwnership
    public var standardError: FileDescriptorOwnership
}
```
