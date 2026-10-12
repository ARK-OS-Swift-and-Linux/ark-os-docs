# Licensing and Compliance Policy

## 1. Project Licensing Status

- **Repository License:** Apache License 2.0
- **File:** [LICENSE](file:///home/aarav/repo/arkos/LICENSE)
- **Copyright Holder:** Aarav Ravindra Kharade (Copyright 2026)

The project's first-party code is licensed under the Apache License, Version 2.0. In accordance with [AGENTS.md](file:///home/aarav/repo/arkos/AGENTS.md) Section 10, agents and contributors must preserve this license and must not replace or alter the project's license without explicit authorization.

---

## 2. Dependency Audit and Licensing Status

Currently, the repository contains no vendored third-party code in `external/` and no pinned dependency revisions in `manifests/sources.lock.toml`.

When implementation begins, ARK-OS will coordinate with external open-source projects. The anticipated dependencies and their legal obligations are documented below:

| Dependency | Upstream Origin | Anticipated License | Source-Distribution Obligations | Pinned Revision Status |
| :--- | :--- | :--- | :--- | :--- |
| Linux Kernel | https://kernel.org | GPL-2.0 | Must provide full corresponding source code for any distributed kernel binary. | Unresolved (None pinned) |
| Swift Core Runtime | https://github.com/swiftlang/swift | Apache-2.0 with LLVM Exception | Retain copyright notices and license text. | Unresolved (None pinned) |
| Clang / LLVM Toolchain | https://llvm.org | Apache-2.0 with LLVM Exception | Retain copyright notices and license text. | Unresolved (None pinned) |

---

## 3. Third-Party Compliance and Tracking Procedures

All contributions introducing or updating third-party software must adhere to the following procedures:

### 3.1 Pinned Source Identification
- Every external dependency must be declared in [manifests/sources.lock.toml](file:///home/aarav/repo/arkos/manifests/sources.lock.toml).
- The declaration must include:
  - Canonical upstream repository URL or release tarball URL.
  - Immutable commit hash or cryptographic checksum (SHA-256).
  - Recognized SPDX license identifier (e.g., `GPL-2.0-only`, `Apache-2.0`).

### 3.2 Modification Tracking
- Direct, unrecorded modifications to upstream source code are strictly prohibited.
- Any necessary patches to external software must be stored as explicit `.patch` files in `external/` or `packaging/recipes/`.
- Each patch file must document its rationale, author, upstream issue reference, and compatibility impact.

### 3.3 Software Bill of Materials (SBOM) and Notices
- All licenses and copyright notices from included dependencies must be mirrored in [THIRD_PARTY_NOTICES.md](file:///home/aarav/repo/arkos/THIRD_PARTY_NOTICES.md).
- When system images are generated, an automated bill of materials recording all included packages and licenses must be generated and stored in `/usr/share/arkos/licenses/` within the root filesystem.

---

## 4. Unresolved Obligations and Disclaimer

The existence of this documentation does not declare or certify the ARK-OS project as legally compliant. Ongoing legal compliance requires:
1. Maintaining compliance with the declared Apache License 2.0 terms in `LICENSE`.
2. Locking and auditing exact upstream source revisions in `manifests/sources.lock.toml` when code is retrieved.
3. Fulfilling all copyleft source-distribution requirements when pre-built kernel or system binaries are published.
