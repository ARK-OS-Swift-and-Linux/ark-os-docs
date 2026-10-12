# Component Specification: System Services

## Overview

- **Component Area:** `components/services/`
- **Role:** Modular Background Daemons and System Services
- **Primary Language:** Swift / C
- **Implementation Status:** Planned (Not implemented)

The `components/services/` directory contains first-party system services that provide shared capabilities to userspace programs and applications.

---

## Core Responsibilities and Service Model

System services in ARK-OS operate under a unified daemon model:

1. **Dedicated Responsibilities:** Each service owns a single, distinct subsystem (e.g., logging, device event dispatch, networking management, or user session management).
2. **Supervised Lifecycle:** All system services are spawned and monitored by `arkrt`. Services must declare dependencies, startup requirements, and restart policies in declarative manifests.
3. **Privilege Separation:** Services must drop privileges immediately after acquiring necessary resources, executing under dedicated service user IDs (UIDs) rather than as `root`.
4. **Input Validation:** All IPC requests originating from applications or other services must be strictly validated at the service boundary.

---

## Planned Core Services

| Service Name | Intended Role | Status |
| :--- | :--- | :--- |
| `logd` | Structured system logging daemon receiving messages via `/dev/log` or domain sockets. | Planned |
| `devd` | Dynamic hardware event monitor consuming kernel uevents from netlink sockets. | Planned |
| `sessiond` | User session and seat manager coordinating console logins and display servers. | Planned |
| `netd` | Network interface configuration and IP address management daemon. | Planned |

---

## Design Invariants and Constraints

- **No Circular Dependencies:** Services must not depend cyclically on one another. Core services (like logging) must not depend on higher-level services (like networking).
- **Communication Protocol:** Services communicate using Unix domain sockets framed with explicit request/response schemas.
- **Resilience:** Unhandled exceptions in one service must not trigger failures in unrelated services or bring down the runtime supervisor.
