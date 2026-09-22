# RECON: 其他记忆系统全景
> 状态：调研完成 · 已过审（2026-09-22 18:56 · 用户原话「通过」，随 v0.3 审核门）（深查组 4 家独立复核 + 速览组 12 家）· 日期：2026-09-22 · 联网验证：全部事实句基于当日 curl/web_fetch 直抓（GitHub API/raw、官方文档、arXiv、产品站）；GitHub `/repos` API 中途限流，stars 数经 HTML/shields.io/search API 双源核对；未能验证处标注「未验证」。本页「本产品」指 Enterprise Agent Memory Platform（MemorySpace / Triage→ChangeSet→Review→Publish / Lineage / CopyInventory / RunGate / ContextManifest，见 `../CONCEPTS.md`）。

---

# A 深查组

## A1 Zep / Graphiti（getzep/graphiti · Zep Cloud）

### 1. 定位
- 双轨：Graphiti = Apache-2.0 开源时序知识图谱（temporal knowledge graph / "Context Graph"）构建与查询框架（Python 库 graphiti-core，自带图库 Neo4j/FalkorDB/Neptune）；Zep Cloud = 托管 agent memory 平台，口号「governed Context Lake of temporal knowledge graphs, sub-200ms」。"Graphiti builds the graph; Zep operates it at scale"。(来源: https://raw.githubusercontent.com/getzep/graphiti/main/README.md ; https://help.getzep.com/llms.txt ; https://help.getzep.com/zep-vs-graphiti.md)
- 自托管 Zep Community Edition 已宣布废弃，替代=Zep Cloud / Graphiti OSS / 企业 BYOC——OSS 只剩引擎层。(来源: https://help.getzep.com/faq.md ; https://blog.getzep.com/announcing-a-new-direction-for-zeps-open-source-strategy/)
- graphiti ≈31.1k stars、278 open issues、最新 v0.30.2（2026-09-08）。(来源: https://github.com/getzep/graphiti ; https://github.com/getzep/graphiti/releases.atom ; 本会话 HTML 抓取核对 31071 stars)

### 2. 记忆模型
- 三层子图：episode 子图（原始消息/文本/JSON，无损留底）→ 语义实体子图（节点=实体，边=fact 三元组，SCREAMING_SNAKE 关系名如 WORKS_AT）→ 社区子图（label propagation 社区检测+LLM 摘要）；episode 经 MENTIONS 边挂到派生实体。(来源: https://arxiv.org/html/2501.13956v1 ; https://help.getzep.com/graphiti/core-concepts/adding-episodes.md)
- 云端把上下文拆为可检索「context types」：facts / entities / episodes / thread summaries / observations / user summary；支持 Pydantic 式自定义实体/边类型（prescribed 或 learned ontology）。(来源: https://help.getzep.com/concepts.md)

### 3. 写入链路与合并策略
- graph.add / add_episode 提交后自动跑 6 步 LLM 管线：抽实体→抽关系与 fact→日期标注（以 episode reference_time 解析"上周"等相对时间）→实体消解→fact 消解（重复合并、矛盾使旧边失效）→实体摘要更新，随后 embed 持久化。全异步（HTTP 202）。(来源: https://help.getzep.com/how-graph-creation-works.md)
- 矛盾消解：LLM 比对新边与语义相关旧边，时间重叠矛盾即把旧边 `invalid_at` 置为新边 `valid_at`——**失效而非删除**；实体消解原则「宁欠合并不过合并」（错合并难撤销，重复节点靠高召回检索兜底）；抽取带 reflexion 式自我复核减少幻觉边。(来源: https://arxiv.org/html/2501.13956v1 ; https://help.getzep.com/how-graph-creation-works.md)
- **无人工审核环节**：抽取即写图，只能事后手工增删改；批量接口 add_episode_bulk 明确不做边失效、仅适合空图回填。(来源: https://help.getzep.com/adding-fact-triplets.md ; https://help.getzep.com/sdk-reference/graph/add-data.md)

### 4. 检索
- 混合三路：语义向量+BM25 全文+图遍历（BFS），RRF 融合；OSS 提供 15 个预置 recipe（MMR/节点距离/episode 提及数/cross-encoder 重排），可搜 edges/nodes/communities。(来源: https://help.getzep.com/graphiti/working-with-data/searching.md)
- 云端 auto search：跨 scope 并行取回+rerank+按 max_characters 预算装配成单个 Context Block（模板如 `%{edges limit=10}`），可 return_raw_results 取回明细；embedder 故障自动降级 BM25。宣称 P95<200ms；LoCoMo 94.7%@155ms、LongMemEval 90.2%@162ms；论文对 MemGPT：DMR 94.8% vs 93.4%。(来源: https://help.getzep.com/searching-the-graph.md ; https://help.getzep.com/retrieval-philosophy.md ; https://help.getzep.com/llms.txt ; https://arxiv.org/abs/2501.13956)
- 哲学明确：默认高召回优先（怕漏关键事实胜过怕噪声），用 filter/limit 拉回精确率。(来源: https://help.getzep.com/retrieval-philosophy.md)

### 5. 失效与时间线
- 每条 fact 边 4 时间戳，bi-temporal：事件线 `valid_at/invalid_at`（事实何时开始/停止为真）+事务线 `created_at/expired_at`（系统何时得知/得知失效）；支持"现在为真"与"任一时点为真"两类查询；失效事实保留可追溯，Context Block 中渲染为 "(Date range: from - present)"。(来源: https://help.getzep.com/facts.md ; https://arxiv.org/abs/2501.13956)

### 6. 治理
- 多租户：user graph / standalone graph 完全隔离无共享状态；OSS 用 group_ids 命名空间。(来源: https://help.getzep.com/faq.md ; https://help.getzep.com/graphiti/core-concepts/graph-namespacing.md)
- 访问控制二分：人→RBAC+企业 SSO；agent/MCP 调用方→挂在 API key 与 UserGroups 上的 ABAC 策略；episode 摄入时的 metadata **投影到派生 fact**，使"按来源限权/过滤"无需另建权限系统。(来源: https://help.getzep.com/governance.md ; https://help.getzep.com/episode-metadata-projection.md)
- 删除合规：user/graph/node/edge/episode/thread 各有删除端点；级联语义明确——删 node 联删边、删边留孤立节点；删 episode 仅当**无其他 episode 支持**时才删派生物，但已触发的失效不回滚、节点摘要不重算。(来源: https://help.getzep.com/sdk-reference/user/delete.md ; https://help.getzep.com/deleting-data-from-the-graph.md)
- 合规：SOC 2 Type II、HIPAA BAA（Enterprise）、trust.getzep.com 状态页、BYOK（客户 AWS KMS 密钥）、BYOM（自带模型凭据）、Cloud/BYOC。GDPR/EU AI Act 专页当前文档未检索到——未验证。(来源: https://help.getzep.com/security-compliance.md)
- 审计：Audit logging 仅 Enterprise 且只覆盖 dashboard 人员操作；API/SDK 调用另走 API logging；Support 可授 72h 只读临时访问。(来源: https://help.getzep.com/audit-logging.md ; https://help.getzep.com/support-access.md)

### 7. 界面与接入
- SDK Python/TS/Go + REST；LangGraph/CrewAI/ADK/AutoGen 集成；Cloud 有 Dashboard（图可视化、debug/API 日志、Playground）；Graphiti OSS 无 UI（仅 MCP server + FastAPI 示例服务）。写入 thread.add_messages / graph.add，读取 Context Block。(来源: https://help.getzep.com/memory-for-agent-frameworks.md ; https://help.getzep.com/zep-vs-graphiti.md ; https://raw.githubusercontent.com/getzep/graphiti/main/README.md)

### 8. 成熟度与公开问题
- 月度发版；v0.28.2 是安全补丁「Harden Search Filters Against Cypher Injection」——检索过滤器曾是注入面。典型 issue：换后端需求集中（Postgres+pgvector #779、Bedrock #459）、Kuzu 归档致依赖风险 #1132、GPT-5 参数兼容 #878。已知局限：摄入重度依赖 LLM 结构化输出（小模型 schema 失败）、默认并发 10 防 429、社区摘要需周期全量刷新、默认开 PostHog 遥测。(来源: https://github.com/getzep/graphiti/releases.atom ; https://github.com/getzep/graphiti/issues?q=is%3Aissue+sort%3Areactions-%2B1-desc ; https://raw.githubusercontent.com/getzep/graphiti/main/README.md)
- 2026-07 官方博客《How Zep tracks provenance in agent memory》把「LLM 抽取破坏来源跟踪，是**所有**基于抽取的记忆系统的共性问题」当作卖点，provenance=图结构本身。(来源: https://blog.getzep.com/how-zep-tracks-provenance-in-agent-memory/)

### 9. 可借鉴（对本平台）
- bi-temporal 4 时间戳 + "失效不删除" → Lineage/revision 的最小充分时间模型。
- episode 不可变留底 + 派生物经图边回溯 episode + 来源 metadata 投影到派生 fact 供 ABAC 评估、删除沿支持关系传播 → 正是 Lineage + CopyInventory 撤回传播的参考实现（含"无其他来源支持才删"的判定规则）。
- 「宁欠合并不过合并」+高召回检索兜底 → ChangeSet 合并策略的安全默认值。
- RBAC（人）/ABAC（agent）二分 + metadata 投影限权 + 时间受限 support 访问 + dashboard/API 双日志分离 → 治理面分层模板。
- 预算化 Context Block（字符预算装配、return_raw 供引用核验）+「检索结果必须按不可信数据通道注入、行动授权留在应用侧」的安全文档 → 直接化为 ContextManifest 输出契约与 RunGate 的必要性论证。

### 10. 不可取（对本平台）
- 全自动 Triage→写图，无 Review/Publish 门：错误合并/幻觉边只能事后删改，与 ChangeSet→Review→Publish 正相反。
- 无行动前准入（文档明言 admission/行动授权归客户），记忆投毒只能靠 prompt 卫生缓解——我们的 RunGate 恰补此洞。
- 失效不可逆、删除不重算摘要：被错误 invalid 的 fact 永久失效、episode 删后节点名/摘要残留——缺"副本闭包+重算"，CopyInventory 需覆盖这类衍生残留。
- 审计日志不覆盖数据面 API、GDPR 条款无专页（未验证）；引擎专有化+CE 废弃=深度审计只能走企业 BYOC，供应商锁定。
- 成本结构：每 episode 多趟 LLM + cross-encoder，摄入成本/限流是运营常态，需在批量与 RunGate 中显式定价。

## A2 Letta（原 MemGPT，letta-ai/letta）

### 1. 定位
- 源自 Berkeley「LLM as OS」研究（MemGPT 论文 2023-10），自我定位"做会学习的 stateful agents 的研究实验室"。(来源: https://raw.githubusercontent.com/letta-ai/letta/main/README.md ; https://arxiv.org/abs/2310.08560)
- **2025-10 起战略转型**：主推 TS harness「Letta Code」（npm @letta-ai/letta-code），旧 Python server 归档（letta-ai/letta 主 README 改为指针，"新项目请用 letta-code"）；2026-03 博客《Letta's Next Phase》确认。(来源: https://raw.githubusercontent.com/letta-ai/letta/main/README.md ; https://raw.githubusercontent.com/letta-ai/letta/archive/README.md ; https://www.letta.com/blog/our-next-phase/)
- 开源 harness/旧 server/UI；Letta Cloud 托管层：BYOK 免费 3 agent、Pro $20/m、API $0.10/active agent/mo + $0.00015/s 工具执行、Enterprise SAML/OIDC。(来源: https://docs.letta.com/pricing)
- stars：letta ≈24.8k、letta-code ≈3.4k。(来源: https://github.com/letta-ai/letta HTML；https://img.shields.io/github/stars/letta-ai/letta.json)

### 2. 记忆模型（两代）
- V1/MemGPT OS 分页隐喻：main context（系统指令+working context FIFO 队列）↔ external context（archival 向量库+recall 对话史，类比磁盘）；LLM 以 function call 在两层级间搬数据；占用达阈值注入 "memory pressure" 警告、超限逐出；request_heartbeat 支持函数链式多步检索。(来源: https://arxiv.org/abs/2310.08560)
- core memory blocks＝上下文内划额命名块：`label/description/value/limit`（persona/human 默认 20k 字符）+ `read_only` + tags；块可跨 agent attach 共享；agent 用 core_memory_append/replace（v2 加 apply_patch/rethink）自编辑。(来源: https://web.archive.org/web/20250711172613/https://docs.letta.com/guides/agents/memory-blocks ; https://raw.githubusercontent.com/letta-ai/letta/archive/letta/schemas/block.py ; https://www.letta.com/blog/memory-blocks/)
- 新模型 MemFS（2026）：**记忆=agent 拥有的 git 仓库**，Markdown+YAML frontmatter 投影为文件；`system/` 下文件每轮全量进 system prompt，其余按需读、文件树常驻 prompt 作路标；label 即路径；共享记忆改为组织所有的 git 仓库（旧共享块列为 legacy 待迁移）。(来源: https://docs.letta.com/concepts/memfs ; https://docs.letta.com/concepts/shared-memory)

### 3. 写入链路与合并策略
- V1 在线自编辑被官方自评为"慢、与业务工具混用不可靠、增量写致记忆脏乱"→ MemGPT 2.0 sleep-time agents（0.7.0，2025-04）：双 agent——primary 无编辑工具，sleep-time agent 独占编辑权、异步整理、anytime 写（主 agent 不必等完成）；两 agent 各自配模型。(来源: https://www.letta.com/blog/sleep-time-compute/)
- Letta Code「dreaming」：后台 subagent 复盘近期对话→合并经验→commit 进 MemFS；触发=每 N 步或上下文压缩时；`/init` 初始化、`/remember` 显式教学、`/doctor` 审计重复/膨胀；可配「Agent reviews before applying」二次自审——**明示不请求人类批准**；reorganize 前自动备份仓库。(来源: https://docs.letta.com/configuration/memory)
- 写入无审核门；唯一硬边界是 git commit（"已提交=生效，未提交≠生效"）；dreaming 用 git worktree 并发写不阻塞主 agent。(来源: https://docs.letta.com/concepts/memfs)

### 4. 检索
- V1：archival_memory_search 语义 top_k + tags + 时间范围；conversation_search 混合（文本+语义）+role/日期；底座 PostgreSQL+pgvector(HNSW)。(来源: https://raw.githubusercontent.com/letta-ai/letta/archive/letta/functions/function_sets/base.py ; https://arxiv.org/html/2310.08560v2)
- MemFS：**默认无向量索引**，agent 用文件搜索/读找记忆；semantic/hybrid 需自装 memfs-search mod+QMD；对话检索云端 full-text+vector+hybrid、本地仅 full-text。(来源: https://docs.letta.com/concepts/memfs)

### 5. 失效与时间线
- 无 bi-temporal、无失效标记；V1 块更新=覆盖写（schema 仅 last_updated_by_id，无版本表）→覆写丢历史；MemFS 以 git commit 提供文件级版本史/冲突解决/回滚，但非语义级血缘。(来源: https://raw.githubusercontent.com/letta-ai/letta/archive/letta/schemas/block.py ; https://docs.letta.com/concepts/memfs)

### 6. 治理
- Cloud 组织：Admin/Editor/Analyst；agent/会话默认私有、opt-in 共享；共享 agent≠共享会话；工具审批按计算机独立生效。(来源: https://docs.letta.com/teams/permissions ; https://docs.letta.com/concepts/shared-memory)
- 合规：Enterprise 档仅列 RBAC+SAML/OIDC；无公开 SOC2/GDPR/信任中心（/security、trust.letta.com 均 404/不可解析）——未验证；审计日志文档未见。(来源: https://docs.letta.com/pricing ; curl 状态码验证)

### 7. 界面与接入
- 形态矩阵：CLI/TUI、桌面 app、chat.letta.com、Slack/Telegram/Discord/WhatsApp/Signal、GitHub Action、ACP 适配器、TypeScript Agent SDK、自托管 App Server（WebSocket 协议+可选 OpenAI 兼容 REST）；V1 时代 ADE 可视化开发环境已随转型淡出；.af AgentFile 导出已弃用。(来源: https://raw.githubusercontent.com/letta-ai/letta-code/main/README.md ; https://docs.letta.com/self-hosting/app-server)

### 8. 成熟度与公开问题
- Python server 停更于 v0.16.8 后归档；letta-code 自 2025-10 已 325 版、月均 20–44 发、仍 0.x；官方论坛 2026-06 弃用；GitHub issue 排行因 API 限流未验证。(来源: https://img.shields.io/github/v/tag/letta-ai/letta.json ; https://registry.npmjs.org/@letta-ai/letta-code ; https://forum.letta.com/top.json)
- HN 质疑 MemFS 路线"记忆归根到底是 markdown 文件，差异在哪"（vs CLAUDE.md 类做法）；V1→V2 剧烈重做本身说明 API-server 形态未达预期；文档整站迁移致旧链接 404。(来源: https://hn.algolia.com/api/v1/search?query=%22letta%22%20memory&tags=comment ; https://www.letta.com/blog/our-next-phase/)

### 9. 可借鉴
- 命名块＝"带显式配额的上下文单元"（label/description/value/limit/read_only）：limit 防记忆挤占、read_only 支持"平台注入 agent 只读"——可映射为 ContextManifest 授权槽位。
- 在线轻写入 + 离线整理双轨（primary 禁编辑、sleep-time/dreaming 专职合并、后台可用更强慢模型）＝ Triage/合并引擎的理想形态。
- 记忆做成 git 仓库：免费获得 ChangeSet/历史/回滚/"未提交≠生效"边界/worktree 并发——我们的 ChangeSet 语义可借其机械层，但必须在其上加人工 Review 门。
- 两级上下文（system/ 常驻 + 按需文件读、文件树即索引）＝ContextManifest 的朴素参考；"默认无向量、文件检索优先"黑盒少、可审计。

### 10. 不可取
- 无人工审核门：agent 自动 commit、dreaming 自审但"不请你批准"——企业场景不可接受，恰是我们 Review/Publish 的差异化。
- V1 覆写即丢历史、无时间线失效、无来源证据链；persona 可被 agent 自改无门控（身份漂移+注入面）。
- 共享仓库/detach 后的本地副本无闭包与撤回传播语义——CopyInventory 正是其空白。
- 把记忆绑在易变应用格式上（.af 弃用、V1 归档、文档迁移 404 遍地）——教训：治理层与存储语义要稳定于产品形态。

## A3 AWS Bedrock AgentCore Memory

### 1. 定位
- AgentCore 家族中与 Runtime/Gateway/Identity/Observability/Evaluations/Harness/Policy 并列的独立数据面服务；任意框架可直调，托管 harness 缺省配 managed memory（CLI 默认关）。(来源: https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/what-is-bedrock-agentcore.html ; https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/harness-memory.html)
- 时间线：2025-07 Preview → **2025-10-13 GA**（9 regions）→ 2025-12 Episodic → 2026-03 record streaming/资源策略/PrivateLink → 2026-04 结构化 metadata 过滤 → 2026-08 IngestData 直投+自定义 namespace 变量+JSON 事件+GovCloud。现 16 Region（含 GovCloud）。(来源: https://aws.amazon.com/blogs/machine-learning/amazon-bedrock-agentcore-is-now-generally-available/ ; release-notes / agentcore-regions.html)
- 定价：短期 $0.25/1k events；长期 built-in $0.75/1k records·月、override/self-managed $0.25/1k；检索 $0.50/1k retrievals。(来源: https://aws.amazon.com/bedrock/agentcore/pricing/)

### 2. 记忆模型
- 三层术语：Memory（顶层容器）→ **Event**（短期不可变，actorId+sessionId 组织；payload=对话消息/JSON ≤100KB/binary；metadata ≤15 键且明示"不敏感内容用、不受 CMK 加密"）→ **MemoryRecord**（长期，content text ≤16KB + ≤20 metadata + 策略 ID，只挂 1 个 namespace ≤1024 字符）。(来源: memory-types.html ; memory-terminology.html ; APIReference/API_Event.html、API_MemoryRecord.html)
- 策略三档：built-in（零配置、存储最贵）/ built-in with overrides（改指令+自选 Bedrock 模型、客户账户内执行、耗客户配额）/ self-managed（SNS+S3 触发、批量写回、schema 自定）——"托管便利 vs 自控便宜"做成价格分层。(来源: memory-strategies.html ; memory-self-managed-strategies.html ; bedrock-capacity.html)
- built-in 四种及输出形态：Semantic=单条事实 JSON{language,fact}；UserPreference=context+preference+categories；Summary=session 级 XML `<topic>`；Episodic=XML 情景（situation/intent/assessment/justification）+跨 episode reflection。(来源: semantic/user-preference/episodic/summary-memory-strategy.html ; API_MemoryStrategy.html)
- namespace 模板：内置 {actorId}/{sessionId}/{memoryStrategyId} + 自定义 {org}/{team}（namespaceKeys ≤5，运行时 extractionConfig.namespaceVariables 注入）。题设"exact message buffering"现役文档不存在——未验证（等价物=短期 raw event）。(来源: specify-long-term-memory-organization.html)

### 3. 写入链路与合并策略
- CreateEvent 或 IngestData（只抽不留原文）触发；**只有策略 ACTIVE 之后的事件才进抽取**（不回填历史）；后台异步"1 分钟或更久"；失败进专用队列：ListMemoryExtractionJobs（failureReason 码表）→ StartMemoryExtractionJob 手动 redrive，FailedExtraction 指标可告警。(来源: long-term-saving-and-retrieving-insights.html ; long-term-ingest-data.html ; long-term-redrive.html)
- 合并语义有公开系统 prompt 可查：consolidation 每步三选一 **AddMemory / UpdateMemory（新信息并入既有记录并保留原文）/ SkipMemory**，矛盾按 confidence 判定；去重/取代以**删除**落地（触发 MemoryRecordDeleted 流事件）——就地改写、无版本链。(来源: memory-system-prompt.html ; memory-user-prompt.html ; memory-record-streaming.html)
- 吞吐硬限：CreateEvent 200 TPS、IngestData 10 TPS、抽取 150k TPM（与 Bedrock 配额强耦合）。(来源: bedrock-agentcore-limits.html)

### 4. 检索
- RetrieveMemoryRecords=namespace（精确）或 namespacePath（前缀）内向量检索：searchQuery ≤10k 字符、topK 默认 10 最大 100、余弦 score（文档强调非百分比）、metadataFilters ≤5 AND；ListMemoryRecords 非语义浏览，可按系统字段 `x-amz-agentcore-memory-createdAt/updatedAt/recordType` 做 BEFORE/AFTER 时间过滤。关键词检索未见支持——未验证；无延迟 SLO。(来源: long-term-retrieve-records.html ; long-term-list-memory-records.html ; long-term-memory-metadata.html)
- 短期侧 ListSessions/ListEvents/GetEvent+metadata 过滤+**branch（会话分叉）**。(来源: memory-types.html ; API_Event)

### 5. 失效与时间线
- event：`eventExpiryDuration` 逐事件 3–365 天，**改配置不影响历史、过期不可延长不可恢复**；record：仅 createdAt/updatedAt，**无 valid/invalid、无 TTL、无版本历史**——回溯只能自建：record→Kinesis streaming（FULL_CONTENT/METADATA_ONLY）落数据湖 + CloudWatch spans 的 event↔record 关联属性。(来源: memory-create-a-memory-store.html ; API_MemoryRecord ; memory-record-streaming.html ; observability-memory-metrics.html)

### 6. 治理
- IAM 数据面动作逐项授权；条件键细到 **namespace 精确 / namespacePath 层级 / namespaceVariable/<key>（写路径强制租户值，含"缺键即 Deny"求值表）**；2026-03 起资源型策略+跨账户数据面。(来源: memory-organization.html ; specify-long-term-memory-organization.html ; resource-based-policies.html ; memory-cross-account-access.html)
- Gateway+OIDC 前置的 **Cedar FGAC**：按终端用户身份强制 per-user/actor/namespace 隔离，deny-by-default；⚠️ BatchCreate/Update/DeleteMemoryRecords 三写 API 不支持 FGAC。(来源: memory-advanced.html ; memory-gateway-fgac.html)
- 合规：随 AWS 体系（HIPAA-eligible、FedRAMP High、SOC/ISO 全家，compliance-validation 页）；**抽取走"同地理内跨区推理"——prompt 可能离开主 Region**（可用 override 自选同区模型规避）；加密 CMK 可选（encryptionKeyArn）但 event metadata 不覆盖；**Memory 专属 CloudTrail 数据事件无文档**——未验证。(来源: compliance-validation.html ; cross-region-inference.html ; data-encryption.html ; enabling-cloudtrail-data-event-logging.html)
- 删除：DeleteEvent / DeleteMemoryRecord（需已知 recordId+namespace）/ BatchDeleteMemoryRecords / DeleteMemory（整资源）。STRICTLY_CONSISTENT metadata 模式：按值分组、抽取永不跨组合并（"hipaa 记录永不与 standard 合并"）——合规隔离的低成本原语。(来源: long-term-delete-memory-records.html ; long-term-memory-metadata.html)

### 7. 界面与接入
- Console 有创建向导（事件过期天数、KMS、策略多选）+详情页内嵌 CloudWatch 指标；**无 record 级浏览/编辑界面**（未验证）。CLI `@aws/agentcore`、boto3 双 client、MemoryClient/MemorySessionManager 高层 SDK、LangGraph checkpointer/Strands 集成、CloudFormation/CDK L2（2026-05 stable）。(来源: memory-create-a-memory-store.html ; memory-get-started.html ; agentcore-sdk-memory.html ; memory-integrate-lang.html ; what-is-bedrock-agentcore.html)

### 8. 成熟度与公开问题
- 配额：150 memories/region、6 strategies/memory、record ≤16KB、metadata ≤20、indexed keys/namespace keys 一旦声明不可删（须先解除策略引用）。限制：semantic/preference 策略只处理 USER/ASSISTANT 消息；无策略=零长期记忆；策略变更有激活滞后；高峰需工单提 TPM。(来源: bedrock-agentcore-limits.html ; long-term-memory-metadata.html ; memory-strategies.html)
- **记忆投毒/prompt 注入被文档划为客户责任**（类比"RDS 安全但客户自防 SQL 注入"），产品侧无内建防护，仅建议入参清洗。(来源: best-practices.html)

### 9. 可借鉴
- **namespace 条件键三件套**（精确/前缀/变量强制）：读写两路共用同一组键——RunGate/ContextManifest 的授权快照可直接套这套语义；`namespaceVariables`"缺键即 Deny"值得照抄为写路径硬约束。
- **record streaming 作一等公民**（Kinesis，变更事件含 recordId/strategyId/namespace/时间戳）= Lineage/CopyInventory 的天然事件源，优于轮询 diff。
- **抽取作业化**：extraction jobs+failureReason 码表+redrive+FailedExtraction 告警——Triage 异步链路照此产品化（我们的"静默不生成"要有显式队列可查）。
- 三档策略价格分层（托管便利贵/自控便宜）把"可控性"做成商品选项——我们 Triage 引擎也该给 built-in vs BYO-model 分层。
- **公开内置系统 prompt** 让抽取/合并行为可审计；STRICTLY_CONSISTENT 分组合并=合规隔离原语。
- Gateway+Cedar 把"终端用户→actor/namespace 准入"从应用代码搬到基础设施，deny-by-default——RunGate 的工业参照。

### 10. 不可取
- **就地改写+去重即删除**：无 revision、无 valid/invalid、无变更前证据链——Lineage/ChangeSet 恰补此洞；撤回只能逐条已知 recordId 删，跨系统副本失控（CopyInventory 空白）。
- 零人工关口：ACTIVE 策略后台 LLM 直接落库，文档自认投毒威胁却把防御外包客户——"记忆变更过人工审核"是对它的直接卖点。
- Batch 三写 API 绕过 FGAC=权限体系有旁路；event metadata 不入 CMK、Memory 数据面 CloudTrail 无文档——审计粒度不达企业线。
- 检索只有"namespace+topK+score"，无装配快照，事后无法证明"当时喂给模型什么、授权态如何"——ContextManifest 差异点。
- 跨区推理使 prompt 出境（须自选模型规避）+150k TPM 与 Bedrock 配额耦合=合规与容量都要客户自运营，平台应把背压/积压重放做成产品能力。
- 无延迟 SLO/无关键词检索/record 16KB 上限：长文档型记忆与团队共享场景扩展性差（6 策略/150 memories 配额同样偏 C 端会话假设）。

## A4 Agent Platform Memory Bank（原 Vertex AI Agent Engine Memory Bank，Google）

### 1. 定位
- Agent Platform（原 Vertex AI Agent Engine，2026-04-22 随"Gemini Enterprise Agent Platform"改名）下的托管记忆服务：「generation, storage, retrieval and embedding」全管。(来源: https://docs.cloud.google.com/gemini-enterprise-agent-platform/scale/memory-bank ; https://cloud.google.com/products/gemini-enterprise-agent-platform/pricing)
- 时间线：2025-07-08 Preview → **2025-12-16 GA**（与 Sessions 同批）→ 2026-01-28 起计费 → 2026-04-22 改名+IngestEvents 流式+不可变 revisions → 2026-06-17 多区域/全局 GA → 2026-07-15 memory profiles GA。(来源: https://cloud.google.com/vertex-ai/generative-ai/docs/release-notes ; https://docs.cloud.google.com/gemini-enterprise-agent-platform/release-notes)
- 资源仍挂 `reasoningEngines/{id}` 下（REST 另有 `projects.locations.memoryBanks`）；官方口径"RAG 是静态外部知识库，Memory Bank 随上下文演化"。(来源: release-notes ; https://docs.cloud.google.com/gemini-enterprise-agent-platform/scale/memory-bank)
- 计费：存储 $0.30/GiB·月（**revisions 也计入**）、读 3M 次=1 vCPU-h($0.085)。(来源: https://cloud.google.com/products/gemini-enterprise-agent-platform/pricing)

### 2. 记忆模型
- **两级**：`Memory`（fact/scope/topics/metadata/memoryType/structuredContent/expireTime|ttl/revision 系列字段）→ 不可变 `MemoryRevision`（快照 fact+**合并前 `extractedMemories` 中间产物**+context+labels）。注意：常见描述的"store→topic→memory 三层"不成立，**topic 只是标签属性**（managed 4 类 USER_PERSONAL_INFO/USER_PREFERENCES/KEY_CONVERSATION_DETAILS/EXPLICIT_INSTRUCTIONS + custom label/description + few-shot）。(来源: https://docs.cloud.google.com/gemini-enterprise-agent-platform/reference/rest/v1beta1/projects.locations.memoryBanks.memories ; https://aiplatform.googleapis.com/$discovery/rest?version=v1beta1 ; https://docs.cloud.google.com/gemini-enterprise-agent-platform/scale/memory-bank/revisions ; https://docs.cloud.google.com/gemini-enterprise-agent-platform/scale/memory-bank/generate-memories)
- **scope=任意 k-v 字典（≤5 对、写入即不可变、检索/合并要求精确同 scope）**——identity 级硬隔离主键。(来源: https://docs.cloud.google.com/gemini-enterprise-agent-platform/scale/memory-bank/fetch-memories)
- 类型枚举 NATURAL_LANGUAGE_COLLECTION/STRUCTURED_COLLECTION/**STRUCTURED_PROFILE**：profile 用固定 schema（可 pydantic 上传），每 scope+schema 单一事实源、每字段一条 memory 独立 TTL/revision，`RetrieveProfiles` 低延迟取合并后画像。(来源: https://docs.cloud.google.com/gemini-enterprise-agent-platform/scale/memory-bank/profiles)

### 3. 写入链路与合并策略
- 三入口：`CreateMemory` 直写不抽取；`GenerateMemories` 抽取+合并（源三选一：payload/vertexSessionSource/**directMemoriesSource 预抽取事实 ≤5 条**——即接受 agent 或 human-in-the-loop 预筛条目）；`IngestEvents` 流式缓冲（stream=scope+stream_id、event_id 自动去重、count/idle/interval/force 触发组合、overlap 跨窗续接、无触发 24h 兜底 flush）。(来源: generate-memories/api-quickstart/ingest-events 同站)
- 两阶段：Extraction（只留命中 topic 的）→ Consolidation（判 duplicative/complementary/contradictory，据此 **ADD/UPDATE/DELETE** 既有记忆）。自定义度很有限：consolidationConfig 只有 revisionsPerCandidateCount（默认 1，调大用历史区分趋势 vs 异常）、metadataMergeStrategy=REQUIRE_EXACT_MATCH、disableConsolidation、allowedTopics；**无自由文本合并指令**。(来源: https://docs.cloud.google.com/gemini-enterprise-agent-platform/scale/memory-bank/generate-memories)

### 4. 检索
- `RetrieveMemories`：scope 必填精确匹配；给 similarity_search_params（top_k 默认 3，欧氏距离升序）做域内向量检索，不给则返回该 scope 全部；`ListMemories` 官方注明不适合低延迟（每页 ≤100）。过滤双轨：AIP-160 filter（fact/时间/topics 系统字段）+ filterGroups（metadata DNF、类型化值）。(来源: fetch-memories ; discovery)

### 5. 失效与时间线
- 时间线=**版本史而非有效期**：每次 create/modify 自动生成不可变 revision（默认开启、可 disable、可打 label 过滤），`RollbackMemory` 回滚、generate 响应带 previous_revision；**被生成删除的 memory，其 revisions 仅存 48h** 可查/回滚。无 valid_at/invalid_at。(来源: https://docs.cloud.google.com/gemini-enterprise-agent-platform/scale/memory-bank/revisions)
- 过期：memory 级 expireTime|ttl 二选一；实例级 default_ttl 或 granular_ttl{create/generate_created/generate_updated}——失效策略与写入来源解耦；revision 级同构 TTL。显式遗忘走 EXPLICIT_INSTRUCTIONS topic + `PurgeMemories`（filter 批量，**force=false dry-run 返回将删条数**）。(来源: setup ; api-quickstart ; revisions)

### 6. 治理
- 专用 IAM 角色 memoryViewer/memoryEditor/memoryUser + **IAM Conditions 按 scope 限权**（CEL `aiplatform.googleapis.com/memoryScope`，项目级、对未来资源生效）；⚠️ **ListMemories/PurgeMemories 不支持条件**——跨 scope 批量操作只能授无条件角色（管理员过宽风险，官方建议正向条件）。(来源: https://docs.cloud.google.com/gemini-enterprise-agent-platform/scale/memory-bank/iam-conditions)
- 加密：**global 端点不能用 CMEK**（合规须 regional/multiregional）；~23 region+us/eu/global。(来源: https://docs.cloud.google.com/gemini-enterprise-agent-platform/release-notes ; https://docs.cloud.google.com/gemini-enterprise-agent-platform/resources/agent-locations)
- 审计：aiplatform.googleapis.com 在 Cloud Audit Logs 支持列表内，但 **Memory Bank 方法级 DATA_READ/WRITE 粒度无文档**——未验证；DSAR 式按用户导出+擦除一键流程未见——未验证。(来源: https://cloud.google.com/logging/docs/audit/services)

### 7. 界面与接入
- Console：Agent Engine 页 **Memory Bank tab**（2025-09-10 上线，查看/管理，无人工审核界面）；SDK 换包 `google-cloud-agentplatform>=2.0.1`；REST v1(GA)/v1beta1(Preview)；ADK `VertexAiMemoryBankService`(BaseMemoryService 实现) + `PreloadMemoryTool`（每轮开场注入）/`LoadMemoryTool`（模型自决）。(来源: release-notes ; setup ; https://docs.cloud.google.com/gemini-enterprise-agent-platform/scale/memory-bank/adk-quickstart)

### 8. 成熟度与公开问题
- 无专属 quota/limits 页（未验证）；官方排障页第一条即「**静默不生成**——LRO 成功但结果为空=判定内容不'有意义'」；行为不一致（SDK generate 默认阻塞 vs ADK 非阻塞；文档默认模型 gemini-3.5-flash vs notebook 里 2.5-flash）；一年内改名+换包+REST 资源面重构+默认模型季度更换=**接入面频繁漂移**；限制清单：scope≤5 对/不支持*、预抽取≤5 条、top_k 默认 3、时间触发须 60s 倍数、删除后 revision 仅 48h。(来源: https://docs.cloud.google.com/gemini-enterprise-agent-platform/troubleshooting/memory-bank ; setup ; release-notes)

### 9. 可借鉴
- **scope 硬隔离主键（不可变、精确匹配、跨 scope 永不合并/检索）+ CEL 条件授权** → MemorySpace 的 API 级租户隔离+space ACL 现成范式。
- **revision 数据形状**：每 mutation 不可变快照 + 合并前 extractedMemories + previous_revision + rollback + revision 级 TTL/labels + 48h 恢复窗——Lineage/ChangeSet 的"最小可用形态"，连成本口径（revision 计费）都替我们想好了怎么被转嫁。
- `PurgeMemories` dry-run（force=false 预览将删集合）→ ChangeSet 预览/发布前 diff 摘要。
- IngestEvents 批处理闸门（缓冲/去重/count+idle+interval 组合/overlap/兜底 flush）≈ Triage 触发层参考实现；granular TTL 按写入来源分类 → 失效模型与 Triage/Publish 通道挂钩。
- Structured profiles（固定 schema、字段级 memory/revision）+ PreloadMemoryTool 开场注入 = ContextManifest 的正确形状；4 个 managed topics 作开箱策略包降低冷启动成本。
- metadata DNF filterGroups 双轨过滤 → CopyInventory 清点/圈选查询语言。

### 10. 不可取
- **无审核门且合并可 DELETE 既有记忆**：判定完直接落库，唯一补救事后 rollback/purge（还只有 48h 窗口）→ 反证 Review/Publish 必须人工、不可自动化；也把"删除可回滚窗口"从免费午餐变成需要自建的 CopyInventory 义务。
- **合并黑盒不可解释**：只配 revisionsPerCandidateCount，不返回"为何删/为何并"→ 我们要求每条 ChangeSet 带可读理由+可重放规则版本。
- **静默不生成=官方成功语义** → Triage 要把"不值得记"做成显式可审计输出类别。
- **默认策略云端漂移**（生成模型/embedding 默认值被换）→ RunGate 必须把 model id+抽取/合并配置 hash 钉进 Lineage 才可复现。
- List/Purge 不吃 IAM condition → 批量操作可穿透租户边界：我们的 Review 与批量入口必须绑 space 权限；topic 非可治理实体（无 store 层）→ 把 Topic/Collection 做成一等治理对象。
- global 与 CMEK 互斥 + 方法级审计无文档 + revisions 成本隐式转嫁 → "区域锁定+CMEK+方法级审计契约+lineage 成本显式估算"可成我们卖点。

---

# B 速览组

### B1 mem0（对照基准，mem0ai/mem0）
- 记忆模型：user/session/agent 三级状态+离散条目+跨条目实体链接；65.8k stars、专项第一、YC S24。(来源: https://raw.githubusercontent.com/mem0ai/mem0/main/README.md)
- 差异化：**2026-04 算法改版**——从"extract+update 两阶段"退到 single-pass **ADD-only**（一次 LLM 调用、只累加不覆盖）+ Temporal Reasoning 排序（当前状态/过去事件/未来计划）替代覆盖式冲突解决；LoCoMo 92.5、LongMemEval 94.4，并自认分数含**专有优化托管版**、OSS 只方向性相似。支持 `mem0 init --agent`（agent 自助取 key）。
- 与本平台关系：它放弃覆盖式合并=承认"自动改写记忆"不可控，反证我们 Review 门路线；但无人工发布、无审计面。

### B2 Cognee（topoteretes/cognee）
- 三库合一（关系+向量+图）记忆层，四动词 remember/recall/improve/forget；写入=Add+Cognify 管线，检索分 semantic/structural(Cypher)/hybrid。(来源: https://raw.githubusercontent.com/topoteretes/cognee/main/README.md ; https://docs.cognee.ai/core-concepts/data-flows.md)
- 治理亮点：pipeline run 全部 **owned**（某 user 对某 dataset，逐层校验访问权）；forget 可删单项/整 dataset；provenance 记在 relational 层。31k stars、v1.6.0（2026-09）；可从 Mem0/Letta/Zep 迁入；原 ECL 命名已弃用（未验证）。
- 关系：数据管线受治+dataset ownership 与 MemorySpace 相近，但无发布审核与时间失效线。

### B3 LangMem / LangGraph Store
- 记忆=Store 里 namespace+key 的 JSON 文档（语义/情景/程序三分法）；**热路径（agent 自调 manage/search 工具）vs 背景整合（memory manager 自动抽取更新）**双模式；无自带服务，成熟度=随 LangGraph Platform 默认提供。(来源: https://docs.langchain.com/oss/python/langchain/long-term-memory ; https://raw.githubusercontent.com/langchain-ai/langmem/main/README.md)
- TTL/失效未验证；1.7k stars。关系：其双模式写入是 Triage 时机设计的免费参照；治理层完全留给宿主。

### B4 A-MEM（arXiv 2502.12110，NeurIPS 2025）
- Zettelkasten 动态笔记网络：新记忆带 contextual description/keywords/tags，自动找相似建链；**memory evolution**=新记忆反写旧记忆的表征与属性。(来源: https://arxiv.org/abs/2502.12110)
- 无过期/删除/权限治理（论文不涉及——未验证）；配套 agirsearch/A-mem ≈1.2k stars。关系：给"链接+演化"提供算法叙事，但离产品最远。

### B5 Memobase（memodb-io/memobase）
- "Memory for User, not Agent"：可设计 topic/sub_topic 画像+user event 时间线；每用户 buffer 会话后批量 flush 抽取，在线读走 SQL（<100ms 宣称）；**blob 处理完默认删除**（不留原文，可配持久化）。(来源: https://github.com/memodb-io/memobase)
- 2.9k stars、Apache-2.0、profile/event/blob API+MCP；event gist 专攻时间性问题自报 LoCoMo SOTA。关系：画像 schema 化+默认删原文的隐私默认值可借鉴。

### B6 supermemory
- 单一"活的事实图"（facts on top of facts）+自动 user profile；learner-1 + "dreaming"后台持续抽取/连边/生成 derives；边类型 updates/extends/derives + `isLatest` 停在当前真相不抹历史；forget=软删、mass-forget 带 dryRun；SOC2 Type II/HIPAA BAA/GDPR，container tag 租户边界；31k stars、闭源平台+开源 SDK、自称 100k+ 组织。(来源: https://raw.githubusercontent.com/supermemoryai/supermemory/main/README.md ; https://supermemory.ai/docs/concepts/graph-memory.md ; https://supermemory.ai/docs/overview/security.md)
- SCIM 未检索到——未验证。关系：updates/isLatest+软删+干跑预览是失效与删除 UX 的好模板；仍无事前审核门。

### B7 HippoRAG 2（arXiv 2502.14802，ICML 2025）
- "From RAG to Memory"：检索=非参数持续学习；诊断图增强 RAG 在基本事实任务上塌陷；Personalized PageRank + 更深 passage 整合，factual/sense-making/associative 三类兼得（关联 +7% 不掉基础）。(来源: https://arxiv.org/abs/2502.14802)
- 无生命周期治理；OSU-NLP-Group/HippoRAG ≈4k stars。关系：是检索算法供给方而非平台竞品。

### B8 ChatGPT memory（openai help）
- 两代并存可切换：saved memories 显式条目 vs improved memory 概括式 summary；来源池含 past chats/Library/connected apps；回答带 **Sources** 可就地纠错删除；temporary chat 不创建不更新。(来源: https://help.openai.com/en/articles/8590148-memory-faq)
- 治理面独特：**删除分层不联动**（删对话不删派生 saved memory；关 Reference chat history 后派生记忆 30 天内删、日志最长 30 天）；regulated workspace 默认关 improved memory；**Project-only memory**（项目内引用不外泄）。
- 关系：官方承认"删除传播延迟+跨来源清理负担"——CopyInventory 要解决的问题在 C 端已被点名。

### B9 Claude memory tool（Anthropic API）
- 文件式记忆：`/memories` 目录下 CRUD 文本文件，客户端完全执行（存储/加密自定），官方安全红线=拒绝越出前缀路径；无服务端 TTL（未验证/不存在）。(来源: https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool)
- 与 Claude Code 区分：CLAUDE.md/AGENTS.md 人写三层指令 + auto memory（Claude 自写、四类 type、`~/.claude/projects/<project>/memory/`、machine-local 不跨机、被排除在 transcript 清理外）。(来源: https://code.claude.com/docs/en/memory)
- 关系："模型发指令+宿主执行存储"=我们的 Connector 边界模型；ZDR 式自控是其卖点也是无治理。

### B10 2026 新玩家（GitHub 发现）
- **volcengine/OpenViking**（38.4k★，2026-01 建仓）：context database=虚拟文件系统，`viking://` URI+ls/read/grep；三层 L0 abstract/L1 overview/L2 原文、目录作用域检索（TrieHI）；核心抽取在内部 VikingMem，开源是子集（AGPLv3）。(来源: https://raw.githubusercontent.com/volcengine/OpenViking/main/README.md)
- **TencentCloud/TencentDB-Agent-Memory**（27.1k★，2026-04 建仓）：L0 Conversation→L1 Atom→L2 Scenario→L3 Persona 金字塔+符号化短期画布；主打**无损回溯**（上层符号可确定性 grep 回 L0 原文，明确反对不可逆有损摘要）；治理参数最直白（retentionDays/去重/触发节奏全可配）。(来源: https://raw.githubusercontent.com/TencentCloud/TencentDB-Agent-Memory/main/README.md)
- **rohitg00/agentmemory**（28.7k★，2026-02 建仓）：coding agent 共享记忆服务器，54 MCP tool+hooks 自动捕获，confidence scoring/lifecycle/KG 混合检索；细节治理未验证。(来源: https://raw.githubusercontent.com/rohitg00/agentmemory/main/README.md)
- GitHub topic `agent memory` 前 15 中仅 5 席真记忆专项（mem0/OpenViking/cognee/agentmemory/TencentDB），9 席是 harness/框架顺带；赛道叙事正从「memory 服务」改名「context/上下文基础设施」。(来源: https://api.github.com/search/repositories?q=agent+memory&sort=stars&order=desc&per_page=15)

---

# C 横向对照表

| 系统 | 记忆分层 | 写入/合并 | 时间/失效 | 治理/审计 | UI 形态 | 与本产品关系一句话 |
|---|---|---|---|---|---|---|
| Zep/Graphiti | episode→实体/fact 图→社区 | 全自动 6 步 LLM 管线，矛盾=旧边 invalid | **bi-temporal 4 戳，失效不删** | RBAC+ABAC+SOC2/HIPAA/BYOK/BYOC，审计仅 dashboard 面 | Cloud Dashboard+图可视化 | 时间线+血缘天花板，缺 Review 门与行动准入 |
| Letta/MemGPT | core blocks/archival/recall → MemFS(git) | agent 自编辑+sleep-time/dreaming 后台整理 | 无失效；git commit 文件级版本 | 组织 RBAC、默认私有；合规无公开面 | CLI/桌面/多渠道/ADE(弃) | 双轨整理+git ChangeSet 隐喻可抄，门与血缘皆缺 |
| AWS AgentCore | Memory→Event(短,3-365d 过期)→Record(长) 双层 | 异步抽取作业化+redrive；consolidation 三操作 Add/Update/Skip，**去重即删除无版本** | record 仅 created/updated，就地改写；回溯靠 Kinesis 流外自建 | IAM namespace 三条件键+Cedar FGAC(批量写旁路)+CMK 部分+CloudTrail 数据面未验证 | Console 向导+CLI/SDK/框架包 | 治理基础设施最强宿主，记忆语义与审计粒度最薄 |
| Google Memory Bank | Memory→**MemoryRevision** 两级+topics 标签（无 store 层）；profiles 形态 | 抽取→consolidate 两阶段，LLM 判矛盾后 **ADD/UPDATE/DELETE**；IngestEvents 批闸门；接受人工预筛条目 | 版本线非时间线：不可变 revision+rollback+删除后 48h 窗；granular TTL；无 valid/invalid | 专用角色+**CEL 按 scope 授权**（List/Purge 不吃条件）；global 无 CMEK；方法级审计未验证 | Console Memory Bank tab+REST/ADK | revisions/scope-IAM 与 Lineage·MemorySpace 最接近，仍无审核门 |
| mem0 | 条目+实体链接(user/session/agent) | ADD-only 累加+时间排序取代覆盖 | Temporal Reasoning（读侧排序） | 云版常规 RBAC | 平台+Library | 放弃覆盖式合并=我们保守默认的行业注脚 |
| supermemory | 事实图+profile(container) | learner-1 dreaming 持续改写 | updates/isLatest，forget=软删 | SOC2/HIPAA，tag 即租户 | SaaS+插件 | 失效 UX（软删/干跑）可借鉴 |
| ChatGPT | saved memories/summary 双态 | 后台自动+用户显式 | 30 天删除传播窗口 | Project-only memory、regulated 默认关 | 产品内 Sources UI | C 端删除闭包难题的公开样本 |
| Claude memory tool | /memories 文件 | 模型指令+客户端执行 | 无 | 宿主全责 | API 工具 | 存储自控无治理=我们的反面 |
| OpenViking | L0/L1/L2 文件树 | session commit→可编辑 Markdown | 未验证 | AGPL，核心开源子集 | 桌面控制台 | 文件隐喻+分层披露与 MemFS 同路 |
| TencentDB-Agent-Memory | L0-L3 金字塔+符号画布 | 每 N 会话批量抽取+去重 | retentionDays 全可配、无损回溯 | 参数级治理最直白 | 插件 | 证据下钻(符号→原文)≈我们的 Lineage 诉求 |

---

# D 全景结论

## 行业趋同点（7 条）
1. **抽取全异步 + LLM 为核**：Zep 6 步管线、AWS IngestData/分钟级、Google GenerateMemories、supermemory dreaming、Letta dreaming——没有一家敢在线同步写全量记忆；"写入后延迟可见"是共同契约（AWS 明说应用要容忍）。
2. **原始层无损留底**：episode/event/session-messages 原文永久或按 TTL 保留，派生物只从原文生长（Zep episode、AWS event+IngestData 二选一、Google vertexSessionSource 指向 Sessions、腾讯 L0 可 grep 回原文并明确反对不可逆摘要）。
3. **失效走向软删/累加**：Zep invalid_at 失效不删、supermemory isLatest+软删、mem0 干脆弃 UPDATE/DELETE 改 ADD-only+读侧时间排序；但 Google consolidate 仍是 LLM 判矛盾的 ADD/UPDATE/DELETE 覆盖式——两代方案并存，**"删除可回滚窗口"成为新痛点**（Google 仅 48h，Zep 干脆不可逆）。
4. **命名空间/scope 是隔离原语**：Zep group_ids/graph、AWS namespace 模板、Google scope 字典、LangMem namespace、supermemory container tag——但全部是**字符串约定**，无一有平台级 schema 治理。
5. **双 agent 分工**：前台轻写入/禁写入、后台专职整理（Letta sleep-time 首倡，Zep/supermemory/腾讯以 cron/batch 变体重现）。
6. **文件/仓库隐喻回归**：Letta MemFS(git)、OpenViking 虚拟 FS(L0/L1/L2)、Claude memory tool `/memories`、agentmemory 全走"markdown+目录+按需读"，对抗向量黑箱；git/不可变 revision（Google）成为默认血缘载体——但全部是**版本史**，无一做到 Zep 式现实时间线。
7. **企业合规卷面=SOC2+BYOK/自带模型+删除 API 三件套**（Zep、supermemory、AWS 继承、Google 继承；Letta 最弱），审计普遍只覆盖管理面，**数据面读取审计几乎无人做全**。

## 我们相对独有卖点（行业空白处）
1. **Triage→ChangeSet→Review→Publish 人工发布门**：全行业零命中——Zep/Letta/supermemory 全部"抽取即生效、错误只能事后回改"；Google 最接近也只是允许人喂入预筛条目、并给 Purge 提供 dry-run。这是最硬的差异化。
2. **RunGate（行动前准入+持续复查）**：没有任何竞品把"记忆影响行动"当治理对象（Zep 官方文档明言行动授权留给客户；AWS 靠 IAM 管不到行动语义）。记忆投毒/TOCTOU 只有我们能系统性回答。
3. **CopyInventory 副本闭包**：ChatGPT 亲证删除传播之苦（30 天+分层不联动）、Zep 删除不重算摘要、Letta detach 后副本语义未定义、AWS embedding/流式副本无闭包视图——"缓存/索引/embedding/备份/Provider/终端"全链路撤回清单无人做。
4. **Purpose/VisibilityContext 双维判定**：竞品 ACL 只回答"谁能看"，不回答"为何用"；AWS condition/Google CEL 只到 scope 粒度，无源/目标空间+revision+敏感性交叉判定。
5. **跨 Harness 中立性**：Zep/Letta/Google/AWS 各自绑自家云或自家 harness；"多执行现场可交接、Lineage 跨系统"仍空位（本会话的 harness 大 repo 榜单反证执行侧内卷、记忆治理侧空白）。

## 我们相对缺口（竞品已做而我们要补的）
1. **检索与延迟工程**：sub-200ms P95+rerank+预算装配（Zep）、混合多路（全组标配）是我们的裸项；ContextManifest 只定义契约，还需检索内核与降级策略（如 Zep embedder 故障降 BM25）。
2. **bi-temporal 时间线**：Zep 的 valid/invalid×created/expired 4 戳与"时点查询"仍是天花板；我们 ChangeSet/Publish 时间≠现实有效时间，Lineage schema 需显式引入双时间线。
3. **在线自编辑的低摩擦体验**：Letta read_only 命名块+配额（limit）、两级上下文（常驻+按需读）是很被认可的 UX；纯审核流可能牺牲"agent 当场记住"手感，需要 ChangeSet 秒级提交+后台批量审核的双速设计。
4. **开发者接入面**：mem0/Zep 三行接入、MCP、各家框架集成全家桶 vs 我们尚无 Connector SDK 生态；supermemory/Claude 插件已进 coding agent 工作流。
5. **画像/结构化模板供给**：Memobase topic 画像、supermemory profile buckets、Google profiles 专型——我们只有 MemoryItem/KnowledgeAsset 抽象，缺开箱模板。
6. **成本与时延透明**：竞品公开基准与限流参数（LoCoMo/LongMemEval、429 并发、1 分钟抽取窗），我们要把"审核门+重算+副本闭包"的成本结构讲清楚，否则会在采购对比中吃亏。

> 基准注：各家基准分数（LoCoMo/LongMemEval/DMR/BEAM）多为厂商自报且托管版与开源版混称（mem0 已公开承认），横向引用时保留"厂商自报"标签，勿作独立验证结论。
