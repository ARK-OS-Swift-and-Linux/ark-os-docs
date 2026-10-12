# Catalog of Architectural Decisions and Open Topics

## Overview

In accordance with [AGENTS.md](file:///home/aarav/repo/arkos/AGENTS.md) Section 2.1 and Section 3.4, architectural decisions are evaluated sequentially and recorded as immutable Architecture Decision Records (ADRs) once accepted.

This catalog tracks the status of major architectural decisions, linking accepted foundational decisions to their records and documenting remaining open or deferred topics.

---

## 1. Status of Foundational Architectural Decisions

The following foundational decisions have been evaluated, accepted, and recorded:

| Architectural Topic | Resolution Status | Canonical ADR |
| :--- | :--- | :--- |
| Kernel Foundation | **Accepted** | [ADR-0001: Adoption of the Linux Kernel](file:///home/aarav/repo/arkos/docs/decisions/ADR-0001-linux-kernel-foundation.md) |
| Primary Userspace Language | **Accepted** | [ADR-0002: Adoption of Swift as Primary Userspace Language](file:///home/aarav/repo/arkos/docs/decisions/ADR-0002-swift-first-userspace.md) |
| Kernel Configuration & Build | **Accepted** | [ADR-0003: Linux Kernel Configuration and Build Strategy](file:///home/aarav/repo/arkos/docs/decisions/ADR-0003-kernel-configuration-and-build-strategy.md) |
| Initial Boot Architecture | **Accepted** | [ADR-0004: Initial Boot Process and Initramfs Architecture](file:///home/aarav/repo/arkos/docs/decisions/ADR-0004-initial-boot-process-initramfs.md) |
| Host Tooling Strategy | **Accepted** | [ADR-0005: Host Tooling and Build Orchestration Strategy](file:///home/aarav/repo/arkos/docs/decisions/ADR-0005-host-tooling-and-build-orchestration.md) |
| High-Level Subsystems Deferral | **Accepted** | [ADR-0006: Staged Development and Deferral of High-Level Subsystems](file:///home/aarav/repo/arkos/docs/decisions/ADR-0006-deferral-of-high-level-subsystems.md) |

---

## 2. Deferred Subsystems

In accordance with [ADR-0006](file:///home/aarav/repo/arkos/docs/decisions/ADR-0006-deferral-of-high-level-subsystems.md), the following subsystems are explicitly deferred until core userspace execution is verified:

### 2.1 Desktop Graphics and Window Compositor
- **Status:** Deferred
- **Context:** A daily-use OS eventually requires a graphical desktop.
- **Approach:** Focus strictly on kernel boot, PID 1 execution, and interactive shell on the console first. Graphics (Wayland compositor vs DRM-KMS) will be addressed after CLI baseline verification.

### 2.2 Application Sandboxing and Packaging Bundles
- **Status:** Deferred
- **Context:** Application isolation models (namespaces, seccomp, Landlock) and bundle formats (`.app`-style directories).
- **Approach:** Initial applications will be standard Linux executables with standard POSIX permissions until system runtime interfaces stabilize.

---

## 3. Active Open Decisions

The following topics remain open and will be resolved as implementation proceeds:

### 3.1 Service IPC Schema and Wire Framing
- **Status:** Open (To be resolved when `components/services/` introduces its first daemon)
- **Context:** First-party services need a structured wire protocol over Unix domain sockets.
- **Options Under Consideration:**
  1. *Line-delimited JSON over Unix domain stream sockets:* Human-readable, simple to debug with `nc` or `socat`, native in Swift via `JSONEncoder`/`JSONDecoder`.
  2. *Binary framed TLV (Type-Length-Value) or CBOR:* Slightly more compact, but harder to inspect without specialized tooling.
- **Required Information:** Message throughput and payload frequency of the initial logging daemon (`logd`).

### 3.2 Contributor Governance: DCO vs. CLA
- **Status:** Open (To be resolved before accepting external contributions)
- **Context:** The repository is licensed under Apache-2.0. The project must establish its contribution certification mechanism.
- **Options Under Consideration:**
  1. *Developer Certificate of Origin (DCO):* Contributors sign off (`Signed-off-by`) certifying rights to contribute under Apache-2.0 (same model as the Linux kernel). Lightweight and standard.
  2. *Contributor License Agreement (CLA):* Requires explicit electronic agreement signing. Higher administrative overhead.
- **Recommendation:** Developer Certificate of Origin (DCO).

### 3.3 Upstream GPL Source Release Automation
- **Status:** Open (To be resolved before distributing compiled system images)
- **Context:** Distributing pre-compiled Linux kernel binaries requires providing complete corresponding source code under GPL-2.0.
- **Requirements:** Establish automated release packaging scripts in `devtools/release/` that archive the exact kernel tree, `.config`, and build scripts alongside binary releases.

---

## 4. Next Implementation Sequence

1. **Step 2: `arkrt` Early Userspace Boundaries:** Specify the exact syscalls and mount operations `arkrt` executes as PID 1 vs kernel defaults.
2. **Step 3: Initial Bring-Up Scripts:** Implement the minimal host build script to retrieve the pinned kernel, compile with `defconfig`, and assemble the test initramfs for QEMU.
3. **Step 4: IPC Protocol Specification:** Resolve Section 3.1 when `logd` or `devd` is introduced.
4. **Step 5: Release Governance:** Finalize Section 3.2 and 3.3 before publishing release images.
