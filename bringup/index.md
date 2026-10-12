# Platform Bring-Up Guide

## Overview

Platform bring-up in ARK-OS refers to the process of booting the system on target hardware platforms or virtual machine environments.

Currently, ARK-OS is at the architectural baseline stage. No kernel images, initramfs archives, or bootable disk images have been built or booted. This guide specifies the planned bring-up configuration, diagnostic requirements, and target emulator setups for when implementation begins.

---

## Target Platforms

| Platform | Machine Type | Firmware | Console | Status |
| :--- | :--- | :--- | :--- | :--- |
| x86_64 Virtual Machine | QEMU standard PC (`q35`) | OVMF (UEFI) or SeaBIOS | Serial `ttyS0` + VGA `tty0` | Planned (Initial Target) |
| ARM64 Virtual Machine | QEMU `virt` machine | AAVMF (UEFI) | Serial `ttyAMA0` | Planned (Future Target) |
| Physical Hardware | Standard x86_64 PC | UEFI 2.x | Serial or Framebuffer | Planned (Future Milestone) |

---

## Planned x86_64 Bring-Up Environment (QEMU)

The initial platform bring-up target is a standardized QEMU x86_64 virtual machine using KVM hardware acceleration.

### Intended QEMU Command Line (Illustrative)

Once boot assets are built, the planned bring-up invocation will be structured as follows:

```bash
qemu-system-x86_64 \
    -enable-kvm \
    -m 2048 \
    -smp 2 \
    -machine q35 \
    -kernel path/to/vmlinuz \
    -initrd path/to/initramfs.cpio.gz \
    -append "console=ttyS0,115200 console=tty0 root=/dev/ram0 init=/sbin/arkrt earlyprintk=serial,ttyS0,115200 loglevel=7" \
    -serial stdio \
    -display none
```

*Note: The command above illustrates the intended target configuration. The referenced kernel and initramfs files do not yet exist in the repository.*

---

## Early Boot Diagnostic Plan

To make startup failures diagnosable during early bring-up:

1. **Kernel Output:** Direct the Linux kernel log buffer to the primary serial port (`earlyprintk=serial,ttyS0,115200` and `console=ttyS0,115200`). This ensures all kernel initialization messages and panics are visible directly in the terminal running QEMU.
2. **Runtime Console Connection:** When `arkrt` launches as PID 1, it must immediately open `/dev/console` and write an initial heartbeat message (e.g., `arkrt: starting initialization`).
3. **Mount Diagnostics:** Emit clear log lines as each virtual filesystem (`/proc`, `/sys`, `/dev`, `/run`) is mounted, reporting success or the specific system error code.
4. **Interactive Console Attachment:** If `arkrt` cannot launch user services or `sash`, it must drop to a rescue shell or halt cleanly with an informative diagnostic message rather than terminating PID 1 (which would cause a kernel panic).

---

## Bring-Up Verification Status

- **Build Verification:** None. No images built.
- **Boot Verification:** None. No boot simulations performed.
- **Supported Hardware Testing:** None. Physical hardware support is not yet established.
