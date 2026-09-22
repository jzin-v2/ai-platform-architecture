# AI 平台架构约束（供 Cursor / Copilot / 其它编码智能体）

> **本文档用途**  
> 本文件是工作区里编码智能体的**最高架构约束**。实现、重构、接入新 Runtime 前必须先读本文与架构图；冲突时以本文 + 架构图为准，不以第三方框架默认部署方式为准。
>
> **配套持续更新文件（必须同步维护）**
> 1. 本文件：`ARCHITECTURE.agents.md`（智能体约束与落地规则）
> 2. 技术架构图：`index.html` → https://jzin-v2.github.io/ai-platform-architecture/
> 3. 功能架构图：`functional.html` → https://jzin-v2.github.io/ai-platform-architecture/functional.html
> 4. 统一资产示意：`asset-model.html` → https://jzin-v2.github.io/ai-platform-architecture/asset-model.html
> 仓库：https://github.com/jzin-v2/ai-platform-architecture  
>
> 变更架构时：**同时更新上述架构文件**，并保持分期（一期/二期/三期）一致。

---

## 0. 工作区约定

预期多根 / 同级目录（名称以实际仓库为准，语义不变）：

```text
workspace/
├── gfast/                 # 业务底座 + AI 控制面（平台主权 SoT）
├── hermes-agent/          # 一期 Runtime（外部现成 Harness，不魔改进 gfast）
├── <future-runtime>/      # 如 opencode / qwenpaw / eino …（三期）
└── ai-platform-architecture/   # ARCHITECTURE.agents.md + index.html + functional.html
```

编码智能体默认假设：

- **业务与平台代码主要写在 `gfast/`**
- **`hermes-agent/` 保持可独立升级的上游项目**；只通过 Adapter / 配置 / 容器调用
- 不要把 Hermes / 其它 Runtime 源码大面积拷贝进 GFast 并深度魔改

---

## 1. 目标与原则

### 1.1 目标

- 多公司（一级部门）AI 业务平台：统一门户、统一身份权限、统一资产与审计；公司边界 = GFast 一级部门
- Agent 能力用**持续更新的现成框架**（先 Hermes，后其它），少自研 Runtime
- 一人可维护：优先 **GFast 模块**，早期**不拆微服务丛林**

### 1.2 硬原则

1. **平台主权（SoT）在 GFast**：用户、RBAC、部门、资产广场、模型路由、Run/审计、配额。**不另建 tenant**；**一级部门（dept）= 公司 = 租户**，资产与 Run 用 `dept_id` 隔离
2. **Runtime 可插拔**：Hermes / OpenCode / QwenPaw / Eino / Dify Legacy 均为外部 Runtime Plugin
3. **单向投影**：平台 → Runtime 下发配置；禁止用户/角色双向全量同步与影子账号泛滥；隔离键用 `dept_id`（一级部门），不用独立 `tenant_id`
4. **工具必经治理门**：业务 Tool / MCP 调用必须经 GFast MCP 治理中间件，禁止 Runtime 直连业务库
5. **不透传主 JWT**：给 Runtime 的是短期、受众绑定的委派 Token / Service Account
6. **危险动作在沙箱**：终端/文件/浏览器等不在 GFast 主进程直接 exec
7. **公司边界用 dept_id**：不引入独立租户表。GFast RBAC 中**一级部门 = 一个公司 = 一个租户**；资产、投影、Run、配额均挂 `dept_id`（一级部门 ID）。子部门用户归属于其一级部门数据范围。

### 1.3 公司边界 = 一级部门（dept_id）

| 概念 | 落点 |
|---|---|
| 公司 / 租户 | GFast 一级部门 |
| 隔离字段 | `dept_id`（不要 `tenant_id`） |
| 数据权限 | 复用 GFast 部门数据权限 |
| 资产归属 | 创建时写入所属一级部门 ID |


---

## 2. 分期（智能体写代码时按当前期实现，勿提前铺三期）

### 一期（当前要做）— Hermes 最小闭环

在 `gfast/` 实现：

- 登录 / RBAC / 一级部门隔离（复用 GFast；dept_id = 公司）
- 资源/资产广场（MCP、Skills、插件、模型元数据）
- Agent 工作台（选择/切换 Runtime，查看 Run，审批入口）
- Run 中心 + 审计字段
- **Projection 单模块**：把模型路由、MCP 白名单、委派身份等推到 Hermes
- **Hermes Adapter**：`start` / `stream` / `cancel` / 事件归一成平台 Run Event
- **MCP 治理门**：权限、风险等级、配额、审计后再转发 Tool
- **任务表异步调度**（不上 Temporal）
- Hermes 以 **Docker/容器** 运行；业务 H5/小程序**不动**

#### 一期编码第一步（GFast 与 Hermes 均已能启动之后）

按此顺序提交代码，**不要四个广场并行开工**：

1. `gfast/`：统一资产模型（`kind/version/dept_id/status/visibility`）+ 资源广场菜单骨架；`dept_id` = 一级部门（公司）
2. `gfast/`：Runtime 注册，先登记 Hermes（地址、健康检查、启用）
3. 广场**先做 MCP/Tool 一种 kind**（含把现有业务接口封装为 Tool 的登记）
4. 同一资产模型再加 Skills、接口/Connector、知识库、插件（页面可后做）
5. 插件 = **平台 Plugin Manifest**；Hermes 定制只在 Projection/Adapter 翻译，不把 Hermes 插件格式当主数据
6. 知识库广场 = **登记 + 绑定外部知识服务**，不在 GFast 自研向量检索
7. MCP/Tool 能登记后，再做 MCP 治理门 → Hermes Adapter → Projection → Run 中心闭环

当前未点名二期/三期时，智能体只实现以上第一步到闭环，不铺 Workflow 画布 / Temporal / A2A。

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

## 3. 分层与代码落点

| 层 | 职责 | 代码落点 | 期次 |
|---|---|---|---|
| ① 展示 | 统一门户、Agent 工作台 | `gfast` 前端 | 一期 |
| ② 身份 | 登录、RBAC、多租户、换票 | `gfast` IAM + Nginx/Caddy | 一期 |
| ③ Core | 资产、模型、Run、审计、业务 MCP Provider | `gfast` AI 模块包 | 一期 |
| ③′ Projection | 配置单向投影到 Runtime | `gfast` **一个** Projection 模块 | 一期→Hermes |
| ④ 编排 | 任务发起 / 长流程 | 一期任务表；二期 Temporal/NATS | 一→二 |
| ⑤ Runtime | Agent 循环 | `hermes-agent` 等**外部进程** | 一期 Hermes |
| ⑥ Gateway | 工具安检 | `gfast` MCP 治理中间件 | 一期 |
| ⑦ Adapter | 协议翻译、事件归一 | `gfast` Hermes Adapter | 一期 |
| ⑧ 沙箱 | 副作用执行边界 | Docker + Hermes 容器 | 一期 |
| ⑨ Providers | 元数据 vs 真实能力 | 广场在 `gfast`；WeKnora/模型/外部 MCP 在外部 | 一期 |

---

## 4. 三个概念不得混淆（实现前自检）

| 概念 | 是什么 | 不是什么 |
|---|---|---|
| **资源广场** | 登记「有哪些」MCP/Skills/模型/插件 | 不是运行时调用 Hermes 的地方 |
| **Projection** | 事先把配置推给 Runtime | 不是双向主数据同步 |
| **Adapter** | Run 时呼叫 Runtime（start/stream/cancel） | 不是资源广场，也不是 Projection |
| **Gateway** | Tool 调用安检门 | 不是 Adapter；不做协议翻译代替 Adapter |
| **Worker/沙箱** | 终端/文件/浏览器实际执行处 | 一期不要在 GFast 主进程里 exec |

---

## 5. 推荐调用链（一期）

```text
用户 → GFast 门户/工作台
    → GFast 鉴权（RBAC/租户）
    → 创建 Run（绑定 asset 版本）
    → 任务表异步执行
    → Hermes Adapter
    → Docker 中 Hermes 实例
    → Hermes 需调 Tool 时 → GFast MCP 治理门 → 业务/外部 MCP
    → 事件/产物回写 Run 中心
```

禁止：

```text
前端 → Hermes Dashboard/API 直连（多租户门户）
Hermes → 直连业务数据库
Hermes → 持有用户主 JWT / 管理员永久 Token
多用户 → 共用同一个 HERMES_HOME / state.db
```

---

## 6. 一期在 `gfast/` 建议模块边界（名称可按项目习惯调整）

```text
gfast/
├── modules/ai_asset/          # 资源广场、版本、上下架
├── modules/ai_runtime/        # Runtime 注册、切换、工作台 API
├── modules/ai_projection/     # 单向投影（先 Hermes）
├── modules/ai_adapter_hermes/ # Hermes Adapter
├── modules/ai_mcp_gateway/    # MCP 治理门
├── modules/ai_run/            # Run、事件、产物元数据、审计
└── modules/ai_job/            # 一期任务表调度
```

`hermes-agent/`：

- 以官方方式容器部署
- 通过环境变量 / 配置接收 Projection 结果（模型 BaseURL/Key 引用、工作目录、MCP 白名单等）
- **不要**把平台 RBAC、租户表、资产主数据搬进 Hermes

---

## 7. 接入新 Runtime 的强制清单（三期或临时实验也适用）

每接入一个框架，至少交付：

1. Runtime 注册项（名称、版本、健康检查）
2. Adapter：`start` / `stream`/`events` / `cancel` / 输出归一
3. Projection 映射表（模型、MCP、Skills、身份如何下发）
4. 工具调用是否全部回灌 MCP 治理门
5. 沙箱/隔离策略
6. 风险等级与审批策略
7. Run 审计字段与产物落点
8. 回滚：关掉该 Runtime 不影响 Core

模板对齐历史架构规范：Adapter 只做映射，不做平台权限决策。

---

## 8. 给编码智能体的操作规则

1. **先读**本文件与 `index.html`（或在线架构图），再改代码  
2. **默认只做一期**；二期/三期需用户明确点名  
3. 优先改 `gfast/` 模块；对 `hermes-agent/` 仅允许配置、Dockerfile、薄封装，禁止深度魔改核心以承载平台业务  
4. 新增 Tool 必须登记资源广场 + 走 MCP 治理门  
5. 不要引入「每个概念一个微服务」  
6. 不要用 Hermes Desktop/Dashboard 替代 GFast 门户  
7. 提交说明里写清：改动落在哪一层、是否触及 Projection/Adapter/Gateway  
8. 若架构变更：同步更新 `ARCHITECTURE.agents.md`、`index.html` 与 `functional.html`

---

## 9. 禁止项速查

- 第三方 Runtime 源码揉进 GFast 大改  
- Runtime 直连平台主库  
- 透传用户主 JWT  
- 绕过 MCP 治理门调高危 Tool  
- 多租户共用同一 Runtime HOME  
- 已发布资产无版本热改  
- 一期同时上 Temporal + A2A + 多 Worker + 多 Runtime  

---

## 10. 维护元数据

| 项 | 值 |
|---|---|
| 技术架构图 | `index.html` / https://jzin-v2.github.io/ai-platform-architecture/ |
| 功能架构图 | `functional.html` / https://jzin-v2.github.io/ai-platform-architecture/functional.html |
| 智能体约束（本文件） | `ARCHITECTURE.agents.md` |
| 当前实施焦点 | **一期第一步：资产骨架 + Hermes 登记 + MCP/Tool** |
| 文档状态 | 与架构图同步持续更新 |

