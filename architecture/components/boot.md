# Component Specification: Boot Architecture and Components

## Overview

- **Component Area:** `components/boot/`
- **Role:** Kernel Handoff, Early Boot Payload Assembly, and Bootloader Configuration
- **Primary Language:** C (GNU-EFI), Shell, Make
- **Implementation Status:** Implemented and verified (`ARKBOOT.EFI` x86_64 UEFI bootloader with Stage A visual dot and Stage B Linux 64-bit handoff; direct QEMU kernel boot supported)

The `components/boot/` area encompasses the first-party UEFI bootloader (`ARKBOOT.EFI`), kernel launch parameters, initramfs assembly rules, and artifact synchronization scripts (`sync.sh`) required to transition from hardware/firmware initialization to the Linux kernel and userspace runtime (`arkrt`).

Per [ADR-0004](file:///home/aarav/repo/arkos/docs/decisions/ADR-0004-initial-boot-process-initramfs.md), ARK-OS boot architecture is strictly divided into two phases: an initial direct-kernel bring-up phase and a subsequent persistent-storage bootloader phase.

---

## Boot Architecture Phases

### 1. Milestone 1: Initial Bring-Up (Direct-Kernel RAM Boot)

For initial userspace bring-up in QEMU, the system uses a **direct-kernel boot** strategy to isolate early verification from block storage and firmware complexity:

- **Launch Mechanism:** QEMU directly loads the compiled Linux kernel (`bzImage`) and standalone initial ramdisk (`initramfs.cpio.gz`) using emulator parameters (`-kernel` and `-initrd`).
- **No Firmware Bootloader:** Firmware bootloaders (such as GRUB or systemd-boot) and EFI system partitions are deliberately omitted during this milestone.
- **Volatile Execution:** The kernel unpacks the initramfs into a RAM-backed root filesystem (`rootfs`) and directly transfers control to the initial userspace process.
- **Kernel Command Line Contract:**
  - `console=ttyS0`: Routes primary console output to the virtual serial interface.
  - `earlyprintk=serial,ttyS0`: Ensures boot messages are visible prior to the full TTY driver initialization.
  - `rdinit=/sbin/arkrt`: Specifies the initial executable to invoke within the unpacked ramdisk.
  - `panic=-1`: Halts immediately on kernel panic to simplify debugging.

### 2. Subsequent Milestones: Persistent Storage and Hardware Boot

Following successful verification of `arkrt` and minimal userspace in the standalone ramdisk, full system images will introduce standard bootloader infrastructure:

- **UEFI Bootloader:** Standard 64-bit UEFI boot protocol utilizing an established bootloader (such as systemd-boot or GRUB 2).
- **Persistent Disk Partitioning:** GPT partition layout containing an EFI System Partition (ESP) and one or more persistent root/data partitions.
- **Two-Stage Handoff:** The initramfs transitions from a self-contained runtime into an intermediate helper responsible for disk discovery, filesystem verification, mounting root by UUID, and executing `switch_root` to transition to persistent storage.

---

## Core Responsibilities

1. **Boot Configuration and Parameters:**
   - Maintaining version-controlled kernel command-line configurations for virtual and hardware targets.
   - Ensuring serial console diagnostics remain enabled by default for debugging.
2. **Early Environment Assembly:**
   - Defining the minimal directory structure (`/dev`, `/proc`, `/sys`, `/sbin`, `/bin`) and essential static nodes required by `arkrt`.
   - Ensuring `rdinit=/sbin/arkrt` is packaged at the root of the initramfs archive.
3. **Architecture Handoff Contracts:**
   - **x86_64 Virtual (Milestone 1):** Linux direct boot protocol loaded via QEMU.
   - **x86_64 Physical (Subsequent):** UEFI 64-bit boot protocol with ESP configuration.
   - **ARM64 (Future):** Device Tree (FDT) or ACPI boot under U-Boot or EDK2.

---

## Design Invariants and Constraints

- **First-Party UEFI Bootloader:** ARK-OS implements a dedicated x86_64 UEFI bootloader (`ARKBOOT.EFI`) using GNU-EFI, implementing Stage A graphical display (centered white dot on black) and Stage B Linux 64-bit boot protocol handoff.
- **Staged Complexity:** Direct QEMU boot (`-kernel` / `-initrd`) remains supported as a fast, lightweight developer test target alongside full UEFI boot.
- **Deterministic Handoff:** The boundary between kernel init and `/sbin/arkrt` must remain explicit, with failures producing clear diagnostic output on the serial console.
