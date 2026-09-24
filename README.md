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
- 一期对话：GFast 壳 + 投影目录 + Adapter 拉起本机 `hermes serve`（tui_gateway）；`embed_url` 保留但本步不用
- 无人工审批；出问题管理员事后处理
