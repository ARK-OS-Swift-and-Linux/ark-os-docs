# ADR-0004: Initial Boot Process and Initramfs Architecture

- **Status:** Accepted
- **Date:** Architectural Baseline Phase
- **Deciders:** ARK-OS Project Maintainers

---

## Context and Problem Statement

When the Linux kernel boots, it requires an initial userspace payload to execute as PID 1 (`/sbin/arkrt`). In traditional Linux systems, the bootloader can either load an initial ramdisk (initramfs) or point the kernel directly to a root block device on disk.

Introducing persistent disk partitioning (GPT), filesystem format drivers (ext4/btrfs), and a two-stage `switch_root` transition during early bring-up introduces multiple compounding failure modes (disk geometry mismatches, block driver initialization timings, rootfs mount failures) before PID 1 has even been verified to run.

---

## Considered Alternatives

1. **Immediate Persistent Disk Image Boot (Single-Stage Direct Block Mount):**
   - *Mechanism:* Format a raw virtual disk image with GPT partitions and launch the kernel with `root=/dev/vda1 init=/sbin/arkrt`.
   - *Advantages:* Persistent storage from day one.
   - *Disadvantages:* Requires complex image formatting tooling on the host; storage driver failures can mask early userspace defects.
2. **Two-Stage Boot with Minimal Intermediate Initramfs:**
   - *Mechanism:* Boot a minimal helper initramfs whose sole purpose is discovering the disk, mounting the persistent root, and executing `switch_root`.
   - *Advantages:* Industry-standard architecture for generic desktop distributions.
   - *Disadvantages:* Premature complexity; requires maintaining two distinct userspaces (the initramfs helper and the real system) before proving either works.
3. **Standalone Minimal Initramfs in QEMU (Selected):**
   - *Mechanism:* Package `arkrt` and essential userspace binaries directly into a CPIO archive loaded into memory by QEMU via `-initrd`. The kernel unpacks this archive into `rootfs` (RAM) and directly executes `/sbin/arkrt`.
   - *Advantages:* Simplest possible bring-up path; completely decouples PID 1 startup from block devices and disk partition tables; fast reboot iteration in QEMU.
   - *Disadvantages:* State is stored in RAM (tmpfs) and does not persist across virtual machine restarts.

---

## Decision

ARK-OS will utilize a **standalone minimal initramfs** for initial system bring-up in QEMU:

1. **Self-Contained Bring-Up Image:** The initramfs will contain the complete minimal system required for early verification:
   - `/sbin/arkrt` (PID 1 runtime)
   - `/bin/sash` (interactive shell)
   - Minimal device nodes and configuration files.
2. **Execution Flow:** The bootloader/QEMU will pass `init=/sbin/arkrt console=ttyS0` to the kernel. Upon unpacking the initramfs, the kernel transfers control directly to `arkrt`.
3. **Deferral of Persistent Block Storage:** Multi-partition disk image generation, disk partitioning recipes, and two-stage `switch_root` boot transitions are explicitly deferred until `arkrt` startup, virtual filesystem mounting, and `sash` execution are verified in the standalone initramfs environment.

---

## Consequences

### Positive
- Fastest, cleanest diagnostic cycle for proving early Swift/Linux userspace.
- Isolates early bring-up errors strictly to kernel configuration and `arkrt` logic.
- Eliminates the need for complex host-side disk partition formatting during early development.

### Negative / Trade-Offs
- The system operates entirely in volatile memory during the bring-up phase. Persistent user files and logs are not saved across QEMU sessions until disk image recipes are introduced.
