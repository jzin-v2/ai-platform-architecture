# AI 平台总体架构

与工作区 `jzin-bk/all-project-doc/` 同步维护。

| 文件 | 用途 | 在线 |
|---|---|---|
| [`index.html`](./index.html) | 技术架构图 | https://jzin-v2.github.io/ai-platform-architecture/ |
| [`ARCHITECTURE.agents.md`](./ARCHITECTURE.agents.md) | 编码智能体最高约束 | 本仓库 |
| [`functional.html`](./functional.html) | 功能架构图 | https://jzin-v2.github.io/ai-platform-architecture/functional.html |
| [`prototype/`](./prototype/) | 早期后台原型（部分已落后于实现） | https://jzin-v2.github.io/ai-platform-architecture/prototype/ |

## 当前关键口径

- `jzin-bk/` = GFast；`hermes-agent/` = 外部 Runtime
- 公司边界 = 一级部门（`dept_id` / `parent_id=0`），无独立 tenant 表
- **GFast 不站在 Hermes 与外部 MCP 之间**：只做 MCP 统一登记 + 按 Runtime 投影映射；各 Runtime 自己连接 MCP
- 一期对话：GFast 壳 + 按（公司，用户，Agent）运行目录投影 + Adapter 拉起 `hermes serve`（tui_gateway）；`embed_url` 保留但本步不用
- 部署：`ai.runtime.placement=local`（本机进程内 serve）或 `remote`（K8s：GFast 无状态多副本 + Hermes StatefulSet 按槽位粘滞，流式经 Redis `ai:chat` 回门户）
- 资源广场：MCP、技能已落地（技能 zip / 单 SKILL.md，整包投影）；插件占位
- 本机 Desktop：`hermes-ext/gfast-bridge` 可把已授权技能下载进个人 profile，**不替代**门户投影；门户对话不读个人 `HERMES_HOME`
- 模型目录可登记；内置模型与会话级模型/推理强度选择已写入架构
- 无人工审批；出问题管理员事后处理
