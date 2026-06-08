# LunarAST Protocol Family Unified Ecosystem White Paper

**——Multi-Layer Heterogeneous Static Contract Standard & "Birth, Transmission, Labeling, Exhibition" Four-Layer Architecture Specification**

**Version**: 1.0 — Official Release
**Last Updated**: 2026-06-08
**Scope**: Interface-level static contract verification for multi-language/multi-repository microservice ecosystems, CI pipeline gating, architectural drift pre-verification, and AI agent zero-knowledge addressing
**Normative Constraints**: Full compliance with Google Naming & JSON Style Guide (`camelCase` naming convention)

---

## Part I: LunarAST Ecosystem Mother Specification

### 1. Physical Role Definitions

This specification defines the physical boundaries and naming contracts of each component in the LunarAST ecosystem:

| Component/Naming | Physical Layer | Core Responsibility |
|:---|:---|:---|
| **LunarAST** | **Standard Specification Layer** | Defines the Base IR specification, extraction protocol, and mathematical comparison algorithm semantics for multi-layer contracts (RouteAST, EventAST, etc.). It is the underlying static contract standard, existing as a static Schema specification and containing no executable code. |
| **`lunar`** | **Data Generation Layer** | Command-line executable binary for local and CI pipelines. Responsible for `lunar init` (initialize draft), `lunar scan` (physical extraction), `lunar diff` (non-privileged comparison), and `lunar sync --apply` (active backup and secure synchronization). Alignment decisions and merge logic are executed entirely here. |
| **`lunar-gateway`** | **Stateless Distribution Layer** | Independently deployed Serverless edge gateway program. Compiled to `wasm32-wasip2`, performs one-way authentication based on security tokens (Ed25519-JWT) and high-concurrency dual-phase isolated cache distribution. The gateway does not run any real-time interface alignment comparison logic. |
| **`lunar-scope`** | **Visualization Presentation Layer** | Purely static front-end multi-layer relationship canvas. Pulls topology JSON already aligned during the build phase from the gateway via standard APIs, rendering a front-end human-machine interface with magnetic prediction dashed lines, breakpoint highlighting, and architectural drift warnings in the browser. |

#### 1.1 Metaphorical Positioning and Physical Correspondence
*   **Data Reflection (LunarAST Standard Layer)**: The system itself does not generate runtime data (does not emit light), and does not participate in any runtime monitoring or performance overhead. It receives physical facts from source code changes during the build phase (the terrain illuminated by developer commits and compilation actions) and statically projects them.
*   **Ecosystem Presentation (lunar-scope Presentation Layer)**: As a purely static front-end multi-layer relationship canvas. It provides architects with multi-angle low-latency observation of contract data, capturing dangling breakpoints and unplanned wild dependencies before code deployment.

---

### 2. Three-Tier Progressive Source of Truth Architecture

```
      ┌────────────────────────────────────────────────────────┐
      │   Tier 1: Physical Facts (AST)                         │  ← 80-90% auto-derived, written to .interfaces-autogen.json
      ├────────────────────────────────────────────────────────┤
      │   Tier 2: Intent & Override Overlay                    │  ← Human-controlled, .lunar/interfaces.yml (primary gate)
      ├────────────────────────────────────────────────────────┤
      │   Tier 3: Escape Hatch (Comments)                      │  ← Only for extremely complex dynamic RPC edge calls
      └────────────────────────────────────────────────────────┘
```

#### 2.1 Tier 1: Physical Facts — Automatic Derivation
*   **Physical File**: **`.lunar/.interfaces-autogen.json`** (This file must be added to the project's `.gitignore`).
*   **Physical Fact Lifecycle**: Since this file does not enter the Git repository, during CI/CD or local compilation phases, the build machine automatically runs `lunar scan` locally to reconstruct this physical fact cache file, without depending on historical version caches.

#### 2.2 Tier 2: Intent & Override Overlay — Field-Level Partial Override and Merge Contract
*   **Physical File**: **`.lunar/interfaces.yml`** in the project root directory.
*   **Properties**: **100% controlled by humans, under version control (Git), tools strictly prohibited from any unauthorized silent modifications**.
*   **Merge Formula ($\oplus$) Field-Level Partial Override Definition**:
    For an interface object located by the same composite primary key `(Path, Method)`, the alignment engine executes the **"Field-Level Partial Override"** rule during compilation:
    *   Let $A$ be an interface object in the physical facts (Actual AST), and $I$ be the interface definition with the same name in the intent overlay.
    *   For any property field $f$ of the interface object (such as `port`, `rawConstraint`, etc.):
        $$\text{Resolved}.f = \begin{cases} 
          I.f, & \text{if } I.f \text{ is specified} \\
          A.f, & \text{otherwise} 
        \end{cases}$$
    *   This rule ensures that when manually overriding specific network parameters or path properties, **other silent metadata in the physical facts** (such as source code physical landmarks `sourceFile`, `lineNumber`, etc.) is preserved as-is.
    *   **Override Boundary Limitation**: For array types (such as `segments`) and nested objects, the overlay **only supports Complete Replacement**, not partial field merging. If both the intent overlay and physical facts are non-empty on such a field and differ in length or sub-keys, `lunar diff` must alert and refuse to execute automatic merge (even in non-strict mode), forcing manual complete rewrite of that array.
    *   **Conflict Detection**: If the overlay changes a core property already present in the physical facts, `lunar diff` must output a highlighted warning log to the terminal for human review, but the toolchain does not block compilation in non-strict mode.

#### 2.3 Tier 3: Escape Hatch
*   **Definition**: Single-line magic comments starting with `// lunar:` or `# lunar:` above source code lines.
*   **Applicable Boundary**: **Only for dynamically assembled RPC/HTTP calls where the target path and method cannot be captured by static AST analysis due to dynamic evaluation. For standard routes, the adapter forcibly extracts physical facts; duplicate declaration via comments is prohibited.**
*   **Syntax Example**:
    ```typescript
    // lunar:consume POST https://api.auth-service/v1/token
    await axios.post(dynamicUrl, data);
    ```

---

### 3. Four-Layer Decoupled Physical Blueprint and Topology Generation

To maintain the absolute robustness of the system kernel, `LunarAST` divides complex distributed dependencies into four mutually independent, physically isolated sub-domain contracts, completely preventing protocol bloat:

1.  **`RouteAST` (Routing & Network Contract)**: Focuses on synchronous network interfaces (REST/gRPC/Nginx). Metadata includes `method`, `segments`, `port`. **Sub-protocol published (see Part II)**.
2.  **`EventAST` (Event & Asynchronous Contract)**: Focuses on asynchronous decoupled event publish/subscribe and request-reply patterns. Metadata includes `brokerType` (e.g., Kafka/NATS/RabbitMQ), `action` (values: `publish` / `subscribe` / `request` / `reply`), `topic`, `payloadSchemaHash`. **Specification planned**.
3.  **`SchemaAST` (Storage & Data Contract)**: Focuses on underlying database tables, object storage buckets, and cache dependencies, used to discover the hidden and fatal "database implicit coupling" in microservices. Metadata includes `storageType` (e.g., PostgreSQL/MongoDB/S3/Redis), `database` (database name/bucket name), `table` (table name/key matching pattern), `operation` (values: `read` / `write` / `join`). **Specification planned**.
4.  **`TypeAST` (Code & Library Contract)**: Focuses on compile-time code reuse and strongly bound interface dependencies. Metadata includes `libraryName` (shared package name), `typeIdentifier` (data DTO or common algorithm formula struct), `interfaceDefinitionFile` (e.g., `.proto` / gRPC Stubs path), `versionConstraint`. **Specification planned**.

#### 3.1 Multi-Repository Ecosystem Topology Generation Process (Static Confluence & Generation Sync Mechanism)
*   **Single Repository (CI Phase)**:
    A single repository executes `lunar scan` during the build phase, extracts the project's `<subdomain>-actual.json` and uploads it to S3/R2. The `<subdomain>` naming convention is defined by each sub-protocol; for RouteAST, it is `route-ast` (producing the file `route-ast-actual.json`).
    *   **Pointer Update Mechanism**: After successfully uploading `<subdomain>-actual.json`, the single repository CI **must immediately update and upload its corresponding repository's pointer file `pointers/latest.json`**, making its `sha` point to the Commit SHA of this submission, thus ensuring the ecosystem orchestration tool can capture the latest available physical version in real-time.
*   **Generation Build Triggering and Atomicity Guarantee (Generation Sync)**:
    To guarantee that a single ecosystem-level build corresponds to a deterministic, atomic, and legitimate global static snapshot during multi-service asynchronous CI builds, the system adopts the **"Ecosystem Build Plan Lockfile"** synchronization mechanism:
    1.  **Plan Generation**: The ecosystem administrator or ecosystem-level automated release tool periodically or based on Commit events polls each project's `latest.json` pointer, generates a target SHA static snapshot file **`ecosystem-plan.json`**, and simultaneously generates a unique `generationId` based on the timestamp and content hash at the time of plan file creation (format: `<timestamp>-<uuid>`).
    2.  **Plan Publishing**: This plan file is uploaded to `ecosystem-config/generations/<generationId>/ecosystem-plan.json`.
    3.  **Central Pipeline Activation**: The upload event of this plan file directly serves as a static signal, unidirectionally triggering the centralized alignment build pipeline, eliminating the "chicken-and-egg" deadlock.
*   **Core Alignment and Verification Boundaries**:
    1.  **Input Compliance Verification (Set Equality)**: **Upon startup, the central pipeline forcibly verifies that the set of all projects defined in `ecosystem-plan.json` must be exactly equal (Set Equality) to the set of projects declared in the ecosystem registration manifest `repos.json`.** If the plan lacks any registered project or contains unregistered projects, the central pipeline must refuse execution, throw an error, and exit, forcing regeneration of a complete lockfile, ensuring every generation snapshot is a complete projection of the entire ecosystem, preventing large-scale false alignment anomalies caused by missing partial snapshots.
    2.  **Build Idempotency Guarantee**: The pipeline first checks if `ecosystem-config/generations/<generationId>/lunar-map.json` already exists. If it exists, the generation alignment is deemed statically ready, and computation is skipped.
    3.  **Generation Fetch and Isolation Determination**: The pipeline fetches all projects' `actual.json` caches in parallel:
        *   If target `actual.json` for all declared projects is ready: the central pipeline aligns the data and generates the global `lunar-map.json`.
        *   **`failed` Status Determination**: For projects that fail to upload `actual.json` within a 5-minute timeout window, mark their `scanStatus: failed` in `lunar-map.json` and forcibly set their `interfaces` field to `null`.
        *   **`stale` Status and Data Adoption Determination**: When a project's `actual.json` cache has been purged by lifecycle, or the difference between its `lastUpdated` timestamp and the current generation compilation time exceeds a preset validity period (default 7 days, controlled by `LUNAR_MAX_STALE_AGE_SECONDS`): **The central pipeline will still adopt its historical data for alignment comparison. However, its `scanStatus` is marked as `stale`, and the final status of all alignment entries generated by this project are forcibly marked as `Unverified`.**
    4.  **Ecosystem-Level Top-Level Pointer Update**: After successfully uploading the alignment results `lunar-map.json` and `meta.json`, the central pipeline **must synchronously and atomically update the ecosystem latest generation pointer located at `ecosystem-config/latest-generation.json`**, pointing its `generationId` and `lastUpdated` to the latest completed generation, enabling downstream clients to achieve stateless dynamic version discovery.
*   **Reverse Derivation Mechanism and Null Value Defense**:
    If the `<subdomain>-actual.json` cache is physically missing from the storage bucket due to network issues or lifecycle expiration, the gateway will forcibly perform data reverse derivation from `projects[].interfaces` in `lunar-map.json`.
    *   **Null Value Exception Defense**: If the corresponding project's `interfaces` has been set to `null` during the build phase due to failure isolation, the gateway determines that the reverse derivation data source for this project is physically unavailable. The gateway immediately interrupts distribution, returns `410 Gone` to the client with an `X-Lunar-Recovery` header guiding rebuild, and carries the `ERR_LUNAR_INTERFACE_DATA_MISSING` error code in the response body.
    *   **Honest Disclosure on Build Failure Isolation and Actual File Existence Inconsistency**: After the central pipeline marks a timed-out, not-ready project as `scanStatus: failed` and `interfaces: null`, that project's CI container might complete the individual upload of `actual.json` at a later moment. At this point, the file physically exists in the storage bucket, but the global topology considers it unavailable due to build isolation. When a client bypasses the topology and directly accesses the project's interface via `GET /commits/<sha>/route-ast-actual.json`, the gateway will return success. Such inconsistency is caused by the failure isolation boundary of distributed compilation; developers must use `lunar doctor` for global state consistency diagnosis.

---

### 4. Data Generation Layer and Compilation Pipeline (lunar)

#### 4.1 Phase One: Detect & Extract — Process Isolation, Adapter Path Override, and Atomicity Guarantee
*   **Adapter Discovery and Override Mechanism**:
    The `lunar` control layer defaults to dynamically searching for executables matching the name `lunar-extract-<lang>` (e.g., `lunar-extract-rust`, `lunar-extract-node`) via the system environment variable `PATH`. Users can specify absolute paths for `adapters` in `.lunar/config.yml` for override, which takes priority over `PATH` auto-discovery.
*   **Line-Delimited JSON Communication and Atomic End Marker**:
    The Orchestrator spawns adapter subprocesses. To defend against heap overflows in large projects, adapters must use streaming output (Line-by-line Flushing). Upon extracting each route, the adapter must immediately write it to `stdout` and execute `flush`, strictly forbidden from accumulating the entire dataset in memory before full serialization.
    *   **Output Stream Atomicity and Count Verification (End-of-Stream Marker)**: To prevent the control layer from receiving an incomplete, corrupted interface list due to an adapter mid-execution crash, **the adapter must output a special end marker line after successfully extracting all routes**:
        `{"_lunar": {"status": "success", "count": 42}}`
        **Upon receiving this marker line, the Orchestrator must rigorously verify that the actual number of parsed routes equals `count`. If they do not match (automatically defending against data phase truncation), the Orchestrator discards all lines and directly triggers `ERR_LUNAR_ADAPTER_CRASH`.** All debug and warning logs must be redirected to `stderr`, strictly forbidden from polluting `stdout`.
*   **Adapter Error Isolation Strategy**:
    If a specific framework's adapter crashes and returns a non-zero exit code, the Orchestrator must gracefully capture it, isolate and skip that project's scan, log a warning, and continue execution without ever causing a main CI process interruption.

#### 4.2 Phase Two: Confirm & Semantic Normalization — Constraint Passing Principle
*   **Semantic Normalization**: Forcefully translate and normalize framework-specific regex and constraint expressions output by different adapters into the standard Rust memory model `RouteAST`.
*   **Constraint Preservation Compromise and Limitation**:
    Performing mathematical equivalence proofs on various irregular regex expressions across languages in the static dimension is not realistically feasible in engineering. At the v0.5.0 stage, the confirmation engine **does not forcibly translate constraint regexes**. `rawConstraint` is preserved as-is and passed as descriptive metadata; the matching algorithm only performs positional and basic type comparison during the alignment phase. **The design limitation is that the system cannot automatically detect and warn of runtime 400 verification mismatch risks caused by asymmetric regex matching sets (non-overlapping matching intervals at both ends) across multi-language frameworks.**

#### 4.3 Zero-Privilege Synchronization Mechanism and Guided Sync
To guarantee developers' 100% control over code, the `lunar` command-line tool refuses any behavior that silently modifies human-maintained files in the background.
*   **`lunar init`**: **Only when detecting that local `.lunar/interfaces.yml` does not exist**, automatically runs physical scanning and creates a draft file. If the file already exists, this command is directly ineffective and does not overwrite any human traces.
*   **`lunar diff`**: Executes comparison between physical facts (AST) and the intent overlay (`interfaces.yml`), outputting a standard Git-diff style change report to the console.
*   **`lunar sync --apply`**: A user-triggered sync merge command, supporting `--dry-run` to preview changes. Before executing the actual merge write, forcibly backs up the old `interfaces.yml` to the local hidden backup directory `.lunar/.backup/interfaces.yml.bak` (this path is automatically written to the project's `.gitignore` by `lunar init`).

---

### 5. Project-Level Intent Overlay and Ecosystem Configuration Specifications (YAML/JSON Schemas)

To ensure uniformity in consumption across multi-language teams, all configuration fields are forcibly required to follow the Google JSON Style Guide standard camelCase format.

#### 5.1 `.lunar/interfaces.yml` Specification
The project-level centralized contract is the first gate for developers to control and define interface boundaries:
```yaml
project: myPaymentService
type: mixed # service / client / mixed
environment: production

exposed:
  - path: /api/v1/payments/refund
    method: POST
    reason: "Refund dedicated interface (planned for release next week)"

consumed:
  - path: /api/v1/auth/verify
    method: POST
    targetProject: authService
    reason: "User transaction pre-session verification"
```

#### 5.2 Ecosystem Registration Manifest: `repos.json`
```json
{
  "version": "0.5.0",
  "comment": "The version field defines the schema compatibility version of this registry configuration itself.",
  "projects": ["myPaymentService", "authService", "billingService"]
}
```

#### 5.3 Ecosystem Centralized Topology Declaration: `ecosystem-topology.json`
```json
{
  "ecosystem": "lunarEcosystem",
  "version": "0.5.0",
  "projects": {
    "myPaymentService": { "layer": "businessOrchestration", "criticality": "high" }
  },
  "relationships": [
    { "from": "myPaymentService", "to": "authService", "type": "rpcSync", "reason": "Session Authentication" }
  ]
}
```

#### 5.4 Ecosystem Build Plan Lockfile: `ecosystem-plan.json`
This file is the decision starting point for the entire ecosystem generation build. It is entirely managed and written by ecosystem-level central automation tools, and hard-locked each time a `generationId` is produced.
```json
{
  "$schema": "https://routeast.dev/schema/v1/ecosystem-plan.schema.json",
  "generationId": "20260608T030000Z-a1b2c3d4",
  "created": "2026-06-08T03:00:00Z",
  "projects": {
    "myPaymentService": { "sha": "abc123e456f789..." },
    "authService": { "sha": "def456a789b123..." }
  }
}
```

---

### 6. Data Exchange Standard Format Specification (The Exchange Contract Spec)

As the physical product output to the entire ecosystem, data must follow a strict static format definition.

#### 6.1 Structured Topology Standard: `lunar-map.json`

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
            "anyOf": [
              {
                "type": "object",
                "required": ["exposed", "consumed"],
                "properties": {
                  "exposed": {
                    "type": "array",
                    "items": {
                      "type": "object",
                      "required": ["path", "method"],
                      "properties": { "path": { "type": "string" }, "method": { "type": "string" } }
                    }
                  },
                  "consumed": {
                    "type": "array",
                    "items": {
                      "type": "object",
                      "required": ["path", "method", "targetProject"],
                      "properties": { "path": { "type": "string" }, "method": { "type": "string" }, "targetProject": { "type": "string" } }
                    }
                  }
                }
              },
              { "type": "null" }
            ]
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
          "status": { "type": "string", "enum": ["Aligned", "ParamNameMismatch", "Unused", "Orphaned", "MethodMismatch", "Unverified"] }
        }
      }
    }
  }
}
```

#### 6.2 AI Context Map Standard: `lunar-map.md` (Markdown Spec)
Rendered dynamically by `lunar-gateway` from `lunar-map.json` in real-time upon request. The gateway performs internal rendering caching for this dynamic endpoint (Cache Key: `<bucket>/cache/<generationId>/md/<style>/<scope>`), supporting parameterized filtering:
*   `GET /lunar-map.md?style=list` (Returns plain text contract list, suitable for small context extraction)
*   `GET /lunar-map.md?style=mermaid` (Returns topology diagram structure, suitable for global chart rendering)
*   `GET /lunar-map.md?scope=project-a` (Sharding fetch, limiting to sub-topology related to a specific project, preventing exceeding AI context window limits)

---

### 7. Open Integration and Third-Party Bridge Isolation Specification

Any external execution sandbox (such as IDE plugins, AI Agents) must follow the **"Bridge Isolation Pattern"** during integration:
*   **Bridge Responsibility**: Third-party integrators must write independent bridge programs (e.g., `routeast-mcp-bridge`), fetch `lunar-map.json` via standard HTTP `GET` interface, and convert it internally into a specific protocol.
*   **Non-Privileged Interaction Specification**: When writing AI-generated alignment suggestions locally, the bridge **is strictly forbidden from silently modifying `interfaces.yml`**. The bridge is only responsible for generating and outputting Git-diff format alignment patch snippets in the terminal, prompting the user to manually run local `lunar sync --apply`.

---

### 8. Stateless Distribution Layer Design and Security Model (`lunar-gateway`)

`lunar-gateway` focuses on high-concurrency, stateless, ultra-fast contract data secure distribution.

#### 8.1 Storage Directory Structure Standard
```
s3://lunar-ast-<organization>/
├── <repo>/
│   ├── commits/
│   │   └── <sha>/
│   │       └── <subdomain>-actual.json
│   └── pointers/
│       └── latest.json
└── ecosystem-config/
    ├── repos.json
    ├── ecosystem-topology.json
    ├── latest-generation.json
    └── generations/
        └── <generationId>/
            ├── meta.json
            ├── ecosystem-plan.json
            └── lunar-map.json
```

#### 8.1.1 Metadata File: `meta.json`
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
          "items": { "type": "string" }
        }
      }
    }
  }
}
```
*   **Property Note**: `compilerVersion` refers to the binary version of the `lunar` core compiler that performs static analysis scanning and compiles output data; it is not bound to the gateway's version.

#### 8.1.2 Generation Global Pointer: `latest-generation.json`
```json
{
  "generationId": "20260608T030000Z-a1b2c3d4",
  "lastUpdated": "2026-06-08T03:00:00Z"
}
```

#### 8.2 Dual-Phase Isolated Cache and Client Tiered Cache Control
*   **Internal Cache**: Uses the complete logical path of the file relative to the Bucket in object storage as the Cache Key.
*   **Security Verification First Principle**: For private resources requiring JWT signatures, **the gateway must unconditionally complete token integrity verification first. Only after authentication is fully passed can the internal cache be matched and read**.
*   **Client Response Differentiation**:
    *   Private resources: `private, no-cache, no-store, must-revalidate`
    *   Public immutable resources (under `/commits/<sha>/` and global topology under `generations/`): `public, max-age=31536000, immutable`
    *   Latest dynamic pointers and latest generation pointers: `public, max-age=300`
*   **Memory Protection Circuit Breaker**: Based on the typical 128MB physical memory constraint safety boundary of Cloudflare Workers, large files (> 2MB) automatically trigger buffer circuit breaking, degrading to stream-through forwarding.

#### 8.3 Security Verification and JWT Signature Contract
*   **Signature Algorithm**: Mandatory use of **Ed25519 (EdDSA)** as the sole signature algorithm, guaranteeing constant-time verification.
*   **Expiration Policy**: The `exp` claim in the JWT is recommended not to exceed 24 hours.
*   **Public Key Cache Rotation**: `MAX_KEYS_PER_REPO = 3` (each physical service/project retains at most the latest and 2 valid historical public key fingerprints), preventing malicious registration from exhausting gateway memory.

#### 8.4 Version Negotiation and Backward Compatibility Strategy
The client carries `Accept: application/vnd.lunar.0.5.0+json`. **The gateway must maintain a data downgrade compatibility window for the currently active major version and the immediately preceding major version.** Only when a predefined, lossless mapping table exists between the new and old versions does the gateway perform automatic downgrade; otherwise, it returns `406 Not Acceptable` and attaches an `X-Lunar-Upgrade-Required: true` header.

---

### 9. Alignment Status Priority and Diagnostic Classification (Diagnostic Short-Circuit)

The alignment engine performs one-way short-circuit evaluation on the input server and client `RouteAST` lists:

$$\text{Unverified} > \text{MethodMismatch} > \text{Orphaned} > \text{Unused} > \text{ParamNameMismatch} > \text{Aligned}$$

*   **Failure Isolation and Stale Node Alignment Mechanism (Unverified Status Isolation)**:
    To prevent data gaps caused by some projects in the ecosystem failing to scan (`scanStatus: failed`) or having stale data due to uncoordinated CI builds (`scanStatus: stale`):
    1.  **Alignment Isolation Extraction**: The alignment engine must retrieve the `scanStatus` from the `projects` array before computation.
    2.  **`stale` Node Handling**: The `exposed` and `consumed` data contained in projects with `stale` status **will still be read and participate in the comparison computation by the alignment engine**. However, all final alignment results produced are **forcibly marked with `status: "Unverified"`**.
    3.  **`failed` Node Handling**: Since `failed` projects cannot fetch valid `interfaces` definitions (they are `null`), no actual comparison can be performed. **The alignment engine must traverse all other healthy projects in the current generation topology. Whenever it detects that a healthy project has initiated interface consumption (`consumed`) targeting this `failed` service, the alignment engine does not perform conventional comparison checks but directly generates an alignment entry with `status: "Unverified"` for it.** Because the `failed` project has no valid interface data (its `interfaces` field is `null`), its own exposed interfaces will not generate any alignment entries in the `alignments` array, and thus will not be erroneously determined as `Unused`.

---

### 10. Observability and Health Check Specification

#### 10.1 Structured Logging Specification (JSON Lines)
The gateway outputs single-line JSON structured logs to `stdout`. `authStatus` state machine: `valid`, `expired`, `invalidSignature`, `missingToken`.

#### 10.2 Metrics Exposure (Prometheus Metrics)
*   `lunar_gateway_requests_total`
*   `lunar_gateway_cache_hits_total`
*   `lunar_gateway_auth_failures_total`
*   `lunar_gateway_version_downgrade_requests_total`
(When exposing to Prometheus, label names like `targetVersion` are automatically converted by the implementation layer to the underscore-delimited naming format `snake_case` conforming to Prometheus best practices, e.g., `target_version`)

#### 10.3 Liveness Health Check
The gateway exposes a fast read-only probe endpoint without signature verification: `GET /healthz`, returning `200 OK` directly upon success, for container and edge runtime liveness detection.

---

### 11. Security Threat Model and Defense Strategy

1.  **Injection Attack Prevention**: Use specific lexical rule regex combined with AST node type joint determination for the target language's physical comment syntax. Any malformed comments not matching preset rules or with extremely non-compliant formats must output the store path, a Warning, and the line number to `stderr`, and be forcibly ignored, not passed to the backend AST tree.
2.  **Key Rotation Overflow Defense**: `MAX_KEYS_PER_REPO = 3`.
3.  **S3 Credential Minimization**: Only `s3:PutObject` and `s3:GetObject`.
4.  **Destructive Action Mistouch Prevention Barrier**: All destructive actions involving the local CLI (like `lunar cleanup`) must undergo **secondary interactive blocking confirmation** in the terminal. Only when the non-interactive override flag (like `--yes`) is explicitly appended in the CLI is the interactive prompt allowed to be skipped.

---

### 12. Core Error Codes and Debugging Guide

| Error Code | HTTP Status Code | Physical Cause | Recommended Recovery Plan |
|:---|:---|:---|:---|
| `ERR_LUNAR_CONFIRM_FAIL` | 400 | Normalization verification failed. | Run `lunar diff`. |
| `ERR_LUNAR_ADAPTER_CRASH` | 422 | Subprocess extractor crashed or count mismatch. | Check CI log stderr. |
| `ERR_LUNAR_PROJECT_NOT_FOUND` | 404 | Target project not defined in `repos.json`. | Add to `repos.json`. |
| `ERR_LUNAR_INTERFACE_NOT_FOUND` | 422 | Target project exists but does not expose a method or path matching the client consumption. | Run `lunar diff`. |
| `ERR_LUNAR_INTERFACE_DATA_MISSING` | 410 | Due to project scan failure or not ready during alignment build phase (scanStatus: failed), and `interfaces` is `null`, the gateway completely lacks a reverse derivation data source when attempting to provide it for the consumer, unable to downgrade recovery. | Run `lunar doctor`, troubleshoot the failed heterogeneous sub-service. |
| `ERR_VERSION_EXPIRED` | 410 | Physical fact cache purged after exceeding 90 days. | Response header carries `X-Lunar-Recovery` header (and optional `X-Lunar-Recovery-Webhook`), guiding and encouraging developers to re-trigger CI build in the corresponding repository. |
| `ERR_GATEWAY_STREAM_FALLBACK_FAILED` | 502 | Downgraded stream-through fallback to origin failed. | Check `lunar-map.json` and origin connectivity. |

---

### 13. Roadmap and Core Evolution Milestones

*   **Milestone 1**: Freeze RouteAST Base IR, implement Rust Confirm Core prototype, provide Rust/Node adapters.
*   **Milestone 2**: Release `lunar` command-line tool, deploy stateless edge gateway `lunar-gateway`, establish security baseline.
*   **Milestone 3**: Release `lunar-scope` intelligent physical magnetic canvas, support `Unused` and `Unverified` interface status visualization.
*   **Milestone 4**: Release TypeAST contract standard, complete full-stack static interface alignment and drift defense.

---

## References and Specification Sources

*   RFC 6570 - URI Template Specification
*   IEEE Std 1471-2000 - Architectural description of software-intensive systems
*   JSON Lines Standard (v1.0)
*   WASI 0.2 (Component Model)
*   ACM TOSEM Vol. 33 - Cross-Language Static Program Analysis
*   NIST FIPS 186-5 - Digital Signature Standard

---

*"Contract supremacy, not a fraction off. Let multi-language routing dialects converge here, achieving zero-intrusion, deterministic network alignment."*
