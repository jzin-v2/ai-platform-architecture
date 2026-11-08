# AI 平台总体架构

与工作区 `jzin-bk/all-project-doc/` 同步维护。

| 文件 | 用途 | 在线 |
|---|---|---|
| [`index.html`](./index.html) | 技术架构图 | https://jzin-v2.github.io/ai-platform-architecture/ |
| [`ARCHITECTURE.agents.md`](./ARCHITECTURE.agents.md) | 编码智能体最高约束 | 本仓库 |
| [`functional.html`](./functional.html) | 功能架构图 | https://jzin-v2.github.io/ai-platform-architecture/functional.html |
| [`prototype/`](./prototype/) | 早期后台原型（部分已落后于实现） | https://jzin-v2.github.io/ai-platform-architecture/prototype/ |

## 当前关键口径

同步日期：2026-11-08（工作区最新附件）。

- `jzin-bk/` = GFast；`hermes-agent/` = 外部 Runtime；统一门户在 `jzin-bk/frontend`
- 公司边界 = 一级部门（`dept_id` / `parent_id=0`），无独立 tenant 表
- **GFast 不站在 Hermes 与外部 MCP 之间**：只做 MCP 统一登记 + 按 Runtime 投影映射；各 Runtime 自己连接 MCP
- 一期对话：GFast 壳 + `ai_agent` 定义 + 按（公司，用户，Agent）运行目录投影 + Adapter 拉起 `hermes serve`（tui_gateway）；`embed_url` 保留但本步不用
- 部署：`ai.runtime.placement=local`（本机进程内 serve）或 `remote`（K8s：GFast 无状态多副本 + Hermes StatefulSet 按槽位粘滞，流式经 Redis `ai:chat` 回门户）
- 资源广场：MCP、技能已落地（技能 zip / 单 SKILL.md，整包投影）；插件占位
- 本机 Desktop：`hermes-ext/gfast-bridge` 可把公司已发布技能/MCP 下载进个人 profile，**不替代**门户投影；门户对话只读投影到 Agent 运行目录的技能
- WeKnora：嵌在 GFast 壳 `WeKnora/frontend/src/gfast-embed/`，同域 `/weknora/`；**不替代** Hermes 对话页
- 高可用：持久数据进数据库，文件进 `upload` 文件服务；Pod 本地只放槽位运行目录投影/会话/记忆；多副本须把 `upload.default` 设成对象存储
- 无人工审批；出问题管理员事后处理

细节与硬原则以 [`ARCHITECTURE.agents.md`](./ARCHITECTURE.agents.md) 为准；图以 [`index.html`](./index.html) 为准。

