# AI 平台总体技术架构

## 持续维护的两个文件（请同步更新）

| 文件 | 用途 |
|---|---|
| [`index.html`](./index.html) | 可视化总架构图 |
| [`ARCHITECTURE.agents.md`](./ARCHITECTURE.agents.md) | 给 Cursor / Copilot 等编码智能体的约束与落地规则 |

**在线架构图：** https://jzin-v2.github.io/ai-platform-architecture/

## 工作区用法

将本仓与 `gfast/`、`hermes-agent/`（及后续其它 Runtime）放在同一工作区；编码前让智能体先读 `ARCHITECTURE.agents.md`。

当前焦点：**一期 = GFast + Hermes-agent 最小闭环**。
