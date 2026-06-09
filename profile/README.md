# LunarAST Ecosystem
## Universal Multi-Layer Static Contract Protocol Suite & System Architecture Visualization Canvas

---

## 1. Ecosystem Vision
With the continuous evolution of distributed and microservice architectures, **API semantic drift** has become a major cause of system failures. LunarAST delivers a **zero-trust, on-demand loading, immutable version** system for code context distribution and contract validation.

It introduces **no runtime monitoring or performance overhead**. Instead, it only reflects physical facts captured at build time, detecting dangling breakpoints and unplanned wild dependencies before code deployment.

- **Ecosystem Whitepaper**: [View full specifications & design philosophy](.github/docs/ecosystem-whitepaper-v1.0.md)

---

## 2. Core Architecture: Generate → Distribute → Specify → Visualize
The entire ecosystem is structurally divided into four fully decoupled layers with clear boundaries. Each component has a single responsibility and physical isolation.

```
                              ┌────────────────────────────────┐
                              │           LunarAST             │  ← 1. Specification Layer (Spec)
                              └──────────────┬─────────────────┘
                                             │
                                             ▼
                              ┌────────────────────────────────┐
                              │             lunar              │  ← 2. Data Generation Layer (Generate)
                              └──────────────┬─────────────────┘
                                             │
                                             ▼
                              ┌────────────────────────────────┐
                              │         lunar-gateway          │  ← 3. Stateless Distribution Layer (Distribute)
                              └──────────────┬─────────────────┘
                                             │
                                             ▼
                              ┌────────────────────────────────┐
                              │          lunar-scope           │  ← 4. Visualization Layer (Visualize)
                              └────────────────────────────────┘
```

### 📂 Core Repositories & Modules

*   **Specification Layer** | [`LunarAST/RouteAST`](https://github.com/LunarAST/RouteAST)
    Official specification of the RouteAST sub-protocol for network and routing contract synchronization.
    This is the first executable interface alignment standard across the ecosystem.

*   **Generator CLI** | [`LunarAST/lunar`](https://github.com/LunarAST/lunar)
    Unified command-line tool written in Rust. It supports project initialization, AST scanning, permission-free diff diagnosis and guided synchronization.
    *   **Rust Adapter** | [`LunarAST/lunar-extract-rust`](https://github.com/LunarAST/lunar-extract-rust)
        Deep AST parser built on `syn::visit`, providing full static extraction for Axum routing definitions.

*   **Distributor** | [`LunarAST/lunar-gateway`](https://github.com/LunarAST/lunar-gateway)
    Serverless edge gateway compiled to `wasm32-wasip2`. It enforces strict JWT authentication, tiered cache control and 2MB memory protection with circuit breaking.

*   **Visualization Layer** | [`LunarAST/lunar-scope`](https://github.com/LunarAST/lunar-scope)
    Pure static frontend canvas built with React + xyflow + elkjs.
    It renders multi-layer topology diagrams, diagnostic alert lines and visualizes dangling connection issues.

---

## 3. Three-Tier Source-of-Truth Model
For static governance, LunarAST adheres to the principles of **physical facts first** and **no overprivileged operations**.
The closed workflow is implemented via the following three progressive data layers:

1.  **Physical Facts**
    Objective facts parsed from source code. Automatically generated and stored in read-only cache:
    `.lunar/.interfaces-autogen.json` (excluded via `.gitignore`).

2.  **Intent Overlay**
    Defined in `.lunar/interfaces.yml` at project root. Fully controlled by developers and tracked by Git.
    Tools are prohibited from making any unauthorized silent modifications.

3.  **Escape Hatch**
    Inline directive `// lunar:consume` placed above source code lines.
    Serves as the final fallback to bypass static analysis limits for untraceable dynamic calls.

---

## 4. Quick Start & Command Reference

Install the `lunar` CLI and corresponding adapters (Rust example):
```bash
cargo install lunar
cargo install lunar-extract-rust
```

Navigate to your project root and execute commands below:
```bash
lunar init                     # Auto-detect tech stack and initialize local config (run when interfaces.yml is missing)
lunar scan                     # Perform static analysis, generate physical facts and write to cache file
lunar diff                     # Show Git-style diff between physical facts and manual intent overlay
lunar sync --apply             # Backup existing data, then apply synchronized changes to config files
lunar doctor                   # Run health check for connectivity, version pointers and topology consistency
```

---

## 5. License & Community Governance
All protocols and components in the LunarAST ecosystem are licensed under **Apache-2.0**.

*   **Contribution Guide**: Read [CONTRIBUTING.md](CONTRIBUTING.md) to learn about adding new adapters or submitting RFC proposals.
*   **Security Disclosure**: Refer to [SECURITY.md](SECURITY.md). Please report security vulnerabilities such as token validation flaws or gateway privilege escalation via private channels.

---

> *"Entities should not be multiplied unnecessarily.
> Let complex code inference fade away, and let deterministic LunarAST contract alignment become the standard."*
