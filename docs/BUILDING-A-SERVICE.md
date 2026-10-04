<div align="center">

# Building an Installable ARC V2 Service

**Create a normal Python package, add the ARC service metadata, implement the `Service` contract, and let Forge, the Service Runner, and Pulse handle the system around it.**

<br>

[![Python](https://img.shields.io/badge/Python-3.11%2B-3776AB?logo=python\&logoColor=white)](#)
[![uv](https://img.shields.io/badge/Build-uv-6F42C1?logo=astral\&logoColor=white)](#)
[![ARC V2](https://img.shields.io/badge/ARC%20V2-Service-blue)](#)
[![Architecture](https://img.shields.io/badge/architecture-service--oriented-2ea44f)](#)

</div>

---

<details>
<summary><strong>Table of Contents</strong></summary>

<br>

* [Getting Started](#getting-started)

  * [1. Create the project](#1-create-the-project)
  * [2. Add the Service Runner](#2-add-the-service-runner)
  * [3. Add ARC service metadata](#3-add-arc-service-metadata)
  * [4. Create the service scaffold](#4-create-the-service-scaffold)
  * [5. Install the service](#5-install-the-service)
  * [6. Verify the service](#6-verify-the-service)
* [Service Configuration](#service-configuration)
* [Service Class](#service-class)
* [Runtime Context](#runtime-context)
* [Readiness and Health](#readiness-and-health)
* [Shutdown](#shutdown)
* [Dependencies](#dependencies)
* [Restart and Health Policies](#restart-and-health-policies)
* [Architecture](#architecture)
* [Project Structure](#project-structure)
* [Next Steps](#next-steps)

</details>

---

## Getting Started

> [!TIP]
> The fastest way to build an ARC service is to start with a normal `uv` package and add the ARC service contract.

### 1. Create the project

Create a package project with `uv`:

```bash
uv init --package arc-v2-test-service
cd arc-v2-test-service
```

A suitable starting structure is:

```text
arc-v2-test-service/
├── pyproject.toml
└── src/
    └── arc_test_service/
        ├── __init__.py
        └── service.py
```

---

### 2. Add the Service Runner

Add the ARC Service Runner as a Python dependency:

```bash
uv add git+https://github.com/ARC-V2-AI/service-runner.git
```

> [!NOTE]
> The runner provides the `Service` base class, runtime context, status types, and the generic process runner used by ARC services.

---

### 3. Add ARC service metadata

Add the following block to `pyproject.toml`:

```toml
[tool.arc.service]
id = "test"
module = "arc_test_service.service"
name = "ARC Test Service"
description = "Minimal service used to test ARC Pulse"
restart = "on-failure"
health = "restart"
depends = []
```

The `module` points to the Python module containing your `Service` subclass. The remaining fields describe the service to ARC.

A complete minimal `pyproject.toml` can look like this:

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "arc-v2-test-service"
version = "0.1.0"
description = "Test service for ARC V2"
requires-python = ">=3.11"
dependencies = [
    "arc-v2-service-runner>=0.1.0,<0.2.0",
]

[tool.hatch.build.targets.wheel]
packages = ["src/arc_test_service"]

[tool.arc.service]
id = "test"
module = "arc_test_service.service"
name = "ARC Test Service"
description = "Minimal service used to test ARC Pulse"
restart = "on-failure"
health = "restart"
depends = []
```

---

### 4. Create the service scaffold

Create `src/arc_test_service/service.py`:

```python
from __future__ import annotations

import asyncio

from arc_service.service import Service


class TestService(Service):
    def __init__(self) -> None:
        super().__init__(
            name="test",
            version="0.1.0",
            description="Minimal ARC test service",
        )

        self._stop_event = asyncio.Event()
        self._ready = False

    async def run(self) -> None:
        assert self.ctx is not None

        self.ctx.logger.info("Test service starting")

        # Initialize your service here.
        self._ready = True

        while not self._stop_event.is_set():
            await asyncio.sleep(1)

    async def ready(self) -> tuple[bool, str | None]:
        if self._ready:
            return True, None

        return False, "still starting"

    async def healthy(self) -> tuple[bool, str | None]:
        return True, None

    async def stop(self) -> None:
        self._stop_event.set()
```

> [!IMPORTANT]
> That is enough for a real installable ARC service.

The framework discovers the `Service` subclass from the configured module; you do not need to write a custom runner or IPC implementation.

---

### 5. Install the service

From the service project:

```bash
arc install .
```

You can also install from another local path or a Git source:

```bash
arc install /path/to/arc-v2-test-service
arc install <git-source>
```

> [!NOTE]
> Forge installs the package into the ARC service runtime and registers the service for Pulse.

---

### 6. Verify the service

List installed services:

```bash
arc list
```

Inspect the service:

```bash
arc info test
```

Remove it again with:

```bash
arc remove test
```

---

## Service Configuration

The ARC-specific configuration lives in `[tool.arc.service]`.

| Field         | Purpose                                         |
| ------------- | ----------------------------------------------- |
| `id`          | Stable ARC service identifier                   |
| `module`      | Python module containing the `Service` subclass |
| `name`        | Human-readable service name                     |
| `description` | Description shown by ARC                        |
| `restart`     | Process restart policy                          |
| `health`      | Action for repeated health failures             |
| `depends`     | Other ARC services required before startup      |

The example service uses the following configuration:

```toml
[tool.arc.service]
id = "test"
module = "arc_test_service.service"
name = "ARC Test Service"
description = "Minimal service used to test ARC Pulse"
restart = "on-failure"
health = "restart"
depends = []
```

### Python dependencies vs ARC dependencies

> [!IMPORTANT]
> These are different concepts:

```toml
dependencies = [
    "arc-v2-service-runner>=0.1.0,<0.2.0",
]
```

installs Python packages.

```toml
depends = ["inference", "memory"]
```

declares dependencies on other ARC services for Pulse.

---

## Service Class

Every ARC service inherits from `Service`:

```python
from arc_service.service import Service
```

The service contract is centered around four methods:

| Method      | Purpose                                            |
| ----------- | -------------------------------------------------- |
| `run()`     | Main long-running work                             |
| `ready()`   | Reports whether initialization is complete         |
| `healthy()` | Reports whether the service is operating correctly |
| `stop()`    | Performs graceful shutdown                         |

> [!IMPORTANT]
> `start(ctx)` is provided by the framework and normally should not be overridden.

### `run()`

Put the actual service work in `run()`.

For a long-running service, it normally remains active until shutdown:

```python
async def run(self) -> None:
    assert self.ctx is not None

    self.ctx.logger.info("Service started")

    while not self._stop_event.is_set():
        await asyncio.sleep(1)
```

Keep blocking work out of the event loop and manage your own internal resources.

---

## Runtime Context

The Service Runner creates a `BaseContext` and makes it available as `self.ctx` before `run()` starts.

```python
self.ctx.logger
self.ctx.env
self.ctx.service_name
self.ctx.process_name
```

Use the context instead of rebuilding ARC runtime behavior inside the service.

For example:

```python
assert self.ctx is not None

self.ctx.logger.info("Starting %s", self.ctx.service_name)

value = self.ctx.env.get("MY_SERVICE_SETTING")
```

> [!NOTE]
> ARC prepares the service environment before the process starts. Services do not load the global ARC `.env` themselves.

---

## Readiness and Health

ARC treats **readiness** and **health** as different states.

### Readiness

`ready()` answers:

> **Is the service initialized and ready for other services to depend on it?**

Example:

```python
async def ready(self) -> tuple[bool, str | None]:
    if self._ready:
        return True, None

    return False, "still starting"
```

Typical readiness conditions include a loaded model, listening server, connected database, initialized workers, or available required resources.

### Health

`healthy()` answers:

> **Is the service currently operating correctly?**

Example:

```python
async def healthy(self) -> tuple[bool, str | None]:
    if self._worker_failed:
        return False, "worker failed"

    return True, None
```

Typical health failures include lost dependencies, failed workers, unavailable required resources, or unrecoverable internal state.

> [!TIP]
> Both methods should stay lightweight because Pulse may call them frequently.

---

## Shutdown

When Pulse stops a service, the Runner calls:

```python
await service.stop()
```

Use `stop()` to release resources and signal your main loop to terminate.

A simple pattern is:

```python
self._stop_event = asyncio.Event()
```

```python
async def stop(self) -> None:
    self._stop_event.set()
```

and in `run()`:

```python
while not self._stop_event.is_set():
    await asyncio.sleep(1)
```

---

## Dependencies

ARC service dependencies are declared in `depends`:

```toml
[tool.arc.service]
id = "agent"
module = "arc_agent.service"
depends = ["inference", "memory"]
```

Pulse uses these dependencies to determine startup order and waits for required dependencies to become ready before starting the dependent service.

For example:

```text
inference ──┐
            ├──► agent
memory ─────┘
```

> [!IMPORTANT]
> Declare only real ARC service dependencies. Do not use `depends` for Python packages.

---

## Restart and Health Policies

### Restart

```toml
restart = "on-failure"
```

| Value        | Meaning                                       |
| ------------ | --------------------------------------------- |
| `always`     | Restart whenever the process exits            |
| `on-failure` | Restart when the process exits unsuccessfully |
| `never`      | Do not restart automatically                  |

### Health

```toml
health = "restart"
```

| Value     | Meaning                                             |
| --------- | --------------------------------------------------- |
| `ignore`  | Record unhealthy state but keep the service running |
| `restart` | Restart after repeated unhealthy checks             |
| `stop`    | Stop after repeated unhealthy checks                |

> [!NOTE]
> These policies are handled by Pulse rather than by the service itself.

---

## Architecture

An installed ARC service sits between the common ARC infrastructure and its own application logic:

```text
                  ARC Core
                     │
              ┌───────┴────────┐
              │                │
            Forge             Pulse
              │                │
         installs        supervises
              │                │
              ▼                │
        ARC Service Runtime    │
              │                │
        Service Runner ◄───────┘
              │
              ▼
         Your Service
```

The responsibilities remain intentionally separate:

> **Forge manages what is installed.**
>
> **Pulse manages what is running.**
>
> **Service Runner provides how a service runs.**
>
> **The service provides the actual capability.**

Forge can install and register a service without supervising its process. Pulse controls lifecycle, dependencies, readiness, health, restart, and shutdown.

> [!NOTE]
> Each service runs as its own independently supervised operating-system process.

---

## Project Structure

A small service can remain very small:

```text
arc-v2-test-service/
├── pyproject.toml
├── README.md
└── src/
    └── arc_test_service/
        ├── __init__.py
        └── service.py
```

As the service grows, add normal Python modules around `service.py`:

```text
src/
└── arc_test_service/
    ├── __init__.py
    ├── service.py
    ├── config.py
    ├── workers.py
    └── api.py
```

> [!NOTE]
> ARC does not require every service to have a special internal project structure. The important integration points are the Python package, the configured `module`, and the `Service` contract.

---

## Next Steps

Once the minimal service works, the next useful additions are usually:

* proper initialization and readiness state
* meaningful health checks
* graceful resource cleanup
* explicit ARC service dependencies
* service-specific environment configuration
* tests for the service's own application logic

For deeper architecture, see the ARC service documentation and the Service Runner contract. The Runner defines the common service API and execution environment, while Core and Pulse remain responsible for installation and supervision.

---

<div align="center">

**ARC V2**

*Build capabilities. Let ARC handle the system around them.*

</div>
