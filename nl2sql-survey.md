# 智能问数调研对照

> 对照成熟 NL2SQL / ChatBI 开源实践，明确本栈「知识底座 ≠ 问数引擎」，给出一期可落地路径：受控 MCP 出数 + WeKnora 口径支撑，Runtime 永不直连业务库。
>
> 配套在线页：[智能问数调研对照](https://cursor.com/canvas/store-681cacfa-37df-490c-9e36-a3891c6a454f/6a065f69-26a5-469e-acc9-35b70b0bca49)
>
> 调研日期：2026-11-08

## 一句话结论

一期已经按考试系统试点落地：口径在 Wren 项目 `exam-wren`，业务库只给 Wren 的只读账号，Wren 自己的 MCP 在 GFast 登记后下发 WeKnora，调用不经过 GFast。WeKnora / RAGFlow 仍是知识底座，不是问数引擎。

---

## 1. 知识底座 vs 问数引擎

队友结论与公开项目共识一致：能「搜文档」不等于能「可信出数」。本栈现有 WeKnora（v0.9.0 能力面：知识库 + Agent + 沙箱/技能/MCP）与本机 RAGFlow 都属于知识底座；真正「自然语言出数」必须另接数仓 / BI / 受控 SQL 或指标服务，经 MCP 暴露给 Agent。

| 维度 | 知识底座（WeKnora / RAGFlow） | 问数引擎（需另接） |
|---|---|---|
| 核心职责 | 业务文档、指标口径、报表说明、示例问答、禁查规则检索 | Schema 对齐、SQL/指标解析、受控执行、结果校验与可视化 |
| 典型能力 | RAG、Agent 编排、沙箱跑代码/技能、MCP 工具链、流式 thought | 语义层 / Headless BI、只读 SQL 网关、行级权限、审计脱敏 |
| 能否当主路径 | **否** — 不可替代专用问数层 | **是** — 出数权威路径 |
| 与本栈硬原则 | 可挂沙箱与技能封装固定问数流程；不承载平台 RBAC/租户 SoT | 必须以 MCP 在 GFast 登记后投影下发；禁止 Runtime 直连业务库 |
| 一期角色 | 口径与文档支撑 + Agent 编排入口（WeKnora 壳层） | 独立受控查询 Provider（只读 SQL / 指标 API）经 MCP 接入 |

**硬边界：** WeKnora 内置 database_query / data_analysis 仅面向知识库元数据与上传 CSV/Excel（DuckDB），不是业务库问数通道。RAGFlow 同样是文档 RAG，结构化问数勿指望它。

---

## 2. 成熟项目对照

星数取自 GitHub API，约 2026-11-08；Vanna 已归档仅作范式参考。

| 项目 | 定位 | 关键能力 | 与本栈契合度 | 备注 |
|---|---|---|---|---|
| WrenAI ~18.1k | GenBI / 治理 Text-to-SQL | MDL 语义层、Schema 检索、SQL 校验、图表与 Dashboard、面向 Agent 的 context layer | 高（理念） | 语义可版本化；一期可借鉴 MDL/口径沉淀，不宜整仓替换门户 |
| DB-GPT ~20.3k | 数据智能体框架 | Text2SQL、RAG、AWEL 编排、Skills、沙箱、MCP 连接器 | 中高（形态接近） | 能力面像「整站数据助手」；与本栈重叠大，宜借鉴 Skills+沙箱，勿整栈迁入 |
| SuperSonic ~5.2k | ChatBI + Headless BI | 语义模型→S2SQL→物理 SQL、多轮、三级权限、可视化 | 中（语义层标杆） | Java SPI 可插拔；完整语义层一期过重，作二期指标层参考 |
| SQLBot ~7.1k | DataEase 系智能问数 | LLM+RAG Text2SQL、术语库、SQL 示例、工作空间隔离、MCP/嵌入 | 高（落地形态） | 明确提供 MCP 调用，最接近「问数服务经 MCP 挂到 Agent」 |
| Chat2DB ~28.4k | AI SQL 客户端 | 多库管理、NL→SQL、解释优化、CLI/MCP | 低（工具位） | 偏 DBA/开发者工作台，不是多租户业务问数门户 |
| Bytebase ~14.7k | 数据库治理 + AI SQL | 访问控制、脱敏、审计；AI 查询走同一治理链；MCP Server | 高（安全范式） | 强调「失败常在权限而非 SQL」；受控执行层必学 |
| Vanna ~23.8k（已归档） | RAG 训练式 Text2SQL 库 | DDL/文档/问答对检索 → 生成 SQL | 低（仅范式） | 2026-03 归档转商业；无持久语义产物，勿新依赖 |
| PandasAI ~23.9k | 对话式 DataFrame/湖仓分析 | CSV/Parquet/SQL 上自然语言分析 | 中（沙箱辅路） | 适合沙箱内对导出结果集二次分析，非企业口径主路径 |
| LangChain SQL Agent | 教程级 SQL Agent | list/schema/check/query 工具循环 + HITL | 中（工具拆分） | 官方警示直连库风险；本栈应把执行挪进受控 MCP，而非 Runtime 持库连接 |

### 行业共识（可学）

- 可信问数 = 语义/口径 + 受控执行 + 权限审计，不只是「模型会写 SQL」
- MCP 成为问数能力对外标准接口（SQLBot / Bytebase / DB-GPT）
- Agent + Skills + 沙箱适合编排与解释，不适合持有业务库凭证

### 对本栈需警惕

- 裸 NL2SQL（Runtime 直连库）违反本栈硬原则，且企业口径不可控
- 整仓引入 WrenAI / SuperSonic / DB-GPT 会破坏 GFast SoT 与投影模型
- 把 WeKnora 或 RAGFlow 当问数主路径会长期校准失败

---

## 3. 行业常见分层

成熟方案普遍把「理解业务」与「执行查询」拆开。下列链路是对照 WrenAI / SuperSonic / SQLBot / Bytebase / LangChain SQL Agent 后的归纳，并标注本栈一期落点。

```text
自然语言问题（门户 / WeKnora 对话）
    → 多轮澄清（歧义指标、时间范围）
    → 知识底座（口径 / 表说明 / 禁查 / few-shot）
    → Schema Linking（表/字段/指标映射）
    → SQL 或指标查询生成（可经语义层降复杂度）
    → 校验与护栏（只读 / 白名单 / LIMIT / 脱敏）
    → 受控执行沙箱（问数 MCP Provider）
    → 解释与可视化（图表 / 摘要 / 产物）
    → 反馈闭环（正确 SQL 回灌示例库）
```

1. **语义层 / 指标层** — 定义指标、维度、口径与禁查；一期用知识库文档 + 轻量表结构说明，二期再评估 SuperSonic/Wren MDL。
2. **Schema Linking** — 从问题检索相关表/字段/示例；由问数 MCP 的 schema 工具 + WeKnora 检索共同完成。
3. **SQL 生成与校验** — 生成后强制只读、表白名单、行数上限、禁止多语句；可参考 Bytebase「生成也走治理链」。
4. **执行沙箱** — 仅问数 Provider 持业务只读凭证；Hermes/WeKnora Runtime 只调 MCP，不持库连接。
5. **结果解释与可视化** — Agent 总结 + 可选沙箱内图表技能；产物回写 Run/会话。
6. **反馈闭环** — 人工确认的问答对进入知识库 few-shot；失败案例进禁查/澄清规则。

---

## 4. 对本栈一期落地路径

### 已落地（2026-11-08）

考试系统按第三方业务系统接入，不改考试模块代码。

- 口径、术语、规则和确认 SQL 在 `exam-wren`（MDL + `knowledge/`），git 管理。WeKnora 知识库不重复放口径。
- 业务库连接只在 Wren。库 `exam_rpt` 的 7 张视图由 `exam_ro` 只读访问，脚本见 `智能问数-考试系统报表视图.sql`。视图去掉密码和 open_id，手机号打码，滤掉软删除行。`dry_run` 的 `EXPLAIN` 另需视图 `SHOW VIEW` 和底表上与视图相同的列级 `SELECT`，不含密码、open_id 和原始手机号。
- Wren `strict_mode` 拒绝模型以外的表和写语句。MCP 只听 `127.0.0.1:8301`，`run-all.ps1` 带 `--allow-write` 启动。
- GFast 登记 MCP `exam-wren` 和技能 `ask-data` 并下发 WeKnora。`store_query` 需审批，用户在对话里确认后才回灌。WeKnora `SSRF_WHITELIST` 包含 `127.0.0.1`。
- 对话入口是 WeKnora 智能体「考试问数」。谁能用这个智能体，谁就能问。考试表没有部门字段，一期不做按人过滤。
- 评测是 `exam-wren/evals/golden.yml` 的 30 题，按结果集对比。审计看 WeKnora 会话里的工具参数。

二期触发条件：要按人/部门过滤、要平台审计表，或 Wren 不能只绑本机。做法是 GFast 原样转发 Wren 的全部工具，并只在 `managed_by=gfast` 的调用上注入身份。切换时只改资产地址。

### 首选方案

问数引擎用 Wren 的语义层和只读执行，不在 GFast 里再写一套 SQL 网关。GFast 只登记和下发。Hermes 本期不作为入口，以后可以复用同一个 MCP。

### 为何选这一层

- **WeKnora Agent + MCP**：一期门户壳层已嵌套 WeKnora，具备沙箱/技能/流式工具链，适合作为问数编排入口；但 WeKnora 本身不是问数引擎，必须外挂受控出数 MCP。
- **Hermes + MCP**：可与 WeKnora 共用同一套已登记 MCP（Projection 下发），适合 Hermes 对话主路径；仍禁止 Hermes 直连业务库。
- **GFast 业务 MCP Provider**：适合把「指标/受控查询」实现成平台侧服务（凭证、审计、部门数据权限落在 GFast），再以 MCP 形态暴露；这是出数实现位，不是对话编排位。

### 落地步骤

0. **划清边界** — 文档与 Agent 提示词写死：WeKnora/RAGFlow=知识底座；出数=受控 MCP。禁止给 Runtime 配置业务库 DSN。
1. **实现受控问数 Provider** — 独立服务或 GFast 模块：只读账号、表白名单、强制 LIMIT、禁 DML/多语句、按部门/角色过滤、全量审计。可借鉴 Bytebase/SQLBot 的治理与 MCP 暴露方式。
2. **MCP 拆分并进资产广场** — 至少拆：`schema_search`（表/字段说明）、`controlled_query`（受控 SQL 或指标查询）、可选 `metric_lookup`（指标元数据）。在 GFast 登记、发布、绑定 Agent，再投影/下发 WeKnora 与 Hermes。
3. **Skills 封装固定流程** — SKILL.md 规定：先检索口径 → 再 schema → 再受控查询 → 再解释/图表；歧义则 clarify。危险动作仍走沙箱。
4. **知识库装填** — 表结构说明、指标口径、示例问答、禁查规则、常见澄清话术；不把原始业务明细灌进 RAG 当「假数仓」。
5. **审计与 Run 衔接** — 每次出数写入 Run 事件：谁、哪个 Agent/资产版本、哪条 MCP 工具、生成 SQL/指标、行数、耗时；广场收藏与个人启用开关沿用现有投影规则。

### 一期 Checklist

- [x] 明确知识底座 vs 问数引擎分工 — Wren 出数，WeKnora 只做对话
- [x] 只读护栏 — exam_rpt 视图、exam_ro、Wren strict_mode、本机绑定
- [x] GFast 登记 exam-wren / ask-data 并下发 WeKnora
- [x] 问数 Skill ask-data — 先口径再查数，确认后才回灌
- [x] 口径放在 Wren knowledge/，不进 WeKnora 知识库
- [ ] 平台审计表覆盖每次出数 — 二期
- [x] WeKnora 与 GFast 配置里没有 exam_ro 密码

### 建议的 MCP / Skills 拆分

| 资产 | 类型 | 职责 | 一期是否必做 |
|---|---|---|---|
| schema_search | MCP Tool | 按问题检索允许的表/字段/注释（可回源元数据服务，不暴露全库） | 必做 |
| controlled_query | MCP Tool | 接受已校验 SQL 或结构化查询；只读执行并返回有限行 + 列类型 | 必做 |
| metric_lookup | MCP Tool | 查指标定义、口径、可用维度（可先由 KB 检索替代） | 可缓 |
| chart_render | Skill / 沙箱 | 对结果集画图或导出；在 WeKnora 沙箱执行，不碰业务库 | 可缓 |
| ask-data-flow | Skill | 编排：口径检索→schema→query→解释；写明禁查与澄清策略 | 必做 |

### 知识库该放什么

- 表/视图业务说明与字段词典（非全量敏感样例行）
- 指标口径：定义、计算公式、统计粒度、常见误用
- 高质量示例问答（问题 → 正确 SQL/指标请求 → 结果说明）
- 禁查规则：禁止表、禁止字段、禁止跨租户、必须带时间窗等
- 报表与看板说明（业务用户语言），供澄清与解释引用

### 与资产广场 / 投影 / 审计如何衔接

- 问数 MCP 与问数 Skill 作为已发布资产进入广场；Agent 绑定编码，pin_version 跟随发布版。
- Projection：Hermes 写入运行目录 mcp_servers；WeKnora 由空间管理员下发，managed_by=gfast。
- 个人启用开关只影响投影，不改广场已发布真相。
- Run 中心记录工具调用与产物；不透传主 JWT，出数侧使用服务账号/短期委派凭证。
- 部门数据权限继续由 GFast `GetAuthDeptWhere` 约束资产可见与 Provider 侧过滤。

---

## 5. 一期不做清单

- Runtime（Hermes / WeKnora Agent 进程）直连业务库或持有业务库长效账号
- 把 WeKnora 或 RAGFlow 当成 NL2SQL / ChatBI 主产品替代问数引擎
- 整仓引入 WrenAI / SuperSonic / DB-GPT 替换门户与资产 SoT
- 一期建设完整 Headless BI 语义层、行级多数据源联邦、Text2SQL 模型微调（SQLCoder 等）
- 依赖已归档的 Vanna 开源仓库作为生产依赖
- 在 GFast 主进程执行长时间查数/画图；或透传用户主 JWT 给出数服务

---

## 6. 参考链接

- [WrenAI](https://github.com/Canner/WrenAI) — GenBI / context layer（~18.1k★）
- [DB-GPT](https://github.com/eosphoros-ai/DB-GPT) — 数据智能体框架（~20.3k★）
- [SuperSonic](https://github.com/tencentmusic/supersonic) — ChatBI + Headless BI（~5.2k★）
- [SQLBot](https://github.com/dataease/SQLBot) — DataEase 智能问数，含 MCP（~7.1k★）
- [Chat2DB](https://github.com/OtterMind/Chat2DB) — AI SQL 客户端（~28.4k★）
- [Bytebase](https://github.com/bytebase/bytebase) — 治理型 AI SQL / MCP（~14.7k★）
- [Bytebase：2026 Text-to-SQL 工具盘点](https://www.bytebase.com/blog/top-text-to-sql-query-tools/)
- [LangChain：Build a SQL agent](https://docs.langchain.com/oss/python/langchain/sql-agent)
- [Vanna](https://github.com/vanna-ai/vanna) — 已归档，仅参考
- [PandasAI](https://github.com/sinaptik-ai/pandas-ai)
- [WeKnora 技能与沙箱文档](https://weknora.weixin.qq.com/docs/03-features/22-skills-sandbox)
- [Wren / DB-GPT / Vanna 对比博客](https://menuagentic.com/zh/blogs/wren-ai-vs-db-gpt-vs-vanna-vs-dataherald/)

---

## 维护元数据

| 项 | 值 |
|---|---|
| 在线 Page | https://cursor.com/canvas/store-681cacfa-37df-490c-9e36-a3891c6a454f/6a065f69-26a5-469e-acc9-35b70b0bca49 |
| 建议落点 | `all-project-doc/智能问数调研对照.md` |
| 对照架构约束 | `ARCHITECTURE.agents.md`、`AI 平台总体技术架构.md` |
| 状态 | 2026-11-08 初稿 |
