# LunarAST Ecosystem

**通用多层静态契约协议家族与系统架构呈现看板**  
*Universal Multi-Layer Static Contract Protocol Family & Architectural Representation Canvas*

---

## 1. 生态愿景

在分布式与微服务架构的持续演进中，接口层面的语义漂移（API Drift）是导致系统崩溃的主要诱因。**LunarAST** 提供了一套**零信任、按需加载、版本不可变**的代码上下文分发与契约校验系统。我们不发光（不引入运行时监控与性能损耗），只反射构建期的物理事实，在代码部署前捕捉悬空的断点与未规划的野生依赖。

- **母协议白皮书（Ecosystem Whitepaper）**：[查看详细规范及设计哲学 (docs/ecosystem-whitepaper-v1.0.zh-CN.md)](docs/ecosystem-whitepaper-v1.0.zh-CN.md)

---

## 2. 物理角色蓝图：生、传、标、展

整个生态在物理边界上彻底重构为标准的四个解耦维度，各组件职责单一，物理隔离：

```
                              ┌────────────────────────────────┐
                              │           LunarAST             │  ← 1. 标准规范层 (标)
                              └──────────────┬─────────────────┘
                                             │
                                             ▼
                              ┌────────────────────────────────┐
                              │             lunar              │  ← 2. 数据生成层 (生)
                              └──────────────┬─────────────────┘
                                             │
                                             ▼
                              ┌────────────────────────────────┐
                              │         lunar-gateway          │  ← 3. 无状态分发层 (传)
                              └──────────────┬─────────────────┘
                                             │
                                             ▼
                              ┌────────────────────────────────┐
                              │          lunar-scope           │  ← 4. 可视化呈现层 (展)
                              └────────────────────────────────┘
```

### 📂 核心代码库与规范目录 (Repositories)

*   **标 (Standard Spec)** | [`LunarAST/RouteAST`](https://github.com/LunarAST/RouteAST) : 同步网络与路由契约（RouteAST）子协议规范，这是生态首个可执行的接口对齐标准。
*   **生 (Generator CLI)** | [`LunarAST/lunar`](https://github.com/LunarAST/lunar) : 统一的命令行可执行工具（Rust 编写），执行初始化、AST扫描、无越权 diff 诊断与 Guided Sync 同步。
    *   *Rust 语言适配器* | [`LunarAST/lunar-extract-rust`](https://github.com/LunarAST/lunar-extract-rust) : 基于 `syn::visit` 的 AST 深度解析器，用于对 Axum 路由进行无死角静态提取。
*   **传 (Distributor)** | [`LunarAST/lunar-gateway`](https://github.com/LunarAST/lunar-gateway) : 编译为 `wasm32-wasip2` 的 Serverless 边缘网关，执行 JWT 强验证、分级缓存控制与 2MB 内存保护熔断。
*   **展 (Visualizer)** | [`LunarAST/lunar-scope`](https://github.com/LunarAST/lunar-scope) : 纯静态的前端关系画布（React + xyflow + elkjs），渲染多图层拓扑、诊断红色警报线与黄色的“悬空线缆物理配线”。

---

## 3. 三级递进真理源模型

在静态治理中，LunarAST 坚守“物理 facts 优先”与“零越权”的底线，通过以下三级递进信息源模型打通此闭环：

1.  **物理事实 (Physical Facts)**：代码的客观事实（扫描自动推导并写入只读缓存 `.lunar/.interfaces-autogen.json`，已 `.gitignore`）。
2.  **意图覆盖层 (Intent Overlay)**：项目根目录下的 **`.lunar/interfaces.yml`**（人类 100% 掌控，Git 版本控制，工具严禁任何未授权的静默修改）。
3.  **末端逃生舱 (Escape Hatch)**：源码行上方的 `// lunar:consume` 魔法单行指令。仅作为最后底线，用于规避静态分析极限的非追踪动态调用。

---

## 4. 快速开始与命令一览 (SOP)

在本地开发机安装 `lunar` CLI 工具及对应的适配器（以 Rust 为例）：

```bash
cargo install lunar
cargo install lunar-extract-rust
```

进入项目根目录：

```bash
lunar init                     # 自动探测技术栈，引导初始化本地底稿（interfaces.yml 不存在时生效）
lunar scan                     # 静态扫描当前项目物理 facts 并写入 .interfaces-autogen.json 缓存
lunar diff                     # 打印物理事实与人工 interfaces.yml 意图覆盖层的标准 Git-diff 报告
lunar sync --apply             # 在自动物理备份旧文件后，将实际代码变更合并写入
lunar doctor                   # 校验项目 S3 连通性、最新指针状态与拓扑一致性
```

---

## 5. 许可证与社区安全治理

LunarAST 协议家族及组件全量遵循 **Apache-2.0** 许可证。
*   **贡献指南**：请阅读 [CONTRIBUTING.md](CONTRIBUTING.md) 以了解如何为协议家族补充适配器或提交 RFC。
*   **漏洞安全披露**：请参阅 [SECURITY.md](SECURITY.md)，任何涉及令牌校验或网关越权安全漏洞，请通过私密渠道上报。

---

*“如无必要，勿增实体。让复杂的代码推导退幕，让确定性的 LunarAST 契约对齐成为事实标准。”*
