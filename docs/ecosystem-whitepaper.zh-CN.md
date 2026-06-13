# LunarAST & lunar-gateway 协议与系统架构规范

**——多层异构静态契约标准与“生、传、标、展”四层架构规范**

**版本**：1.6 — 规范标准设计与重构升级版（完全闭环版）  
**最后更新**：2026-06-13  

---

## 1. 物理角色定义

本规范定义 LunarAST 生态中各组件的物理边界与命名契约 [5]：

| 组件/命名 | 物理层级 | 核心职责 |
|:---|:---|:---|
| **LunarAST** | **标准规范层** | 制定多层契约（RouteAST、EventAST 等）的 Base IR 规范、提取协议与数学比对算法语义 [5]。它是底层的静态契约标准，作为静态 Schema 规范存在，本身不包含可运行代码。 |
| **`lunar`** | **数据生成层** | 用户本地与 CI 管道的命令行可执行二进制。负责 `lunar init`（初始化底稿）、`lunar scan`（物理提取）、`lunar diff`（无越权比对）和 `lunar sync --apply`（主动备份并安全同步） [2]。对齐决策与合并逻辑完全在此执行。 |
| **`lunar-serve`** | **本地只读分发层** | 本地运行的极轻量只读 HTTP 分发层二进制。依赖 `lunar-interface`，提供开发态下的 `lunar-map.json` 本地高保真渲染，并支持在零人工配置状态下，通过 Fallback 机制安全、按需、免归档地为 AI Agent 直接提供本端物理工作区的源码镜像直读。 |
| **`lunar-gateway`** | **无状态分发层** | 独立部署的 Serverless 边缘网关程序。编译为 `wasm32-wasip2` [4]，执行基于安全令牌（Ed25519-JWT）的单向鉴权与高并发双阶段隔离缓存分发 [2]。网关不运行任何实时接口对齐比对逻辑。 |
| **`lunar-scope`** | **可视化呈现层** | 纯静态的前端多层关系画布。通过标准 API 从网关拉取已在构建期对齐完备的拓扑 JSON，在浏览器中渲染具有磁吸预测虚线、断点高亮与架构漂移警告的前端人机交互界面。 |

### 1.1 隐喻定位与物理对应
*   **数据反射（LunarAST 标准层）**：系统本身不产生运行期数据（不发光），不参与任何运行时监控与性能损耗。它在构建期接收源码变更中的物理事实（由于开发者的提交和编译动作照亮的地貌），并将其静态投影出来。
*   **生态呈现（lunar-scope 呈现层）**：作为纯静态的前端多图层关系画布。它为架构师提供多角度的契约数据低延迟观测，在代码部署前捕捉悬空的断点与未规划的野生依赖 [2]。

---

## 2. 三级递进真理源架构

```
      ┌────────────────────────────────────────────────────────┐
      │   第一级：物理事实 (Physical Facts / AST)               │  ← 80-90% 自动推导，写入 .interfaces-autogen.json
      ├────────────────────────────────────────────────────────┤
      │   第二级：意图覆盖层 (Intent & Override Overlay)        │  ← 人工管控，.lunar/interfaces.yml (首要关卡)
      ├────────────────────────────────────────────────────────┤
      │   第三级：末端逃生舱 (Escape Hatch / Comments)          │  ← 仅用于极其复杂的动态 RPC 边缘调用
      └────────────────────────────────────────────────────────┘
```

### 2.1 第一级：物理事实 (Physical Facts) — 自动推导
*   **物理文件**：**`.lunar/.interfaces-autogen.json`**（此文件必须加入项目的 `.gitignore`）。
*   **物理事实生命周期**：由于该文件不进入 Git 库，在 CI/CD 或本地编译阶段，构建机自动在本地运行 `lunar scan` 重建此物理事实缓存文件，不依赖历史版本缓存。

### 2.2 第二级：意图覆盖层 (Intent & Override Overlay) —— 字段级部分覆盖与合并契约
*   **物理文件**：项目根目录下的 **`.lunar/interfaces.yml`**。
*   **属性**：**100% 由人类掌控、处于版本控制（Git）下、工具严禁任何未授权的静默修改**。
*   **合并公式 ($\oplus$) 的字段级部分覆盖定义**：
    对于同一个联合主键 `(Path, Method)` 定位的接口对象，对齐引擎在编译期执行 **“字段级部分覆盖（Partial Field Override）”** 规则：
    *   定义 $A$ 为物理事实（Actual AST）中的某个接口对象，$I$ 为意图覆盖层中的同名接口定义。
    *   对于接口对象的任意属性字段 $f$（如 `port`、`rawConstraint` 等）：
        $$\text{Resolved}.f = \begin{cases} 
          I.f, & \text{if } I.f \text{ is specified} \\
          A.f, & \text{otherwise} 
        \end{cases}$$
    *   该规则确保了在人工覆盖特定网络参数或路径属性时，**原样保留物理事实中的其他静默元数据**（如源码物理地标 `sourceFile`、`lineNumber` 等） [2]。
    *   **覆盖边界限制**：对于数组类型（如 `segments`）及嵌套对象，覆盖层**仅支持全量替换（Complete Replacement）**，不支持部分字段合并。若意图覆盖层与物理事实在该字段上均非空且长度或子键不一致，`lunar diff` 必须告警并拒绝执行自动合并（即使在非严格模式下），强制要求人工对该数组执行完整重写。
    *   **冲突检测**：若覆盖层改变了物理事实中已存在的核心属性，`lunar diff` 必须向终端输出高亮警告（Warning）日志，提请人类审核，但工具链不阻断非严格模式下的编译。

### 2.3 第三级：末端逃生舱 (Escape Hatch)
*   **定义**：源码行上方的以 `// lunar:` 或 `# lunar:` 开头的单行魔法指令 [2]。
*   **适用边界**：**仅用于动态拼接的 RPC/HTTP 调用，其中目标路径和方法由于动态求值导致无法被静态 AST 分析捕获。对于标准路由，适配器强制提取物理事实，禁止通过注释重复声明。**
*   **语法范例**：
    ```typescript
    // lunar:consume POST https://api.auth-service/v1/token
    await axios.post(dynamicUrl, data);
    ```

---

## 3. 四层解耦物理蓝图与拓扑生成

为了保持系统内核的绝对鲁棒性，`LunarAST` 将复杂的分布式依赖关系切割为四个相互独立、物理隔离的子领域契约，彻底杜绝协议膨胀 [5]：

1.  **`RouteAST` (路由与网络契约)**：专注同步网络接口（REST/gRPC/Nginx）。元数据包含 `method`, `segments`, `port`。**当前处于 v0.5.0 规范设计草案阶段**。
2.  **`EventAST` (事件与异步契约)**：专注异步解耦的事件发送/订阅及请求-响应模式。元数据包含 `brokerType`（如 Kafka/NATS/RabbitMQ）, `action`（取值 `publish` / `subscribe` / `request` / `reply`）, `topic`, `payloadSchemaHash`。**规范规划中**。
3.  **`SchemaAST` (存储与数据契约)**：专注底层的数据库表、对象存储桶及缓存依赖，用于发现微服务中隐秘且致命的“数据库隐式耦合”。元数据包含 `storageType`（如 PostgreSQL/MongoDB/S3/Redis）, `database`（库名/桶名）, `table`（表名/键匹配模式）, `operation`（取值 `read` / `write` / `join`）。**规范规划中**。
4.  **`TypeAST` (代码与类库契约)**：专注编译期的代码级复用与接口强绑定依赖。元数据包含 `libraryName`（共享包名）, `typeIdentifier`（数据 DTO 或公共算法公式结构体）, `interfaceDefinitionFile`（如 `.proto` / gRPC Stubs 路径）, `versionConstraint`。**规范规划中**。

### 3.1 多仓库生态的拓扑生成流程（静态合流与世代同步机制）
*   **单仓库（CI 阶段）**：
    单个仓库在构建期执行 `lunar scan`，提取出该项目的 `<subdomain>-actual.json`（例如 `route-ast-actual.json`）并上传至 S3/R2 [2]。
    *   **指针更新机制**：单仓库 CI 在成功上传 `<subdomain>-actual.json` 之后，**必须立即更新并上传其对应仓库的指针文件 `pointers/latest.json`**，使其 `sha` 指向本次提交的 Commit SHA，从而确保生态编排工具能实时捕捉最新可用的物理版本。
*   **世代构建触发与原子性保证（Generation Sync）**：
    为了在多服务异步 CI 构建时，保证生态级别一次构建对应一个确定、原子且合法的全局静态快照，系统采用 **“生态构建计划锁文件（Ecosystem Lockfile）”** 同步机制：
    1.  **计划生成**：生态管理员或生态级自动化发布工具通过定期或基于 Commit 事件轮询各项目 `latest.json` 指针，生成一份目标 SHA 静态快照文件 **`ecosystem-plan.json`**，同时赋予该世代唯一的 `generationId`（该 ID 严格由生态发布工具在计划文件创建时基于时间戳和内容哈希生成，格式为：`<timestamp>-<uuid>`）。
    2.  **计划发布**：该计划文件被上传至 `ecosystem-config/generations/<generationId>/ecosystem-plan.json`。
    3.  **中心管道激活**：该计划文件的上传事件直接作为静态信号，单向触发中心化对齐构建管道，消除“先有鸡先有蛋”的死锁 [2]。
*   **核心对齐与检验边界**：
    1.  **输入合规校验（Set Equality）**：**中心管道在启动时强制校验 `ecosystem-plan.json` 中定义的所有项目集合必须与生态注册清单 `repos.json` 中声明的项目集合完全相等（Set Equality）。** 若计划中缺少任何已注册项目 or 包含未注册项目，中心管道必须拒绝执行、抛出错误并退出，强制要求重新生成完整的锁文件，确保每一次世代快照均为全生态的完整投影，防范局部快照缺失引发的大规模虚假对齐异常。
    2.  **构建幂等性保证**：管道首先检查 `ecosystem-config/generations/<generationId>/lunar-map.json` 是否已存在。若已存在，判定该世代对齐已静态就绪，直接跳过计算。
    3.  **世代拉取与隔离判定**：管道并行拉取所有项目的 `actual.json` 缓存：
        *   若所有声明项目的目标 `actual.json` 均已就绪：中心管道对齐数据并生成全局 `lunar-map.json`。
        *   **`failed` 状态判定**：对于在 5 分钟超时窗口内未完成 `actual.json` 上传的项目，在 `lunar-map.json` 中标记其 `scanStatus: failed`，并强制将其 `interfaces` 字段置为 `null` [2]。
        *   **`stale` 状态与数据采纳判定**：若该项目在对象存储中仍留存有历史 `actual.json`，但其 `lastUpdated` 时间戳与当前世代编译时间的差值超过预设有效期（默认 7 天，由 `LUNAR_MAX_STALE_AGE_SECONDS` 控制）：**中心管道正常采纳其历史数据参与对齐计算，但将其 `scanStatus` 标记为 `stale`，且该项目产生的所有对齐条目的最终状态强行标记为 `unverified` [2]。**
    4.  **生态级顶级指针更新**：中心管道在成功上传对齐结果 `lunar-map.json` 与 `meta.json` 之后，**必须同步、原子地更新位于 `ecosystem-config/latest-generation.json` 的生态最新代指针**，将其 `generationId` 与 `lastUpdated` 指向本次完成的最新世代，以供下游客户端实现无状态的动态版本发现。
*   **反向推导机制与空值防御**：
    若 `<subdomain>-actual.json` 缓存由于网络或生命周期过期在存储桶中物理缺失，网关将强制从 `lunar-map.json` 中的 `projects[].interfaces` 执行数据反向推导。
    *   **空值异常防御**：若对应项目的 `interfaces` 已在构建期因为失败隔离被置为 `null`，网关判定该项目反向推导数据源物理不可用。网关立即中断分发，向客户端返回 `410 Gone` 并附加 `X-Lunar-Recovery` 头部引导重新构建，并在响应体中携带 `ERR_LUNAR_INTERFACE_DATA_MISSING` 错误码。
    *   **构建失败隔离与实际文件存在的不真实性批注（Honest Disclosure）**：在中心管道将某个超时未就绪项目标记为 `scanStatus: failed`  且 `interfaces: null` 后，该项目的 CI 容器可能在稍后时刻完成了 `actual.json` 的单独上传。此时，存储桶中该文件物理存在，而全局拓扑由于构建隔离将其视为不可用。当客户端绕过拓扑、直接通过 `GET /commits/<sha>/route-ast-actual.json` 访问该项目接口时，网关将返回成功。此类不一致由分布式编译的失败隔离边界引起，开发人员必须使用 `lunar doctor` 进行全局状态的一致性诊断。

### 3.2 零摩擦源码投影与降级寻路规范（Zero-Friction Source Code Projection）

为了在契约拓扑（Topology）之外，为 AI Agent 提供免归档、零摩擦、按需加载的源码镜像消费能力，系统必须支持“URL 投影”与“路径 Fallback”机制：

1.  **路由别名等价化（Route Aliasing）**：
    网关及本地只读服务在提供 GitHub 镜像访问时，必须将 `/blob/` (网页文件查看) 路径与 `/raw/` (原始文本直读) 路径在路由级别完全等价化处理。AI 可无视 URL 语法差异，直接通过将 `github.com` 替换为生态分发域名，获取物理文件内容 [1.2]。
2.  **绝对路径两级回落（Base Path Fallback Priority）**：
    服务端在解析项目物理工作区路径时，强制执行以下降级链路，拒绝任何全局强制硬编码：
    $$\text{ResolvedPath} = \begin{cases} 
      \text{Registry.path}, & \text{if specified in repos.json} \\
      \text{Topology.path}, & \text{else if automatically discovered in lunar-map.json} \\
      \text{Error (400)}, & \text{otherwise} 
    \end{cases}$$
3.  **大小写自适应归一化（Case-Insensitive Normalisation）**：
    在匹配 GitHub 网络坐标 `{owner}/{repo}/{branch}` 时，网关与服务层必须在内存中强制进行小写归一化（Lowercase Normalisation）哈希映射，彻底消除跨平台大小写命名差异导致的分发断层 [1.2]。

### 3.3 去中心化 AI 任务协同看板与密码学防伪校验规范（Decorrelated AI Handover Scratchpad）

为了在无状态、零信任的公网分发环境下，允许外部 AI 代理自主、增量地协助人类完善 `interfaces.yml` 契约，系统确立“去中心化 AI 任务协同看板”规范：

1. **状态事实载体（ai-todo.json）**：
   项目当前正在执行的 AI 开发进度及待合并补丁，统一持久化于本地项目隐藏目录 `.lunar/ai-todo.json` 下。该文件仅保留当前正处于 `pending` 状态的活跃任务，已合并的任务将在合并瞬间被物理擦除并剪枝，确保文件永远保持极小体积以节省 AI 传输 Token。
2. **Ed25519 任务指纹防伪（Cryptographic Verification）**：
   外部 AI 代理在向本端 `POST /api/v1/projects/:name/todo` 提交看板建议或 YAML 契约补丁时，必须强制使用其持有的 AI 私钥，对 `patch` 负载执行 **Ed25519 密码学数字签名**。
   本地 `lunar` 客户端在执行 `lunar pull` 一键拉取时，强制在本地加载对应项目的公钥对该签名进行数学验签。
   *   **安全防御边界**：任何未带签名、签名已过期、或与注册指纹不匹配的看板提交，本地 CLI 拒绝合并并强力熔断，彻底阻断公网接口恶意对齐注入与后门投毒风险。
3. **宏观里程碑结晶化折叠展示（Crystallized Milestone Rendering）**：
   在渲染 `/tree` 路由的 Markdown 数据时，`lunar-serve` 执行“微观剪枝，宏观结晶”规则。
   *   对于已完成的里程碑（Milestones），网关不展开任何已完成的微观子任务，仅以单行已完成勋章化状态标记渲染，为 AI 提供 100% 的宏观地标视野（Directional Context）而保持 0% 的微观 Token 噪声。

---

## 4. 数据生成层与编译流水线 (lunar)

### 4.1 阶段一：判断 —— 基础多语言自适应探测与降级事实生成
*   **基础多语言自适应探测与降级事实生成（Base Multi-Language Sniffer & Fallback Fact Generation）**：
    当 `lunar scan` 启动时，如果项目根目录下没有 `Cargo.toml`（非 Rust 项目），控制层会执行词法嗅探，检查是否存在：
    *   `requirements.txt` / `pyproject.toml` / `Pipfile` $\rightarrow$ 判定为主语言：`Python`
    *   `go.mod` $\rightarrow$ 判定为主语言：`Go`
    *   `package.json` $\rightarrow$ 判定为主语言：`Node.js`
    *   `nginx.conf` $\rightarrow$ 判定为主语言：`Nginx`
    若对应主语言的编译期 AST 提取器（如 `lunar-extract-python`）尚未安装在系统 PATH 中，控制层绝对不执行阻断性报错，而是自动启动**“声明式降级事实生成器”**，在本地 `.lunar/` 自动写入一个合法的空物理事实文件 `.interfaces-autogen.json`，并将 `projectType` 标记为对应语言。
    *   **架构收益**：该设计消灭了词法提取器缺失带来的生命期中断。它允许异构的多语言项目（如 Python、Go、Nginx）无障碍接入，通过本地人肉或 AI 维护的 `interfaces.yml` 意图覆盖层，零开销、100% 确定性地将非 Rust 微服务并入到 `lunar-scope` 拓扑画布中。
*   **适配器路径覆盖机制**：
    `lunar` 控制层默认通过系统环境变量 `PATH` 动态检索命名匹配为 `lunar-extract-<lang>` 的可执行二进制。用户可通过在 `.lunar/config.yml` 中显式指定 `adapters` 路径进行绝对路径覆盖，其优先级高于 `PATH` 自动发现。
*   **行分隔 JSON 通信（JSON Lines）与原子性结束标记**：
    Orchestrator 启动适配器子进程。为了防御大型项目下的堆溢出，适配器必须采用流式输出（Line-by-line Flushing） [3]。适配器每提取一条路由，立即将其写入 `stdout` 并执行 `flush`，严禁在内存中累积整个数据集再进行全量序列化 [3]。
    *   **输出流原子性与计数校验（End-of-Stream Marker）**：为了防止适配器执行中途崩溃（Crash）导致控制层误接收不完整、发生破损的接口列表，**适配器必须在所有路由提取成功后，在最后一行输出一条特殊的结束标记行**：
        `{"_lunar": {"status": "success", "count": 42}}`
        **Orchestrator 接收到此标记行后，必须强力校验实际接收解析出的路由总数是否等于 `count`。若不一致（暗示数据发生截断或丢失），Orchestrator 丢弃所有行并直接触发 `ERR_LUNAR_ADAPTER_CRASH`。** 所有的调试与警告日志一律重定向输出至 `stderr`，严禁污染 `stdout` [1.2.1]。
*   **适配器错误隔离策略**：
    如果某个特定框架的适配器崩溃并返回非零退出码，Orchestrator 必须优雅捕获，隔离并跳过该项目的扫描，记录警告日志后继续执行，绝不发生主 CI 进程阻断。

### 4.2 阶段二：确认 (Confirm & Semantic Normalization) —— 约束传递原则
*   **语义归一化（Semantic Normalization）**：将不同适配器输出的框架特定正则与约束表达式（如 Express 的 `:id(\\d+)`、FastAPI 的 `{id:int}`），强行翻译并归一化为标准的 Rust 内存模型 `RouteAst` [1.1.1]。
*   **约束保留妥协与局限**：
    在静态维度对跨语言的各种不规则正则表达式执行数学等价证明在工程上是不现实的。在 v0.5.0 阶段，确认引擎**不对约束正则进行强行翻译**。`rawConstraint` 被原样保留并作为描述性元数据进行传递，匹配算法在对齐阶段仅进行位置和基本类型比对。**该设计局限在于系统无法自动检测和预警跨多语言框架时由于正则匹配集非对称（即两端约束定义不一致）导致的运行时 400 校验失配风险。**

### 4.3 零越权同步机制与 Guided Sync
为了保障开发者对代码的 100% 控制权，`lunar` 命令行工具拒绝任何在后台静默修改人工维护文件的行为。
*   **`lunar init`**：**仅当检测到本地 `.lunar/interfaces.yml` 不存在时**，自动运行物理扫描，创建底稿文件。若文件已存在，此命令直接无效，不覆盖任何人工痕迹。
*   **`lunar diff`**：执行物理事实（AST）与意图覆盖层（`interfaces.yml`）的比对，在控制台输出标准的 Git-diff 风格变更报告。
*   **`lunar sync --apply`**：用户主动触发的同步合并命令，支持 `--dry-run` 预览更改。在执行实际合并写入前，强制对旧的 `interfaces.yml` 进行备份并写入本地隐藏备份目录 `.lunar/.backup/interfaces.yml.bak`（此路径自动由 `lunar init` 写入项目的 `.gitignore` 中）。
*   **AI 建议补丁机制**：`.lunar/suggestions/` 目录用于存放人类或 AI 生成的意图覆盖建议补丁（YAML 格式）。`lunar sync --apply` 在合并 `interfaces.yml` 时会自动检测并处理此目录下的补丁文件，处理后移入 `merged/` 子目录。

---

## 5. 项目级意图覆盖层与生态配置规范 (YAML/JSON Schemas)

为了确保多语言团队消费的统一性，所有的配置字段强制遵循 Google JSON Style Guide 标准驼峰格式。

### 5.1 `.lunar/interfaces.yml` 规范
项目级集中契约是开发者控制和定义接口边界的第一道关卡：
```yaml
# ===================================================================
# LunarAST Project Interface Contract
# This file is owned and maintained by humans.
# ===================================================================

project: myPaymentService
type: mixed # 角色定义：service / client / mixed
environment: production

# 手动声明的 API
exposed:
  - path: /api/v1/payments/refund
    method: POST
    reason: "退款专用接口（计划下周发布）"

# 复杂动态调用的手动契约覆盖
consumed:
  - path: /api/v1/auth/verify
    method: POST
    targetProject: authService  # 必须在生态 repos.json 中存在，否则 ldg doctor 报错
    reason: "用户交易前置会话校验"
```

### 5.2 生态注册清单：`repos.json`
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

### 5.3 生态集中式拓扑声明：`ecosystem-topology.json`
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
      "reason": "会话鉴权"
    }
  ]
}
```

### 5.4 生态构建计划锁文件：`ecosystem-plan.json`
该文件是整个生态世代构建的决策起点。它完全由生态级中心自动化工具管理与写入，并随每次 `generationId` 产生时进行硬性锁定。
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

## 6. 数据交换标准格式规范 (The Exchange Contract Spec)

作为整个生态对外输出的物理产品，数据必须遵循严格的静态格式定义，确保高密度与高解析效率。

### 6.1 结构化拓扑标准：`lunar-map.json`
以下为 `lunar-map.json` 的顶层 Schema 契约规范（全量遵循 Google驼峰式命名）：

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

### 6.2 AI 上下文地图标准：`lunar-map.md` (Markdown Spec)
在请求时，存储桶不支持静态存储 `lunar-map.md`，而是**由 `lunar-gateway` (或边缘 `lunar-serve`) 在接收到客户端请求时从 `lunar-map.json` 中实时解析、翻译并动态渲染输出**。支持通过以下查询参数实施参数化过滤以降低 AI 的 Token 消耗：
*   `GET /lunar-map.md?summary=true`（返回极简摘要，最省 Token，适合 AI 首次接入） [1.2]。
*   `GET /lunar-map.md?style=list`（返回纯文本契约列表，适合小上下文快速提取）。
*   `GET /lunar-map.md?style=mermaid`（返回拓扑图结构，适合全局图表渲染）。
*   `GET /lunar-map.md?scope=project-a`（通过分片获取，限定特定项目相关的子拓扑，防止超出 AI 上下文窗口限制）。

---

## 7. 开放集成与第三方桥接器隔离规范

LunarAST 保持高度的无状态与数据主权。任何外部执行沙箱（如 IDE 插件、AI Agent）在集成时，均必须遵循 **“桥接器隔离模式”** [2]：

*   **桥接器职责**：第三方集成方必须编写独立的桥接程序（如 `routeast-mcp-bridge`），通过标准 HTTP `GET` 接口拉取 `lunar-map.json`，并由桥接器在其内部转化为特定协议。
*   **不越权交互规范**：桥接器在将 AI 生成的对齐建议写入本地时，**严禁静默修改 `interfaces.yml`**。桥接器只负责在终端生成并输出 Git-diff 格式 of 对齐补丁片段，并提示用户手动运行本地的 `lunar sync --apply` [7]。

```rust
// 桥接器与 LunarAST 之间的最小交互契约
#[async_trait]
pub trait LunarMcpBridge {
    /// 1. 获取纯净的静态拓扑，支持分片限制 (?scope=projectX) 降低 Token 消耗
    async fn fetch_lunar_map(&self, gatewayUrl: &str, scope: Option<&str>) -> Result<LunarMapPayload, BridgeError>;
    
    /// 2. 翻译为目标协议的规范的 Tools 与 Resources 契约 (JSON-RPC)
    fn translate_to_mcp_tools(&self, payload: LunarMapPayload) -> Vec<McpToolSchema>;
}
```

---

## 8. 无状态分发层设计与安全模型 (`lunar-gateway`)

`lunar-gateway` 专注于高并发、无状态的极速契约数据安全分发 [2]：

### 8.1 存储目录结构标准
*   **推荐 S3 Bucket 命名格式**：`lunar-ast-<organization>`。
*   **全局拓扑存放定位**：**`lunar-map.json` 属于生态全局产物，严禁存放于单一服务的 commits 目录下**。网关在 `ecosystem-config/` 下按“构建代（Generations）”进行统一存储归档。
```
s3://lunar-ast-<organization>/
├── <repo>/
│   ├── commits/
│   │   └── <sha>/
│   │       └── <subdomain>-actual.json# 物理 facts 缓存（例如 route-ast-actual.json）
│   └── pointers/
│       └── latest.json                # 包含最新 SHA 的复合指针
└── ecosystem-config/
    ├── repos.json                     # 生态注册项目白名单
    ├── ecosystem-topology.json        # 编排与拓扑声明
    ├── latest-generation.json         # 指向当前最新活跃代信息的静态指针
    └── generations/
        └── <generationId>/
            ├── meta.json              # 该世代的元数据摘要
            ├── ecosystem-plan.json    # 该世代的构建计划锁文件
            └── lunar-map.json         # 该世代的全局对齐拓扑
```

#### 8.1.1 元数据文件：`meta.json` 规范体
在构建期，由 CI/CD 引擎生成并强制同步写入的 `meta.json` 文件必须严格遵循以下 Schema：
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
*   **属性说明**：`compilerVersion` 指代执行静态分析扫描并编译输出该数据的 `lunar` 编译器二进制版本，不与网关的版本绑定。当 `downgradeMap` 缺省、或不存在与请求版本兼容的降级转换子树时，网关拒绝有损降级。

#### 8.1.2 世代全局指针：`latest-generation.json` 规范体
```json
{
  "generationId": "20260608T030000Z-a1b2c3d4",
  "lastUpdated": "2026-06-08T03:00:00Z"
}
```

### 8.2 双阶段隔离缓存与客户端分级缓存控制
为了在“每次请求强制验签”与“零对象存储穿透”之间取得物理平衡，`lunar-gateway` 采用双阶段隔离缓存机制 [2]：
*   **内部缓存**：网关以对象存储中文件**相对于 Bucket 的完整逻辑路径作为 Cache Key**。
*   **安全验证优先原则**：对于需要 JWT 签名的私有资源（如私有项目的 actual.json），**网关必须首先无条件完成令牌完整性校验（包括签名与时钟偏差）[1.3.3]。只有在鉴权完全通过后，方可去匹配和读取内部缓存**。内部缓存 Key 依旧采用去 Token 化的物理逻辑路径以供多客户端共享，但验签鉴权步骤绝不可被任何缓存机制绕过。
*   **客户端响应差异化配置与全局拓扑缓存头**：
    *   **私有资源（需要 JWT，如私有项目的 actual.json）**：网关在返回时强制抹去所有边缘缓存头，改写为 `private, no-cache, no-store, must-revalidate`，防止令牌劫持。网关内部仍可通过去 Token 化的逻辑路径作为 Key 进行 `immutable` 边缘缓存匹配。
    *   **公开不可变资源（无需令牌，如 `/commits/<sha>/` 下资源及 `generations/` 下全局拓扑 `lunar-map.json`、`ecosystem-plan.json` 和 `meta.json`）**：网关向客户端返回的响应允许其进行长期本地缓存：`Cache-Control: public, max-age=31536000, immutable`，最大化边缘性能。
    *   **最新动态指针 `/pointers/latest.json` 及最新世代指针 `latest-generation.json`**：网关允许客户端短期本地缓存：`Cache-Control: public, max-age=300`，保证版本一致性更新。
*   **网关动态渲染缓存与 OOM 防御**：
    *   **动态渲染端点缓存**：网关对 `lunar-map.md` 的查询请求，在内存中以 `(Accept, scope, generationId)` 作为复合 Key，将 Markdown 的渲染结果缓存。其缓存生存期与对应的 `lunar-map.json` 版本保持强一致。
    *   **内存保护熔断**：基于 Cloudflare Workers 典型的 128MB 物理内存约束安全边界，对大体积文件（> 2MB），网关自动熔断“内存 Bytes 缓冲”，并基于 Cloudflare Workers 128MB 限制的安全边界，降级为流式直通（Stream-through）转发，牺牲一次 R2 回源以保护网关 Isolate 不发生崩溃。支持通过网关环境变量 `LUNAR_GATEWAY_BUFFER_LIMIT_BYTES` 进行热值覆盖。

### 8.3 安全验证与 JWT 签名契约
*   **签名算法规范**：强制采用 **Ed25519 (EdDSA)** 作为单一签名算法，保证恒定时间验证 [1.3.3]。
*   **过期时间策略**：JWT 令牌中的 `exp` 声明建议不超过 24 小时。
*   **公钥分发方式与 KV 缓存轮转策略**：公钥支持静态模式（Worker 环境变量）或动态模式（Cloudflare KV）。网关内存限制最大存储开销，通过设定 `MAX_KEYS_PER_REPO = 3`（每个逻辑项目最多保留最新及 2 个跨版本有效历史公钥指纹）防止恶意注册公钥耗尽网关内存。
*   **URL 参数令牌停用警告**：生产环境绝对不推荐默认支持在 URL 参数中传递 JWT 令牌。网关检测到 URL 传参时，在 `stderr` 中输出安全警告。

### 8.4 版本协商与向后兼容策略 (Version Negotiation)
当 `lunar-map.json` 的 `version` 进行主版本号变更时（如从 `0.5.0` 升级至 `1.0.0`），客户端通过 HTTP 媒体类型参数执行向后兼容的版本协商：
*   **协商路由规范与窗口**：
    客户端发起请求时，携带 Header `Accept: application/vnd.lunar.0.5.0+json`。**网关必须对当前活跃的主版本和紧邻的上一个主版本（即共两个主版本）保持数据降级兼容窗口。**
    *   **降级边界**：仅当新旧版本之间存在预定义的、无信息丢失的映射表（由 `meta.json` 里的 `downgradeMap` 描述）时，网关才执行自动降级；若主版本间发生必填字段变更、枚举值增删或语义不兼容，网关不得执行自动降级，必须返回 `406 Not Acceptable`，并附加 `X-Lunar-Upgrade-Required: true` 头部。

### 8.5 Crates 物理重组与 Workspace 依赖治理（Decoupled Workspace Spec）

为了防御生态系统膨胀带来的模块死锁与不必要编译负担，LunarAST 采用 Cargo Workspace 多 Crate 隔离治理规范：

1. **契约标准库物理分离（The Interface Crate）**：
   提取出零依赖的静态契约库 `lunar-interface`。该库仅保留 `RouteEntry`、`ActualJson`、`LunarMap` 等核心数据模型以及 `generate_lunar_map` 图对齐逻辑。
2. **CLI 与 Server 端依赖去耦（Decoupling CLI from Serving Layers）**：
   分发层（`lunar-serve`、`lunar-gateway`）仅依赖 `lunar-interface` 契约库，严禁直接或间接依赖包含 `clap`、`rust-s3`、`ed25519-dalek` 等 CLI 专属载荷的 `lunar` 单体二进制。从而最大化保证编译速度与服务端的轻量化运行。

---

## 9. 对齐状态优先级与诊断分类 (Diagnostic Short-Circuit)

对齐引擎在进行路径参数匹配时，对输入的服务端与客户端 `RouteAst` 列表执行单向短路（Short-circuit）评估。单个接口对仅能归属于一个最终状态，状态优先级为：
$$\text{MethodMismatch} > \text{Orphaned} > \text{Unused} > \text{ParamNameMismatch} > \text{Aligned}$$

*   **失败隔离与陈旧节点对齐机制（Unverified 状态隔离）**：
    为了避免因为生态中某些项目扫描失败（`scanStatus: failed`）或因 CI 构建不协调导致数据陈旧（`scanStatus: stale`）产生数据缺失，进而引发大规模虚假的对齐状态异常（如消费者被误判定为 `Orphaned`，生产者被误判定为 `Unused`）：
    1.  **对齐隔离提取**：对齐引擎必须在计算前检索 `projects` 数组中的 `scanStatus` [2]。
    2.  **`stale` 节点处理**：处于 `stale` 状态的项目所包含的 `exposed` 与 `consumed` 数据**仍会被对齐引擎读取并参与比对计算**。但其产生的所有最终对齐结果，**其 `status` 均被强制标注为 `"unverified"`** [2]。
    3.  **`failed` 节点处理**：由于 `failed` 项目无法拉取到有效的 `interfaces` 定义（为 `null`），无法进行任何实际比对。**对齐引擎必须遍历当前世代拓扑中所有其余健康项目。只要探测到有健康项目发起了对该 `failed` 服务的接口消费（`consumed`），对齐引擎不对其进行常规的 MethodMismatch 或 Orphaned 检查，而是为其直接生成一条 `status: "unverified"` 的对齐条目**。同时，由于 `failed` 项目无有效接口数据（其 `interfaces` 字段为 `null`），其自身暴露（Exposed）的接口不会在 `alignments` 数组中产生任何对齐条目，因此不会地被错误地判定为 `Unused`。
    4.  **未校验契约语义**：被标记为 `unverified` 的条目（优先级 5），在 `lunar-scope` 呈现时予以模糊黄线警示，在数据流和门禁审计中不判定为契约异常，仅代表数据源过时或未就绪。
*   **诊断判定原则（按严重级别单向短路）**：
    1.  **MethodMismatch（优先级 1）**：只要路径结构完全一致，但客户端使用了 `POST`，服务端仅暴露了 `GET`。对齐引擎立即抛出 MethodMismatch 并停止后续检查。
    2.  **Orphaned（优先级 2）**：针对特定目标服务发起了调用消费，但生态拓扑 `repos.json` 注册的所有服务中，找不到任何满足路径基和方法契约的节点（断点）。
    3.  **Unused（优先级 3）**：**（拓扑级全局计算状态）**。服务端暴露了接口，但在整个生态拓扑中没有任何客户端调用它。此状态由 `lunar` 在生成 `lunar-map.json` 时的**全局后处理（Post-processing）阶段**中，自左向右整个生态扫描，遍历所有项目的 `exposed` 与全量 `consumed` 交叉对比后进行精准标注，对清理幽灵契约、收窄攻击面具有极高的架构治理价值 [2]。
    4.  **ParamNameMismatch（优先级 4）**：参数位置与类型对齐，但命名不一致。触发前端磁吸虚线 [2]。

---

## 10. 可观测性与健康检查规范

### 10.1 结构化日志规范（JSON Lines）
`lunar-gateway` 与本地只读服务必须向 `stdout` 输出单行换行分隔的标准 JSON 结构化日志，其格式全量采用 `camelCase` 并包含以下基本属性：
```json
{"timestamp":"2026-06-12T01:00:00Z","level":"INFO","method":"GET","path":"/public/repo-a/commits/sha-123/lunar-map.json","status":200,"durationMs":12,"cache":"HIT","authStatus":"valid","clientIp":"12.34.56.78"}
```
*   **`authStatus` 状态机定义**：仅允许取值：`valid`（合规通过）、`expired`（令牌过期）、`invalidSignature`（签名未通过）、`missingToken`（未带令牌）。

### 10.2 指标暴露（Prometheus Metrics）
网关必须在内存中统计并暴露符合 Prometheus 标准的静态 `/metrics` 接口。**为了保证在主流指标监控系统（Prometheus 规范）中的生态对齐，网关在向外部暴露这些指标时，必须由实现层自动将属性中的 camelCase 标签名称转换为符合 Prometheus 最佳实践的下划线分隔命名格式（snake_case，如 `targetVersion` $\rightarrow$ `target_version`）**：
*   `lunar_gateway_requests_total`（请求总计数器，标签：`method`, `status`）。
*   `lunar_gateway_cache_hits_total`（缓存命中计数器，标签：`cacheType` (internal/client)）。
*   `lunar_gateway_auth_failures_total`（鉴权失败计数器，标签：`reason` (expired/signatureInvalid/missingToken)）。
*   `lunar_gateway_version_downgrade_requests_total`（版本向后兼容降级请求计数器，标签：`targetVersion`）。用于在过渡期监控和评估因客户端工具升级滞后产生的数据翻译有损降级流量。

### 10.3 活性健康检查
网关暴露极速不经过验签鉴权的只读探针端点：`GET /healthz`，成功直接返回 `200 OK`，用于容器及边缘运行时的活性探测。

---

## 11. 安全威胁模型与防御策略 (Security Threat Model)

为了保障 LunarAST 在零信任环境下的稳定性，建立如下四道物理防线：

1.  **注入攻击防护与词法规则**：
    由于魔法注释（`// lunar:consume`）由适配器词法扫描，可能存在恶意代码注入非白名单注释的情况。**适配器应针对目标语言的物理注释语法采用对应的词法过滤规则与 AST 节点联合判定。扫描器在工作阶段执行严格的物理词法拦截，仅提取符合预设规则（如 Python 使用 `#\s*lunar:(expose|consume)` 正则，HTML 模版采用 `<!--\s*lunar:(expose|consume)\s*-->` 等）的注释行。所有不匹配预设规则或格式极不合规的畸形注释必须向 `stderr` 输出 store 路径、不兼容警告（Warning）和行号。所有此类不合规行强行忽略，不传入后端 AST 树，阻断注入风险。**
2.  **密钥轮换溢出防御**：
    网关在内存中对公钥进行软 TTL 缓存。内存软 TTL 缓存强制限制最大存储开销，通过设定 `MAX_KEYS_PER_REPO = 3`（每个逻辑项目最多保留最新及 2 个跨版本有效历史公钥指纹）防止恶意注册大量废弃公钥耗尽网关内存。
3.  **S3 凭证最小化**：
    CI Action 所持有的 S3 凭证仅具备对应 Bucket 路径下的 `s3:PutObject` and `s3:GetObject` 权限，不具备删除桶或修改桶 IAM 策略的越权能力。
4.  **销毁性动作防误触屏障与 CLI 权限划分**：
    所有本地 CLI 的销毁或生态全局清理动作（如 `lunar cleanup`），强制要求在终端进行 **二次交互式阻断确认**。只有在 CLI 中显式追加非交互式覆盖标志（如 `--yes`）时，才允许跳过交互提示，确保 CI/CD 自动化集成的顺利执行。通过此机制隔离破坏性指令，彻底阻断越权误触风险 [2]。

---

## 12. 核心错误码与调试指南 (Error Codes)

| 错误码 | HTTP 状态码 | 物理成因 | 推荐恢复方案 |
|:---|:---|:---|:---|
| `ERR_LUNAR_CONFIRM_FAIL` | 400 | 阶段二归一化校验不通过，通配符格式不合规。 | 运行 `lunar diff`，检查提示的语法不合规段。 |
| `ERR_LUNAR_ADAPTER_CRASH` | 422 | 阶段一子进程提取器崩溃（如 Node 解析器语法错误），或者流式读取的实际条目总数与结束标记行中的 `count` 校验值失配（数据发生截断）。 | 检查 CI 日志中对应适配器的 stderr 堆栈信息。 |
| `ERR_LUNAR_PROJECT_NOT_FOUND` | 404 | 目标项目未在生态注册清单 `repos.json` 中定义。 | 将目标项目添加至生态 `repos.json` 清单并重新触发构建。 |
| `ERR_LUNAR_INTERFACE_NOT_FOUND` | 422 | 目标项目存在，但未暴露符合客户端消费的方法 or 路径。 | 运行 `lunar diff`，检查客户端消费与服务端暴露的差异，修改 interfaces.yml 并进行同步。 |
| `ERR_LUNAR_INTERFACE_DATA_MISSING` | 410 | 由于对齐构建期项目扫描失败或未就绪（scanStatus: failed），且 `interfaces` 字段为 `null`，网关在试图为消费者提供反向推导数据源时彻底缺失，无法降级恢复。 | 运行 `lunar doctor`，排查构建失败的异构子服务。 |
| `ERR_VERSION_EXPIRED` | 410 | 物理事实缓存已在对象存储中超过 90 天被自动清除。 | 终端提示 `Hanging Pointer`。网关在返回该错误时，响应头部必须附带 `X-Lunar-Recovery: Trigger CI pipeline for <repo>` 及预配置的 CI 自动化 Trigger Webhook 路径。**指引并鼓励开发者在对应仓库重新触发 CI 构建。** |
| `ERR_GATEWAY_STREAM_FALLBACK_FAILED` | 502 | 契约文件体积超限（>2MB），降级流式转发时回源失败。 | 检查 `lunar-map.json` 是否混入了非必要的前端静态资源，并检查存储源站连通性。 |

---

## 13. 路线图与核心演进里程碑

整个生态的演进放弃具体时间线的硬性绑定，采用具有明确阶段性交付件的里程碑模型推进：

*   **Milestone 1 (RouteAST 基础实现)**：
    *   冻结 `RouteAST Base IR v0.5.0` 契约规范。
    *   实现纯 Rust 编写的确认内核（Confirm Core）原型。
    *   提供 Rust (Axum) 和 Node (Express) 轻量级适配器。
*   **Milestone 2 (完整四层架构与安全分发)**：
    *   发布用户侧工具 `lunar` 命令行工具，实现 `init`, `scan`, `diff`, `sync --apply` 核心指令。
    *   部署无状态边缘网关 `lunar-gateway`（支持双阶段缓存、分级缓存控制、边缘内存保护与熔断、可观测性日志及 Prometheus 监控）。
    *   **【重构升级达成】**：实现多 Crates 模块化 Workspace 架构解耦，抽离轻量化 `lunar-interface` 核心模型，使 `lunar-serve` 与命令行应用依赖彻底解绑，完成大小写自适应和本地工作区路径自动探测两级寻路回落 [1.2]。
*   **Milestone 3 (多维呈现与事实标准建立)**：
    *   发布前端基于 `MatchResult` 状态优先级机制的 **lunar-scope 智能物理磁吸画布**，支持 `Unused` 和 `unverified` 接口状态可视化 [2]。
    *   冻结 `EventAST` 与 `SchemaAST` 规范标准，发布官方事件/数据适配器。
*   **Milestone 4 (全静态代码级深度审计与开放生态体系)**：
    *   发布 `TypeAST` 契约标准与跨项目代码编译级依赖审计。
    *   向外部生态及安全审计平台提供标准的 `lunar-map.json`，完成全栈静态接口对齐与漂移防御。

---

## 附录 B：参考文献与规范出处

*   RFC 6570 - URI Template Specification for parameter standardization.
*   IEEE Std 1471-2000 - Systems and software engineering - Recommended practice for architectural description of software-intensive systems.
*   JSON Lines Standard (v1.0) - Line-Delimited JSON streaming format.
*   WASI 0.2 (Component Model) - WebAssembly System Interface specification.
*   ACM TOSEM Vol. 33 - Cross-Language Static Program Analysis on Microservice Topologies.
*   [1.3.3] NIST FIPS 186-5 - Digital Signature Standard (DSS) guidelines for Ed25519 system signature integration and curve verification.

---

### A.5 CLI 快捷命令参考一览 (Cheat Sheet)

```bash
lunar init                     # 自动探测技术栈，引导初始化本地底稿（ interfaces.yml 不存在时生效）
lunar scan                     # 静态扫描当前项目物理 facts 并写入 .interfaces-autogen.json 缓存
lunar diff                     # 打印物理事实与人工 interfaces.yml 意图覆盖层的标准 Git-diff 报告
lunar sync --dry-run           # 预览对齐变更的同步路径
lunar sync --apply             # 在自动物理备份旧文件后，将实际代码变更合并写入
lunar doctor                   # 校验项目 S3 连通性、最新指针状态与拓扑一致性
lunar cleanup --all            # 交互式引导清除当前项目的所有 S3/R2 数据（删除 commits/ 和 pointers/ 下该项目的所有对象，不影响 ecosystem-config/ 中的全局配置）。若需跳过交互，可追加 `--yes` 标志（风险高，需谨慎操作）
