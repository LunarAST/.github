# LunarAST & lunar-gateway Protocol and System Architecture Specification

**— A Multi-Layer Heterogeneous Static Contract Standard and the “Generate, Distribute, Specify, Visualize” Four‑Layer Architecture Specification**

**Version**: 1.6 — Definitive Specification and Refactored Upgrade (Fully Closed Loop)  
**Last Updated**: 2026-06-13  

---

## 1. Physical Role Definition

This specification defines the physical boundaries and naming contracts of every component in the LunarAST ecosystem [5]:

| Component / Name | Physical Layer | Core Responsibility |
| :--- | :--- | :--- |
| **LunarAST** | **Specification Layer** | Defines the Base IR specifications of multi‑layer contracts (RouteAST, EventAST, etc.), extraction protocols, and mathematical alignment semantics [5]. It is a static schema specification that contains no executable code. |
| **`lunar`** | **Data Generation Layer** | Command‑line binary executed locally and in CI pipelines. Responsible for `lunar init` (initialization), `lunar scan` (physical extraction), `lunar diff` (non‑overprivileged comparison) and `lunar sync --apply` (proactive backup and safe synchronisation) [2]. All alignment decisions and merge logic reside here. |
| **`lunar-serve`** | **Local Read‑only Distribution Layer** | A very lightweight, read‑only HTTP distribution binary running locally. Depends on `lunar-interface`, provides local high‑fidelity rendering of `lunar-map.json` during development, and supports a fallback mechanism to securely, on‑demand, and archivelessly serve the source code mirror of the local physical workspace directly to AI agents with zero manual configuration. |
| **`lunar-gateway`** | **Stateless Distribution Layer** | An independently deployed serverless edge gateway program. Compiled to `wasm32-wasip2` [4]; enforces one‑way authentication based on security tokens (Ed25519‑JWT) and high‑concurrency two‑stage isolated cache distribution [2]. The gateway does **not** execute any real‑time interface alignment logic. |
| **`lunar-scope`** | **Visualisation Layer** | A purely static, multi‑layer frontend canvas. Pulls the ready‑to‑use topology JSON (already aligned at build time) from the gateway via standard APIs and renders a human‑machine interface with magnetic predictive dotted lines, breakpoint highlighting, and architecture drift warnings. |

### 1.1 Metaphor and Physical Correspondence
*   **Data reflection (LunarAST Standard Layer)**: The system does **not** generate runtime data (it does not emit light) and does **not** participate in any runtime monitoring or performance overhead. During the build phase it receives physical facts from source code changes (the landscape illuminated by developer commits and compilation actions) and statically projects them.
*   **Ecosystem visualisation (lunar-scope Visualisation Layer)**: As a purely static, multi‑layer relational canvas, it provides architects with low‑latency, multi‑perspective observation of contract data, catching dangling breakpoints and unplanned wild dependencies before code deployment [2].

---

## 2. Three‑Tier Source‑of‑Truth Architecture

```
      ┌────────────────────────────────────────────────────────┐
      │   Tier 1: Physical Facts (AST)                         │  ← 80‑90% automatic, written to .interfaces-autogen.json
      ├────────────────────────────────────────────────────────┤
      │   Tier 2: Intent & Override Overlay                    │  ← human‑controlled, .lunar/interfaces.yml (primary gate)
      ├────────────────────────────────────────────────────────┤
      │   Tier 3: Escape Hatch (Comments)                      │  ← only for extremely complex dynamic RPC edge cases
      └────────────────────────────────────────────────────────┘
```

### 2.1 Tier 1 – Physical Facts (AST) — Automatic Derivation
*   **Physical file**: **`.lunar/.interfaces-autogen.json`** (this file **must** be added to the project’s `.gitignore`).
*   **Lifecycle**: Because this file is not stored in Git, during CI/CD or local builds the build machine automatically runs `lunar scan` locally to rebuild this physical fact cache, never relying on historical cache.

### 2.2 Tier 2 – Intent & Override Overlay — Field‑level Partial Override and Merge Contract
*   **Physical file**: **`.lunar/interfaces.yml`** at the project root.
*   **Properties**: **100% human‑controlled, under version control (Git), and tools are strictly forbidden from any unauthorised silent modification.**
*   **Merge formula ($\oplus$) – field‑level partial override definition**:  
    For an interface object identified by the composite primary key `(Path, Method)`, the alignment engine executes at compile time the **“Partial Field Override”** rule:
    *   Let $A$ be an interface object from the physical facts (Actual AST), and $I$ be the same‑named interface definition from the intent overlay.
    *   For any attribute field $f$ of the interface object (e.g. `port`, `rawConstraint`):
        $$\text{Resolved}.f = \begin{cases} 
          I.f, & \text{if } I.f \text{ is specified} \\
          A.f, & \text{otherwise} 
        \end{cases}$$
    *   This rule ensures that when a human overrides specific network parameters or path attributes, **other silent metadata from the physical facts** (e.g. source file location `sourceFile`, `lineNumber`) are **preserved as‑is** [2].
    *   **Override boundary restrictions**: For array types (e.g. `segments`) and nested objects, the overlay supports **only Complete Replacement**; partial field merges are not allowed. If both the overlay and the physical facts are non‑empty for such a field and their lengths or sub‑keys differ, `lunar diff` **must** warn and **reject** automatic merging (even in non‑strict mode), forcing the human to manually rewrite the whole array.
    *   **Conflict detection**: If the overlay changes a core property that already exists in the physical facts, `lunar diff` **must** output a highlighted warning to the terminal for human review, but the toolchain does **not** block compilation in non‑strict mode.

### 2.3 Tier 3 – Escape Hatch
*   **Definition**: A single‑line magic comment starting with `// lunar:` or `# lunar:` placed immediately above a source code line [2].
*   **Applicability boundary**: **Only for dynamically constructed RPC/HTTP calls where the target path and method cannot be captured by static AST analysis because of dynamic evaluation. For standard routes, the adapter MUST forcibly extract physical facts and MUST NOT allow duplicate declarations via comments.**
*   **Syntax example**:
    ```typescript
    // lunar:consume POST https://api.auth-service/v1/token
    await axios.post(dynamicUrl, data);
    ```

---

## 3. Four‑Layer Decoupled Physical Blueprint and Topology Generation

To keep the system core absolutely robust, `LunarAST` partitions the complex distributed dependency relationships into four mutually independent, physically isolated sub‑domain contracts, completely preventing protocol bloat [5]:

1.  **`RouteAST` (Routing and Network Contract)**: Focuses on synchronising network interfaces (REST/gRPC/Nginx). Metadata includes `method`, `segments`, `port`. **Currently at v0.5.0 draft design stage.**
2.  **`EventAST` (Event and Asynchronous Contract)**: Focuses on asynchronous decoupled event publish/subscribe and request‑reply patterns. Metadata includes `brokerType` (e.g. Kafka/NATS/RabbitMQ), `action` (values `publish` / `subscribe` / `request` / `reply`), `topic`, `payloadSchemaHash`. **Specification in planning.**
3.  **`SchemaAST` (Storage and Data Contract)**: Focuses on low‑level database tables, object storage buckets, and cache dependencies, used to detect hidden and dangerous “implicit database coupling” in microservices. Metadata includes `storageType` (e.g. PostgreSQL/MongoDB/S3/Redis), `database` (database/bucket name), `table` (table/key pattern), `operation` (values `read` / `write` / `join`). **Specification in planning.**
4.  **`TypeAST` (Code and Library Contract)**: Focuses on compile‑time code reuse and strong interface binding dependencies. Metadata includes `libraryName` (shared package name), `typeIdentifier` (DTO or common algorithm structure), `interfaceDefinitionFile` (e.g. `.proto` / gRPC stubs path), `versionConstraint`. **Specification in planning.**

### 3.1 Multi‑Repository Topology Generation Flow (Static Merge and Generation Synchronisation)
*   **Single repository (CI phase)**:  
    During the build phase, the repository executes `lunar scan` to extract its `<subdomain>-actual.json` (e.g. `route-ast-actual.json`) and uploads it to S3/R2 [2].  
    *   **Pointer update mechanism**: After successfully uploading the `<subdomain>-actual.json`, the CI **MUST immediately update and upload its corresponding pointer file `pointers/latest.json`**, making its `sha` point to the commit SHA of this build, so that ecosystem orchestration tools can always capture the latest available physical version.
*   **Generation build trigger and atomicity (Generation Sync)**:  
    To guarantee that each ecosystem‑level build corresponds to a deterministic, atomic, and valid global static snapshot even when multiple services build asynchronously, the system uses an **Ecosystem Lockfile** synchronisation mechanism:
    1.  **Plan generation**: An ecosystem administrator or automated ecosystem release tool periodically polls each project’s `latest.json` pointer (or listens to commit events) and generates a static snapshot file **`ecosystem-plan.json`** with target SHAs. A unique `generationId` is assigned (strictly generated by the ecosystem release tool when the plan file is created, based on a timestamp and content hash, format: `<timestamp>-<uuid>`).
    2.  **Plan publication**: The plan file is uploaded to `ecosystem-config/generations/<generationId>/ecosystem-plan.json`.
    3.  **Central pipeline activation**: The upload event of the plan file directly acts as a static signal that unidirectionally triggers the centralised alignment build pipeline, eliminating the “chicken‑and‑egg” deadlock [2].
*   **Core alignment and validation boundaries**:
    1.  **Input compliance check (Set Equality)**: **The central pipeline MUST enforce at startup that the set of all projects defined in `ecosystem-plan.json` is exactly equal to the set of projects declared in the ecosystem registry `repos.json`.** If the plan misses any registered project or contains an unregistered project, the pipeline MUST refuse execution, throw an error, and exit, forcing a complete lock file to be regenerated. This ensures every generation snapshot is a full projection of the whole ecosystem, preventing large‑scale false alignment anomalies caused by partial snapshots.
    2.  **Idempotency guarantee**: The pipeline first checks whether `ecosystem-config/generations/<generationId>/lunar-map.json` already exists. If it does, it considers that generation statically ready and skips computation.
    3.  **Generation pull and isolation decision**: The pipeline pulls the `actual.json` caches of all projects in parallel:
        *   If all declared projects have their target `actual.json` ready: the pipeline aligns the data and generates the global `lunar-map.json`.
        *   **`failed` state decision**: For projects that have not completed uploading their `actual.json` within a 5‑minute timeout window, the pipeline marks their `scanStatus: failed` in `lunar-map.json` and forcibly sets their `interfaces` field to `null` [2].
        *   **`stale` state and data adoption decision**: If a project still has a historical `actual.json` in object storage, but its `lastUpdated` timestamp difference from the current generation compilation time exceeds the predefined validity period (default 7 days, configurable via `LUNAR_MAX_STALE_AGE_SECONDS`): **the pipeline still adopts the historical data for alignment computation, but marks its `scanStatus` as `stale`, and forces the final `status` of every alignment entry involving that project to `unverified` [2].**
    4.  **Ecosystem‑level top pointer update**: After successfully uploading the alignment result `lunar-map.json` together with `meta.json`, the central pipeline **MUST synchronously and atomically update the `ecosystem-config/latest-generation.json` pointer** to point its `generationId` and `lastUpdated` to the latest completed generation, enabling downstream clients to do stateless dynamic version discovery.
*   **Reverse derivation mechanism and null‑value defence**:  
    If an `<subdomain>-actual.json` cache is physically missing from object storage (due to network issues or lifecycle expiry), the gateway **MUST** forcibly derive the data from `projects[].interfaces` inside `lunar-map.json`.  
    *   **Null‑value exception defence**: If the corresponding project’s `interfaces` was already set to `null` at build time because of failure isolation, the gateway determines that the reverse‑derivation data source is physically unavailable. It immediately interrupts distribution, returns `410 Gone` to the client with an `X-Lunar-Recovery` header guiding a rebuild, and includes the `ERR_LUNAR_INTERFACE_DATA_MISSING` error code in the response body.  
    *   **Honest disclosure of inconsistency between build failure isolation and actual file existence**: After the central pipeline marks a timeout‑failed project as `scanStatus: failed` and `interfaces: null`, that project’s CI container may later finish uploading its `actual.json`. The file then physically exists in the bucket, but the global topology treats it as unavailable due to build isolation. When a client bypasses the topology and directly accesses that project’s interface via `GET /commits/<sha>/route-ast-actual.json`, the gateway will return success. Such inconsistency is caused by the isolation boundary of distributed builds; developers **must** use `lunar doctor` to perform global consistency diagnosis.

### 3.2 Zero‑Friction Source Code Projection and Fallback Path Resolution Specification

To provide AI agents with an archiveless, zero‑friction, on‑demand source code mirroring capability **in addition to** the contract topology, the system MUST support “URL projection” and “path fallback” mechanisms:

1.  **Route aliasing equivalence**:  
    When providing GitHub‑style mirror access, the gateway and local read‑only service MUST treat `/blob/` (web file view) and `/raw/` (raw text read) paths as completely equivalent at the routing level. An AI can simply replace `github.com` with the ecosystem distribution domain to obtain the physical file content, ignoring the URL syntax difference [1.2].

2.  **Absolute path two‑level fallback priority**:  
    When resolving the physical workspace path of a project, the server MUST enforce the following fallback chain and reject any global hard‑coding:
    $$\text{ResolvedPath} = \begin{cases} 
      \text{Registry.path}, & \text{if specified in repos.json} \\
      \text{Topology.path}, & \text{else if automatically discovered in lunar-map.json} \\
      \text{Error (400)}, & \text{otherwise} 
    \end{cases}$$

3.  **Case‑insensitive normalisation**:  
    When matching GitHub network coordinates `{owner}/{repo}/{branch}`, the gateway and service layer MUST force lower‑case normalisation hashing in memory, completely eliminating distribution gaps caused by cross‑platform case‑sensitivity differences [1.2].

### 3.3 Decentralised AI Task Handover Scratchpad and Cryptographic Anti‑Forgery Specification

To allow external AI agents to autonomously and incrementally help humans improve the `interfaces.yml` contract in a stateless, zero‑trust public distribution environment, the system establishes the **Decentralised AI Task Handover Scratchpad** specification:

1.  **State fact carrier (`ai-todo.json`)**:  
    The current AI development progress and pending merge patches for a project are persistently stored in the local hidden directory `.lunar/ai-todo.json` under the project root. This file only retains active tasks with `pending` status. Once a task is merged, it is physically erased and pruned at the moment of merge, ensuring the file stays extremely small to save AI transmission tokens.

2.  **Ed25519 task fingerprint anti‑forgery (Cryptographic Verification)**:  
    When an external AI agent submits scratchpad suggestions or YAML contract patches via `POST /api/v1/projects/:name/todo`, it MUST enforce the use of its own AI private key to compute an **Ed25519 cryptographic digital signature** over the `patch` payload.  
    The local `lunar` client, when executing the one‑click `lunar pull`, MUST load the corresponding project’s public key locally and mathematically verify the signature.  
    *   **Security defence boundary**: Any scratchpad submission without a signature, with an expired signature, or with a fingerprint that does not match the registered key **MUST** be rejected by the local CLI with a hard fuse, completely blocking malicious alignment injection and backdoor poisoning through public endpoints.

3.  **Crystallised milestone rendering**:  
    When rendering the Markdown data for the `/tree` route, `lunar-serve` applies the **“micro‑pruning, macro‑crystallisation”** rule.  
    *   For completed milestones, the gateway does **not** expand any completed micro‑tasks; it renders them as a single‑line, checked achievement badge. This provides the AI with 100% macro‑landmark context (Directional Context) while keeping 0% micro‑token noise.

---

## 4. Data Generation Layer and Build Pipeline (`lunar`)

### 4.1 Phase 1: Detect — Base Multi‑Language Sniffer and Fallback Fact Generation
*   **Base Multi‑Language Sniffer and Fallback Fact Generation**:  
    When `lunar scan` starts and the project root does **not** contain a `Cargo.toml` (i.e., it is a non‑Rust project), the controller performs lexical sniffing and checks for the existence of:
    *   `requirements.txt` / `pyproject.toml` / `Pipfile` → language = `Python`
    *   `go.mod` → language = `Go`
    *   `package.json` → language = `Node.js`
    *   `nginx.conf` → language = `Nginx`
    If the compile‑time AST extractor for the detected language (e.g. `lunar-extract-python`) is **not** installed in the system `PATH`, the controller does **not** abort with an error. Instead, it automatically activates a **Declarative Fallback Fact Generator**, which writes a valid empty physical fact file `.interfaces-autogen.json` into the local `.lunar/` directory, and sets the `projectType` to the detected language.
    *   **Architectural benefit**: This design eliminates lifecycle interruptions caused by missing lexical extractors. It allows heterogeneous multi‑language projects (Python, Go, Node.js, Nginx) to join the ecosystem without friction. Through the `interfaces.yml` intent overlay maintained by humans or AI, non‑Rust microservices are incorporated into the `lunar-scope` topology canvas with zero overhead and 100% determinism.

*   **Adapter path override mechanism**:  
    By default, the `lunar` controller dynamically discovers executables named `lunar-extract-<lang>` via the system `PATH` environment variable. Users can explicitly specify adapter paths in `.lunar/config.yml` (using the `adapters` field), which takes precedence over `PATH` auto‑discovery.

*   **Line‑delimited JSON (JSON Lines) communication and atomic end‑of‑stream marker**:  
    The orchestrator launches the adapter as a subprocess. To defend against heap overflow in large projects, the adapter **must** stream output line‑by‑line (line‑by‑line flushing) [3]. Each extracted route is immediately written to `stdout` and flushed – the adapter **must not** accumulate the whole dataset in memory before serialising it [3].  
    *   **Output stream atomicity and count validation (End‑of‑Stream Marker)**: To prevent the orchestrator from accidentally receiving an incomplete or corrupted interface list due to a crash in the middle of the adapter, **the adapter MUST output a special end‑of‑stream marker line as its very last line after successfully extracting all routes**:
        `{"_lunar": {"status": "success", "count": 42}}`
        **After receiving this marker line, the orchestrator MUST strictly verify that the total number of routes actually parsed equals the `count` value. If they differ (implying truncation or data loss), the orchestrator discards all lines and triggers `ERR_LUNAR_ADAPTER_CRASH`.** All debug and warning logs **must** be redirected to `stderr`; they are strictly forbidden from polluting `stdout` [1.2.1].

*   **Adapter error isolation strategy**:  
    If an adapter for a particular framework crashes and returns a non‑zero exit code, the orchestrator **must** gracefully catch it, isolate it, skip scanning that project, log a warning, and continue execution – it **must never** block the main CI process.

### 4.2 Phase 2: Confirm & Semantic Normalisation – Constraint Propagation Principle
*   **Semantic normalisation**: Framework‑specific regular expressions and constraint expressions (e.g. Express `:id(\\d+)`, FastAPI `{id:int}`) from different adapters are forcibly translated and normalised into the standard Rust in‑memory model `RouteAst` [1.1.1].
*   **Constraint preservation compromise and limitation**:  
    Statically proving the mathematical equivalence of various irregular regular expressions across languages is infeasible in practice. At v0.5.0, the confirmation engine **does not forcibly translate constraint regular expressions**. `rawConstraint` is kept as‑is and passed as descriptive metadata; the matching algorithm during alignment performs only position and basic type comparison. **The limitation is that the system cannot automatically detect and warn about runtime 400 validation mismatches caused by asymmetric regular expression sets (i.e., inconsistent constraint definitions between two ends) across multi‑language frameworks.**

### 4.3 Zero‑Overprivileged Synchronisation Mechanism and Guided Sync
To guarantee developers 100% control over their code, the `lunar` CLI refuses any silent modifications of human‑maintained files in the background.
*   **`lunar init`**: **Only when local `.lunar/interfaces.yml` does NOT exist**, it automatically runs a physical scan and creates a draft file. If the file already exists, this command does nothing and does **not** overwrite any manual work.
*   **`lunar diff`**: Performs a comparison between the physical facts (AST) and the intent overlay (`interfaces.yml`), outputting a standard Git‑diff style change report to the console.
*   **`lunar sync --apply`**: A user‑triggered synchronisation merge command that supports `--dry-run` to preview changes. Before actually writing the merge, it **forces** a backup of the old `interfaces.yml` into a local hidden backup directory `.lunar/.backup/interfaces.yml.bak` (this path is automatically added to the project’s `.gitignore` by `lunar init`).
*   **AI‑generated patch mechanism**: The `.lunar/suggestions/` directory is used to store YAML‑format intent overlay suggestion patches generated by humans or AI. When merging `interfaces.yml`, `lunar sync --apply` automatically detects and processes patch files in this directory, then moves processed files into the `merged/` subdirectory.

---

## 5. Project‑level Intent Overlay and Ecosystem Configuration Schemas (YAML/JSON)

To ensure uniform consumption by multi‑language teams, all configuration fields strictly follow the Google JSON Style Guide’s standard camelCase format.

### 5.1 `.lunar/interfaces.yml` Specification
The project‑level central contract is the first gate where developers control and define interface boundaries:
```yaml
# ===================================================================
# LunarAST Project Interface Contract
# This file is owned and maintained by humans.
# ===================================================================

project: myPaymentService
type: mixed # role: service / client / mixed
environment: production

# Manually declared APIs
exposed:
  - path: /api/v1/payments/refund
    method: POST
    reason: "Refund‑specific endpoint (planned for next week)"

# Manual contract override for complex dynamic calls
consumed:
  - path: /api/v1/auth/verify
    method: POST
    targetProject: authService  # must exist in ecosystem repos.json, otherwise ldg doctor will error
    reason: "Pre‑transaction session verification for user"
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

### 5.3 Ecosystem Centralised Topology Declaration: `ecosystem-topology.json`
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

### 5.4 Ecosystem Build Plan Lock File: `ecosystem-plan.json`
This file is the starting point of every ecosystem generation build. It is strictly created and written by the centralised ecosystem automation tool and is hard‑locked each time a `generationId` is produced.
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

## 6. Data Exchange Standard Format Specification (The Exchange Contract Spec)

As the physical product exported by the whole ecosystem, the data must follow a strict static format definition to ensure high density and high parsing efficiency.

### 6.1 Structured Topology Standard: `lunar-map.json`
Below is the top‑level schema contract specification for `lunar-map.json` (fully follows Google camelCase naming):

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

### 6.2 AI Context Map Standard: `lunar-map.md` (Markdown Spec)
On request, the bucket does **not** store a static `lunar-map.md`. Instead, **`lunar-gateway` (or the edge `lunar-serve`) parses, translates, and dynamically renders it from `lunar-map.json` in real time** when the client requests it. The following query parameters are supported to perform parametric filtering and reduce AI token consumption:
*   `GET /lunar-map.md?summary=true` (returns a very short summary, token‑saving, suitable for AI first access) [1.2].
*   `GET /lunar-map.md?style=list` (returns a plain text list of contracts, suitable for quick extraction in small contexts).
*   `GET /lunar-map.md?style=mermaid` (returns a Mermaid topology diagram, suitable for global chart rendering).
*   `GET /lunar-map.md?scope=project-a` (sharded fetch, limits the result to the sub‑topology related to a specific project, preventing the AI context window from being exceeded).

---

## 7. Open Integration and Third‑Party Bridge Isolation Specification

LunarAST maintains high statelessness and data sovereignty. Any external execution sandbox (e.g. IDE plugin, AI agent) must follow the **“Bridge Isolation Pattern”** [2]:

*   **Bridge responsibilities**: The third‑party integrator must write an independent bridge program (e.g. `routeast-mcp-bridge`) that pulls `lunar-map.json` via standard HTTP `GET` interfaces and then internally translates it into the target protocol.
*   **Non‑overprivileged interaction specification**: When writing AI‑generated alignment suggestions to local storage, the bridge **must not** silently modify `interfaces.yml`. The bridge’s only responsibility is to generate and output Git‑diff‑style alignment patch fragments to the terminal and suggest that the user manually run `lunar sync --apply` locally [7].

```rust
// Minimal interaction contract between a bridge and LunarAST
#[async_trait]
pub trait LunarMcpBridge {
    /// 1. Fetch the clean static topology, optionally with a shard limit (?scope=projectX) to reduce token consumption
    async fn fetch_lunar_map(&self, gatewayUrl: &str, scope: Option<&str>) -> Result<LunarMapPayload, BridgeError>;
    
    /// 2. Translate to standard Tools & Resources contracts (JSON‑RPC) of the target protocol
    fn translate_to_mcp_tools(&self, payload: LunarMapPayload) -> Vec<McpToolSchema>;
}
```

---

## 8. Stateless Distribution Layer Design and Security Model (`lunar-gateway`)

`lunar-gateway` focuses on high‑concurrency, stateless, extremely fast secure contract data distribution [2]:

### 8.1 Storage Directory Structure Standard
*   **Recommended S3 bucket naming**: `lunar-ast-<organization>`.
*   **Global topology location**: **`lunar-map.json` is an ecosystem‑wide product and MUST NOT be stored under a single service’s `commits/` directory**. The gateway stores it in `ecosystem-config/` grouped by **generations**.
```
s3://lunar-ast-<organization>/
├── <repo>/
│   ├── commits/
│   │   └── <sha>/
│   │       └── <subdomain>-actual.json   # physical facts cache (e.g. route-ast-actual.json)
│   └── pointers/
│       └── latest.json                   # composite pointer containing the latest SHA
└── ecosystem-config/
    ├── repos.json                         # ecosystem project allowlist
    ├── ecosystem-topology.json            # orchestration and topology declaration
    ├── latest-generation.json             # static pointer to the current active generation
    └── generations/
        └── <generationId>/
            ├── meta.json                  # metadata summary for this generation
            ├── ecosystem-plan.json        # lock file for this generation’s build plan
            └── lunar-map.json             # global alignment topology for this generation
```

#### 8.1.1 Metadata file: `meta.json` specification
The `meta.json` file generated by the CI/CD engine during the build and forcibly written synchronously **must** follow this schema:
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
*   **Attribute description**: `compilerVersion` refers to the version of the `lunar` compiler binary that performed the static analysis and compiled the output data; it is **not** bound to the gateway’s version. If `downgradeMap` is absent or does not contain a downgrade conversion subtree compatible with the requested version, the gateway **must** refuse to perform a lossy downgrade.

#### 8.1.2 Generation global pointer: `latest-generation.json` specification
```json
{
  "generationId": "20260608T030000Z-a1b2c3d4",
  "lastUpdated": "2026-06-08T03:00:00Z"
}
```

### 8.2 Two‑Stage Isolated Cache and Client‑Side Differentiated Cache Control
To balance the requirements of “enforce signature verification on every request” and “zero object‑storage penetration”, `lunar-gateway` adopts a two‑stage isolated cache mechanism [2]:
*   **Internal cache**: The gateway uses the **full logical path of the file relative to the bucket** as the cache key.
*   **Security‑first principle**: For private resources that require JWT signatures (e.g. `actual.json` of private projects), **the gateway MUST unconditionally complete token integrity checks (including signature and clock skew) first [1.3.3]. Only after authentication passes may it match and read from the internal cache**. The internal cache key still uses the de‑tokenised logical path so that the cache can be shared among multiple clients, but the authentication step must never be bypassed by any caching mechanism.
*   **Differentiated client‑side caching headers**:
    *   **Private resources (require JWT, e.g. `actual.json` of private projects)**: The gateway forcibly strips all edge caching headers and rewrites them to `private, no-cache, no-store, must-revalidate`, preventing token hijacking. Internally, the gateway may still use the de‑tokenised logical path as the key for `immutable` edge cache matching.
    *   **Public immutable resources (no token required, e.g. resources under `/commits/<sha>/`, global topology `lunar-map.json`, `ecosystem-plan.json` and `meta.json` under `generations/`)**: The gateway allows clients to perform long‑term local caching: `Cache-Control: public, max-age=31536000, immutable`, maximising edge performance.
    *   **Latest dynamic pointers `/pointers/latest.json` and the latest generation pointer `latest-generation.json`**: The gateway allows short‑term local caching: `Cache-Control: public, max-age=300`, ensuring timely version consistency.
*   **Gateway dynamic rendering cache and OOM defence**:
    *   **Dynamic rendering endpoint cache**: The gateway caches Markdown rendering results for `lunar-map.md` queries in memory using the composite key `(Accept, scope, generationId)`. The cache lifetime is strongly tied to the version of the corresponding `lunar-map.json`.
    *   **Memory protection circuit breaker**: Based on the typical 128 MB physical memory constraint of Cloudflare Workers, for large files (> 2 MB) the gateway automatically breaks the “in‑memory byte buffer” and, respecting the 128 MB limit, falls back to stream‑through forwarding. This sacrifices one R2 fetch to protect the gateway isolate from crashing. Override is possible via the environment variable `LUNAR_GATEWAY_BUFFER_LIMIT_BYTES`.

### 8.3 Security Verification and JWT Signature Contract
*   **Signature algorithm specification**: **Ed25519 (EdDSA)** is forced as the sole signature algorithm, guaranteeing constant‑time verification [1.3.3].
*   **Expiration policy**: The `exp` claim in JWT tokens SHOULD NOT exceed 24 hours.
*   **Public key distribution and KV cache rotation strategy**: Public keys can be supplied in static mode (Worker environment variables) or dynamic mode (Cloudflare KV). The gateway memory limits maximum storage by setting `MAX_KEYS_PER_REPO = 3` (each logical project keeps at most the latest key plus at most 2 historically valid public key fingerprints across versions), preventing malicious public key registrations from exhausting gateway memory.
*   **Warning against URL parameter tokens**: In production, supporting JWT tokens passed in URL parameters is **strongly discouraged**. If the gateway detects a token in the URL, it MUST output a security warning to `stderr`.

### 8.4 Version Negotiation and Backward Compatibility Strategy
When the major version of `lunar-map.json` changes (e.g. from `0.5.0` to `1.0.0`), clients perform backward‑compatible version negotiation using HTTP media type parameters:
*   **Negotiation routing specification and window**:  
    The client sends a request with the header `Accept: application/vnd.lunar.0.5.0+json`. **The gateway MUST maintain a data downgrade compatibility window for the currently active major version and the immediately preceding major version (i.e., two major versions in total).**  
    *   **Downgrade boundary**: The gateway performs automatic downgrade **only if** a pre‑defined, lossless mapping table (described by the `downgradeMap` inside `meta.json`) exists between the old and new versions. If a major version change introduces mandatory field changes, enum additions/removals, or semantic incompatibilities, the gateway **must not** perform automatic downgrade; it must return `406 Not Acceptable` and include the `X-Lunar-Upgrade-Required: true` header.

### 8.5 Crates Physical Reorganisation and Workspace Dependency Governance (Decoupled Workspace Specification)

To prevent ecosystem bloating from causing module deadlocks and unnecessary compile‑time burden, LunarAST adopts a Cargo Workspace multi‑crate isolation governance specification:

1.  **Contract library physical separation (The Interface Crate)**:  
    A zero‑dependency static contract library `lunar-interface` is extracted. It only contains the core data models (`RouteEntry`, `ActualJson`, `LunarMap`, etc.) and the graph alignment logic `generate_lunar_map`.
2.  **Decoupling CLI from serving layers**:  
    The distribution layers (`lunar-serve`, `lunar-gateway`) depend **only** on the `lunar-interface` contract library; they **must not** directly or indirectly depend on the monolithic `lunar` binary that carries CLI‑specific payloads (e.g. `clap`, `rust-s3`, `ed25519-dalek`). This maximises compile speed and keeps the serving layers lightweight.

---

## 9. Alignment Status Priority and Diagnostic Classification (Diagnostic Short‑Circuit)

When performing path parameter matching, the alignment engine applies a single‑path short‑circuit evaluation over the server‑side and client‑side `RouteAst` lists. A single interface pair can belong to only one final status, and the status priority is:
$$\text{MethodMismatch} > \text{Orphaned} > \text{Unused} > \text{ParamNameMismatch} > \text{Aligned}$$

*   **Failure isolation and stale‑node alignment mechanism (`unverified` state isolation)**:  
    To avoid large‑scale false alignment anomalies (e.g. consumers incorrectly flagged as `Orphaned`, producers incorrectly flagged as `Unused`) caused by some projects having `scanStatus: failed` or stale data due to CI build inconsistencies:
    1.  **Alignment isolation extraction**: The alignment engine MUST consult the `scanStatus` inside the `projects` array before computation [2].
    2.  **`stale` node handling**: For projects with `stale` status, their `exposed` and `consumed` data **are still read by the alignment engine and participate in comparison**. However, **all final alignment results involving such a project have their `status` forcibly set to `"unverified"`** [2].
    3.  **`failed` node handling**: Because a `failed` project cannot provide a valid `interfaces` definition (it is `null`), no actual comparison is possible. **The alignment engine MUST iterate over all other healthy projects in the current generation topology. Whenever it detects that a healthy project has issued a `consumed` reference to a service belonging to a `failed` project, it does NOT perform the usual MethodMismatch or Orphaned checks. Instead, it directly generates an alignment entry with `status: "unverified"` for that pair.** At the same time, because the `failed` project has no valid interface data (its `interfaces` field is `null`), its own exposed endpoints do **not** produce any alignment entries in the `alignments` array, so they are not mistakenly flagged as `Unused`.
    4.  **Unverified contract semantics**: Entries marked `unverified` (priority 5) are shown with a fuzzy yellow line in `lunar-scope` and are not considered contract anomalies in data flow or gate audits – they merely indicate outdated or not‑ready data sources.
*   **Diagnostic judgement principles (single‑path short‑circuit by severity)**:  
    1.  **MethodMismatch (priority 1)**: As long as the path structures are identical but the client uses `POST` while the server only exposes `GET`, the alignment engine immediately raises MethodMismatch and stops further checks.
    2.  **Orphaned (priority 2)**: A call is made to a specific target service, but among all services registered in the ecosystem topology `repos.json`, no node satisfies the path base and method contract.
    3.  **Unused (priority 3)**: **(Topology‑level global computed state)**. An endpoint is exposed by the server but no client in the whole ecosystem topology calls it. This state is accurately annotated by `lunar` during the **post‑processing phase** of generating `lunar-map.json`: it scans the whole ecosystem left to right, cross‑correlates every project’s `exposed` with the full set of `consumed` dependencies, and provides high architectural governance value for cleaning up ghost contracts and narrowing the attack surface [2].
    4.  **ParamNameMismatch (priority 4)**: Parameter positions and types align, but the names differ. Triggers the magnetic dotted line in the frontend [2].

---

## 10. Observability and Health Check Specification

### 10.1 Structured Logging Specification (JSON Lines)
`lunar-gateway` and the local read‑only service MUST output newline‑delimited JSON structured logs to `stdout`. The format fully follows camelCase and includes at least the following basic attributes:
```json
{"timestamp":"2026-06-12T01:00:00Z","level":"INFO","method":"GET","path":"/public/repo-a/commits/sha-123/lunar-map.json","status":200,"durationMs":12,"cache":"HIT","authStatus":"valid","clientIp":"12.34.56.78"}
```
*   **`authStatus` state machine definition**: Allowed values only: `valid`, `expired`, `invalidSignature`, `missingToken`.

### 10.2 Metric Exposure (Prometheus Metrics)
The gateway MUST maintain in‑memory statistics and expose a static `/metrics` endpoint compliant with the Prometheus standard. **To ensure ecosystem alignment with mainstream monitoring systems (Prometheus specification), when exposing these metrics to the outside, the implementing layer MUST automatically convert camelCase label names to the snake_case format recommended by Prometheus best practices (e.g. `targetVersion` $\rightarrow$ `target_version`)**:  
*   `lunar_gateway_requests_total` (total request counter, labels: `method`, `status`).
*   `lunar_gateway_cache_hits_total` (cache hit counter, labels: `cacheType` (internal/client)).
*   `lunar_gateway_auth_failures_total` (authentication failure counter, labels: `reason` (expired/signatureInvalid/missingToken)).
*   `lunar_gateway_version_downgrade_requests_total` (version backward‑compatibility downgrade request counter, label: `targetVersion`). Used during the transition period to monitor and evaluate lossy downgrade traffic caused by outdated client tooling.

### 10.3 Liveness Health Check
The gateway exposes a very fast read‑only probe endpoint that does **not** go through authentication: `GET /healthz`. On success it directly returns `200 OK`, suitable for container and edge runtime liveness probing.

---

## 11. Security Threat Model and Defence Strategies

To ensure the stability of LunarAST in a zero‑trust environment, four physical defence lines are established:

1.  **Injection attack protection and lexical rules**:  
    Because magic comments (`// lunar:consume`) are lexically scanned by the adapter, there is a risk of malicious code injecting non‑whitelisted comments. **The adapter SHOULD apply the corresponding language‑specific lexical filtering rules combined with AST node determination.** During scanning, it performs strict physical lexical interception and only extracts comment lines that match predefined rules (e.g. for Python the regex `#\s*lunar:(expose|consume)`, for HTML templates `<!--\s*lunar:(expose|consume)\s*-->`, etc.). All malformed comments that do not match the preset rules or are severely non‑compliant MUST output the store path, a warning, and the line number to `stderr`. All such non‑compliant lines are forcibly ignored and **not** passed into the backend AST tree, thereby blocking injection risks.
2.  **Key rotation overflow defence**:  
    The gateway caches public keys in memory with a soft TTL. The soft TTL cache forcibly limits maximum storage overhead by setting `MAX_KEYS_PER_REPO = 3` (each logical project keeps at most the latest key plus at most 2 historically valid public key fingerprints across versions), preventing malicious registration of many obsolete public keys from exhausting gateway memory.
3.  **S3 credential minimisation**:  
    The S3 credentials held by CI Actions only have `s3:PutObject` and `s3:GetObject` permissions on the corresponding bucket paths; they do not have overprivileged capabilities such as deleting buckets or modifying bucket IAM policies.
4.  **Destructive action anti‑accident barrier and CLI permission separation**:  
    For all local CLI destructive operations or ecosystem‑wide cleanup actions (e.g. `lunar cleanup`), **interactive, blocking confirmation** is enforced in the terminal. Only when the user explicitly adds a non‑interactive override flag (e.g. `--yes`) may the interactive prompt be skipped, ensuring smooth CI/CD automation integration. This mechanism isolates destructive commands and completely blocks the risk of overprivileged accidental operations [2].

---

## 12. Core Error Codes and Debugging Guide

| Error Code | HTTP Status | Physical Cause | Recommended Recovery Action |
| :--- | :--- | :--- | :--- |
| `ERR_LUNAR_CONFIRM_FAIL` | 400 | Phase 2 normalisation validation fails, e.g. invalid wildcard format. | Run `lunar diff` and check the reported syntactically invalid sections. |
| `ERR_LUNAR_ADAPTER_CRASH` | 422 | Phase 1 subprocess extractor crashed (e.g. Node parser syntax error), or the total number of routes parsed from the stream does not match the `count` in the end‑of‑stream marker line (data truncation). | Check the `stderr` stack trace of the corresponding adapter in the CI logs. |
| `ERR_LUNAR_PROJECT_NOT_FOUND` | 404 | The target project is not defined in the ecosystem registry `repos.json`. | Add the target project to the ecosystem `repos.json` registry and trigger a new build. |
| `ERR_LUNAR_INTERFACE_NOT_FOUND` | 422 | The target project exists but does not expose a path/method matching the client’s consumption. | Run `lunar diff`, examine the differences between client consumption and server exposure, modify `interfaces.yml`, and sync. |
| `ERR_LUNAR_INTERFACE_DATA_MISSING` | 410 | Because the project failed scanning or was not ready during the alignment build (`scanStatus: failed`) and its `interfaces` field is `null`, the gateway completely lacks the reverse‑derivation data source and cannot fall back. | Run `lunar doctor` to diagnose the heterogeneous sub‑service that failed to build. |
| `ERR_VERSION_EXPIRED` | 410 | The physical fact cache has been automatically purged from object storage after more than 90 days. | The terminal prompts “Hanging Pointer”. The gateway MUST include an `X-Lunar-Recovery: Trigger CI pipeline for <repo>` header with a pre‑configured CI automation trigger webhook path. **Guide and encourage the developer to re‑trigger the CI build for the corresponding repository.** |
| `ERR_GATEWAY_STREAM_FALLBACK_FAILED` | 502 | The contract file size exceeded the limit (>2 MB) and the fallback stream‑through forwarding failed to fetch from the origin. | Check whether `lunar-map.json` contains unnecessary frontend static assets, and verify the connectivity of the storage origin. |

---

## 13. Roadmap and Core Evolution Milestones

The evolution of the whole ecosystem abandons hard‑bound timelines and adopts a milestone model with clearly defined deliverable artefacts:

*   **Milestone 1 (RouteAST basic implementation)**:
    *   Freeze the `RouteAST Base IR v0.5.0` contract specification.
    *   Implement a pure‑Rust prototype of the Confirm Core.
    *   Provide lightweight adapters for Rust (Axum) and Node.js (Express).
*   **Milestone 2 (Complete four‑layer architecture and secure distribution)**:
    *   Release the user‑side `lunar` CLI tool, implementing the core commands `init`, `scan`, `diff`, `sync --apply`.
    *   Deploy the stateless edge gateway `lunar-gateway` (supporting two‑stage caching, differentiated caching control, edge memory protection and circuit breaking, observability logs, and Prometheus monitoring).
    *   **Refactoring upgrade achieved**: Multi‑crate modular Workspace architecture decoupled; lightweight `lunar-interface` core model extracted; `lunar-serve` completely decoupled from the CLI application dependencies; case‑insensitivity and two‑level fallback for local workspace path auto‑detection implemented [1.2].
*   **Milestone 3 (Multi‑dimensional presentation and de facto standard establishment)**:
    *   Release the frontend **lunar-scope intelligent physical magnetic canvas** based on the `MatchResult` status priority mechanism, supporting visualisation of `Unused` and `unverified` interface states [2].
    *   Freeze the `EventAST` and `SchemaAST` specification standards and publish official event/data adapters.
*   **Milestone 4 (Full static deep code‑level auditing and open ecosystem)**:
    *   Publish the `TypeAST` contract standard and cross‑project compile‑time dependency auditing.
    *   Provide the standard `lunar-map.json` to the external ecosystem and security auditing platforms, completing full‑stack static interface alignment and drift defence.

---

## Appendix B: References and Specification Sources

*   RFC 6570 - URI Template Specification for parameter standardization.
*   IEEE Std 1471-2000 - Systems and software engineering - Recommended practice for architectural description of software‑intensive systems.
*   JSON Lines Standard (v1.0) - Line‑Delimited JSON streaming format.
*   WASI 0.2 (Component Model) - WebAssembly System Interface specification.
*   ACM TOSEM Vol. 33 - Cross‑Language Static Program Analysis on Microservice Topologies.
*   [1.3.3] NIST FIPS 186‑5 - Digital Signature Standard (DSS) guidelines for Ed25519 system signature integration and curve verification.

---

### A.5 CLI Quick Reference Cheat Sheet

```bash
lunar init                     # Auto‑detect technology stack and bootstrap local draft (only when interfaces.yml does not exist)
lunar scan                     # Statically scan the current project’s physical facts and write .interfaces-autogen.json cache
lunar diff                     # Print a standard Git‑diff report between physical facts and the manual interfaces.yml intent overlay
lunar sync --dry-run           # Preview the changes that would be synchronised
lunar sync --apply             # After automatically backing up the old file, merge the actual code changes into the config file
lunar doctor                   # Validate project S3 connectivity, latest pointer status, and topology consistency
lunar cleanup --all            # Interactively guide removal of all S3/R2 data for the current project (deletes all objects under commits/ and pointers/ for this project, does not affect ecosystem-config/). To skip interaction, add the `--yes` flag (high risk, use with caution)
```
