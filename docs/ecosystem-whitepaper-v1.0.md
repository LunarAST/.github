# LunarAST & lunar-gateway Protocol and System Architecture Specification
## — Multi-layer Heterogeneous Static Contract Standard & Four-tier Architecture: Capture, Transmit, Tag, Present
**Version**: 1.6 — Specification Redesign & Full Logical Closure Release
**Last Updated**: 2026-06-12

---

## 1. Physical Role Definitions
This specification defines the physical boundaries and naming conventions for all components within the LunarAST ecosystem.

| Component / Name | Physical Tier | Core Responsibilities |
|:---|:---|:---|
| **LunarAST** | **Standard Specification Layer** | Defines the base IR specifications, extraction protocols and mathematical comparison semantics for multi-layer contracts (RouteAST, EventAST, etc.). As the underlying static schema standard, it contains no executable code itself. |
| **`lunar`** | **Data Generation Layer** | Native command-line binary for local workstations and CI pipelines. Implements `lunar init` (initialize templates), `lunar scan` (physical extraction), `lunar diff` (read-only comparison) and `lunar sync --apply` (backup & secure synchronization). All alignment decision and merge logic runs here. |
| **`lunar-serve`** | **Local Read-only Distribution Layer** | Lightweight local HTTP binary for read-only access. Depends on `lunar-interface`. Renders high-fidelity `lunar-map.json` for development use, and provides on-demand, archive-free direct access to local workspace source code for AI Agents via built-in fallback mechanisms with zero manual configuration. |
| **`lunar-gateway`** | **Stateless Distribution Layer** | Independently deployed Serverless edge gateway. Compiles to `wasm32-wasip2` [4]. Implements one-way authentication based on Ed25519-JWT tokens and high-concurrency dual-isolation cache distribution [2]. No real-time interface alignment or comparison logic is executed on the gateway. |
| **`lunar-scope`** | **Visualization Presentation Layer** | Pure static frontend multi-relational canvas. Pulls pre-aligned topological JSON from the gateway via standard APIs, and renders an interactive UI with magnetic prediction dashed lines, breakpoint highlighting and architecture drift warnings in browsers. |

### 1.1 Metaphor Mapping & Physical Correspondence
*   **Data Reflection (LunarAST Standard Layer)**: The system generates no runtime data (no active monitoring) and introduces zero performance overhead at runtime. It captures physical facts from source code changes triggered by developer commits and compilation, then projects them as static artifacts.
*   **Ecosystem Presentation (lunar-scope Presentation Layer)**: A pure static multi-layer relational canvas. It delivers low-latency observation of contract data for architects, and detects orphaned breakpoints and unintended ad-hoc dependencies before code deployment [2].

---

## 2. Three-tier Progressive Source of Truth Architecture
```
      ┌────────────────────────────────────────────────────────┐
      │   Tier 1: Physical Facts (AST)                           │  ← 80-90% auto-generated, stored in .interfaces-autogen.json
      ├────────────────────────────────────────────────────────┤
      │   Tier 2: Intent & Override Overlay                      │  ← Human-controlled, stored in .lunar/interfaces.yml (primary governance layer)
      ├────────────────────────────────────────────────────────┤
      │   Tier 3: Escape Hatch (Inline Comments)                 │  ← Reserved only for extreme edge cases of dynamic RPC calls
      └────────────────────────────────────────────────────────┘
```

### 2.1 Tier 1: Physical Facts — Auto-Generated Data
*   **Physical File**: **`.lunar/.interfaces-autogen.json`** (This file **must** be added to the project `.gitignore`).
*   **Lifecycle of Physical Facts**: Since this file is excluded from Git, CI/CD pipelines or local compilation will automatically execute `lunar scan` to rebuild the physical fact cache. No historical cache dependencies are required.

### 2.2 Tier 2: Intent & Override Overlay — Partial Field Override & Merge Rules
*   **Physical File**: **`.lunar/interfaces.yml`** located in the project root directory.
*   **Attributes**: Fully human-managed, tracked by Git. Toolchains are strictly prohibited from making silent unauthorized modifications.
*   **Merge Operator ($\oplus$) & Partial Field Override Definition**:
    For an interface object uniquely identified by the composite primary key `(Path, Method)`, the alignment engine enforces **partial field override** rules during compilation:
    *   Let $A$ = an interface object from Physical Facts (raw AST), $I$ = the corresponding interface definition from the Intent Overlay.
    *   For any field $f$ of the interface object (e.g. `port`, `rawConstraint`):
        $$\text{Resolved}.f = \begin{cases} 
          I.f, & \text{if } I.f \text{ is specified} \\
          A.f, & \text{otherwise} 
        \end{cases}$$
    *   This rule ensures that when humans override specific network parameters or path attributes, **all other implicit metadata from Physical Facts (e.g. source file paths `sourceFile`, line numbers `lineNumber`) are preserved intact** [2].
    *   **Override Boundary Restriction**: For array-type fields (e.g. `segments`) and nested objects, the overlay only supports **full replacement**, not partial field merging. If both the overlay and physical facts contain non-empty values with mismatched lengths or child keys, `lunar diff` must throw an error and reject automatic merging (even in non-strict mode), forcing manual full rewrite of the array content.
    *   **Conflict Detection**: If the overlay modifies core properties from physical facts, `lunar diff` outputs highlighted warning logs to the terminal for human review. Compilation will not be blocked in non-strict mode.

### 2.3 Tier 3: Escape Hatch
*   **Definition**: Single-line inline directives starting with `// lunar:` or `# lunar:` above source code lines [2].
*   **Usage Boundary**: **Reserved exclusively for dynamically assembled RPC/HTTP calls, where target paths and methods cannot be captured by static AST analysis due to runtime dynamic evaluation. For standard routes, adapters must extract physical facts; duplicate declarations via comments are forbidden.**
*   **Syntax Example**:
    ```typescript
    // lunar:consume POST https://api.auth-service/v1/token
    await axios.post(dynamicUrl, data);
    ```

---

## 3. Four-tier Decoupled Architecture & Topology Generation
To guarantee maximum robustness of the core system, LunarAST splits complex distributed dependencies into four independent, physically isolated subdomain contracts to eliminate protocol bloat [5]:

1.  **`RouteAST` (Routing & Network Contract)**: Manages synchronous network interfaces (REST/gRPC/Nginx). Metadata includes `method`, `segments`, `port`. **Current status: v0.5.0 draft specification**.
2.  **`EventAST` (Event & Asynchronous Contract)**: Manages asynchronous event publish/subscribe and request-response patterns. Metadata includes `brokerType` (e.g. Kafka/NATS/RabbitMQ), `action` (enum: `publish` / `subscribe` / `request` / `reply`), `topic`, `payloadSchemaHash`. **Specification in planning**.
3.  **`SchemaAST` (Storage & Data Contract)**: Manages underlying database tables, object storage buckets and cache dependencies, used to detect hidden and critical implicit database coupling between microservices. Metadata includes `storageType` (e.g. PostgreSQL/MongoDB/S3/Redis), `database` (bucket/DB name), `table` (table name / key pattern), `operation` (enum: `read` / `write` / `join`). **Specification in planning**.
4.  **`TypeAST` (Code & Library Contract)**: Manages compile-time code reuse and strongly bound library dependencies. Metadata includes `libraryName` (shared package name), `typeIdentifier` (DTO / public structure), `interfaceDefinitionFile` (e.g. `.proto` / gRPC stub path), `versionConstraint`. **Specification in planning**.

### 3.1 Multi-repo Ecosystem Topology Generation (Static Merging & Generation Synchronization)
*   **Single Repository (CI Phase)**:
    Each repository runs `lunar scan` during build to extract `<subdomain>-actual.json` (e.g. `route-ast-actual.json`) and upload it to S3/R2 [2].
    *   **Pointer Update Mechanism**: After successful upload of `<subdomain>-actual.json`, the single-repo CI pipeline **must immediately update and upload the repository pointer file `pointers/latest.json`**, where the `sha` field points to the current commit SHA. This ensures ecosystem orchestrators can capture the latest valid physical version in real time.
*   **Generation Build Trigger & Atomicity Guarantee**:
    To ensure one ecosystem-wide build maps to a single atomic, valid global static snapshot during asynchronous CI execution across multiple services, the system adopts an **Ecosystem Lockfile** synchronization mechanism:
    1.  **Plan Generation**: Ecosystem administrators or ecosystem automation tools periodically poll the `latest.json` pointers of all projects to generate a static snapshot file named **`ecosystem-plan.json`**. A unique `generationId` is assigned to each generation (format: `<timestamp>-<uuid>`), generated by the ecosystem publisher based on timestamp and content hash upon file creation.
    2.  **Plan Publication**: The lockfile is uploaded to `ecosystem-config/generations/<generationId>/ecosystem-plan.json`.
    3.  **Central Pipeline Activation**: The upload event of the lockfile acts as a static trigger to activate the centralized alignment pipeline, eliminating circular dependency deadlocks [2].
*   **Core Alignment & Validation Rules**:
    1.  **Input Validation (Set Equality Check)**: **On startup, the central pipeline enforces strict set equality: the full project set defined in `ecosystem-plan.json` must exactly match the registered project set in `repos.json`.** If any registered project is missing or unregistered projects are included, the pipeline terminates with an error and refuses execution. This guarantees every generation snapshot is a complete projection of the entire ecosystem and prevents false alignment caused by partial snapshots.
    2.  **Idempotency Guarantee**: The pipeline first checks if `ecosystem-config/generations/<generationId>/lunar-map.json` already exists. If so, the generation is marked ready and computation is skipped directly.
    3.  **Generation Fetch & Status Classification**: The pipeline fetches `actual.json` caches of all projects in parallel:
        *   **All artifacts ready**: The pipeline aligns data and generates the global `lunar-map.json`.
        *   **`failed` Status**: For projects whose `actual.json` is not uploaded within the 5-minute timeout window, mark their `scanStatus: failed` in `lunar-map.json` and set the `interfaces` field to `null` forcibly [2].
        *   **`stale` Status**: If historical `actual.json` artifacts exist for a project, but the time difference between its `lastUpdated` timestamp and the current generation build time exceeds the predefined retention period (default: 7 days, controlled by `LUNAR_MAX_STALE_AGE_SECONDS`): The pipeline uses the historical data for alignment, marks `scanStatus: stale`, and forces all alignment entries generated from this project to `unverified` [2].
    4.  **Global Ecosystem Pointer Update**: After uploading the aligned `lunar-map.json` and `meta.json`, the central pipeline **atomically updates the global top-level pointer `ecosystem-config/latest-generation.json`**, setting its `generationId` and `lastUpdated` to the newly completed generation for stateless clients to discover the latest version dynamically.
*   **Reverse Lookup & Null Safety Mechanism**:
    If `<subdomain>-actual.json` is physically missing from the storage bucket due to network issues or expiration, the gateway performs reverse lookup using `projects[].interfaces` inside `lunar-map.json`.
    *   **Null Exception Handling**: If a project’s `interfaces` is set to `null` due to build failure isolation, the gateway determines the source data is unavailable. It terminates the request, returns `410 Gone` with the `X-Lunar-Recovery` response header for rebuild guidance, and includes the error code `ERR_LUNAR_INTERFACE_DATA_MISSING` in the response body.
    *   **Failure Isolation & Consistency Disclosure**: After the central pipeline marks a project as `scanStatus: failed` and `interfaces: null`, the project’s standalone CI may still upload `actual.json` later. In this case, the file exists physically in storage but is treated as unavailable by the global topology due to build isolation rules. If clients bypass the topology and directly access the artifact via `GET /commits/<sha>/route-ast-actual.json`, the request will succeed. Such inconsistencies are inherent to distributed build isolation; developers must use `lunar doctor` to diagnose global state consistency.

### 3.2 Zero-Friction Source Code Projection & Fallback Routing Specification
To enable AI Agents to access source code on-demand without archiving overhead, the system implements URL projection and path fallback mechanisms:

1.  **Route Alias Normalization**:
    When serving GitHub-style file access, the gateway and local read-only service treat `/blob/` (web file view) and `/raw/` (raw text access) as fully equivalent routes. AI clients can replace `github.com` with the ecosystem domain to retrieve raw file content without adjusting URL syntax [1.2].
2.  **Two-tier Base Path Fallback Priority**:
    When resolving a project’s physical workspace path, the service enforces the following fallback chain. Hardcoded global paths are strictly prohibited:
    $$\text{ResolvedPath} = \begin{cases} 
      \text{Registry.path}, & \text{if defined in repos.json} \\
      \text{Topology.path}, & \text{else if auto-discovered in lunar-map.json} \\
      \text{Error (400)}, & \text{otherwise} 
    \end{cases}$$
3.  **Case-Insensitive Normalization**:
    When matching GitHub-style coordinates `{owner}/{repo}/{branch}`, the gateway and service layer convert all paths to lowercase for hash mapping in memory, eliminating routing failures caused by cross-platform case sensitivity differences [1.2].

---

## 4. Data Generation Layer & Compilation Pipeline (`lunar`)

### 4.1 Phase 1: Detect & Extract — Process Isolation, Adapter Override & Atomicity Guarantee
*   **Adapter Discovery & Override Mechanism**:
    The `lunar` controller dynamically scans the system `PATH` for binaries named `lunar-extract-<lang>`. Users can explicitly define `adapters` paths in `.lunar/config.yml` for absolute path override, which takes higher priority over automatic `PATH` discovery.
*   **JSON Lines Streaming & End-of-Stream Atomic Marker**:
    The orchestrator spawns adapter child processes. To prevent heap overflow for large projects, adapters must adopt line-by-line streaming output with immediate flushing [3]. Each extracted route is written to `stdout` and flushed instantly; in-memory full-data accumulation and one-time serialization are forbidden [3].
    *   **Stream Integrity & Count Validation**: To avoid corrupted partial data caused by adapter crashes, **adapters must output a dedicated end marker line after all routes are successfully extracted**:
        `{"_lunar": {"status": "success", "count": 42}}`
        After receiving this marker, the orchestrator **strictly verifies whether the total parsed route count matches the `count` value**. If mismatched (indicating truncated data), the orchestrator discards all output and throws `ERR_LUNAR_ADAPTER_CRASH`. All debug and warning logs are redirected to `stderr` and must not pollute `stdout` [1.2.1].
*   **Adapter Failure Isolation**:
    If an adapter for a specific framework exits with a non-zero code (crash), the orchestrator catches the error gracefully, skips scanning for the current project, logs a warning, and continues execution. The main CI process will not be interrupted.

### 4.2 Phase 2: Confirm & Semantic Normalization — Constraint Propagation Rule
*   **Semantic Normalization**: Translate framework-specific regex and constraint expressions (e.g. `:id(\\d+)` for Express, `{id:int}` for FastAPI) into unified Rust-native `RouteAst` models [1.1.1].
*   **Constraint Limitation & Compromise**:
    Mathematically proving equivalence for arbitrary cross-language regular expressions is impractical in engineering. In v0.5.0, the confirmation engine **does not recompile constraint regex**. The original `rawConstraint` value is preserved as descriptive metadata. The alignment engine only compares path positions and basic data types during matching.
    *   Limitation: The system cannot automatically detect or alert runtime 400 errors caused by asymmetric regex matching rules across multi-language frameworks.

### 4.3 Zero-Privilege Sync & Guided Sync Mechanism
To ensure full developer control over source code, the `lunar` CLI is forbidden from silently modifying human-maintained files in the background.
*   **`lunar init`**: Only runs automatic scanning and generates template files if `.lunar/interfaces.yml` does not exist locally. No existing manual configurations will be overwritten.
*   **`lunar diff`**: Compares physical AST facts with the intent overlay file, and outputs standard Git-diff style change reports to the terminal.
*   **`lunar sync --apply`**: Manually triggered merge command; supports `--dry-run` for change preview. Before writing changes, the tool automatically backs up the old `interfaces.yml` to the hidden backup directory `.lunar/.backup/interfaces.yml.bak`. This backup path is automatically added to `.gitignore` by `lunar init`.
*   **AI Suggestion Patch Mechanism**: The `.lunar/suggestions/` directory stores YAML patches generated by humans or AI for intent overlay updates. During `lunar sync --apply`, the CLI automatically processes patch files in this directory and moves processed patches to the `merged` subdirectory.

---

## 5. Project-level Intent Overlay & Ecosystem Configuration (YAML/JSON Schemas)
All configuration fields follow the Google JSON Style Guide (**camelCase**) for cross-team and cross-language consistency.

### 5.1 `.lunar/interfaces.yml` Specification
The project-level centralized contract acts as the primary governance layer for developers to define interface boundaries:
```yaml
# ===================================================================
# LunarAST Project Interface Contract
# This file is owned and maintained by humans.
# ===================================================================

project: myPaymentService
type: mixed # Role: service / client / mixed
environment: production

# Manually declared APIs
exposed:
  - path: /api/v1/payments/refund
    method: POST
    reason: "Refund dedicated endpoint (scheduled for next week release)"

# Manual contract override for complex dynamic calls
consumed:
  - path: /api/v1/auth/verify
    method: POST
    targetProject: authService  # Must exist in ecosystem repos.json, otherwise ldg doctor throws an error
    reason: "Pre-transaction session verification"
```

### 5.2 Ecosystem Registry: `repos.json`
```json
{
  "version": "0.5.0",
  "comment": "The version field defines the schema compatibility version of this registry configuration itself.",
  "projects": [
    "myPaymentService",
    "authService",
    "billingService"
  ]
}
```

### 5.3 Ecosystem Topology Declaration: `ecosystem-topology.json`
```json
{
  "ecosystem": "lunarEcosystem",
  "version": "0.5.0",
  "projects": {
    "myPaymentService": {
      "layer": "businessOrchestration",
      "criticality": "high"
    }
  },
  "relationships": [
    {
      "from": "myPaymentService",
      "to": "authService",
      "type": "rpcSync",
      "reason": "Session authentication"
    }
  ]
}
```

### 5.4 Ecosystem Build Lockfile: `ecosystem-plan.json`
This file is the single source of truth for ecosystem generation builds. It is fully managed and written by ecosystem automation tools and locked upon each new `generationId`.
```json
{
  "$schema": "https://routeast.dev/schema/v1/ecosystem-plan.schema.json",
  "generationId": "20260608T030000Z-a1b2c3d4",
  "created": "2026-06-08T03:00:00Z",
  "projects": {
    "myPaymentService": {
      "sha": "abc123e456f789..."
    },
    "authService": {
      "sha": "def456a789b123..."
    }
  }
}
```

---

## 6. Data Exchange Format Specification (The Exchange Contract Spec)
All externally exposed artifacts follow strict static format definitions to maximize parsing efficiency and data density.

### 6.1 Structured Topology Schema: `lunar-map.json`
Top-level schema definition for `lunar-map.json` (fully compliant with Google camelCase naming):
```json
{
  "$schema": "https://routeast.dev/schema/v1/lunar-map.schema.json",
  "type": "object",
  "required": ["version", "projects", "alignments"],
  "properties": {
    "version": { "type": "string", "pattern": "^\\d+\\.\\d+\\.\\d+$" },
    "projects": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["name", "type", "sha", "interfaces", "scanStatus"],
        "properties": {
          "name": { "type": "string" },
          "type": { "type": "string", "enum": ["service", "client", "mixed"] },
          "sha": { "type": "string" },
          "scanStatus": { "type": "string", "enum": ["success", "failed", "stale"] },
          "interfaces": {
            "type": ["object", "null"],
            "required": ["exposed", "consumed"],
            "properties": {
              "exposed": {
                "type": "array",
                "items": {
                  "type": "object",
                  "required": ["path", "method"],
                  "properties": {
                    "path": { "type": "string" },
                    "method": { "type": "string" }
                  }
                }
              },
              "consumed": {
                "type": "array",
                "items": {
                  "type": "object",
                  "required": ["path", "method", "targetProject"],
                  "properties": {
                    "path": { "type": "string" },
                    "method": { "type": "string" },
                    "targetProject": { "type": "string" }
                  }
                }
              }
            }
          }
        }
      }
    },
    "alignments": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["clientProject", "serverProject", "path", "method", "status"],
        "properties": {
          "clientProject": { "type": "string" },
          "serverProject": { "type": "string" },
          "path": { "type": "string" },
          "method": { "type": "string" },
          "status": { "type": "string", "enum": ["Aligned", "ParamNameMismatch", "Unused", "Orphaned", "MethodMismatch", "unverified"] }
        }
      }
    }
  }
}
```

### 6.2 AI Context Markdown: `lunar-map.md` (Markdown Spec)
`lunar-map.md` is not persisted as a static file in storage. Instead, **`lunar-gateway` (or local `lunar-serve`) dynamically parses and renders it from `lunar-map.json` upon client requests**. Query parameters are supported for filtering to reduce AI token consumption:
*   `GET /lunar-map.md?summary=true`: Return condensed summary (minimum token usage, for initial AI access) [1.2].
*   `GET /lunar-map.md?style=list`: Return plain contract list (for fast extraction in small contexts).
*   `GET /lunar-map.md?style=mermaid`: Return topology graph (for global visualization).
*   `GET /lunar-map.md?scope=project-a`: Sharded access for specific projects, preventing AI context window overflow.

---

## 7. Open Integration & Third-party Bridge Isolation Specification
LunarAST maintains full statelessness and data sovereignty. All external sandboxes (IDE plugins, AI Agents) must follow the **bridge isolation pattern** for integration [2]:

*   **Bridge Responsibilities**: Third-party integrators build standalone bridge applications (e.g. `routeast-mcp-bridge`), which pull `lunar-map.json` via standard HTTP `GET` requests and convert it into target protocols internally.
*   **No Unauthorized Modification Rule**: When converting AI-generated alignment suggestions into local changes, bridges are **strictly forbidden from silently modifying `interfaces.yml`**. Bridges only output Git-diff style patch snippets to the terminal and prompt users to run `lunar sync --apply` manually.

```rust
// Minimal interaction contract between bridges and LunarAST
#[async_trait]
pub trait LunarMcpBridge {
    /// 1. Fetch pure static topology, support sharding via (?scope=projectX) to reduce token usage
    async fn fetch_lunar_map(&self, gatewayUrl: &str, scope: Option<&str>) -> Result<LunarMapPayload, BridgeError>;
    
    /// 2. Convert topology into standard Tools & Resources schema for JSON-RPC
    fn translate_to_mcp_tools(&self, payload: LunarMapPayload) -> Vec<McpToolSchema>;
}
```

---

## 8. Stateless Distribution Layer & Security Model (`lunar-gateway`)

`lunar-gateway` is dedicated to high-concurrency, stateless, secure distribution of contract data [2].

### 8.1 Standard Storage Directory Structure
*   **Recommended S3 Bucket Naming**: `lunar-ast-<organization>`.
*   **Global Topology Storage Rule**: `lunar-map.json` is an ecosystem-wide artifact and **must not be stored in the `commits` directory of any single service**. The gateway stores all generation artifacts uniformly under `ecosystem-config/`.

```
s3://lunar-ast-<organization>/
├── <repo>/
│   ├── commits/
│   │   └── <sha>/
│   │       └── <subdomain>-actual.json # Physical fact cache (e.g. route-ast-actual.json)
│   └── pointers/
│       └── latest.json                # Composite pointer for the latest SHA
└── ecosystem-config/
    ├── repos.json                     # Ecosystem registered project allowlist
    ├── ecosystem-topology.json        # Orchestration & topology declaration
    ├── latest-generation.json          # Static pointer to the current active generation
    └── generations/
        └── <generationId>/
            ├── meta.json              # Metadata summary for this generation
            ├── ecosystem-plan.json    # Build lockfile for this generation
            └── lunar-map.json         # Global aligned topology
```

#### 8.1.1 Metadata File: `meta.json` Schema
Generated and synced by CI/CD pipelines during build:
```json
{
  "sha": "abc123e456f789...",
  "lastUpdated": "2026-06-08T03:00:00Z",
  "compilerVersion": "0.5.0",
  "downgradeMap": {
    "type": "object",
    "additionalProperties": {
      "type": "object",
      "properties": {
        "fieldMappings": {
          "type": "array",
          "items": {
            "type": "object",
            "required": ["sourceField", "targetField"],
            "properties": {
              "sourceField": { "type": "string" },
              "targetField": { "type": "string" }
            }
          }
        },
        "removedEnums": {
          "type": "array",
          "items": {
            "type": "string"
          }
        }
      }
    }
  }
}
```
*   **Field Description**: `compilerVersion` refers to the version of the `lunar` binary that performed static analysis and export. It is decoupled from the gateway version. If `downgradeMap` is missing or no compatible downgrade rule exists for the client version, the gateway rejects lossy downgrade.

#### 8.1.2 Global Generation Pointer: `latest-generation.json` Schema
```json
{
  "generationId": "20260608T030000Z-a1b2c3d4",
  "lastUpdated": "2026-06-08T03:00:00Z"
}
```

### 8.2 Dual-isolation Cache & Client Cache Control
To balance mandatory authentication and zero object storage penetration, `lunar-gateway` adopts a dual-isolation caching strategy:
*   **Internal Cache**: The gateway uses the full logical path of each object in the bucket as the cache key.
*   **Authentication First Rule**: For private resources requiring JWT authentication (e.g. private project `actual.json`), **the gateway completes full token validation (signature & timestamp check) first before accessing internal cache**. Authentication can never be bypassed by cache. Internal cache uses token-free logical paths as keys for cross-client sharing.
*   **Differentiated Client Cache Headers**:
    *   **Private Resources (JWT required)**: Override all edge cache headers to `private, no-cache, no-store, must-revalidate` to prevent token hijacking. Internal cache still uses immutable rules with token-free keys.
    *   **Public Immutable Resources (No token required)**: Artifacts under `/commits/<sha>/`, `generations/` (including `lunar-map.json`, `ecosystem-plan.json`, `meta.json`) return `Cache-Control: public, max-age=31536000, immutable` for long-term client caching.
    *   **Dynamic Pointers (`/pointers/latest.json` & `latest-generation.json`)**: Allow short-lived client caching with `Cache-Control: public, max-age=300` to balance performance and version consistency.
*   **Dynamic Render Cache & OOM Protection**:
    *   **Render Cache**: For `lunar-map.md` requests, the gateway caches rendered Markdown content using composite keys `(Accept, scope, generationId)`. Cache lifetime is strictly aligned with the corresponding `lunar-map.json` version.
    *   **Memory Circuit Breaker**: Following the 128MB memory limit of Cloudflare Workers, files larger than 2MB trigger a circuit breaker: in-memory buffering is disabled, and **stream-through forwarding** is used instead to avoid isolate crashes. The threshold can be customized via the environment variable `LUNAR_GATEWAY_BUFFER_LIMIT_BYTES`.

### 8.3 Authentication & JWT Signature Contract
*   **Signature Algorithm**: **Ed25519 (EdDSA)** is enforced as the sole signing algorithm for constant-time verification [1.3.3].
*   **Expiry Policy**: The JWT `exp` claim is recommended to be set to a maximum of 24 hours.
*   **Public Key Distribution & KV Cache Rotation**: Public keys can be deployed via static Worker environment variables or dynamic Cloudflare KV. To control memory usage, the gateway limits each project to store a maximum of 3 active public keys (`MAX_KEYS_PER_REPO = 3`) to prevent memory exhaustion from malicious key registration.
*   **URL Token Warning**: Passing JWT tokens via URL parameters is strongly discouraged in production. The gateway outputs security warnings to `stderr` when URL-borne tokens are detected.

### 8.4 Version Negotiation & Backward Compatibility
When the major version of `lunar-map.json` is incremented (e.g. 0.5.0 → 1.0.0), clients negotiate compatibility via HTTP content type parameters:
*   **Negotiation Rule**: Clients send requests with the header `Accept: application/vnd.lunar.0.5.0+json`. The gateway maintains a compatibility window for **the current major version and the immediate previous major version (total 2 versions)**.
*   **Downgrade Boundary**: Automatic downgrade is only allowed if lossless mapping rules exist in `downgradeMap` inside `meta.json`. If breaking changes are introduced (required fields removed, enums modified), the gateway returns `406 Not Acceptable` with the header `X-Lunar-Upgrade-Required: true` and refuses downgrade.

### 8.5 Crate Refactoring & Cargo Workspace Dependency Governance
To avoid module deadlocks and excessive compilation overhead as the ecosystem scales, LunarAST adopts decoupled Cargo Workspace rules:
1.  **Isolate Contract Crate**: Extract the zero-dependency core crate `lunar-interface`, which only contains core data models (`RouteEntry`, `ActualJson`, `LunarMap`) and the `generate_lunar_map` topology alignment logic.
2.  **Decouple CLI & Serving Layers**: Distribution layers (`lunar-serve`, `lunar-gateway`) depend **only** on `lunar-interface`. Direct or indirect dependencies on CLI-only crates (e.g. `clap`, `rust-s3`, `ed25519-dalek`) are forbidden. This maximizes compilation speed and keeps serving layers lightweight.

---

## 9. Alignment Status Priority & Diagnostic Rules (Diagnostic Short-Circuit)
The alignment engine evaluates client-side and server-side `RouteAst` entries with short-circuit logic. Each interface pair maps to exactly one final status, with priority defined as:
$$\text{MethodMismatch} > \text{Orphaned} > \text{Unused} > \text{ParamNameMismatch} > \text{Aligned}$$

*   **Handling Failed & Stale Nodes (`unverified` Status)**:
    To prevent false alignment errors caused by incomplete or outdated data from failed/stale CI builds:
    1.  **Pre-check**: The alignment engine reads the `scanStatus` field of all projects in the `projects` array before computation [2].
    2.  **`stale` Nodes**: Data from `stale` projects is included in alignment computation, but **all resulting alignment entries are forced to `unverified`** [2].
    3.  **`failed` Nodes**: Projects marked `failed` have `interfaces: null` and cannot participate in regular comparison. The engine scans all other healthy projects: if any healthy project consumes endpoints from the `failed` service, a dedicated alignment entry with `status: "unverified"` is generated. Exposed interfaces from `failed` projects do not generate alignment entries and will not be misclassified as `Unused`.
    4.  **`unverified` Semantics**: Entries marked `unverified` are rendered as yellow warning lines in `lunar-scope`. They are not treated as formal contract violations during auditing, indicating only outdated or unavailable data sources.
*   **Diagnostic Rules (Short-circuit by Severity)**:
    1.  **MethodMismatch (Priority 1)**: Path structure matches, but HTTP methods differ (e.g. client uses `POST`, server exposes `GET`). Evaluation terminates immediately.
    2.  **Orphaned (Priority 2)**: A client calls an endpoint, but no registered service in `repos.json` provides a matching path and method (broken endpoint).
    3.  **Unused (Priority 3)**: Topology-level global status: A server exposes an endpoint with no clients consuming it. Identified in the post-processing phase of `lunar-map.json` generation. Critical for architecture governance and attack surface reduction.
    4.  **ParamNameMismatch (Priority 4)**: Path position and data type match, but parameter names differ. Triggers magnetic dashed warning lines in the frontend.

---

## 10. Observability & Health Check Specification

### 10.1 Structured Logging (JSON Lines)
`lunar-gateway` and local read-only services output single-line JSON structured logs to `stdout` (camelCase fields):
```json
{"timestamp":"2026-06-12T01:00:00Z","level":"INFO","method":"GET","path":"/public/repo-a/commits/sha-123/lunar-map.json","status":200,"durationMs":12,"cache":"HIT","authStatus":"valid","clientIp":"12.34.56.78"}
```
*   **`authStatus` Enums**: `valid` / `expired` / `invalidSignature` / `missingToken`.

### 10.2 Prometheus Metrics
The gateway exposes a standard `/metrics` endpoint for in-memory statistics. All `camelCase` labels are converted to `snake_case` to follow Prometheus conventions:
*   `lunar_gateway_requests_total`: Total request counter, labels: `method`, `status`.
*   `lunar_gateway_cache_hits_total`: Cache hit counter, labels: `cacheType` (`internal`/`client`).
*   `lunar_gateway_auth_failures_total`: Authentication failure counter, labels: `reason` (`expired`/`signatureInvalid`/`missingToken`).
*   `lunar_gateway_version_downgrade_requests_total`: Downgrade request counter, label: `targetVersion`. Used to track client version migration progress.

### 10.3 Liveness Health Check
The gateway exposes a token-free liveness probe: `GET /healthz`, returns `200 OK` on success for container and edge runtime health monitoring.

---

## 11. Security Threat Model & Mitigation Strategies
Four layers of defense for zero-trust environments:

1.  **Injection Attack Prevention & Lexical Rules**:
    Since inline `// lunar:` / `# lunar:` directives are parsed lexically, strict lexical filtering combined with AST node validation is enforced for each language. Adapters only parse lines matching predefined regex rules (e.g. `#\s*lunar:(expose|consume)` for Python, `<!--\s*lunar:(expose|consume)\s*-->` for HTML). Malformed or unmatched lines are logged to `stderr` with file path and line numbers, then discarded to block injection risks.
2.  **Key Rotation & Memory Protection**:
    The gateway uses soft TTL cache for public keys and limits `MAX_KEYS_PER_REPO = 3` to prevent memory exhaustion from excessive obsolete public keys.
3.  **Least-privilege S3 Credentials**:
    CI Action S3 credentials are granted only `s3:PutObject` and `s3:GetObject` permissions for the target bucket. Bucket deletion or IAM policy modification permissions are strictly denied.
4.  **Destructive Operation Confirmation**:
    All destructive or ecosystem-wide cleanup commands (e.g. `lunar cleanup`) require interactive confirmation in the terminal. The `--yes` flag skips prompts only for CI/CD automation, preventing accidental misuse.

---

## 12. Core Error Codes & Troubleshooting Guide

| Error Code | HTTP Status Code | Root Cause | Recommended Resolution |
|:---|:---|:---|:---|
| `ERR_LUNAR_CONFIRM_FAIL` | 400 | Normalization validation failed; wildcard/constraint syntax invalid. | Run `lunar diff` and fix syntax errors in reported segments. |
| `ERR_LUNAR_ADAPTER_CRASH` | 422 | Adapter subprocess crashed, or parsed entry count mismatches the `count` value in the end marker (truncated data). | Check adapter stack traces in CI `stderr` logs. |
| `ERR_LUNAR_PROJECT_NOT_FOUND` | 404 | Target project not registered in `repos.json`. | Add the project to `repos.json` and retrigger build. |
| `ERR_LUNAR_INTERFACE_NOT_FOUND` | 422 | Target project exists, but no exposed method/path matches the client request. | Run `lunar diff`, align client consumption and server exposure rules, then update `interfaces.yml` and sync. |
| `ERR_LUNAR_INTERFACE_DATA_MISSING` | 410 | Source data unavailable due to build failure (`scanStatus: failed`, `interfaces: null`). | Run `lunar doctor` to diagnose build failures across distributed services. |
| `ERR_VERSION_EXPIRED` | 410 | Physical fact artifacts auto-purged after 90 days retention. | The response includes the `X-Lunar-Recovery` header with CI webhook URL; retrigger CI pipeline for the target repository. |
| `ERR_GATEWAY_STREAM_FALLBACK_FAILED` | 502 | Stream-through forwarding failed for oversized files (>2MB). | Remove non-essential static assets from `lunar-map.json` and verify storage backend connectivity. |

---

## 13. Roadmap & Core Evolution Milestones
Timeline is not strictly defined; delivery is milestone-driven with tangible artifacts:

*   **Milestone 1 (RouteAST Core Implementation)**:
    *   Freeze `RouteAST Base IR v0.5.0` specification.
    *   Implement pure Rust confirmation core prototype.
    *   Release lightweight adapters for Rust (Axum) and Node.js (Express).
*   **Milestone 2 (Full Four-tier Architecture & Secure Distribution)**:
    *   Release `lunar` CLI with core commands: `init`, `scan`, `diff`, `sync --apply`.
    *   Deploy `lunar-gateway` with dual-cache, tiered cache control, memory circuit breaker, structured logging and Prometheus metrics.
    *   Complete multi-crate Workspace decoupling with standalone `lunar-interface`. Implement case normalization and two-tier path fallback.
*   **Milestone 3 (Multi-dimensional Visualization & Standard Finalization)**:
    *   Release `lunar-scope` interactive canvas with magnetic positioning and full status visualization for `Unused` / `unverified` endpoints.
    *   Freeze `EventAST` & `SchemaAST` specifications and release official adapters.
*   **Milestone 4 (Full Static Code Audit & Open Ecosystem)**:
    *   Release `TypeAST` specification and cross-project compile-time dependency auditing.
    *   Expose standard `lunar-map.json` for third-party security audit platforms; implement full-stack static alignment and drift prevention.

---

## Appendix B: References & Specification Sources
*   RFC 6570 - URI Template Specification for parameter standardization.
*   IEEE Std 1471-2000 - Systems and software engineering - Recommended practice for architectural description of software-intensive systems.
*   JSON Lines Standard (v1.0) - Line-Delimited JSON streaming format.
*   WASI 0.2 (Component Model) - WebAssembly System Interface specification.
*   ACM TOSEM Vol. 33 - Cross-Language Static Program Analysis on Microservice Topologies.
*   [1.3.3] NIST FIPS 186-5 - Digital Signature Standard (DSS) guidelines for Ed25519 system signature integration and curve verification.

---

### A.5 CLI Quick Reference (Cheat Sheet)
```bash
lunar init                     # Auto-detect tech stack and initialize local template (only if interfaces.yml does not exist)
lunar scan                     # Perform static scan for physical facts and write to .interfaces-autogen.json
lunar diff                     # Print Git-diff style comparison between physical facts and intent overlay
lunar sync --dry-run           # Preview sync changes before applying
lunar sync --apply             # Backup old config then merge and write changes
lunar doctor                   # Verify S3/R2 connectivity, pointer status and global topology consistency
lunar cleanup --all            # Interactive cleanup of all S3/R2 artifacts for current repo (commits/ + pointers/); add --yes to skip prompt (high risk)
```
