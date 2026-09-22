# RECON: mem0

> 状态: 调研稿 · 已过审（2026-09-22 18:56 · 用户原话「通过」，随 PRODUCT-DESIGN v0.3 / §1.5 审核门；标「未验证」项不得被下游引用）
> 日期: 2026-09-22
> 联网验证: 已联网验证。GitHub REST API 对本会话出口 IP 间歇限流（403 rate limit），已改用 `raw.githubusercontent.com`、`img.shields.io` 计数徽标、`github.com` HTML 页面/Atom feed 与重试后的 API 调用完成核验；文档站 `docs.mem0.ai/<page>.md` 可取到完整 Markdown 原文。所有事实句后附来源；取不到或无法交叉验证的写「未验证」。
> 一句话画像: mem0 是「抽取式记忆层」的品类定义者与最大声量玩家（66k★），但其价值主张正在从**显式 LLM 合并（ADD/UPDATE/DELETE）**退回到**只追加 + 检索期排序**；治理面（审核、血缘、副本闭包、执行准入）基本为空，恰是我们方案的差异化空间。

---

## 1 定位速览

### 1.1 三种交付形态 + 一个本地/MCP 支线

| 形态 | 是什么 | 关键差别 |
|---|---|---|
| **Mem0 Platform（managed）** | 托管 API + 控制台（`app.mem0.ai`，`api.mem0.ai`），Mem0 代跑向量库/LLM/embedder/reranker (来源: https://docs.mem0.ai/platform/platform-vs-oss) | 独占 v3 能力：原生 Graph Memory、Memory Decay、Temporal Reasoning、Dream 后台整理、自定义类别、webhooks、记忆导出、batch 操作、反馈、摘要 (来源: https://docs.mem0.ai/platform/platform-vs-oss) |
| **Mem0 Open Source** | 同一套记忆引擎自托管：作为库（`pip/npm install mem0ai`）或 Docker server（REST + 控制台） (来源: https://docs.mem0.ai/open-source/overview) | 无 org/project 概念、无项目级事件日志、无 graph（v3 已移除）、无 decay/temporal/dream/webhooks/export/batch/feedback；25 向量库 / 18 LLM / 11 embedder 自选 (来源: https://docs.mem0.ai/platform/platform-vs-oss) |
| **OpenMemory（MCP 支线）** | 营销页把 OpenMemory 定位为「面向编码 agent 的持久 MCP 记忆层」：装插件 / 接 MCP，自动捕获、按类型打标签、按 project 取用、有访问日志 (来源: https://mem0.ai/openmemory) | 见 1.2，该名称当前处于**语义漂移**状态 |
| **Agent 插件家族** | 编码 harness 侧的插件（Claude Code / Codex / Cursor / Kimi / OpenCode / OpenClaw / Antigravity / DeepSeek Harness / Pi Agent），共享一套脚本、skills 与记忆库 (来源: https://docs.mem0.ai/changelog/highlights) | 通过 hooks 自动 capture/recall，非显式 tool call |

「两者跑同一套核心抽取与检索逻辑，Platform 只加托管 + 一小撮 v3-only 能力 + 管理面」是官方自述 (来源: https://docs.mem0.ai/platform/platform-vs-oss)。但基准分数一侧明确补充：**分数来自 managed platform，含 OSS 拿不到的专有优化，OSS 用户只应期待"方向性相似、数字不同"** (来源: https://docs.mem0.ai/core-concepts/memory-evaluation, https://raw.githubusercontent.com/mem0ai/mem0/main/README.md)。

### 1.2 OpenMemory 是什么（现状需要小心）

- `github.com/mem0ai/openmemory` 仓库**当前**的描述是「开源 CLI & TUI，把 AI 编码会话在 Claude Code / Codex / OpenCode 之间搬运」，36★、MIT、创建于 2026-07-06，README 通篇讲 session 搬运与 autosync 路线图，**不提记忆抽取/检索** (来源: https://raw.githubusercontent.com/mem0ai/openmemory/main/README.md, https://api.github.com/repos/mem0ai/openmemory)。
- 独立站 `openmemory.ai` 则是一个「Better Memory for Claude Code」的**候补名单式托管产品页**（Capture / Organize / Deliver 三段，含 Dashboard、Access Logs，页脚「© 2026 Mem0」） (来源: https://openmemory.ai/)。
- 旧版定位（本地优先、记忆不出设备、按客户端授权访问）仍留在 `mem0.ai/openmemory-mcp-3` 落地页，且其「Full Memory Control（决定什么被存、何时过期、哪个 MCP client 能访问）」被标为 **UPCOMING FEATURE** (来源: https://mem0.ai/openmemory-mcp-3)。
- `docs.mem0.ai` 的 sitemap 中**没有** OpenMemory 文档页，只有 `platform/mem0-mcp`（托管 MCP，`https://mcp.mem0.ai/mcp`，需 Platform API key，记忆存在账号里而非本机）(来源: https://docs.mem0.ai/sitemap.xml, https://docs.mem0.ai/platform/mem0-mcp)。
- **未验证**: 早期那个「本地 Docker 栈（Qdrant + Postgres + Neo4j + Next.js 控制台）的 OpenMemory」是否已被改名/归档/并入 server；本次未能取得可引用的迁移或归档公告。

### 1.3 商业与合规姿态

- 分档：Free（1 project）、Pro（unlimited projects）、Enterprise（**Audit logs**、custom integrations & SSO） (来源: https://mem0.ai/pricing)。
- 站页脚合规声明：HIPAA Ready、SOC 2 Type I、GDPR Ready，另有 Trust Center 站点 (来源: https://mem0.ai/openmemory, https://trust.mem0.ai/)。
- Platform 概览把「Audit logs and workspace governance ship by default」当卖点 (来源: https://docs.mem0.ai/platform/overview)；但 OSS 侧「支持 SLA 等合同性承诺目前未见于文档」也是官方自己写的 (来源: https://docs.mem0.ai/platform/platform-vs-oss)。

---

## 2 记忆模型

### 2.1 「记忆类型」名不副实——官方自己承认

`MemoryType` 枚举有三个值，**只有一个是真的** (来源: https://docs.mem0.ai/core-concepts/memory-types)：

| 类型 | 状态 | 备注 |
|---|---|---|
| `procedural_memory` | **已实现** | 仅 Python OSS（`Memory`/`AsyncMemory`），需 `agent_id`；Platform `MemoryClient` 与 TS SDK 均无 (来源: https://docs.mem0.ai/core-concepts/memory-types) |
| `semantic_memory` | **未实现** | 只在枚举里定义，代码其他处从未读取；传给 `add()` 抛校验错误；仓库内无路线图日期 (来源: 同上) |
| `episodic_memory` | **未实现** | 同上，无文档化路线图 (来源: 同上) |

- 也没有 working / associative 这一层概念：`docs.mem0.ai` 全站无此分类（已用 sitemap 全量核对）(来源: https://docs.mem0.ai/sitemap.xml)。
- 除 procedural 外，**记忆不按命名类型分派，只按标识符分作用域** (来源: https://docs.mem0.ai/core-concepts/memory-types)。

### 2.2 多层作用域（真正的分层机制）

| 坐标 | 语义 | 可用性 |
|---|---|---|
| `user_id` | 跟随一个人/账号的长期偏好、画像、历史行为 | OSS + Platform |
| `agent_id` | 某个 agent 人格/工具自身的上下文 | OSS + Platform |
| `run_id` | 一次会话、工单、实验等短生命周期流程 | OSS + Platform |
| `app_id` | 白标应用/租户维度的额外切分 | **仅 Platform** |

`add()` 至少要求一个标识符；同时传多个会进一步收窄 (来源: https://docs.mem0.ai/core-concepts/memory-types)。写入语义有个坑值得记：**默认抽取路径下，每条被抽取出的事实按「谁说的」归属，所以一条记录带 `user_id` 或 `agent_id`，不会同时带两个**；而读取时未被提及的字段**不等于 `null` 约束**，只传 `user_id` 会把同样带 `app_id`/`run_id` 的记录也匹配进来 (来源: https://docs.mem0.ai/platform/features/entity-scoped-memory)。

组织层再叠 `Organization → Project`，org/project 由 API key 自动解析（v3 起构造函数不再接受 `org_id`/`project_id`，改用 project 专用 key）；成员角色只有两档：`OWNER` 与 `READER` (来源: https://docs.mem0.ai/api-reference/organizations-projects, https://docs.mem0.ai/api-reference/organization/add-org-member, https://docs.mem0.ai/api-reference/project/add-project-member)。

### 2.3 三个存储面（source of truth 与投影分离，是它做对的一件事）

| 存储 | 装什么 | 用途 |
|---|---|---|
| **SQL 数据库** | 事实与 metadata；**history log（ADD 事件）+ 滚动消息窗口** | 每条记忆的 source of truth；审计轨迹；抽取期去重上下文 |
| **向量数据库** | 记忆文本、embedding、metadata（时间戳、hash、categories、attributed_to） | 主存储 + 语义检索 |
| **实体存储** | 实体 + embedding + 关联的 memory ID | 图上连接与实体召回加权 |

(来源: https://docs.mem0.ai/core-concepts/memory-evaluation, https://docs.mem0.ai/core-concepts/how-it-works)

### 2.4 Graph Memory：从「有类型图」退到「共现图」

- **论文版（2025）**：有向标注图，节点=实体（带实体类型分类、embedding、创建时间戳），边=有类型关系（如 `lives_in`）；两阶段抽取（实体抽取 → 三元组），检索期实体链接 + 关系三元组；底层 **Neo4j**；更新期做冲突检测，**LLM update resolver 决定哪些关系作废，标记 invalid 而非物理删除以支持时间推理** (来源: https://arxiv.org/html/2504.19413v1)。
- **当前 Platform 版**：图是**内置、常开、无 schema**；节点是抽取出的实体，边只表示「在同一批记忆里共现」；**明确不给关系打类型标签**（"it won't record a 'manages' edge"）；`relations` 字段**恒为空数组**，图只影响 `score` 排序，不返回独立图载荷 (来源: https://docs.mem0.ai/platform/features/graph-memory)。
- **OSS 当前**：外部图存储集成（Neo4j / Memgraph / Kuzu / Apache AGE / Neptune）随 v3 **整体移除**，`enable_graph` 与 `graph_store` 配置项作废，无可查询图、无 `relations`；实体只参与加权 (来源: https://docs.mem0.ai/migration/oss-v2-to-v3, https://docs.mem0.ai/platform/platform-vs-oss)。
- 计费门槛：图本身**全档位常开**，但控制台里的交互式 **Graph view 只在 Pro / Enterprise** (来源: https://docs.mem0.ai/platform/features/graph-memory)。

**判断**：它把「关系语义」换成了「零配置 + 零运维」，代价是丢掉了论文卖点里最有治理价值的部分——带类型的关系与「作废而非删除」的时间有效性标记。

---

## 3 写入链路

### 3.1 招牌机制 v2（2024–2025）：抽取 → 比对 → 四操作合并

论文原文（也是当年宣传的「招牌」）：候选事实与检索回的相似旧记忆一起，通过 function-calling（"tool call"）交给 LLM，**由 LLM 自己决定四个操作之一**：`ADD`（无语义等价记忆时新建）、`UPDATE`（补充既有记忆）、`DELETE`（与既有记忆矛盾时删除）、`NOOP`（无需改动）；不用单独分类器，直接借 LLM 推理判定 (来源: https://arxiv.org/html/2504.19413v1)。

Platform 文档目前仍按这个模型描述抽取三步：**① 拉同 scope 最近消息做上下文 → ② 把新消息 embed 后在同 scope 向量检索"可能需要变的候选记忆" → ③ 单次 LLM 调用比对新消息与候选，逐条事实决定 ADD / UPDATE / DELETE 或不动** (来源: https://docs.mem0.ai/core-concepts/memory-types)。

### 3.2 现行 v3（2026-04 起）：单趟 ADD-only，招牌被自己拆掉

README 的「New Memory Algorithm (April 2026)」与迁移指南一致：**Single-pass ADD-only extraction——一次 LLM 调用，没有 UPDATE/DELETE；记忆累积，不覆盖任何东西** (来源: https://raw.githubusercontent.com/mem0ai/mem0/main/README.md, https://docs.mem0.ai/migration/platform-v2-to-v3, https://docs.mem0.ai/migration/oss-v2-to-v3)。

| 维度 | 旧（v2） | 新（v3） |
|---|---|---|
| 抽取 | 两次 LLM pass（抽取 + 合并） | 单趟 ADD-only（一次 LLM 调用） |
| 记忆变更 | ADD / UPDATE / DELETE | **只有 ADD**，不覆盖不删除 |
| 抽取延迟 p50 | ~2.0s | **~1.0s** |
| LoCoMo | 71.4 | 92.5（README/评估页）/ 91.6（迁移指南口径） (来源: 同上，两处数字不一致) |

（来源: https://docs.mem0.ai/migration/platform-v2-to-v3, https://docs.mem0.ai/migration/oss-v2-to-v3, https://raw.githubusercontent.com/mem0ai/mem0/main/README.md）

现行官方六阶段抽取管道 (来源: https://docs.mem0.ai/core-concepts/memory-evaluation)：
1. **存入新消息**（agent 回复后异步进入管道）
2. **上下文查找**（找相关既有记忆以免重复存）
3. **蒸馏**：单趟 LLM 抽取，产出 ADD-only 事实
4. **去重 + embedding**：基于 hash 去重，再向量化
5. **Graph Memory（实体链接）**：抽专名/引号文本/复合名词短语并跨记忆连边
6. **Temporal Reasoning**：独立一遍时间推理（可与写入异步），抽取「事件何时发生 / 进行中还是已完成 / 时间精度 / 记忆类型（event、state、plan、preference、relationship、absence）」并存为 metadata

关键架构自述：**「新事实与旧事实并存，谁也不被覆盖」是刻意决策，目的是保留时间上下文、避免过早合并造成信息损失** (来源: https://docs.mem0.ai/core-concepts/memory-evaluation)。

### 3.3 调用面与成本

- `add(messages, user_id=..., metadata=..., infer=True)`；`infer=False` 原文入库并**会落重复**（官方 Warning：对同一事实混用两种模式会存两份）(来源: https://docs.mem0.ai/core-concepts/memory-operations/add)。
- Platform 写入**异步**：返回 `status: "PENDING"` + `event_id`，需轮询 `GET /v1/event/{event_id}/`（v3 起 `async_mode` 参数被删，默认异步）(来源: https://docs.mem0.ai/api-reference/memory/add-memories, https://docs.mem0.ai/migration/platform-v2-to-v3)。
- Platform 自动补上下文：只发新轮即可，抽取期会自动拉同标识符的较早消息做指代消解（"He" → Biscuit）(来源: https://docs.mem0.ai/core-concepts/memory-operations/add)。
- 成本重心在「一次（或旧版两次）LLM 调用 + embedding」，读取侧平均 <7,000 tokens/query（top_200 预算）(来源: https://docs.mem0.ai/core-concepts/memory-evaluation)。
- **体积无界增长**由用户承担：官方直说「记忆数会随时间增长而非被合并，这是设计意图，相关性交给检索处理；若你对数量有硬限制，请用 expiration 或定期清理」(来源: https://docs.mem0.ai/migration/platform-v2-to-v3)。

### 3.4 合并策略变更的代价（公开证据）

- **矛盾并存**：`#4956`（open, 19 评论, 2026-04-24）——mutable 状态类事实（当前雇主、当前城市、关系状态）在 ADD-only 下持续累积矛盾记忆；检索的语义/BM25/实体信号不含 recency，旧值可能排在前 (来源: https://github.com/mem0ai/mem0/issues/4956)。
- 同一问题 `#5867`（open, 8 评论, 2026-06-25, 标签 `bug`/`enhancement`/`P2-medium`）附了最小复现（Ronaldo→Messi 两条都在）(来源: https://github.com/mem0ai/mem0/issues/5867)。
- `#5352`（open, 50 评论, 2026-06-03）：因 `#4896`/`#4904` 的语义冲突提案被官方以 **not planned** 关闭，自托管用户自己写了第三方插件 + **每周 cron 卫生脚本**（cosine≥0.82 聚类、LLM 合并、新日期优先、补 `update`/`delete` 工具与时间前缀）来救 (来源: https://github.com/mem0ai/mem0/issues/5352)。
- **抽取质量**：`#4573`（closed, 24 评论）——某团队生产 32 天、单 agent 单人类、Qdrant 后端，拉出 10,134 条逐条审：2,468 条精确 hash 重复与「幻觉类别簇」（如凭空生成"software developer at Google"），668 条同一个反馈环幻觉副本，2,943 条近似重复；最终仅 224 条存活（**97.8% 是垃圾**），其中 186 条还得删掉重写，全库只有 **38 条原样可用** (来源: https://github.com/mem0ai/mem0/issues/4573)。
- **静默丢失**：`#5245`（open, 22 评论, 标签 `P1-high` + `accepted`）——v3 add 管道第 3 阶段批量 embedding 失败后逐条重试，个别仍失败时该文本不进 `embed_map`，下游只遍历 `embed_map` 的键，于是**LLM 已抽出的事实被静默丢弃，只记 WARNING，不向调用方抛异常**（同一模式还出现在实体 embedding 与 boost 路径）(来源: https://github.com/mem0ai/mem0/issues/5245)。
- 同类静默失败：`#4985`（换 embedding provider 因维度不匹配静默丢写）、`#7347`（Azure AI Search 遥测辅助函数把用户记忆静默改挂到 telemetry id 上）、`#7113`（Qdrant 静默丢弃 `*` 通配）、`#7117`（FAISS `_load()` 失败被吞导致索引与 docstore 失联）(来源: https://github.com/mem0ai/mem0/issues/4985, https://github.com/mem0ai/mem0/issues/7347, https://github.com/mem0ai/mem0/issues/7113, https://github.com/mem0ai/mem0/issues/7117)。

---

## 4 检索与装配

### 4.1 多信号融合

Platform v3 = **四路并行打分后 rank 融合**：语义（向量）、关键词（BM25，带动词形态 lemmatization）、实体（查询实体命中加权）、时间（写时存的时间 metadata 对查询时间意图打分）(来源: https://docs.mem0.ai/core-concepts/how-it-works, https://docs.mem0.ai/core-concepts/memory-evaluation)。

- 时间分是**加性**的且语义相关性始终占主导：只微调排序、不过滤候选、不覆盖强语义匹配 (来源: https://docs.mem0.ai/core-concepts/memory-evaluation)。
- 返回的 `score` 是归一到 `[0,1]` 的合成分；图「只影响排序、不改响应形状」(来源: https://docs.mem0.ai/api-reference/memory/search-memories, https://docs.mem0.ai/platform/features/graph-memory)。
- OSS 侧没有图，只有 semantic + 可选 keyword + 实体重叠加权；需要 `mem0ai[nlp]`(spaCy) 与（Qdrant）`fastembed` 才启用实体与 BM25，缺依赖时**静默降级为纯语义**（只打一条 warning）(来源: https://docs.mem0.ai/migration/oss-v2-to-v3)。
- 按查询类型的信号主导关系被明确写进文档（概念型→semantic、精确事实→BM25、实体中心→entity、时间→temporal）(来源: https://docs.mem0.ai/core-concepts/memory-evaluation)。

### 4.2 参数面

| 参数 | 默认 / 取值 |
|---|---|
| `top_k` | 默认 **10**（范围 1–1000）；OSS v3 默认从 100 降到 **20** |
| `threshold` | 默认 **0.1**（传 `0.0` 关闭；OSS v3 之前是无过滤，且强制落在 `[0,1]`） |
| `rerank` | 默认 **false**（OSS v3 之前默认 true；rerank 增加 ~200–400ms） |

(来源: https://docs.mem0.ai/api-reference/memory/search-memories, https://docs.mem0.ai/migration/oss-v2-to-v3, https://docs.mem0.ai/migration/platform-v2-to-v3)

### 4.3 filters 与 metadata

- 标识符**必须放进 `filters`**（顶层 kwargs 在 Platform 返回 400、在 OSS v3 抛 `ValueError`），且至少一个标识符 (来源: https://docs.mem0.ai/api-reference/memory/search-memories)。
- 支持 `AND` / `OR` / `NOT` 包裹 + 比较算子 `in / nin / gte / gt / lte / lt / ne / contains / icontains` + 通配 `*` (来源: https://docs.mem0.ai/api-reference/memory/search-memories, https://docs.mem0.ai/platform/features/v2-memory-filters)。
- **Platform 用固定 allowlist 校验顶层键**（`user_id`、`agent_id`、`app_id`、`run_id`、`created_at`、`updated_at`、`timestamp`、`expiration_date`、`categories`、`metadata`、`keywords`、`memory_ids`），其它键直接 400；且日期字段拒绝显式 `eq`，Platform 无 `nin`。**OSS 任意 metadata 键可直接过滤，无 allowlist**，实际算子支持度取决于你选的向量库 (来源: https://docs.mem0.ai/platform/platform-vs-oss, https://docs.mem0.ai/platform/features/v2-memory-filters, https://docs.mem0.ai/open-source/features/metadata-filtering)。
- 过期记忆默认隐藏，`show_expired` 才返回；`get(memory_id)` 不受过期影响 (来源: https://docs.mem0.ai/api-reference/memory/search-memories)。
- 可解释性：OSS `explain=True` 返回 `score_details`（semantic、归一化 BM25、entity boost、原始合成分、最大可达分、最终分、用于过滤的 threshold）(来源: https://docs.mem0.ai/core-concepts/memory-operations/search)。

### 4.4 装配（context assembly）

**没有装配层。** 官方心智模型是「你 `add`，`search` 后**由你的应用自己决定把哪些记忆塞进 prompt**」(来源: https://docs.mem0.ai/core-concepts/how-it-works)。相关能力（摘要、分类、rerank、decay）都是打分/裁剪，不存在「本次 Run 用了哪些证据」的清单化产物。

补充：Memory Decay（Platform，**按 project 显式开启、默认关**）给每条记忆 0.3×–1.5× 的软性缩放，"绝不置零、绝不过滤"，且只对 v3 search 生效 (来源: https://docs.mem0.ai/platform/features/memory-decay)。

---

## 5 生命周期与治理

### 5.1 update / delete / reset 语义

- `update(memory_id, text=..., metadata=..., timestamp=..., expiration_date=...)`：直接**覆盖**存储值并重建索引；OSS 允许只更 metadata；`data` 是 `text` 的弃用别名；被标 `immutable` 的记忆必须删了重加 (来源: https://docs.mem0.ai/core-concepts/memory-operations/update)。（注：v3 迁移把 SDK 客户端的 `immutable` 参数列入**已删除参数**清单 (来源: https://docs.mem0.ai/migration/oss-v2-to-v3)）
- `batch_update` / `batch_delete`：**Platform 独有**，单请求最多 1,000 条 (来源: https://docs.mem0.ai/platform/platform-vs-oss, https://docs.mem0.ai/core-concepts/memory-operations/update)。
- `delete_all(user_id=... | agent_id= | run_id= | metadata=)`；**破坏性收紧**：过去无 filter 调用 `delete_all` 会清空整个 project，现在**直接报错**；批量删要把某字段显式设 `"*"`，全项目清库要求**四个 filter 全为 `"*"`** (来源: https://docs.mem0.ai/core-concepts/memory-operations/delete)。
- `delete_entities` / `list_entities` 作为 MCP 工具存在（删一个 user/agent/app/run 实体及其全部记忆）(来源: https://docs.mem0.ai/platform/mem0-mcp)。
- **`reset()` 存在，但只存在于代码、不在文档**：OSS `Memory.reset()` / `AsyncMemory.reset()` 会**删除向量集合 + 重建 history 库**（`self.db.reset()`，随后 `SQLiteManager(history_db_path)` 重建），并清空实体索引；对不支持 reset 的向量库退化为 `delete_col()` + 重新 create (来源: https://raw.githubusercontent.com/mem0ai/mem0/main/mem0/memory/main.py:2130-2152, :3828)。`docs.mem0.ai` 全站文档、sitemap 与 `llms.txt` 中**检索不到 reset** (来源: https://docs.mem0.ai/sitemap.xml, https://docs.mem0.ai/llms.txt)。治理含义：一条文档未记载的 API 会连带抹掉本应作为审计轨迹的 history 库。
- 值得记的一处正面细节：显式 `update()` 会向 history 表写入 `(memory_id, prev_value, new_value, "UPDATE", created_at, updated_at, actor_id, role)`——即**逐条旧值/新值/操作者留痕是有的**，只是它不覆盖 LLM 抽取路径（v3 只记 ADD 事件）(来源: https://raw.githubusercontent.com/mem0ai/mem0/main/mem0/memory/main.py:2078-2090, https://docs.mem0.ai/core-concepts/memory-evaluation)。

### 5.2 审计与血缘——有日志，无血缘

| 能力 | 实际强度 |
|---|---|
| 单条记忆变更史 `history(memory_id)` | Platform + OSS 都有；OSS v3 下 SQL history 记的是 **ADD 事件**（因不再 UPDATE/DELETE）(来源: https://docs.mem0.ai/api-reference/memory/history-memory, https://docs.mem0.ai/core-concepts/memory-evaluation) |
| 项目级事件流 `GET /v1/events/` | **仅 Platform**，官方明说可用于 dashboard / alerting / **audit**，含 add/search/delete 事件；OSS 只有 per-memory history，无项目级事件日志 (来源: https://docs.mem0.ai/api-reference/events/get-events, https://docs.mem0.ai/platform/platform-vs-oss) |
| Webhooks | 仅 Platform，project 级订阅 `memory_add / memory_update / memory_delete / memory_categorize`；OSS 不可用 (来源: https://docs.mem0.ai/platform/features/webhooks, https://docs.mem0.ai/platform/platform-vs-oss) |
| 自托管审计 | Docker server 自带**请求审计日志**（控制台 Requests 页：每条 API 调用的状态、延迟、鉴权模式）+ 每用户 API key（只存前缀 + bcrypt hash）(来源: https://docs.mem0.ai/open-source/setup) |
| 记忆→来源证据的 Lineage | **不存在**。抽取时用到哪些消息不记录；唯一的例外是 Dream 的 pattern memory 保留「回链到被蒸馏的具体来源记忆」(来源: https://docs.mem0.ai/platform/features/dream) |
| 导出可携证明 | 无。社区 RFC `#7376` 明确要求「带 proof（命令/结果/退出码/输出 hash）与 chain of custody、跨未验证跳数做 trust decay」的可携记忆包，指出当前「记忆离开 Mem0 只剩文本，没有 state/history/provenance」(来源: https://github.com/mem0ai/mem0/issues/7376) |
| 完整性对账 | 缺失。`#7316`（closed）：history 表与 payload 里存的 md5 都从不与向量库实际内容对账，绕过 API 直接改 Qdrant 表后，`get_all`/`search`/`history` 全部照原样返回且无任何异常 (来源: https://github.com/mem0ai/mem0/issues/7316) |
| DELETE 的时间戳 | 曾是缺口：`#4467` 请求给 history 的 DELETE 操作补时间戳（closed）(来源: https://github.com/mem0ai/mem0/issues/4467) |

### 5.3 过期、撤回与副本闭包

- **Expiration 是隐藏不是删除**：记录原地保留，`search`/`get_all` 跳过，按 ID 仍可取，清掉日期就回来；`YYYY-MM-DD`、**按 UTC 判定**、含当日（设 2030-01-31 则该日全天仍可见，次日消失）；无日期即永不过期（默认）；**日期格式坏掉 fail-open**（存了无法解析的值 → 视为未过期，"坏日期决不会让记忆悄悄消失"）(来源: https://docs.mem0.ai/platform/features/memory-expiration)。
- 文档侧把 delete 明确映射到合规诉求：「满足用户 erasure（GDPR/CCPA）」「清理过期会话/retention deadline 后的清理」(来源: https://docs.mem0.ai/core-concepts/memory-operations/delete)。
- **但删除闭包实测不成立**（这是企业侧最硬的教训）：
  - `#6627`（closed）：`delete_all()` 内部取 ID 列表时没传 `top_k`，多数向量库默认 100 → **只删前 100 条却返回 "Memories deleted successfully!"**，被报为 GDPR「被遗忘权」表面成功而个人数据仍在库 (来源: https://github.com/mem0ai/mem0/issues/6627)。
  - `#5734`（closed）：`delete_all()` 跳过实体 ID 校验，导致**静默 no-op 删除** (来源: https://github.com/mem0ai/mem0/issues/5734)。
  - `#7226`（open）：删除路径读私有 `_entity_store` 属性而非懒初始化 property，任何「本进程未写过实体」的删除会跳过实体索引清理，**留下永久悬空链接** (来源: https://github.com/mem0ai/mem0/issues/7226)。
  - `#3245`（closed）：删记忆不清理 Neo4j 图数据（历史遗留问题）(来源: https://github.com/mem0ai/mem0/issues/3245)。
- 生命周期提案仍停留在社区：`#5330`（closed, 17 评论）用 Ebbinghaus 遗忘曲线做访问频次/按寿命清理，被指出「Mem0 此前没有内建过期或 decay 机制，本地部署会积累陈旧条目」(来源: https://github.com/mem0ai/mem0/issues/5330)。

### 5.4 多租户与权限做到什么程度

- 有：org → project 两级容器 + 成员管理（角色只有 `OWNER` / `READER`）+ project 级 API key 解析 + 「只有成员才能访问其 org/project 作用域内的记忆和数据」的自述 (来源: https://docs.mem0.ai/api-reference/organizations-projects, https://docs.mem0.ai/api-reference/organization/add-org-member, https://docs.mem0.ai/api-reference/project/add-project-member)。
- 应用/租户切分靠 `app_id`（**仅 Platform**；OSS 无 `app_id`）(来源: https://docs.mem0.ai/platform/platform-vs-oss)。
- 无：**记忆级/字段级 ACL、用途(Purpose)声明、可见性上下文、审批门、跨空间发布授权、敏感级别、投影/脱敏**。`user_id` 之类的作用域是**过滤约定而非授权模型**——它自己也提醒「不传 user_id 过滤会跨用户串味」(来源: https://docs.mem0.ai/core-concepts/memory-operations/search)。
- 治理类操作按档位收口：交互式 Graph view、Audit logs、SSO 都在 Pro/Enterprise 之后 (来源: https://docs.mem0.ai/platform/features/graph-memory, https://mem0.ai/pricing)。

### 5.5 一条重要设计旁证：Dream（后台整理）

Dream 是「把 v3 拆掉的 UPDATE/DELETE 换个位置做」的尝试：三件事——**Supersede**（新事实矛盾时把旧事实标为过时，加记忆时触发，全档常开）、**Merge**（重复折成一条 canonical，全档常开）、**Synthesis**（把反复出现的信号蒸馏成更高层 pattern memory，按排期跑、**opt-in 且 Pro 以上**）；pattern memory 与来源记忆保持链接、可追溯证据、幂等不重复生成 (来源: https://docs.mem0.ai/platform/features/dream)。**OSS 完全没有 Dream** (来源: https://docs.mem0.ai/platform/platform-vs-oss)。

---

## 6 界面形态

### 6.1 Platform dashboard（`app.mem0.ai`）

已验证存在的界面能力：记忆浏览与搜索、实体视图、API keys 管理、**Dashboard Graph view**（Pro/Enterprise）、Dream 运行审阅页（`/dashboard/dream`：近期 synthesis 运行与产出 pattern）、Copilot、CLI/agent-signup 之外的「无需 dashboard 的 agent 自助注册」路径 (来源: https://docs.mem0.ai/core-concepts/memory-operations/search, https://docs.mem0.ai/platform/features/graph-memory, https://docs.mem0.ai/platform/features/dream, https://docs.mem0.ai/introduction, https://docs.mem0.ai/platform/overview)。计费与治理项：Projects 数量、Audit logs、SSO 分档 (来源: https://mem0.ai/pricing)。

### 6.2 自托管 server dashboard（OSS）

`make up` 起 REST(:8888) + dashboard(:3000)，六页 (来源: https://docs.mem0.ai/open-source/setup)：

| 页 | 能力 |
|---|---|
| **Requests** | 默认落地页；**实时审计日志**：每条 API 调用的状态、延迟、鉴权模式 |
| **Memories** | 浏览与搜索已存记忆 |
| **Entities** | 去重后的 `user_id`/`agent_id`/`run_id` + 记忆计数 + **级联删除** |
| **API Keys** | 逐用户发 key、打标签、吊销 |
| **Configuration** | 运行时覆盖 LLM / embedder（落库、重启仍生效） |
| **Settings** | 账号与会话控制 |

一次性 setup wizard 建首个 admin（之后关闭注册）、`JWT_SECRET` 必填、key 只存前缀 + bcrypt、第 5 步现场 `curl` 冒烟测试并能在审计页看到；`make bootstrap` 提供「agent-first，不开浏览器」路径 (来源: https://docs.mem0.ai/open-source/setup)。

### 6.3 OpenMemory UI

宣传页给的是：Dashboard（Projects / Global Preferences / Organize Memories）、**按类型打标签**（preferences、decisions、patterns、project context、troubleshooting、team conventions）、按 project 切分取用、**Access Logs**（"See every memory that was added, edited, or served" + "Tag, version, and set visibility rules"），演示里还显示某条回答「Auto Fetched 2 Memories」并标注由哪个 agent 访问（Claude / Cursor / Codex / Copilot）(来源: https://openmemory.ai/, https://mem0.ai/openmemory)。注意 `openmemory-mcp-3` 旧页里「什么被存/何时过期/哪个 client 可访问」的 Full Memory Control 仍标 **UPCOMING** (来源: https://mem0.ai/openmemory-mcp-3)。**该 UI 是否已交付、以及三套 OpenMemory 命名之间的关系：未验证。**

---

## 7 集成与生态

- **SDK / 协议面**：Python SDK（`mem0`、`AsyncMemory`、`MemoryClient`）、TypeScript SDK（含 `mem0ai/oss`）、REST API（`/v3/memories/add/`、`/v1/...`，自托管走 `server/`）、**OpenAI 兼容层**（OSS feature 页）、托管 MCP（`https://mcp.mem0.ai/mcp`，暴露 11 个工具：`add_memory`、`search_memories`、`get_memories`、`get_memory`、`update_memory`、`delete_memory`、`delete_all_memories`、`delete_entities`、`list_entities`、`list_events`、`get_event_status`）、Python/Node CLI (来源: https://docs.mem0.ai/platform/platform-vs-oss, https://docs.mem0.ai/open-source/features/openai-compatibility, https://docs.mem0.ai/platform/mem0-mcp, https://github.com/mem0ai/mem0/releases.atom)。
- **框架/工具适配器**：docs 首页写「Setup guides for **22 tools**，含 LangChain、CrewAI、LlamaIndex、Vercel AI SDK」(来源: https://docs.mem0.ai/introduction)；`/integrations` 下实际有 **37 个**独立集成文档页（agents/autogen/agno/camel/crewai/dify/flowise/langchain/langgraph/llama-index/mastra/n8n/openai-agents-sdk/openclaw/opencode/pi-agent/pipecat/raycast/respan/strands/vercel/zapier/aws-bedrock/livekit/elevenlabs/hermes/kimi/cursor/codex/claude-code/claude-ai/antigravity/deepseek-plugin/chatdev/agentops/langchain-tools/google-ai-adk 等）(来源: https://docs.mem0.ai/sitemap.xml)。
- **组件矩阵（OSS 可选后端）**：27 个向量库文档页 / 17 个 LLM 模型页 / 11 个 embedder 页 / 5 个 reranker 页（Cohere、HuggingFace、LLM-reranker、sentence-transformer、ZeroEntropy）；官方正文口径为「25 vector stores、18 LLM providers、11 embedders」(来源: https://docs.mem0.ai/sitemap.xml, https://docs.mem0.ai/platform/platform-vs-oss, https://docs.mem0.ai/components/rerankers/overview)。
- **编码 harness 插件家族**：Claude Code / Cursor / Codex / Kimi Code / OpenCode / OpenClaw / Pi Agent / Antigravity / **DeepSeek Harness**；Claude Code 插件在 `integrations/claude-code-plugin/`（v0.3.1，纯 stdlib Python，本地只暴露一个 `search_memories` MCP 工具 + 6 个 `/mem0:*` skills，用 lifecycle hooks 本地采证、后台 worker 抽取）(来源: https://docs.mem0.ai/introduction, https://docs.mem0.ai/llms.txt)。
- **与本 harness 直接相关**：2026-08-24 发布 **DeepSeek Harness 原生 Cordis 插件** `@mem0/deepseek-plugin`，注册 `search_memory` / `add_memory` 两个工具、声明 `inject = ['tools']` 走 `ctx.tools.register()`、卸载自动摘除；**开发预览状态，自动捕获/自动召回尚未实现** (来源: https://docs.mem0.ai/changelog/highlights)。
- **周边仓库**：`mem0ai/memory-benchmarks`（113★，开源评测框架，跑 LoCoMo/LongMemEval/BEAM，含结果 web UI）、`mem0ai/mem0-mcp`（662★）、`mem0ai/mem0-chrome-extension`（679★）、`mem0ai/dolphinbench`（新，2026-09-22 仍在推）、`mem0ai/eve-mem0-template`（Vercel 模板）(来源: https://api.github.com/orgs/mem0ai/repos, https://docs.mem0.ai/core-concepts/memory-evaluation)。
- 无代码/低代码：n8n 社区节点（Memory 资源覆盖 Add/Search/Get/Get Many/Update/Delete，并可挂到 AI Agent 当工具）、Zapier app（Add/Search/Get/Delete 四个 action，**尚未上架 Zapier 公共 App Directory**）(来源: https://docs.mem0.ai/changelog/highlights)。

---

## 8 公开问题与成熟度

### 8.1 项目健康度（核验于 2026-09-22）

| 指标 | 值 | 来源 |
|---|---|---|
| Stars | 65,822（shields 取整 66k） | https://api.github.com/repos/mem0ai/mem0 · https://img.shields.io/github/stars/mem0ai/mem0.json |
| Forks / Contributors | 7,729 / 399 | https://api.github.com/repos/mem0ai/mem0 · https://img.shields.io/github/contributors/mem0ai/mem0.json |
| Open issues+PRs | 766 | https://api.github.com/repos/mem0ai/mem0 |
| 创建 / 最近 push | 2023-06-20 / 2026-09-21 | https://api.github.com/repos/mem0ai/mem0 |
| 许可证 | Apache-2.0 | https://api.github.com/repos/mem0ai/mem0 |
| 语言构成 | Python 4.51M / TypeScript 3.19M 字符为主 | https://api.github.com/repos/mem0ai/mem0/languages |
| Python 包 | `mem0ai` 最新 2.1.0（2026-09-18），历史 190 个发布；近月节奏 8/11、8/24、9/02、9/18 | https://pypi.org/pypi/mem0ai/json |
| 发布形态 | 2026-09-18 同日多组件发版：Python SDK v2.1.0、Node SDK v3.2.0、Vercel AI SDK provider v3.0.3、Pi/OpenCode/OpenClaw/DeepSeek 插件、Python/Node CLI | https://github.com/mem0ai/mem0/releases.atom |
| 最近动态 | 2026-09 议题集中在 v3 可靠性、向量后端缺陷、export 可携性 RFC、faithfulness hook 提案 | https://github.com/mem0ai/mem0/issues?q=sort%3Aupdated |

### 8.2 LoCoMo 基准争议（两边都取到了原文）

**Zep 方**（《Lies, Damn Lies, & Statistics: Is Mem0 Really SOTA in Agent Memory?》2025-05-06，2026-06-03 更新）：
- 自我更正：早期算错自家分，更正后 **Zep 75.14% ± 0.17**，比 Mem0 最佳配置（Mem0 Graph）**相对高约 10%**；而 Mem0 论文里报给 Zep 的是 **65.99%** (来源: https://blog.getzep.com/lies-damn-lies-statistics-is-mem0-really-sota-in-agent-memory/)。
- 指 Mem0 **评测 Zep 的实现有误**：① 给双方都设 `user` 角色（把单用户图搞成"每句话换一个身份"）；② 时间戳拼进消息文本而非用 Zep 的 `created_at`，破坏其时间推理；③ 串行而非并行检索，人为抬高 Zep 的检索延迟（0.778s → 实测并发 p95 0.632s）(来源: 同上)。
- 指 **LoCoMo 本身无效**：会话平均仅 16k–26k tokens（在当代窗口内），**Mem0 自家结果里 full-context 基线 ~73% 反超 Mem0 最佳 ~68%**；不测知识更新；Category 5 缺 ground truth 无法用；BLIP 图像描述缺信息导致多模态题不可答；说话人归属错误；问题欠定（七月八月都露营）(来源: 同上)。
- Zep 主张改用 LongMemEval（平均 115k tokens、显式考时间推理与状态变更、人工策展）(来源: 同上)。

**Mem0 论文侧的原始数字**（2025-04-28）：full-context **72.90**、Mem0^g **68.44**、Mem0 **66.88**、Zep 65.99、OpenAI 52.90、LangMem 58.10、A-Mem 48.38；Mem0 检索 p50 0.148s / p95 0.200s，memory tokens 1764（Zep 3911，full-context 26031）；摘要口径「相对 OpenAI +26%、图版再 +2%、p95 降 91%、省 >90% token」(来源: https://arxiv.org/html/2504.19413v1, https://arxiv.org/abs/2504.19413)。**即「full-context 打赢 Mem0」这张表是 Mem0 自己发表的**，Zep 只是把它读出来了。

**Mem0 方**（2026-04-28 发布、2026-09-03 更新的对比文）：
- 承认「LoCoMo 上两队存在**活跃的评测方法学争议**」(来源: https://mem0.ai/blog/zep-vs-mem0-which-ai-memory-layer-should-you-choose)。
- 换战场到 LongMemEval（**Mem0 93.4 vs Zep 71.2**，+22.2）与新推的 **BEAM**（1M/10M；自认「唯一在真实生产上下文体量上跑的公开基准」）；承认未发 DMR 分数；**承认 Zep 在每查询 token 上更省（~1.6K vs Mem0 ~6.9K）** (来源: 同上)。
- 自家文档同步转向「基准可信度」叙事：明确写「LoCoMo、LongMemEval 这类较小基准可以被激进的检索策略、更大窗口或前沿模型刷高，并不意味着底层记忆系统变好了」；并声明**分数反映含专有优化的 managed platform，OSS 用户不应对齐** (来源: https://docs.mem0.ai/core-concepts/memory-evaluation)。
- 还留了一句难得的诚实话：「Knowledge update（93.6）是 **ADD-only 架构**最难的一类——旧事实被保留而非覆盖，语义相近的旧事实仍会与新事实一起冒出来」；以及承认 open-domain（72.7）"not benefiting and actively being tuned"、BEAM@10M 上 temporal reasoning 16.3 / event ordering 20.2 / multi-session 26.1 属短板 (来源: https://docs.mem0.ai/core-concepts/memory-evaluation)。
- 复现类问题长期存在：`#2967`（closed，2025-06）第三方复现 Zep 基线时时间戳在写入中丢失；议题列表里另有「Failed to reproduce the accuracy on LOCOMO via Mem0 platform」(来源: https://github.com/mem0ai/mem0/issues/2967, https://github.com/mem0ai/mem0/issues?q=locomo)。
- 评测可复核性一侧做得到位：评测框架开源在 `mem0ai/memory-benchmarks`，支持 Cloud/OSS 两种后端、可调 answerer/judge 模型、`--top-k-cutoffs 10,20,50,200`，并注明 ±1 分的 judge 抖动 (来源: https://docs.mem0.ai/core-concepts/memory-evaluation)。

### 8.3 其余公开风险（按治理影响排序）

1. **抽取期无忠实度校验**：社区 `#7283`（open，10 评论，2026-09-22 仍在更新）请求「写入前 faithfulness check」并提供可插拔 verification hook；当前官方链路里没有这一门 (来源: https://github.com/mem0ai/mem0/issues/7283)。
2. **垃圾进、垃圾留**：`#4573` 的 97.8% junk 与 668 条同一反馈环幻觉副本 (来源: https://github.com/mem0ai/mem0/issues/4573)。
3. **静默降级文化**：`#5245`（P1-high、已 accepted 仍 open）、`#7117`、`#7113`、`#4985`、`#7347`——共同模式是「失败被吞、只打日志、返回值看起来成功」，与「unknown ≠ zero」原则直接冲突 (来源: 各 issue 链接)。
4. **删除不完备 + 副本残留**：`#6627`、`#5734`、`#7226`、`#3245` (来源: 各 issue 链接)。
5. **历史与真值不对账**：`#7316` (来源: https://github.com/mem0ai/mem0/issues/7316)。
6. **安全**：`#4875`（PGVector / Azure MySQL / Neptune 经 `collection_name` 的 SQL/Cypher 注入，CWE 标注）、`#3963`（CVE-2026-0994）、`#5151`（共享图记忆的**投毒**级高危漏洞的负责任披露）(来源: https://github.com/mem0ai/mem0/issues/4875, https://github.com/mem0ai/mem0/issues/3963, https://github.com/mem0ai/mem0/issues/5151)。文档也自认「别把 secret 与未脱敏 PII 存进来——记忆按设计就是可被检索的」(来源: https://docs.mem0.ai/core-concepts/memory-types)。
7. **契约不稳**：v3 一次性改名/删除大量参数（`enable_graph`、`graph_store`、`immutable`、`force_add_only`、`includes`/`excludes`、`api_version`、`output_format`、`async_mode`、`keyword_search`、`batch_size`、`custom_update_memory_prompt`、顶层 entity kwargs），`top_k`/`threshold`/`rerank` 默认值全部改动，`get_all` 响应形状改为分页信封，`relations` 保留但恒空 (来源: https://docs.mem0.ai/migration/oss-v2-to-v3, https://docs.mem0.ai/migration/platform-v2-to-v3)。TS/Python 内部字段命名分叉（`text_lemmatized` vs `textLemmatized`）导致同一 collection 跨语言不可用 (来源: https://docs.mem0.ai/migration/oss-v2-to-v3)。
8. **能力披露与营销脱节**：`memory_type` 的 semantic/episodic 定义在枚举里却不实现且传值报错 (来源: https://docs.mem0.ai/core-concepts/memory-types)；`delete_all` 破坏性收紧、Graph view 付费墙同理。文档里也自曝「User Profiles 功能仍在内部定稿，因此本页不把它列为 Platform 卖点」(来源: https://docs.mem0.ai/platform/platform-vs-oss)。
9. **内部数字不一致**：LoCoMo 新算法在 README/评估页为 **92.5**，在两份迁移指南为 **91.6**；平台对比页称「25 vector stores / 18 LLM / 11 embedder」，sitemap 实数 27/17/11；首页称 22 个集成工具，`/integrations` 有 37 页 (来源: 上述各页)。做竞品汇报时不能整段照搬它的数字。

---

## 9 可借鉴点

> 每条都落到 MemorySpace / ChangeSet / Lineage / CopyInventory / RunGate 上的具体动作。

1. **「抽取期只追加，冲突交给检索期」是个已被验证的失败方案——但它反证了我们的候选→审核→发布链路。**
   证据：v2 的 LLM 四操作合并被官方在 v3 里整体撤掉（"Two LLM passes → Single-pass ADD-only"）(来源: https://docs.mem0.ai/migration/platform-v2-to-v3)；撤掉后立刻出现 `#4956`/`#5867`/`#5352` 的矛盾并存与"用户自建 cron 卫生脚本"，且相关提案被 not planned (来源: https://github.com/mem0ai/mem0/issues/4956, .../5867, .../5352)；官方评估页承认 knowledge update 是自家 ADD-only 架构最难一类 (来源: https://docs.mem0.ai/core-concepts/memory-evaluation)。
   结论：变更裁决必须有**归属明确的执行者**（同步 LLM 合并、或异步整理任务、或人），三选一但不能缺位；我们把它做成显式 ChangeSet + Review + Publish，正是它的缺口。

2. **把破坏性动作从写入热路径挪到"异步整理任务"——但要给它加上我们要的可审阅性。**
   Dream 的 Supersede（标旧事实为过时）/ Merge（折重复）在 Platform 全档常开、Synthesis 按需排期，且 **pattern memory 保留指向来源记忆的链接**（它整个产品里唯一像 Lineage 的东西）(来源: https://docs.mem0.ai/platform/features/dream)。
   借鉴：我们的知识化（5.2 记忆→Wiki/Skill/RCA）应有一个"整理任务"角色，输出是**候选 ChangeSet**而不是就地改写；每条派生资产强制带来源集合 + 触发者 + 排期，且幂等（Dream 的"additive and idempotent"值得抄进设计）。

3. **三个存储面分离（SQL 真值 / 向量投影 / 实体投影）+ 把 history 放在 SQL 侧，是 CopyInventory 的最小可行结构。**
   证据：SQL 存 facts+metadata+**history log**+滚动消息窗口；向量库存 embeddings；实体存储存 entity↔memory 关联，各司其职 (来源: https://docs.mem0.ai/core-concepts/memory-evaluation, https://docs.mem0.ai/core-concepts/how-it-works)。
   借鉴：CopyInventory 至少按「真值 / 语义投影 / 关键词投影 / 实体索引 / 派生模式」分型登记，每型各有独立处置与验证路径——`#7226` 的教训正是「实体索引这条副本在删除路径上根本没被走到」(来源: https://github.com/mem0ai/mem0/issues/7226)。

4. **作用域坐标 ≠ 治理容器：把这两层分开摆，和我们的 MemorySpace vs Project 一致。**
   它的 user/agent/app/run 是**归属与过滤坐标**，org/project 才是**权限与配额容器**，且官方明确二者不是一回事（graph entity ≠ entity ID）(来源: https://docs.mem0.ai/platform/features/graph-memory, https://docs.mem0.ai/platform/features/entity-scoped-memory)。
   借鉴：我们的 User/Team/Project/Agent 四维 MemorySpace 归属边界应显式声明「归属不自动等于可见性」，与核心不变量 1（owner ≠ visibility ≠ ACL ≠ execution authority）对齐。

5. **写入异步化 + event_id 可轮询 + 项目级事件流，是审计闭环最省事的骨架。**
   `add` 返回 `status: PENDING` 与 `event_id`，`GET /v1/event/{event_id}/` 给出 SUCCEEDED/FAILED，`GET /v1/events/` 被官方定位成 dashboard/alerting/**audit** 三用 (来源: https://docs.mem0.ai/api-reference/memory/add-memories, https://docs.mem0.ai/api-reference/events/get-events)。
   借鉴：我们的 Evidence 摄取与 Triage 应暴露同样的「受理回执 → 状态可查 → 项目级事件流」三段；并且**不要学它的反面**：OSS 没有项目级事件日志（只有 per-memory history）(来源: https://docs.mem0.ai/platform/platform-vs-oss)——对我们这恰恰应是默认必有。

6. **破坏性批量操作要"显式意图化"。**
   `delete_all` 从「无 filter 清空全项目」改成「无 filter 直接报错」，批量删需把字段显式设 `"*"`，全项目清库要求四个 filter 同时 `"*"` (来源: https://docs.mem0.ai/core-concepts/memory-operations/delete)。
   借鉴：这条正是 RunGate 的"高副作用动作需逐项声明"思路，可以直接引为行业先例；但我们的门槛还多一层：它只是参数仪式，不记录发起人、理由与 Purpose，也不产出处置证明。

7. **"隐藏 ≠ 删除"这条不变量，它可以背书，但方向要反过来。**
   expiration 只是让 `search`/`get_all` 不再返回，记录原地保留、按 ID 仍可取、清日期即恢复 (来源: https://docs.mem0.ai/platform/features/memory-expiration) ——完全对应我们的 `retained ≠ service-visible`。
   要反过来的地方：它「坏日期 fail-open（照旧可见）」，我们的高敏复用/导出/清理声明在证据不可证时必须 **fail-closed**（不变量 6、7）；`YYYY-MM-DD` + 固定 UTC + 含当日的粗颗粒语义也不满足审计需要，我们应存显式时区与判定时刻。

8. **写入期就抽取「时间语义」，并让它在检索期参与打分。**
   Temporal Reasoning 抽出「事件何时发生 / ongoing 还是 completed / 时间精度 / 记忆类型(event, state, plan, preference, relationship, absence)」并作为可查询 metadata；时间分加性且不改候选集 (来源: https://docs.mem0.ai/core-concepts/memory-evaluation, https://docs.mem0.ai/platform/features/temporal-reasoning)。
   借鉴：直接可用于 CodeScenario 的"症状→根因→修复"时序与验收；其中 `absence`（缺席事实）对我们"unknown 与 zero 的区分"很有价值。

9. **检索分数要有可核对的分解，且评测必须报 token 成本。**
   OSS `explain=True` 输出 `score_details`（semantic / 归一化 BM25 / entity boost / 原始合成分 / 最大可达分 / 最终分 / threshold）(来源: https://docs.mem0.ai/core-concepts/memory-operations/search)；评估页强制「报分必报每查询平均 tokens」「同预算同模型才可比」「小基准刷分不等于系统变好」(来源: https://docs.mem0.ai/core-concepts/memory-evaluation)。
   借鉴：ContextManifest 每条记忆带命中信号分解 + 当时可见的策略/模型版本 + 分数；VALIDATION.md 的口径也照此写「token 预算与检索深度必须与基线同约束」。

10. **诚实披露"定义了但没实现"，是产品可信度的正向做法。**
    memory-types 页直接写「semantic/episodic 未实现、传值会报错、仓库里找不到路线图日期」(来源: https://docs.mem0.ai/core-concepts/memory-types)；platform-vs-oss 页写「需要合同性支持 SLA 请先问，目前未作为 Platform 福利记录」(来源: https://docs.mem0.ai/platform/platform-vs-oss)。
    借鉴：我们的文档模板应强制「实现状态列 + 无路线图则注明」，禁止用枚举/营销词冒充能力。

11. **接口层用白名单拒绝未知过滤键（400），比"任何键都接受"更接近治理。**
    Platform 顶层过滤键走固定 allowlist，未知键 400；OSS 任意 metadata 键可过滤，算子支持度还取决于底层向量库 (来源: https://docs.mem0.ai/platform/platform-vs-oss, https://docs.mem0.ai/platform/features/v2-memory-filters)。
    借鉴：ContextManifest/检索接口应 schema 化可过滤字段与算子，并把"字段支持度取决于后端"这类降级显式化，而不是静默少过滤（`#7113` 就是静默丢通配）(来源: https://github.com/mem0ai/mem0/issues/7113)。

12. **自托管版的审计面与鉴权基线值得当作"最低配"抄。**
    请求级实时审计日志（状态/延迟/鉴权模式）+ 逐用户 key（只存前缀 + bcrypt）+ 一次性 admin wizard + `AUTH_DISABLED` 仅限本地 + `JWT_SECRET` 缺失即 500 (来源: https://docs.mem0.ai/open-source/setup)。
    借鉴：企业版若走私有化交付，这是我们对齐线，且必须再加：谁在何 Purpose 下读了哪条（它的 Access Log 只到 API 层）。

13. **可自证的评测框架 + 组件化多包发布，是可复现性的组织形式。**
    `memory-benchmarks` 开源、支持 cloud/oss 双后端、top-k 分档、judge 抖动 ±1 (来源: https://docs.mem0.ai/core-concepts/memory-evaluation)；同一 repo 内按 SDK/插件/CLI 分别打 tag 同日发版 (来源: https://github.com/mem0ai/mem0/releases.atom)。
    借鉴：我们的验收包应同样进仓库可复跑，且发布粒度按「契约 / 后端 / 前端 / agent 插件」分开，避免 v3 那种一次性全量破坏改名。

---

## 10 不可取点

1. **隐式 LLM 合并（v2 式 ADD/UPDATE/DELETE/NOOP）绝不能进我们的写入链路。**
   证据：机制出自论文（"The LLM itself determines which of four distinct operations to execute"）(来源: https://arxiv.org/html/2504.19413v1)；一次误判即不可见地改写/删除真值，无候选、无审阅者、无回滚。官方自己撤了它 (来源: https://docs.mem0.ai/migration/platform-v2-to-v3)。
   对照：我们坚持 抽取产候选 → Triage 定归属 → ChangeSet → Review → Publish；裁决可以是 LLM 建议，但发布权在人与策略。

2. **没有"候选态"，任何抽取结果直接对检索可见。**
   它的链路是 `add` 即产记忆并立即可搜 (来源: https://docs.mem0.ai/core-concepts/how-it-works)，`infer=False` 还额外把原文入库。`#4573` 里 668 条同一幻觉副本、被 agent"记住"了从未被告知的事，就是这条无闸门链路的后果 (来源: https://github.com/mem0ai/mem0/issues/4573)。
   对照：candidate → reviewed → published 三态 + `stale/withdrawn/superseded` 不可省。

3. **没有 Lineage：记忆不携带来源证据、来源 revision 与置信度。**
   抽取用到哪些消息不落库（只有"滚动消息窗口"当去重上下文），SQL history 记 ADD 事件 (来源: https://docs.mem0.ai/core-concepts/memory-evaluation)；社区 `#7376` 直接指出"记忆离开系统只剩文本，没有 state/history/provenance" (来源: https://github.com/mem0ai/mem0/issues/7376)。
   对照：MemoryItem / KnowledgeAsset / ExportAsset / CopyInventory 全链路强制绑定源 revision——否则 5.1 跨 Harness 交接与 5.4 撤回闭包无法成立。

4. **删除没有闭包，"成功"不可信。**
   `delete_all` 静默只删 100 条并报成功 (来源: https://github.com/mem0ai/mem0/issues/6627)；`delete_all` 缺校验导致 no-op 删除 (来源: https://github.com/mem0ai/mem0/issues/5734)；实体索引在纯删除进程里永不清理 (来源: https://github.com/mem0ai/mem0/issues/7226)；历史上删记忆不清 Neo4j (来源: https://github.com/mem0ai/mem0/issues/3245)。
   对照：CopyInventory 必须逐副本（online/cache/index/embedding/manifest/backup/provider/offline）出**处置任务 + 独立验证证据 + unknown 边界**，`requested ≠ executed ≠ verified`。

5. **失败一律降级为"看起来成功"。**
   embedding 部分失败→事实被丢但只 WARNING、调用方无感 (来源: https://github.com/mem0ai/mem0/issues/5245)；换 embedding provider 因维度不匹配静默丢写 (`#4985`)；`_load()` 失败被吞 (`#7117`)；通配被丢 (`#7113`)；BM25 缺依赖时静默降级 (`migration/oss-v2-to-v3`)。
   对照：任何"部分成功/不确定"必须落进状态矩阵的 `partial / unknown / retained`，并在 Home 的 unknown 队列里可见（不变量 4）。

6. **权限模型过薄：2 个角色 + org/project 容器，没有记忆级/字段级 ACL，也没有 Purpose 概念。**
   成员角色仅 `OWNER`/`READER` (来源: https://docs.mem0.ai/api-reference/organization/add-org-member, https://docs.mem0.ai/api-reference/project/add-project-member)；防跨用户靠"你记得传 user_id 过滤"的建议 (来源: https://docs.mem0.ai/core-concepts/memory-operations/search)；Platform 过滤 allowlist 是"字段白名单"而非"授权" (来源: https://docs.mem0.ai/platform/features/v2-memory-filters)。
   对照：VisibilityContext + Purpose + 召回前过滤（不变量 3）必须内建，作用域坐标永远只是索引、不是授权。

7. **没有 RunGate / 执行准入这一层，且把删除类工具直接交给模型。**
   MCP 一把梭暴露 `delete_all_memories`、`delete_entities` 等给 agent，文档鼓励"AI agent 可在数据过时或用户要求时自己删记忆" (来源: https://docs.mem0.ai/platform/mem0-mcp, https://docs.mem0.ai/core-concepts/memory-operations/delete)；同时没有任何"默认 blocked / unknown 即冻结扩大副作用"的语义。
   对照：ActionProposal 默认不可执行，须过当前 RunGate；补偿/回滚/换 Provider 都是新的受授权动作（不变量 8）。

8. **抽取式改写让"记忆与原文不一致"成为常态，且无忠实度闸门。**
   默认存抽取产物而非逐字（`"I prefer aisle seats"` → `"User prefers aisle seats"`）(来源: https://docs.mem0.ai/core-concepts/how-it-works)；写前 faithfulness 检查只是社区待议 PR (来源: https://github.com/mem0ai/mem0/issues/7283)；`#2967` 显示时间戳等属性在抽取中丢失 (来源: https://github.com/mem0ai/mem0/issues/2967)。
   对照：Evidence 原文只读留存 + MemoryItem 存抽取产物，两侧同屏并显示归因；Triage 提供脱敏/忠实度预览。

9. **敏感数据靠"提示用户别存"，不是结构性控制。**
   两处文档都是 warning："避免存 secret/未脱敏 PII，记忆按设计可检索，请先加密或哈希" (来源: https://docs.mem0.ai/core-concepts/how-it-works, https://docs.mem0.ai/core-concepts/memory-types)。
   对照：脱敏与字段投影必须在召回/图分析/渲染之前，且不可被放宽（ToolPolicy 不能被 RunGate 放宽）。

10. **契约留僵尸字段与破坏性大改。**
    `relations` 恒空但保留 (来源: https://docs.mem0.ai/platform/features/graph-memory)；`data` 作为 `text` 弃用别名仍在 (来源: https://docs.mem0.ai/core-concepts/memory-operations/update)；v3 一次性改名/删数十参数并改默认值与响应形状 (来源: https://docs.mem0.ai/migration/oss-v2-to-v3)。

11. **用基准叙事带产品，分数与交付面不匹配。**
    「分数来自 managed platform，OSS 不保证对齐」(来源: https://docs.mem0.ai/core-concepts/memory-evaluation)；同代数字在两页里 92.5 / 91.6 不一致 (来源: README vs migration 页)；README 与旧文档并存两套抽取描述（how-it-works 说 additive，memory-types 说 ADD/UPDATE/DELETE）(来源: https://docs.mem0.ai/core-concepts/how-it-works, https://docs.mem0.ai/core-concepts/memory-types)。
    对照：我们的 VALIDATION.md 每个指标标「测的是 Platform 还是可自托管」、保留可复跑脚本与口径版本。

12. **「图」被卖两次但当前形态是共现加权，不是可查询关系图。**
    v3 后无 typed edges、无 `relations` 载荷、OSS 无可查图 (来源: https://docs.mem0.ai/platform/features/graph-memory, https://docs.mem0.ai/migration/oss-v2-to-v3)；对照论文版才有「实体=节点、关系=带类型边、冲突标记 invalid 而非删除」(来源: https://arxiv.org/html/2504.19413v1)。
    对照：CodeScenario 的因果/依赖/归属关系必须显式建模、可查询、可授权过滤——共现图做不到多跳审计。

13. **同名产品（Platform / OSS / OpenMemory / openmemory.ai / mem0ai/openmemory）语义已经漂移，用户无法判断数据落在哪。**
    同一名字对应 CLI 搬运工具、候补名单式托管产品、本地 MCP 落地页、以及"Upcoming"的控制能力四种状态（见 1.2 来源）。
    对照：命名唯一 + 每个 MemorySpace 明确「数据在哪、谁能读、留多久」；概念模型已在 CONCEPTS.md 落定。

---

## 11 来源清单

**代码仓库与元数据**
- https://api.github.com/repos/mem0ai/mem0 — stars/forks/许可证/时间线（API 间歇限流，已重试取得）
- https://api.github.com/repos/mem0ai/mem0/languages · https://api.github.com/orgs/mem0ai/repos · https://api.github.com/repos/mem0ai/openmemory
- https://img.shields.io/github/stars/mem0ai/mem0.json · /contributors · /last-commit — 66k★、399 贡献者、最近提交"上周"
- https://raw.githubusercontent.com/mem0ai/mem0/main/README.md — New Memory Algorithm (April 2026)、基准表、改动清单
- https://raw.githubusercontent.com/mem0ai/openmemory/main/README.md — 当前为会话搬运 CLI
- https://github.com/mem0ai/mem0/releases.atom — 2026-09-18 多组件同日发版
- https://pypi.org/pypi/mem0ai/json — `mem0ai` 2.1.0 / 2026-09-18 / 190 个发布
- https://docs.mem0.ai/sitemap.xml · https://docs.mem0.ai/llms.txt — 页面全量清单与集成叙述

**官方文档（均以 `<url>.md` 取原文）**
- 概念：/introduction · /core-concepts/how-it-works · /core-concepts/memory-types · /core-concepts/memory-operations/{add,search,update,delete} · /core-concepts/memory-evaluation
- 定位与差异：/platform/overview · /platform/platform-vs-oss · /open-source/overview · /open-source/setup · /open-source/features/{overview,metadata-filtering,reranker-search,rest-api,openai-compatibility}
- 能力：/platform/features/{graph-memory,entity-scoped-memory,advanced-retrieval,custom-categories,dream,memory-decay,memory-expiration,memory-export,feedback-mechanism,group-chat,temporal-reasoning,v2-memory-filters,webhooks,direct-import,multimodal-support}
- **文档死链（实测 404，本身就是不可取点第 9 条的证据）**：`/platform/advanced-memory-operations` 返回 404，却被 /core-concepts/memory-operations/add（"Advanced Memory Operations 指南"）与 /core-concepts/memory-types（同名卡片）当作下一步链接抛出 (来源: https://docs.mem0.ai/core-concepts/memory-operations/add, https://docs.mem0.ai/core-concepts/memory-types, HTTP 状态核验 2026-09-22)。另 `/platform/security` 亦为 404（安全面在 docs 中无独立页，合规声明只见于营销页与 trust.mem0.ai）。
- API：/api-reference/memory/{add-memories,search-memories,update-memory,delete-memory,batch-delete,batch-update,get-memories,get-memory,history-memory,feedback,create-memory-export,get-memory-export} · /api-reference/organizations-projects · /api-reference/organization/add-org-member · /api-reference/project/add-project-member · /api-reference/events/get-events · /api-reference/entities/delete-user · /api-reference/webhook/*
- 迁移（含旧机制与成本）：/migration/oss-v2-to-v3 · /migration/platform-v2-to-v3 · /migration/oss-to-platform · /migration/server-pgvector-upgrade
- 生态：/platform/mem0-mcp · /integrations · /changelog/highlights · /components/{vectordbs,llms,embedders,rerankers}/*

**论文与基准**
- https://arxiv.org/abs/2504.19413 · https://arxiv.org/html/2504.19413v1 — ADD/UPDATE/DELETE/NOOP 四操作、Neo4j 有向标注图与"标 invalid 不物删"、Table 2 全量分数与延迟
- https://github.com/snap-research/locomo · https://github.com/xiaowu0162/LongMemEval · https://github.com/mem0ai/memory-benchmarks
- https://mem0.ai/blog/zep-vs-mem0-which-ai-memory-layer-should-you-choose · https://mem0.ai/blog/understanding-memory-benchmark-for-production-ai-agents · https://mem0.ai/compare/mem0-vs-zep

**基准争议（对家口径）**
- https://blog.getzep.com/lies-damn-lies-statistics-is-mem0-really-sota-in-agent-memory/ — Zep 更正后 75.14%±0.17、三项实现错误、LoCoMo 缺陷清单、LongMemEval 主张

**产品与合规**
- https://mem0.ai/pricing — Free/Pro/Enterprise 与 audit logs / SSO 分档
- https://openmemory.ai/ · https://mem0.ai/openmemory · https://mem0.ai/openmemory-mcp-3 — OpenMemory 三种并存叙述
- https://trust.mem0.ai/ · https://mem0.ai/（页脚 HIPAA / SOC 2 Type I / GDPR 声明）

**议题（问题证据）**
- https://github.com/mem0ai/mem0/issues/4573 （97.8% junk）· /issues/4956 · /issues/5867 · /issues/5352 （ADD-only 矛盾与社区自救）
- /issues/5245 （静默丢写，P1 accepted）· /issues/4985 · /issues/7113 · /issues/7117 · /issues/7347
- /issues/6627 · /issues/5734 · /issues/7226 · /issues/3245 （删除闭包与副本残留）
- /issues/7316 · /issues/4467 · /issues/6512 （history/真值对账与可配置性）
- /issues/7376 （export 可携性 RFC）· /issues/7283 （写入前 faithfulness 校验）· /issues/5330 （遗忘曲线清理提案）
- /issues/4875 · /issues/3963 · /issues/5151 （注入、CVE、共享图记忆投毒披露）
- /issues/2967 · /issues?q=locomo （复现类）
- https://raw.githubusercontent.com/mem0ai/mem0/main/mem0/memory/main.py — 源码直读核验：`reset()` 实现（2130-2152）、`update()` 写 history 的字段（2078-2090）、v3 ADD-only 管道与批量 embedding 回退路径（770-783 见 #5245 引用）
- https://github.com/Ardem2025/mem0-temporal-hygiene （#5352 中第三方卫生脚本，用于佐证缺口）

---

### 一页结论（给后续决策用）

1. mem0 的技术叙事已经**从"智能合并"自我反转为"只追加"**；这不是小改良，而是它把招牌机制撤下。它替我们证明了：写入期 LLM 裁决（无论做还是不做）都不如一条**显式候选→审核→发布**的链路。
2. 它真正做对的三件事可以拿去：真值与投影分面、写入受理回执 + 事件流、时间语义在写入期落地；以及诚实标注"未实现"的文档风格。
3. 它的治理空白几乎就是我们的产品定义：无候选态、无血缘、无删除闭包、无字段投影、无 Purpose、无执行准入、权限两档、失败一律降级——**`#6627`（删 100 条报成功）、`#4573`（97.8% junk）、`#5352`（用户自建 cron 清理）三个证据足以支撑对外叙事**。
4. 生态与规模差距真实存在（66k★、190 次发布、37 集成页、9 个 harness 插件、含本 harness 的 Cordis 插件）；我们的可辩护点不在「记忆抽取质量」，而在「可审计、可撤回、可执行准入」。
5. 它的基准数字不可直接引用（平台专有优化、92.5/91.6 双口径、full-context 反超、Zep 75.14% 争议 + 对家自我更正），对外竞品叙述必须双边引用、标注争议未决。
