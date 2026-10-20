# HERMES-AGENT-JZIN-BK 工作区 · AI 编码智能体系统提示词

> **文档用途**  
> 本文件是工作区 **HERMES-AGENT-JZIN-BK** 的最高架构约束。实现、重构、接入新 Runtime 前必须先读本文与架构图；冲突时以本文 + 架构图为准，不以第三方框架默认部署方式为准。
>
> **配套持续更新文件（必须同步维护）**
> 1. 本文件：工作区 `jzin-bk/all-project-doc/ARCHITECTURE.agents.md` ≡ 本仓 `ARCHITECTURE.agents.md`
> 2. 技术架构图：工作区 `jzin-bk/all-project-doc/AI 平台总体技术架构.html` ≡ 本仓 `index.html`  
>    在线：https://jzin-v2.github.io/ai-platform-architecture/
> 3. 功能/原型（本仓保留）：`functional.html`、`prototype/`
> 4. 详细规范（可选）：工作区 `all-project-doc/(旧)AI平台总体架构与服务融合规范方案.md`
>
> 仓库：https://github.com/jzin-v2/ai-platform-architecture  
> 变更架构时：**工作区文档与本仓同步推送**，并保持分期（一期/二期/三期）一致。

---

## 0. 工作区约定

### 0.1 多根目录语义

```text
workspace/ HERMES-AGENT-JZIN-BK
├── jzin-bk/               # = GFast 业务底座 + AI 平台控制面（平台主权 SoT）
│   ├── internal/app/      # GoFrame 后端模块（exam、kb、llm、mcp、ai_* 等）
│   ├── frontend/          # Vue 3 统一门户（与后端平级，别名 @/）
│   └── all-project-doc/   # 架构文档（本文件所在目录）
└── hermes-agent/          # 一期 Agent Runtime（外部现成 Harness，可独立升级）
```

**命名映射：** 文档中的 `gfast/` = 本仓库 `jzin-bk/`。

### 0.2 编码智能体默认假设

- **业务与平台代码主要写在 `jzin-bk/`**
- **`hermes-agent/` 保持可独立升级的上游项目**；仅通过 Adapter / 配置 / 容器调用
- 不要把 Hermes / 其它 Runtime 源码大面积拷贝进 jzin-bk 并深度魔改
- 前端统一门户在 **frontend**，Hermes Desktop/Dashboard **不作为多租户门户**

### 0.3 开工前必读（强制）

每次接到架构相关任务，**先读**：

1. `all-project-doc/ARCHITECTURE.agents.md`（本文件）
2. `all-project-doc/AI 平台总体技术架构.html`（或同目录 `.md` 摘要）
3. 若涉及长期融合 / 多 Provider / Workflow DSL：`(旧)AI平台总体架构与服务融合规范方案.md`

**不读架构文档不得开始写代码。**

---

## 1. 目标与原则

### 1.1 目标

- 多租户 AI 业务平台：统一门户、统一身份权限、统一资产与审计
- Agent 能力用**持续更新的现成框架**（先 Hermes，后 OpenCode / QwenPaw / Eino 等），少自研 Runtime
- 一人可维护：优先 **GFast 模块**，早期**不拆微服务丛林**

### 1.2 硬原则（不可违反）

| # | 原则 | 说明 |
|---|------|------|
| 1 | **平台主权（SoT）在 jzin-bk** | 租户、用户、RBAC、资产广场、模型路由、Run/审计、配额。配额仍未做 |
| 2 | **Runtime 可插拔** | Hermes / OpenCode / QwenPaw / Eino / Dify Legacy 均为外部 Runtime Plugin |
| 3 | **单向投影** | 平台 → Runtime 下发配置；禁止用户/角色双向全量同步与影子账号泛滥 |
| 4 | **MCP 只登记和映射** | GFast 统一保存 MCP 并按 Runtime 下发。不拦截 Hermes 或其他 Runtime 对外部 MCP 的调用；各 Runtime 用自己的方式连接。禁止 Runtime 直连业务库 |
| 5 | **不透传主 JWT** | 目标凭证是短期、受众绑定的委派 Token / Service Account。当前只在未使用的 embed 会话接口上签发，对话主路径不携带 |
| 6 | **危险动作在沙箱** | 终端/文件/浏览器等不在 GFast 主进程直接 exec |
| 7 | **资产版本不可变** | 已发布版本不可热改；每次 Run 绑定具体 asset 版本 |
| 8 | **不魔改第三方核心** | WeKnora、Dify、Hermes 等禁止承载平台 RBAC/租户/资产主数据 |

---

## 2. 分期实施（默认只做一期）

### 一期（当前焦点）— Hermes 最小闭环

在 `jzin-bk/` 实现：

- 登录 / RBAC / 多租户（复用 GFast）
- 资源/资产广场（MCP、Skills、插件、模型元数据）
- Agent 定义与工作台（选择 Agent，查看 Run）。主路径无审批；`risk_level` 仅预留
- Run 中心 + 审计字段
- **Projection 单模块**：模型路由、MCP、技能包、工具集名称 → Hermes。委派身份未做
- **Hermes Adapter**：`start` / `stream` / `cancel` / 事件归一成平台 Run Event
- **MCP**：GFast 做增删改查和映射，不做 Hermes 与外部 MCP 之间的治理门
- **任务表异步调度**（不上 Temporal）
- Hermes 以 **Docker/容器** 运行；业务 H5/小程序**不动**

#### 一期第一步（当前代码范围）

只做 GFast 壳上的资源广场、Agent 定义、目录投影和薄对话，不把 Hermes Dashboard 当多租户后台：

- **公司边界**是一级部门（`parent_id = 0`），不新建租户表。投影目录、编码唯一、跨资源绑定按该公司的部门子树。身份上下文里的 `tenant_id` 映射为这个一级部门 id。
- **数据权限**用 GFast `GetAuthDeptWhere`（部门数据权限，不随创建人调岗迁移）。新建行的 `dept_id` 是登录人当时的部门，`created_by` 是创建人。我的列表、详情、修改、删除，以及发布、下架、新版本、测连通，都带这个条件。超管不限制；角色没配数据范围时兜底为仅本人。数据范围选「全部」的角色可以看到各公司数据。广场只展示本公司一级部门子树里已发布的记录，不套个人数据范围。
- **Agent 定义**在 `internal/app/ai_agent`，门户页面在 `frontend/src/views/ai/agent`。一条 Agent 绑定已发布的 MCP 和技能，再交给 Projection。插件选择器占位，不能勾选。
- **Agent 对话**是 GFast 自己的对话页：选择已启用的 Agent，先把已发布资产写成配置模板，再铺进该用户这个 Agent 的运行目录（留下 Hermes 的 `state.db`、记忆和附件），由适配器拉起该运行目录上的本机 `hermes serve`，用 tui_gateway JSON-RPC 续聊。发送前把上传的文件和文件夹复制到运行目录的 `attachments/chat-<会话>/`，供 Docker 只读挂载到 `/root/.hermes/attachments`；图片仍走 `images`。流式正文、思考、工具事件，以及 `clarify` 追问，经 GFast 已有 WebSocket 推到页面。追问由门户回答后写回同一轮，适配器不把 `clarify` 答成空字符串。生成文件的预览和下载也走 GFast：只读该用户运行目录里的允许类型，浏览器不读本机磁盘。平台记录会话和 Run，会话只对创建人可见。不把用户主 JWT 交给 Hermes，浏览器也不直连 serve。Runtime 注册里的 `embed_url` 仍保留，本步不使用。短期委派 Token 只在这个未使用的 embed 会话接口上签发，不交给 `hermes serve`。对话链路也不携带第 6 节的身份上下文。配额未做。
- **资源广场**目录下是 MCP、技能、插件三个菜单，页面在 `frontend/src/views/ai/plaza/`。MCP 和技能都有「我的」和「广场」。连接方式、地址和请求头是各 Runtime 共用的；工具过滤、超时、证书、OAuth、Stdio 参数和环境变量标成 Hermes 独有，只在投影到 Hermes 时写入 `mcp_servers`。技能用 zip 登记，包内只能有一个 `SKILL.md`，投影时整包拷进该 Agent 的 `skills/{编码}/`。GFast 不运行技能，也不配置 docker 或 local。插件只占位。REST 可登记，不写入 Hermes。工具集 `toolset` 可在后端登记名称列表，投影时写入该目录 `config.yaml` 的 `tools.cli.enabled`；门户没有工具集菜单，Agent 表单也不绑定它。
- **本机 Desktop 技能广场**不替代上面的投影。`jzin-bk/hermes-ext/gfast-bridge` 是发给个人 Hermes Desktop 的统一插件，不进 Hermes 核心，也不做多租户门户。用户用 GFast 账号登录后浏览本公司已发布技能。别人的技能先订阅再下载；自己发布的技能不订阅，本机没有同名目录才下载，已有则不覆盖。包从 `GET /api/v1/aiAsset/asset/skillPackage` 取得，只写入当前 profile 的 `skills/{编码}/`，并留下 `.jzin-subscribed`。本机自建技能可上传为草稿再发布。门户对话仍只读投影进该用户 Agent 运行目录的技能，不读这份个人 `HERMES_HOME`。插件完整包放在 `%LOCALAPPDATA%\hermes\plugins/gfast-bridge`（或对方机器上的同一相对位置）；`desktop-plugins/gfast-bridge` 只是桌面启动时复制出的 `plugin.js`，不能单独分发。
- **Projection**把已绑定的已发布资产写成独立目录，不使用个人 Hermes 家目录。配置模板在 `{homeRoot}/{公司一级部门}/{agentCode}/`，只放 `config.yaml`、`.env`、技能和插件，忽略个人的启用开关。真正的 Hermes 家目录是 `{homeRoot}/{公司一级部门}/users/{userId}/agents/{agentCode}/`：每次对话前从模板铺配置，并套上该用户关掉的资产，留下 `memories/`、`state.db`、工作区和附件。旧的共享目录不自动拆进某个用户。
- **记忆**在平台上一个用户一行主体（`ai_memory_subject`），正文不入库。`USER.md` 只在该 Agent 运行目录里还不存在时，用姓名、账号、部门种一次。`MEMORY.md` 由 Hermes 自己写。门户「我的记忆」只读写当前用户这两份文件，下一场新会话才会注入。会话列表只看本人。
- **Runtime 注册**保存 API Base URL、health、embed_url 与启用状态。
- **模型目录**可登记对话、Embedding、ReRank、视觉和语音。接入项是服务商、模型名称、显示名称、Base URL、API Key 和自定义请求头；高级项按类型保存上下文窗口、思考参数格式、向量维度和并发上限。服务商含 25 家固定厂商和自定义 OpenAI 兼容接口；选中厂商后按当前模型类型填入默认 Base URL，并预选思考参数格式。占位地址不能直接保存。对话和视觉未填上下文窗口时保存为 200000。部署配置 `ai.builtinModels` 在进程启动时同步为内置模型：`builtin=1`、`dept_id=0`，不走部门数据权限，能打开模型列表的登录用户都能看到和使用。门户不能修改或删除；配置里去掉并重启后，对应行软删除。复制会另存为本公司模型，且不带原密钥。新建和编辑时可测试连接：按类型向 OpenAI 兼容接口发一次最小请求，不写库；Embedding 测通后回填向量维度。已启用的对话模型会投影成该 Agent 目录里的 Hermes 具名提供商，密钥只写入该目录的 `.env`。对话页可以在输入框右侧选择这些模型，并选择推理强度，默认是中档。选择记在会话上，不改投影里的默认模型。只接受 low、high、max 的接口会把中档写成 high。Embedding、ReRank、视觉和语音仍只留在目录里。不再复制本机 Hermes 的模型路由和个人密钥。
- **Run 中心**只做列表/详情骨架。主路径无人工审批；`risk_level` 仅预留展示。
- 部署分两种。`ai.runtime.placement=local`（本机开发）仍由 GFast 进程内拉起 `hermes serve`。`placement=remote`（Kubernetes）时，GFast 是无状态 Deployment，可以多副本；Hermes 跑在 StatefulSet `hermes-runtime` 上，多个用户按槽位固定在同一个 Pod，每个运行目录只有一个写者。Pod 崩溃后用同一序号挂上原来的盘再起，已写入 `state.db` 的会话可以继续，进行中的那一轮要重发。200 人同时在线大约 8～10 个 Runtime Pod。技能包目录用只读共享卷，Hermes 家目录不放在共享卷上。
- 任务表调度、按公司上 Docker 沙箱、Temporal 仍未做。Hermes 与外部 MCP 之间不设治理门。对话这一步的 Adapter 负责 `hermes serve` 的 start / stream / cancel，并把 `clarify` 交回门户回答。

### 二期 — 编排与隔离强化

- 引入 **Temporal**（长任务、暂停恢复、人工审批）
- 按需 **NATS JetStream**
- 每租户 / 每 Run 强化 Hermes 进程与目录隔离
- 观测（成本、告警）；可选企业 SSO（Keycloak 等）前置

### 三期 — 多 Runtime 与自动化

- 复制 Adapter 模式接入 OpenCode、QwenPaw、Eino 等
- A2A Gateway
- 独立 Browser/Code Worker；E2B 等沙箱
- Dify 仅 Legacy；UE 等专项 Worker 按需

**智能体规则：** 未明确要求二期/三期时，**只提交一期所需代码与配置**。

---

## 3. 九层架构与代码落点

| 层 | 职责 | 代码落点 | 期次 |
|---|---|---|---|
| ① 展示 | 统一门户、Agent 工作台 | `jzin-bk/frontend` | 一期 |
| ② 身份 | 登录、RBAC、多租户、换票 | `jzin-bk` IAM + Nginx/Caddy | 一期 |
| ③ Core | 资产、模型、Run、审计、业务 MCP Provider | `jzin-bk/internal/app/ai_*` 模块包 | 一期 |
| ③′ Projection | 配置单向投影到 Runtime | `jzin-bk` **一个** Projection 模块 | 一期→Hermes |
| ④ 编排 | 任务发起 / 长流程 | 一期任务表；二期 Temporal/NATS | 一→二 |
| ⑤ Runtime | Agent 循环 | `hermes-agent` **外部进程** | 一期 Hermes |
| ⑥ Gateway | 工具调用拦截 | 不做。各 Runtime 自己调用 MCP | 不在本期 |
| ⑦ Adapter | 协议翻译、事件归一 | `jzin-bk` Hermes Adapter | 一期 |
| ⑧ 沙箱 | 副作用执行边界 | Docker + Hermes 容器 | 一期 |
| ⑨ Providers | 元数据 vs 真实能力 | 广场在 jzin-bk；WeKnora/模型/外部 MCP 在外部 | 一期 |

### 3.1 一期建议模块边界（jzin-bk）

```text
jzin-bk/internal/app/
├── ai_asset/          # MCP、技能、工具集登记。技能包与工具集名称可投影。插件占位
├── ai_agent/          # Agent 定义。绑定已发布 MCP 与技能；插件选择器占位
├── ai_model/          # 模型目录。对话模型投影到 Hermes；其他类型仍只在目录
├── ai_runtime/        # Runtime 注册、切换、工作台 API
├── ai_projection/     # 单向投影（先 Hermes）。模板按 Agent，运行目录按用户
├── ai_memory/         # 用户记忆主体。正文在 Hermes 运行目录
├── ai_adapter_hermes/ # Hermes Adapter
├── ai_run/            # Run、事件、产物元数据、审计
└── ai_job/            # 一期任务表调度（目录尚未建立）
```

**已有模块复用：** `llm/`、`mcp/`、`kb/` 等可作为 Provider 或过渡期能力，新 AI 平台能力优先按上表分包，避免概念混叠。

### 3.2 hermes-agent 边界

- 以官方方式容器部署
- 通过环境变量 / 配置接收 Projection 结果（模型 BaseURL/Key 引用、工作目录、MCP 白名单等）
- **允许：** Dockerfile、部署配置、薄封装脚本、Projection 消费端配置
- **禁止：** 把平台 RBAC、租户表、资产主数据搬进 Hermes；深度魔改 agent 核心以承载平台业务

---

## 4. 五个概念不得混淆（实现前自检）

| 概念 | 是什么 | 不是什么 |
|---|---|---|
| **资源广场** | 登记 MCP 和技能（门户已落地）、工具集（后端可登记并投影，门户无菜单）、插件（占位）。技能只投影目录 | 运行时调用 Hermes 的地方 |
| **Projection** | 事先把配置推给 Runtime | 双向主数据同步 |
| **Adapter** | Run 时呼叫 Runtime（start/stream/cancel） | 资源广场，也不是 Projection |
| **Gateway** | 本期不做。GFast 不拦截 Runtime 的 MCP 调用 | 各 Runtime 自己的 MCP 客户端 |
| **Worker/沙箱** | 终端/文件/浏览器实际执行处 | 一期不要在 GFast 主进程里 exec |

---

## 5. 一期推荐调用链

```text
用户 → frontend / GFast 门户
    → GFast 鉴权（RBAC/租户）
    → 创建 Run（绑定 asset 版本）
    → 任务表异步执行
    → Hermes Adapter
    → Docker 中 Hermes 实例
    → Hermes 按投影的 mcp_servers 自己连接外部 MCP
    → 事件/产物回写 Run 中心
```

当前对话还没走到任务表和 Docker 沙箱。已落地的链路是：门户登录 → 部门数据权限 → 配置模板 → 铺进该用户这个 Agent 的运行目录 → 适配器拉起该目录的 `hermes serve` → tui_gateway JSON-RPC → 事件回到 Run 和会话。本机模式下 serve 在 GFast 进程内；Kubernetes 模式下 GFast 把这一轮交给槽位对应的 Runtime Pod，流式事件经 Redis 频道 `ai:chat` 回到持有该用户 WebSocket 的 GFast Pod。上传的文件和文件夹在发送前复制到运行目录的 `attachments/`，供沙箱只读读取。`clarify` 由门户写回同一轮。生成文件的预览和下载走 GFast：local 读本机运行目录，remote 向持有该盘的 Pod 取回字节。浏览器只连 GFast。配置模板按 Agent 共用；`HERMES_HOME`、`state.db` 和记忆按（公司，用户，Agent）分开。门户会话只列出创建人本人的记录。

### 5.2 Kubernetes 部署形状

```text
浏览器 → GFast Service（Deployment，无状态，副本可增减）
    → MySQL：账号、资产、会话、Run、ai_runtime_slot
    → 按槽位把 turn / answer / interrupt / 文件 / 记忆交给
       hermes-runtime-{序号}.hermes-runtime
    → 该 Pod 上的 hermes serve（一块 RWO 盘，多个运行目录）
    → 流式事件发布到 Redis ai:chat
    → 任意 GFast Pod 推给连在本进程上的浏览器
```

槽位表一行对应一个（公司，用户，Agent）运行目录。第一次按 `userId % replicas` 定序号，之后以表里的序号为准，扩容不搬已有目录。只有序号相同的 Pod 能续约。同一目录不会被两个 Pod 同时写。副本数由 StatefulSet 手工设置，并与配置 `ai.runtime.replicas` 保持一致。

### 5.1 禁止调用链

```text
前端 → Hermes Dashboard/API 直连（多租户门户）
Hermes → 直连业务数据库
Hermes → 持有用户主 JWT / 管理员永久 Token
多用户 → 共用同一个 HERMES_HOME / state.db
GFast → 拦截并代替 Runtime 调用外部 MCP
已发布资产 → 无版本热改
```

---

## 6. 统一身份上下文（目标契约）

目标是每一次 AI 调用、Tool 调用、Worker Job 都携带：

```json
{
  "tenant_id": "tenant_xxx",
  "workspace_id": "workspace_xxx",
  "actor_type": "user",
  "actor_id": "user_xxx",
  "run_id": "run_xxx",
  "delegation_chain": [
    "user:user_xxx",
    "agent:xxx@1.0.0",
    "tool:gfast.xxx.create@1.0.0"
  ]
}
```

这是目标契约。当前对话链路还没有携带这份 JSON。短期委派 Token 只在 Runtime 的 embed 会话接口上签发；对话主路径不使用 `embed_url`，也不把该 Token 交给 `hermes serve`。

---

## 7. Tool 风险等级（预留，本期不实现治理门）

| 等级 | 类型 | 示例 | 默认策略 |
|---|---|---|---|
| L0 | 只读 | 查询、读取、检索 | 可自动执行 |
| L1 | 可逆写入 | 创建草稿、待审核数据 | 可自动或抽样审批 |
| L2 | 业务提交 | 发布、发送通知、正式记录 | 必须人工确认 |
| L3 | 高风险 | 删除、改权限、资金、生产发布 | 默认禁止或双人审批 |

GFast MCP Tool 必须调用业务服务/API，**禁止绕过业务规则直连数据库**。

---

## 8. jzin-bk 开发规范（与架构对齐）

### 8.1 后端（GoFrame）

- 新模块遵循 **gfast-module-dev** 技能：api → controller → service → logic → model/dao → router → boot 注册
- **零 `init()`**：logic 通过 `Register()` + `internal/app/boot/` 挂载
- 数据权限显式注入，不假设框架自动隔离。AI 业务表用 `GetAuthDeptWhere`，不用按创建人迁移的 `GetAuthWhere`
- 多语言：后端消息写中文 + `manifest/i18n/en-US.yaml` 补词条

### 8.2 前端（frontend）

- 静态路由注册、菜单权限 SQL、`v-auth` 与 API 路径一致
- Agent 工作台、资产广场、Run 中心是**统一门户**，不嵌入 Hermes TUI 作为主体验
- 新页面 i18n：禁止硬编码中文 UI 文案

### 8.3 hermes-agent 侧改动

- 仅配置、容器、Projection 消费、事件协议对接
- 保持与上游 hermes-agent 可合并/可升级
- 若需平台专属能力：在 jzin-bk 做 Adapter/Gateway，不在 Hermes 核心写业务逻辑

---

## 9. 接入新 Runtime 强制清单（三期或实验也适用）

每接入一个框架，至少交付：

1. Runtime 注册项（名称、版本、健康检查）
2. Adapter：`start` / `stream`/`events` / `cancel` / 输出归一
3. Projection 映射表（模型、MCP、Skills、身份如何下发）
4. 工具调用不回灌 GFast。本期不设治理门；各 Runtime 按投影的 `mcp_servers` 自己连接。高风险审批留到二期
5. 沙箱/隔离策略
6. 风险等级与审批策略
7. Run 审计字段与产物落点
8. 回滚：关掉该 Runtime 不影响 Core

**Adapter 只做映射，不做平台权限决策。**

---

## 10. 给编码智能体的操作规则

1. **先读**本文件与 `AI 平台总体技术架构.html`，再改代码
2. **默认只做一期**；二期/三期需用户明确点名
3. 优先改 `jzin-bk/` 模块；对 `hermes-agent/` 仅允许配置、Dockerfile、薄封装
4. 新增 MCP 登记在资源广场，按 Runtime 映射下发，不在 GFast 里转发调用
5. 不要引入「每个概念一个微服务」
6. 不要用 Hermes Desktop/Dashboard 替代 frontend 门户
7. 提交说明里写清：改动落在哪一层、是否触及 Projection/Adapter/Gateway
8. 若架构变更：同步更新本文件与架构 HTML/MD
9. 写 jzin-bk 业务模块时启用 **gfast-module-dev** 与 **goframe-v2** 技能
10. 设计外部服务接入方案时，按旧规范 §21 模板输出（目标、架构位置、身份权限、版本、测试、风险）

---

## 11. 禁止项速查

- 第三方 Runtime 源码揉进 jzin-bk 大改
- Runtime 直连平台主库
- 透传用户主 JWT
- 在 GFast 里拦截并代替 Runtime 调用外部 MCP
- 多租户共用同一 Runtime HOME
- 已发布资产无版本热改
- 一期同时上 Temporal + A2A + 多 Worker + 多 Runtime
- 前端直连 WeKnora / Dify / 外部 MCP / 外部 Agent
- 将 Dify/WeKnora 内部 DSL 作为平台长期标准
- 在 API 主进程直接执行长时间浏览器、渲染或代码构建

---

## 12. 维护元数据

| 项 | 值 |
|---|---|
| 工作区名 | HERMES-AGENT-JZIN-BK |
| 平台仓库 | `jzin-bk/`（= GFast） |
| Runtime 仓库 | `hermes-agent/` |
| 架构图（HTML） | `all-project-doc/AI 平台总体技术架构.html` |
| 架构摘要 | `all-project-doc/AI 平台总体技术架构.md` |
| 智能体约束（本文件） | `all-project-doc/ARCHITECTURE.agents.md` |
| 长期融合规范 | `all-project-doc/(旧)AI平台总体架构与服务融合规范方案.md` |
| 当前实施焦点 | **一期：jzin-bk + Hermes-agent** |
| 文档状态 | 与架构图同步持续更新 |
