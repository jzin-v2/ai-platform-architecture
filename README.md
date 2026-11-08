# AI 平台总体架构

与工作区 `jzin-bk/all-project-doc/` 同步维护。

| 文件 | 用途 | 在线 |
|---|---|---|
| [`index.html`](./index.html) | 技术架构图 | https://jzin-v2.github.io/ai-platform-architecture/ |
| [`ARCHITECTURE.agents.md`](./ARCHITECTURE.agents.md) | 编码智能体最高约束 | 本仓库 |
| [`BRANCHING.md`](./BRANCHING.md) | 分支与仓库管理 | 本仓库 |
| [`nl2sql-survey.md`](./nl2sql-survey.md) | 智能问数调研对照 | 本仓库 |
| [`functional.html`](./functional.html) | 功能架构图 | https://jzin-v2.github.io/ai-platform-architecture/functional.html |
| [`prototype/`](./prototype/) | 早期后台原型（部分已落后于实现） | https://jzin-v2.github.io/ai-platform-architecture/prototype/ |

## 当前关键口径

同步日期：2026-11-08 晚（工作区最新附件）。

- `jzin-bk/` = GFast；`hermes-agent/` = 外部 Runtime；统一门户在 `jzin-bk/frontend`
- 公司边界 = 一级部门（`dept_id` / `parent_id=0`），无独立 tenant 表
- **GFast 不站在 Hermes 与外部 MCP 之间**：只做 MCP 统一登记 + 按 Runtime / WeKnora 下发；各 Runtime 自己连接 MCP；禁止 Runtime 直连业务库
- 一期对话：GFast 壳 + `ai_agent` 定义 + 按（公司，用户，Agent）运行目录投影 + Adapter 拉起 `hermes serve`（tui_gateway）
- 部署：`ai.runtime.placement=local` 或 `remote`（K8s + Redis `ai:chat`）
- 模型：按目标下发；Hermes 读对话目标，WeKnora 推送型（`managed_by=gfast`，远端 `gf-{id}`）
- MCP/技能：门户工具箱「我的/广场」；空间管理员可下发到本公司 WeKnora 空间；REST 不写 Hermes、也不下发 WeKnora
- WeKnora：乾坤壳层嵌套（七个菜单共用保活子应用），同域 `/weknora/`；**不替代** Hermes 对话页
- **考试问数（一期）**：口径在 Wren `exam-wren`，业务库只给 Wren 只读账号；MCP `exam-wren` + 技能 `ask-data` 下发 WeKnora；GFast 不在调用链上；按人过滤/平台审计留二期
- 高可用：持久数据进数据库，文件进 `upload`；多副本须对象存储
- 协作：个人长期分支 `feature/<名字>` + 集成分支，用 merge，不 rebase / force push（见 `BRANCHING.md`）

细节与硬原则以 [`ARCHITECTURE.agents.md`](./ARCHITECTURE.agents.md) 为准；图以 [`index.html`](./index.html) 为准。

