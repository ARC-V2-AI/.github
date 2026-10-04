# Contributing to ARC-V2-AI

ARC-V2-AI projects are developed as parts of a larger system. Contributions should therefore improve the individual repository without weakening the architectural boundaries around it.

Individual repositories may define additional requirements. When they do, those requirements take precedence over these organisation-wide defaults.

## Before making changes

1. Read the repository README and relevant documentation.
2. Understand which architectural layer the change belongs to.
3. Search existing issues and pull requests.
4. For significant changes, discuss the design before implementation.
5. Keep unrelated cleanup out of feature or bug-fix changes.

## Architectural boundaries

ARC V2 deliberately separates responsibilities.

### Installation

**Forge** manages installed components and services: sources, metadata, packages, registration, configuration known to the runtime, and installed state.

### Supervision

**Pulse** manages running services: startup order, lifecycle, dependencies, readiness, health, restart behaviour, and shutdown.

### Execution

The **Service Runner** provides the common runtime and communication mechanism used to execute ARC services.

### Capability

**Services** contain the actual application or system capability. Core infrastructure should not need to know the internal implementation of a service beyond its contract.

A contribution should preserve these boundaries unless changing the architecture itself is the explicit purpose of the work.

## Code changes

Prefer:

- small, focused changes
- explicit interfaces and responsibilities
- simple implementations over unnecessary abstraction
- existing project conventions over introducing new conventions
- changes that are easy to test, review, and remove

Avoid:

- unrelated refactoring
- hidden coupling between architectural layers
- duplicate lifecycle or installation logic
- silently changing public behaviour
- adding dependencies without a clear reason

## Tests and validation

Run the checks documented by the repository before opening a pull request.

When automated tests do not exist for the affected area, provide meaningful manual validation and describe exactly what was checked.

If a change affects a public command, service contract, configuration value, architecture, or documented behaviour, update the relevant documentation.

## Documentation

Documentation is part of the implementation. Architectural changes should explain:

- what responsibility changed
- why the boundary changed
- how the affected components interact
- what users or service authors need to know

Do not document speculative behaviour as if it were implemented.

## Issues

Use the provided issue templates when possible.

A useful bug report should contain a reproducible problem, expected and actual behaviour, the relevant environment, and enough logs or configuration to investigate it without exposing secrets.

Feature proposals should explain the problem before prescribing the implementation.

## Pull requests

A pull request should make the change understandable without requiring the reviewer to reconstruct the entire development process.

Include:

- what changed
- why it changed
- how it was validated
- documentation changes, when applicable
- important compatibility or migration considerations

Keep the PR focused. Split unrelated work into separate pull requests.

## Commits

Use concise commit messages that describe the change. Follow repository-specific commit conventions when they exist.

Do not commit:

- credentials or tokens
- `.env` files containing secrets
- private keys
- local machine configuration
- generated caches
- large generated artifacts unless explicitly required

## Licensing

Check the license of the repository before contributing. ARC V2 Core is licensed under **AGPL-3.0**; other repositories may have repository-specific licensing terms.
