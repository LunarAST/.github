# Contributing to LunarAST

Thank you for your interest in contributing to the LunarAST protocol family. This document outlines the conventions and processes for contributing to LunarAST specifications, adapters, and tooling.

## Scope of Contributions

We welcome contributions in the following areas:

- **Specification Improvements**: Corrections, clarifications, or extensions to existing protocol documents (Mother Specification, RouteAST, etc.).
- **New Sub-Protocol Proposals**: Proposals for new sub-domain contracts (e.g., EventAST, SchemaAST, TypeAST).
- **Adapter Implementations**: New language/framework adapters that extract RouteAST data from source code.
- **Tooling Contributions**: Improvements to the `lunar` CLI, `lunar-gateway`, or `lunar-scope`.
- **Bug Reports and Security Issues**: See `SECURITY.md` for vulnerability reporting.

## Before You Start

- For substantial changes or new sub-protocol proposals, please open a **Protocol Proposal** Issue using the provided template before writing code or specification text.
- Adapter implementations must comply with the LDJSON communication protocol and atomicity guarantees defined in the RouteAST specification.

## Pull Request Process

1. Fork the relevant repository and create a feature branch.
2. Ensure your changes align with the naming conventions (Google JSON Style Guide `camelCase` for all JSON/YAML properties; `kebab-case` for filenames).
3. For specification changes, update the version number and `Last Updated` date accordingly.
4. All PRs require approval from at least one CODEOWNER before merging.
5. Schema changes under `schemas/` must pass the automated JSON Schema validation CI check.

## Naming Conventions

LunarAST follows the Google Naming & Style Guides:

- All JSON/YAML property keys use **camelCase**.
- All executable binaries and public configuration filenames use **kebab-case**.
- File extensions for project-level intent overlays must be **.yml** (e.g., `.lunar/interfaces.yml`).

## Community

- Follow the [Code of Conduct](CODE_OF_CONDUCT.md) in all interactions.
- For real-time discussion, join our community channels (links in the organization profile).

## License

By contributing, you agree that your contributions will be licensed under the Apache-2.0 License.
