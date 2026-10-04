<div align="center">

# ARC-V2-AI

**Building the systems around local AI.**

*Local-first · service-oriented · persistent by design*

</div>

---

## Table of Contents

* [What is ARC-V2-AI?](#what-is-arc-v2-ai)
* [Projects](#projects)

  * [ARC V2 Core](#arc-v2-core)
  * [ARC V2 Inference](#arc-v2-inference)
* [Architecture Principles](#architecture-principles)

  * [Local-first](#local-first)
  * [Service-oriented](#service-oriented)
  * [Explicit separation](#explicit-separation)
  * [Persistent by design](#persistent-by-design)
  * [Linux-native](#linux-native)
* [Development Status](#development-status)
* [References & Documentation](#references--documentation)

  * [ARC V2 Documentation](#arc-v2-documentation)
  * [Development Guides](#development-guides)
  * [Repository Documentation](#repository-documentation)
* [Contributing](#contributing)

---

## What is ARC-V2-AI?

ARC-V2-AI develops the system architecture and infrastructure required to build **persistent local AI environments** rather than isolated model interfaces.

The organisation's projects focus on making capabilities composable: a system should be able to install components, run them as independent services, supervise their lifecycle, and connect them through explicit interfaces.

The current architecture is centred around **ARC V2**.

```mermaid
flowchart LR
    Core["ARC V2 Core"] --> Pulse["Pulse\nSupervision"]
    Forge["Forge\nInstallation"] --> Services["Installed Services"]
    Pulse --> Runner["Service Runner"]
    Runner --> Services
    Services --> Capabilities["ARC Capabilities"]
    Inference["ARC V2 Inference"] --> Capabilities
```

## Projects

### [ARC V2 Core](https://github.com/ARC-V2-AI/ARC-V2-Core)

**The operating core of ARC V2.**

ARC V2 Core provides the foundational system layer: boot, service installation, lifecycle management, supervision, dependency handling, health and readiness management, and the service runtime boundary.

Its architecture deliberately separates three concerns:

* **Forge** — manages what is installed.
* **Pulse** — manages what is running.
* **Service Runner** — provides how a service runs.

Services remain independent components that provide the actual capabilities of the system.

### [ARC V2 Inference](https://github.com/ARC-V2-AI/ARC-V2-Inference)

Local inference infrastructure for ARC V2, providing the model-serving layer used by the wider system.

---

## Architecture Principles

### Local-first

AI computation, state, and services should be able to operate locally and remain under the user's control.

### Service-oriented

Capabilities are implemented as focused services with explicit contracts instead of being coupled into one monolithic runtime.

### Explicit separation

Installation, supervision, execution, and application capability are separate concerns. Each layer should have a clear responsibility.

### Persistent by design

ARC is intended to operate as a persistent environment. State, memory, lifecycle, health, and recovery are system concerns rather than temporary additions around a model call.

### Linux-native

ARC V2 is designed around Linux as the primary operating environment and uses system-level primitives where they provide a better foundation than platform-agnostic abstractions.

---

## Development Status

ARC V2 is under **active development**. Architecture, interfaces, and implementation details may change as the system develops.

Repositories should be treated according to their own documented stability and compatibility guarantees.

---

## References & Documentation

This section collects the documentation used to understand, develop, and extend the ARC V2 system.

### ARC V2 Documentation

* **[ARC V2 Core](https://github.com/ARC-V2-AI/ARC-V2-Core)** — Operating core, Forge, Pulse, service lifecycle, installation, and runtime architecture.
* **[ARC V2 Service Runner](https://github.com/ARC-V2-AI/ARC-V2-Service-Runner)** — Service contract, runtime context, runner behavior, readiness, health, and lifecycle.
* **[ARC V2 Inference](https://github.com/ARC-V2-AI/ARC-V2-Inference)** — Local inference and model-serving infrastructure.

### Development Guides

* **[Building an ARC V2 Service](docs/BUILDING-A-SERVICE.md)** — Guide for creating, packaging, installing, and developing an independently installable ARC V2 service.
* **[ARC V2 Services](https://github.com/ARC-V2-AI/ARC-V2-Services)** — Service documentation and examples.

### Repository Documentation

For implementation details, configuration, development setup, and repository-specific conventions, refer to the documentation included in each individual repository.

The organization README provides the architectural overview; individual repositories remain the authoritative source for their own APIs, configuration, and development requirements.

---

## Contributing

Contributions are welcome, particularly improvements that strengthen the architecture, reliability, documentation, developer experience, and individual services.

For significant architectural changes, discuss the design before implementing a large change. Keeping boundaries explicit is more important than adding abstraction for its own sake.

See the organisation's [`CONTRIBUTING.md`](https://github.com/ARC-V2-AI/.github/blob/main/CONTRIBUTING.md) and the contributing documentation of the individual repository for project-specific requirements.

---

<div align="center">

**ARC-V2-AI**

*Building the systems around local AI.*

</div>
