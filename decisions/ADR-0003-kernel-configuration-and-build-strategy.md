# ADR-0003: Linux Kernel Configuration and Build Strategy

- **Status:** Accepted
- **Date:** Architectural Baseline Phase
- **Deciders:** ARK-OS Project Maintainers

---

## Context and Problem Statement

Following [ADR-0001](file:///home/aarav/repo/arkos/docs/decisions/ADR-0001-linux-kernel-foundation.md), ARK-OS utilizes the Linux kernel as its system foundation. However, the project must determine how the kernel source is obtained, how its configuration is maintained, and how kernel binaries are compiled for target images.

Using an external distribution's pre-compiled kernel package (e.g., from Debian or Alpine) introduces third-party dependencies, includes unnecessary drivers, and reduces reproducibility. Conversely, maintaining a full downstream kernel fork introduces heavy maintenance overhead.

---

## Considered Alternatives

1. **Distribution Kernel Package Integration:**
   - *Mechanism:* Download pre-compiled kernel packages from an existing distribution.
   - *Advantages:* Zero kernel compile time during development.
   - *Disadvantages:* Couples ARK-OS to an external package ecosystem; includes hundreds of unused drivers; reduces control over boot parameters and virtual filesystem behavior.
2. **Downstream Kernel Fork:**
   - *Mechanism:* Maintain an independent Git repository or fork with custom patches.
   - *Advantages:* Maximum customization.
   - *Disadvantages:* Severe patch rebasing overhead against upstream Linux releases; violates the principle of avoiding unnecessary divergence.
3. **Upstream Source with In-Tree Minimal Configuration (Selected):**
   - *Mechanism:* Retrieve pinned upstream Linux kernel sources directly from `kernel.org` (recorded in `manifests/sources.lock.toml`). Maintain a minimal, version-controlled `defconfig` in the repository tailored strictly for target hardware (initially QEMU/KVM virtual devices).
   - *Advantages:* Fully reproducible, minimal kernel footprint, completely auditable, zero external distribution lock-in.
   - *Disadvantages:* Requires host build time to compile the kernel during full system builds.

---

## Decision

ARK-OS will build its kernel from upstream Linux kernel releases using an in-tree, version-controlled minimal configuration (`defconfig`):

1. **Source Tracking:** Kernel releases will be tracked with immutable commit/tag references and cryptographic checksums in [manifests/sources.lock.toml](file:///home/aarav/repo/arkos/manifests/sources.lock.toml).
2. **Minimal Target Profile:** The initial kernel configuration for x86_64 will enable only essential subsystems:
   - Processor architecture support (x86_64 `CONFIG_64BIT=y`).
   - Essential virtual filesystems (`CONFIG_DEVTMPFS=y`, `CONFIG_DEVTMPFS_MOUNT=y`, `CONFIG_PROC_FS=y`, `CONFIG_SYSFS=y`, `CONFIG_TMPFS=y`).
   - Initramfs support (`CONFIG_BLK_DEV_INITRD=y`).
   - Serial console output (`CONFIG_SERIAL_8250=y`, `CONFIG_SERIAL_8250_CONSOLE=y`).
   - QEMU virtualized I/O devices (`CONFIG_VIRTIO=y`, `CONFIG_VIRTIO_PCI=y`, `CONFIG_VIRTIO_NET=y`, `CONFIG_VIRTIO_BLK=y`, `CONFIG_VIRTIO_CONSOLE=y`).
   - ELF binary execution (`CONFIG_BINFMT_ELF=y`).
3. **No Out-of-Tree Patches by Default:** Kernel code will remain vanilla upstream unless an explicit architectural decision requires a patch.

---

## Consequences

### Positive
- The operating system foundation remains independent of third-party Linux distributions.
- Fast boot times and low memory overhead due to a minimal kernel tailored strictly to target devices.
- Kernel configuration is tracked directly in version control alongside system manifests.

### Negative / Trade-Offs
- Building a full system from scratch requires compiling the Linux kernel on the host development machine.
- Physical hardware bring-up will require expanding driver options in target profiles as tested devices are added.
