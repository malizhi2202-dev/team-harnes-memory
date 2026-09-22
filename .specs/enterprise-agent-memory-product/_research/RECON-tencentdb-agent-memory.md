# RECON: TencentDB Agent Memory

> 状态: 调研稿 · 已过审（2026-09-22 18:56 · 用户原话「通过」，随 PRODUCT-DESIGN v0.3 / §1.5 审核门；标「未验证」项不得被下游引用）· 日期: 2026-09-22 · 联网验证: 是
> 验证方式: curl 抓取 raw.githubusercontent.com 文档副本 + curl/web_fetch 抓 GitHub 页面（repo 首页/issues/releases/tree 页）+ shields.io JSON 徽章 + 对 README 截图做 vision 转录。`web_search` 未使用（已损坏）。GitHub API 因 IP 限流基本未用。
> 注意：仓库存在**两条产品线、两套 README**——`main` 分支是 v1.x「单机 OpenClaw/Hermes 插件」线，`master`（及真实默认分支 `feat/server_team`，二者 README 内容一致）是 v2.x「Team Memory Hub」线。本文以 v2 团队线为主、v1 线补充核心内核事实。仓库首页 JSON 显示 defaultBranch 为 `feat/server_team`（来源: https://github.com/TencentCloud/TencentDB-Agent-Memory 页面内嵌数据）。

---

## 1 定位速览

- 官方一句话定位（仓库 About/description）：「TencentDB Agent Memory 是面向 AI Agent 的 **team-level memory hub**，把对话、文档、代码转化为四种可复用记忆资产（Chat Memory、Skill、LLM-Wiki、Code-Graph），在 Agent 与框架之间被治理、共享、装配」(来源: https://github.com/TencentCloud/TencentDB-Agent-Memory 页面标题/描述)。
- 四种资产分别是什么（来源: https://raw.githubusercontent.com/TencentCloud/TencentDB-Agent-Memory/master/README.md ；中文措辞 https://raw.githubusercontent.com/TencentCloud/TencentDB-Agent-Memory/master/README_CN.md）：
  - **Chat Memory**：保留偏好、事实、决策与交互历史；每个 Agent 创建时自动获得一块；按 L0 Conversation → L1 Atom → L2 Scenario → L3 Persona 逐层蒸馏。数据形态：L0 消息（role/content/timestamp）、L1 原子（type∈episodic|persona|instruction，content≤8192 字）、L2 场景（按路径组织的 Markdown 文件）、L3 画像（`persona.md` 单文件）(来源: https://raw.githubusercontent.com/TencentCloud/TencentDB-Agent-Memory/master/MemoryCore/v3-api-memorycore-doc.md)。
  - **Skill**：从对话与工具调用中沉淀「可执行经验」。形态 = `SKILL.md` 全文（含 frontmatter，name≤64 且必须与 frontmatter.name 一致）+ 资源文件（≤100 个，可带 encoding/base64、mime、is_executable）；有版本、owner、team、status(active/archived)；官方文案强调"Skill 不只是 prompt 片段，它有版本、资源文件、触发边界、执行步骤、验证规则"，「个人 Skill 默认私有，审核后可分享给团队」(来源: master README.md；MemoryCore v3 文档 §3.2)。
  - **LLM-Wiki**：把产品文档/设计规范/运维手册转成「结构化页面 + 链接图谱」，灵感注明来自 Karpathy 的 LLM 知识库 gist。形态 = raw 源文件 + 抽取后 pages（ref/title/path）+ graph（nodes/edges/communities），BM25 检索支持图谱 hop 扩展 (来源: master README.md Acknowledgements & §知识地图；https://raw.githubusercontent.com/TencentCloud/TencentDB-Agent-Memory/master/MemoryKnowledge/v3-api-memoryknowledge-doc.md §3.1；https://raw.githubusercontent.com/TencentCloud/TencentDB-Agent-Memory/master/MemoryKnowledge/openapi.yaml 头注释"Standalone knowledge service for Code-Graph + LLM-Wiki, 28 endpoints")。
  - **Code-Graph**：索引代码符号、文件、调用关系与影响路径；形态 = repo/branch/commit + stats{files,nodes,edges}，查询工具 search/explore/callers/callees/impact/node/status/files 返回文本块。实现注明复用了开源项目 colbymchenry/codegraph 的代码 (来源: MemoryKnowledge v3 文档 §3.2；master README.md Acknowledgements)。
- v1 线（main 分支）的核心卖点：「符号化短期记忆 + 分层长期记忆」——工具日志 offload 到 `refs/*.md`、中间层 jsonl 步级摘要、顶层 Mermaid 画布 + `node_id` 回溯；宣称集成 OpenClaw 后 WideSearch token −61.38%、通过率相对 +51.52%、PersonaMem 48%→76%（注明为长会话连续任务实测，非单轮）(来源: https://raw.githubusercontent.com/TencentCloud/TencentDB-Agent-Memory/main/README.md)。v2 线保留 PersonaMem 48→76 这一项 benchmark (来源: master README.md Benchmark)。
- 产品哲学：「记忆资产，不是聊天记录仓库。RAG 回答"能查到什么"，Team Memory 还回答"谁能用、哪个版本有效、该装给哪个 Agent"」，并给出 Chat History / Standard RAG / 本产品 的三列对比表（跨会话理解、蒸馏可执行经验、文档结构、代码调用图、归属/版本/状态、团队共享与配装、private/team/ACL 七个维度）(来源: master README.md "Memory Assets, Not a Chat Log Warehouse" 节)。**未找到**与 mem0 等具体开源项目的公开对比（在已下载的全部 master/main 文档副本中 grep "mem0" 零命中；第三方评测未验证）。
- 目标客户叙事：一人公司/小团队组建 Agent 小队（Scout/Builder/Reviewer 各配不同记忆装备），冷启动=导入代码库、文档、历史对话即继承"存档" (来源: master README.md Cold Start & One Play Style 节)。

## 2 记忆模型

- **隔离与归属（v3 数据面强制）**：`team_id / agent_id / user_id / task_id` 四元组，可走 body 或 `x-tdai-*` Header（body 优先）；v3 强制 team+agent+user 三元组，缺省回落 `default` 桶；上层还有实例维度 `service_id`（`x-tdai-service-id`，即租户/记忆实例路由键，KS 端"所有接口按 service_id 收敛，跨租户资源统一返回 404 不暴露存在性"）(来源: MemoryCore v3 文档 §1.4；MemoryKnowledge v3 文档 §1.3/1.4)。
- **分层语义**：L0 原始对话（核对原话/时间戳/来源）→ L1 Atom（可行动事实，精确召回）→ L2 Scenario（场景知识块，快速恢复工作上下文）→ L3 Core/Persona（长期画像与稳定模式）。生成与检索都分层：日常用 L2/L3 引导上下文，需要具体事实时 BM25+向量+RRF 落回 L1/L0 (来源: master README.md Technical Implementation §1)。
- **统一资产注册（meta_asset）**：四类资产都登记为 Memory Asset，字段含 `asset_id, team_id, asset_type(skill|llm_wiki|code_graph|chat_memory), name, owner_user_id, source_type, source_ref, version, visibility, status, confidence, expires_at, last_used_at, usage_count, content_ref, metadata_json`。即：**版本、可见性、状态、置信度、过期时间、使用计数、内容引用、来源引用全部在资产壳上，与内容体分离** (来源: MemoryCore v3 文档 §3.7.9)。
- **记忆块（Chat Memory 面板对象）**：`chat_memory-{team}-{agent}`（每 Agent 一块）或自建 `mem-xxx`；含 title、owner、scope(team/private)、layer_counts{L0,L1,L2,L3} (来源: https://raw.githubusercontent.com/TencentCloud/TencentDB-Agent-Memory/master/MemoryPanel/panel-api-doc.md §3.4)。
- **可见性四档**：`private`（仅 Owner 可读，团队管理员也不行）/ `team`（成员可读，Owner/Admin 可管理）/ `restricted`（User/Role/Agent ACL 精确授权）/ `agent`（同团队内定向配装专用）；Panel 的知识资产 set-visibility 枚举还多一个 `task` (来源: master README.md "Shared Experience, Not Shared Privacy" 表；MemoryPanel panel-api-doc §3.10 set-visibility)。
- **版本与编辑**：L1 更新即版本自增（但 `update` 返回字符串 `"v2"`、`query` 返回数字，官方文档自认类型漂移"根源在代码…文档无法同时满足"）；L2 场景/L3 画像按文件版本读取（可传 version）；Skill 是完整版本链：`version` 单调递增 + `is_head` + `expected_version` 乐观锁（update/patch/files 写全走乐观锁，冲突 40901 带 current_version）+ versions 列表带 `is_expired`；Wiki/CodeGraph 有字符串 version 与状态机 (来源: MemoryCore v3 文档 §3.1 atomic/update 注、§3.2、§3.3；MemoryKnowledge v3 文档 §1.6/§3.1/§3.2)。
- **角色模型（两层）**：全局 `System Admin`（管用户/团队、可用全部资产管理功能）；Team 内 `admin/member/reviewer`（ACL 授权时 subject_type 为 user/team_role/agent）；资产归属靠 Owner 标记，Owner 自动获得管理权 (来源: master README.md；MemoryCore v3 文档 §3.7.4、§3.7.11)。
- **实体全家桶（meta 面 55 接口）**：User、User-Key（API key，可设过期/吊销，列表脱敏仅 create 回显一次）、Team、Team-Member、Agent（含 prompt、visibility、archive）、Task（source_type manual/tapd/github/other、risk_level、linked_agents、auto_assign_floating_assets）、Task-Agent 关联、Participation-Log（追加式参与事件）、Asset、Agent-Fixed-Asset 绑定、ACL、Auth-Verify、Instance-Quota、用户 Config (来源: MemoryCore v3 文档 §3.7)。

## 3 写入链路

- **对话 → Chat Memory（L0→L1→L2→L3 异步管线）**：
  - 入口一：MemoryProxy 拦截 Agent 的全部 LLM 流量，在会话中自动捕获（每轮对话写 L0）；入口二：直接 `POST /v3/conversation/add`（1–100 条/次），写成功后异步 `notifyPipeline` 触发 L1 抽取 (来源: INSTALL.md Step 4/5 https://raw.githubusercontent.com/TencentCloud/TencentDB-Agent-Memory/master/INSTALL.md；MemoryCore v3 文档 §3.1)。
  - 触发策略（内核配置 schema）：`pipeline.everyNConversations=5`（每 5 轮触发 L1 批处理）、`enableWarmup=true`（新会话 1→2→4 翻倍提前触发）、`l1IdleTimeoutSeconds=600`（用户停聊 600s 触发）、L1 完成后延迟 10s 触发 L2、L2 最小间隔 900s / 最大轮询 3600s / session 活跃窗口 24h；单次 L1 每 session 最多 20 条 (来源: https://raw.githubusercontent.com/TencentCloud/TencentDB-Agent-Memory/master/MemoryCore/openclaw.plugin.json configSchema.pipeline/extraction/persona)。
  - **去重/冲突**：`extraction.enableDedup=true`——L1「智能去重（基于向量相似度或关键词进行冲突检测）」(来源: openclaw.plugin.json；main README.md 配置表亦同)。
  - **合并/归纳**：每新增 50 条 L1 触发 L3 画像生成；L2 最大场景数 15；画像备份保留 3、场景块备份 10（滚动备份，非不可变历史）(来源: openclaw.plugin.json persona)。
  - 已知可靠性缺陷（issue）：L1 抽取失败时 L0→L1 游标仍前进、对话被静默丢弃（#1395）；抽取可能把第三方人名归因给用户、prompt 缺 name-source 规则（#1402）(来源: https://github.com/TencentCloud/TencentDB-Agent-Memory/issues)。
- **对话 → Skill**：`/v3/skill/conversation/add` 每轮同步做 buffer 拼接+阈值判定，满足即归档切片异步走「五角色 skill 抽取管线」（Pi 系列 issue 标题提及）；归档触发原因枚举 `tool_calls | bytes | compressed | oversize`；可 `extract` 手动直触发、`force-archive` 跳阈值；对话结束提示语内含"LLM 判断是可复用 how-to 则自动提炼" (来源: MemoryCore v3 文档 §3.2；INSTALL.md Step 5；README master §Skill)。
- **文档 → Wiki（两段式）**：先 `wiki/raw/write` 上传源文件（≤10 个/次、单文件 ≤512KB、总 ≤5MB），再 `wiki/ingest`（202 受理）；状态机 `draft→pending→processing→ready/failed`；空 wiki 拒绝 ingest；processing 期写/删一律 409；进度经 S2S 回调 `event=ingest_progress`（phase/completed/percent）推给 Panel；`ready` 时 Panel 才把明细写入内核并注册 meta_asset（failed 不注册）；页面可人工 `page/write` 覆盖。定时 auto-sync 调度器可配 scanInterval/maxConcurrentSyncs (来源: MemoryKnowledge v3 文档 §3.1、§3.5；MemoryPanel panel-api-doc §3.11)。
- **代码 → Code-Graph**：`code-graph/create(repo_url, branch=main)` 幂等（同 repo+branch 返回已有），创建即自动 build，`sync` 重建索引；官方注明「当前优先支持公开 HTTPS 仓库，私有仓库/SSH 凭证仍在完善」(来源: MemoryKnowledge v3 文档 §3.2；master README.md Notes)。
- **历史导入（冷启动）**：Panel `chat-memory/import` 把 ≤100 条历史消息写进目标 Agent 记忆块的 L0（不新建资产）；代码库/文档/会话均可作为导入源 (来源: MemoryPanel panel-api-doc §3.4；master README.md Cold Start)。

## 4 检索与装配

- **内核召回策略**：`keyword` / `embedding` / `hybrid`（BM25+向量+RRF 融合，默认推荐）；`recall.maxResults=5`、`scoreThreshold=0.3`、单条 L1 注入字符上限与整轮总字符预算（默认 0=不限）、`timeoutMs=5000` 超时**跳过注入不阻塞会话** (来源: openclaw.plugin.json recall；main README.md 配置表)。BM25 中文用 jieba 分词 (来源: main README.md bm25.language)。
- **分层检索语义**：常规靠 L2/L3 快速进入上下文；需要具体事实才下钻 L1/L0；「上带判断方向、下带证据精度」，保证从抽象到原文的确定性 drill-down 路径（Persona→Scenario→Atom→Conversation / MMD→jsonl→refs）(来源: master README.md Technical Implementation；main README.md Core Technology)。
- **装配 = Fixed Binding + ACL 先行**：Agent 与资产的绑定（injection_mode、priority）先按 Team/User/Agent/可见性收敛候选集，再在当前 query 上检索——"先划定谁能用，再检索用什么" (来源: master README.md "Memory isn't a global prompt"；MemoryCore v3 文档 §3.7.10 agent-fixed-asset/set、list-with-detail 带 apply_visibility_filter/touch_usage)。
- **注入路径（Proxy）**：Proxy 管线 `auth(user_key) → sessionInit(交互式选 Team→Agent→Task) → injection(把该 Agent 的 L2/L3 记忆 + Skill + Knowledge 混入 system prompt) → 转发上游 LLM`；会话内可用 `mem:sync` 重刷注入、`mem:create-skill` 就地沉淀；session 级缓存刷新接口按 hook（memory/knowledge/skill 列表 refreshed/skipped）(来源: INSTALL.md Step 4 & "Using Proxy with Agents" & ROADMAP.md mem: 命令表；MemoryProxy v3 文档 §3.3 refresh-cache)。
- **按需取用（知识）**：Wiki/CodeGraph 不整块注入，Agent 先 `POST /v3/tools/list` 自发现该知识资源可用工具（wiki 7 个 / code-graph 9 个只读工具白名单，管理动作不暴露），再 `/v3/tools/call` 执行——文档官方称 "progressive-exposure"；分配给 Agent 的知识资产默认 `injection_mode='tool'` (来源: MemoryKnowledge v3 文档 §3.3；MemoryPanel panel-api-doc §3.10)。
- **Agent 侧接口形态**：① LLM 反代（Anthropic Messages / OpenAI ChatCompletions / OpenAI Responses 三协议 + 9 种客户端画像：Claude Code、Codex、CodeBuddy、WorkBuddy、**DeepSeek Harness(dsh)**、OpenCode、Hermes、OpenClaw、Pi + 通用 header preselect），宣称"零代码、零插件、零 MCP 接入"；② HTTP v3 RPC（全 POST、统一信封 `{code,message,request_id,data}`）+ TypeScript/Python SDK；③ Agent tools：`tdai_memory_search` / `tdai_conversation_search`（插件线注册为主机工具）。**未发现官方 MCP server 形态**（文档只强调免 MCP）(来源: INSTALL.md；MemoryCore README.md https://raw.githubusercontent.com/TencentCloud/TencentDB-Agent-Memory/master/MemoryCore/README.md；main README.md Features 表；dsh 适配器文档 https://raw.githubusercontent.com/TencentCloud/TencentDB-Agent-Memory/master/agents/dsh/README.md)。
- **Skill 注入形态**：`/v3/skill/listing` 直接生成 `<available_skills>` prompt 块，char_budget 默认 8000，超限自动降级为 search 模式返回命中 (来源: MemoryCore v3 文档 §3.2 listing)。

## 5 生命周期与治理

做到什么：
- **多租户**：`service_id` 实例收敛 + team/agent/user/task 四元组数据面强制隔离；KS 跨租户 404 不暴露存在性；Bearer apiKey 常量时间比较；网关鉴权四层（数据面/元数据面/内部运维面/实例销毁）分级 (来源: MemoryKnowledge v3 文档 §1.4；MemoryCore v3 文档 §1.4)。
- **权限**：visibility 四档 + 通用 ACL（grant/revoke/list/check；subject=user|team_role|agent；permission=read/write/delete/assign/share/use；effect=allow/deny 支持显式拒绝）；list-accessible 按人聚合可访问资产 (来源: master README.md；MemoryCore v3 文档 §3.7.11/3.7.9)。
- **身份运维**：user_key 体系（创建仅一次回显全文、可设 expires_at、revoke、last_used_at）；2.0.2-beta 起 Panel 登录兼容 OAuth2 对接企业 OA/SSO，OA 账号与 user_key 打通 (来源: MemoryCore v3 文档 §3.7.2；CHANGELOG.md https://raw.githubusercontent.com/TencentCloud/TencentDB-Agent-Memory/master/CHANGELOG.md)。
- **过期/清理**：资产壳有 `expires_at`；`capture.l0l1RetentionDays`（默认 0=不清理；非 0 强制 ≥3 天，设 1–2 天需显式开 `allowAggressiveCleanup`）+ 每日 `cleanTime` 定点执行 (来源: MemoryCore v3 文档 §3.7.9；openclaw.plugin.json capture)。
- **撤回/删除**：分层清晰——`chat-memory/clear` 一键清空 L0/L1/L2/L3 **内容但保留资产壳（归属/绑定/ACL 不变）**；`layer-delete` 分层批删；`unbind` 解绑、共享可撤回（patch-scope team→private）；Agent `delete-cascade`（先级联清 skill 再 archive）；wiki/code-graph delete 级联清元数据+磁盘+注销引擎；`instance/destroy` 三服务联动销毁全部数据（state/store/COS/quota/元数据分库）(来源: MemoryPanel panel-api-doc §3.4/§3.7/§3.8-3.9；MemoryCore v3 文档 §3.4/§3.9；MemoryProxy v3 文档 §3.1)。
- **溯源/审计（部分）**：`memory-generation-log`——L1/L2/L3 每次生成的日志，支持按 `memory_id+layer` 反查（"这条记忆是哪次生成产生的"）；`memory-prompt` 自定义抽取 prompt 有版本、按 instance/team/agent 三层生效、**apply/replace/clear 操作日志**；`participation-log` 追加式 task 参与事件；资产 `usage_count/last_used_at` touch-usage；`request_id` 跨服务日志链路；可选 Analytics（ClickHouse）记录 memory/skill/wiki/code-graph 工具调用命中率与 token 分布，"Proxy/Knowledge 写、Core/Knowledge 查、Panel 渲染" (来源: MemoryCore v3 文档 §3.5/§3.6/§3.7.8；MemoryPanel panel-api-doc §1.7；INSTALL.md Analytics 节)。
- **配额/频控**：instance-quota、429 语义、Proxy 全局+`(instance_id, model_id)` 维度频控 (来源: MemoryCore v3 文档 §1.5/§3.7.13；MemoryProxy v3 文档 §3.2)。

没做到什么（证据性判断）：
- **无 ChangeSet/Review/Publish 状态机**："审核"只出现在 README 文案（"审核后可分享给团队"、"Team 内 reviewer 角色"）；API 面仅 team-member role 枚举里有 `reviewer`，无评审单、无审批队列、无发布对象；共享动作=一键 `patch-scope` 改可见性 (来源: master README.md vs MemoryCore v3 文档 §3.7.4 与 MemoryPanel panel-api-doc §3.4 全量接口中无 review/approval 端点)。
- **鉴权外包给调用方**：数据面 delete/search 类接口「信任 Bearer + `x-tdai-service-id` 即可，不做用户级 Owner 校验——Owner 校验由面板转发前完成」；KS 整体是内网信任模型（仅必填 service-id header，Bearer 可选）；Proxy 的 session-refresh / force-archive / rate-limits 运维接口**无鉴权**（文档明言注释声称有 admin auth 但实现未调用）；wiki status-callback S2S 无鉴权 (来源: MemoryCore v3 文档 §3.1 注、§3.4 鉴权注；MemoryKnowledge v3 文档 §1.3；MemoryProxy v3 文档 §1.3、§4.2)。
- **删除不可恢复**：skill delete 于 2026-07 由软删改为**物理删除全部版本+清 storage+级联清 meta/ACL/绑定**，且响应字段 `archived` 语义被复用为"删除成功"；无回收站/tombstone/恢复接口 (来源: MemoryCore v3 文档 §3.2 delete)。
- **访问审计缺位**：generation/prompt/participation 日志覆盖"生成与生效"，但**读路径（谁召回了什么、注入了什么、谁看过哪条 L1）无不可变审计**——Analytics 是可选统计埋点而非审计；ContextManifest（本轮实际注入清单）没有可导出的正式产物，仅 refresh-cache 返回 hook 级 refreshed/skipped (来源: MemoryProxy v3 文档 §3.3；INSTALL.md Analytics 节，未见更细接口)。
- **跨实例迁移未交付**：Portable memory（跨 Agent/框架/设备导入导出与在线迁移）在 main 分支 Roadmap 中仍是未勾选未来项；sqlite 与 MongoDB 后端互不迁移、官方迁移工具"后续版本提供" (来源: https://raw.githubusercontent.com/TencentCloud/TencentDB-Agent-Memory/main/README.md Roadmap；INSTALL.md MongoDB 节)。

## 6 界面形态

- **有 Web Console**：Memory Hub（代码名 MemoryPanel，"Team Memory Control"），默认 `http://localhost:8125`；定位是控制面板而非展示板：Team Up（建团队/加人加 Agent/定共享边界）、Asset Library（浏览/搜索/审核/管理四类资产）、Agent Loadout（按 Agent 配装、调优先级与使用模式）、Knowledge Workshop（建 Wiki/CodeGraph、监控处理状态）、Access Control（private/team/ACL 切换、撤回共享）(来源: master README.md "Control Panel" 表；MemoryPanel/README.md https://raw.githubusercontent.com/TencentCloud/TencentDB-Agent-Memory/master/MemoryPanel/README.md)。
- **登录**：首访输入 `user_key`（部署脚本自动生成的 `sk-mem-...`，存 `.admin-key` 文件）；2.0.2-beta 起支持 OAuth2/OA 对接；Panel **无服务端登录会话**（无状态，凭证即 API key）(来源: INSTALL.md Step 1；CHANGELOG.md；MemoryPanel/README.md 项目定位)。
- **页面清单（来自截图 vision 转录，非猜测）**：左侧栏 = Task Board；Organization（Members / Agents / API Key）；Asset Management（Wiki Knowledge Base / Code_Graph / Skills / Chat_Memory）；顶栏含团队切换、语言切换、用户头像 (来源: https://raw.githubusercontent.com/TencentCloud/TencentDB-Agent-Memory/master/assets/images/asset.png 、 chat_memory.png 、 wiki.png 三张截图经 vision 转录)。
- 截图所见交互：资产页三 tab（Team Assets / Agent Assets / My Assets），每个 Chat Memory 块显示 `chat_memory-{team}-{agent}` ID、上传者、时间；详情面板 = **L0 Raw 1583 / L1 Atomic 166 / L2 Scenario 7 / L3 Core 1 四层计数卡 + 场景 .md 文件日期列表**（即 L2 白盒文件直接呈现给用户）；操作按钮 Allocate to Agent / Upload Memory / Shared⇄Private 开关 / Export Skill / Import Skill；资产列表显示 `sk1-` 前缀 Skill ID (来源: chat_memory.png、asset.png vision 转录)。Wiki 详情页 = 面包屑 + `Ready · 25 pages` + Add/Ingest 按钮 + Overview/Graph/Pages/Search 四 tab，Graph 为社区着色的节点-边图（例：20 nodes · 61 edges，节点含 SQLite、Memory Hierarchy(L0-L3)、Data Isolation Model 等）(来源: wiki.png vision 转录)。
- Analytics 数据大盘页（可选启用后渲染）：工具调用频次/命中率、token 分布、跨团队跨 Agent 对比 (来源: INSTALL.md Analytics 节；CHANGELOG.md)。
- README 内嵌 3 段 demo 视频（OpenClaw×、Hermes×、Team 演示）——内容未逐帧验证（未验证）。v1 插件线无 console，白盒目录 `~/.openclaw/memory-tdai/` 直接开文件查（其 Roadmap 中"可视化调试与记忆观测面板"当时未勾选，后被 v2 Panel 实现）(来源: main README.md)。

## 7 技术栈

- **服务拓扑（v2 全栈）**：`memory-core`（:8420，记忆/元数据/Skill/鉴权数据面）+ `MemoryKnowledge` KS（文档默认 :8421、部署脚本用 :8424，wiki/code-graph 内容面）+ `MemoryProxy`（:8096，LLM 反代+注入，Anthropic/OpenAI 双协议多路由）+ `MemoryPanel`（:8125，React 控制台）；`deploy/global-images/start-all.sh` 一键起容器并自动 init-admin (来源: MemoryCore/MemoryKnowledge/MemoryProxy 三份 v3 文档卷首；INSTALL.md 端口表——注意 KS 文档默认 8421 与部署文档 8424 不一致，属文档/配置漂移实例)。
- **语言/框架**：Node.js ≥22.16 + TypeScript（Hono、tsx、Vitest；Panel 前端 React 18 + Vite + Tailwind + Zustand）；Python 用于迁移脚本/hermes supervisor；Docker 镜像（docker.io/agentmemory/memory-hub 等）；SDK：TypeScript + Python (来源: MemoryPanel/README.md 技术栈；MemoryCore/README.md Requirements & SDK；INSTALL.md docker 命令；main README.md badge)。
- **存储与是否绑腾讯 DB**：**不绑定、双层可换**。Standalone（开源单机）= SQLite + sqlite-vec 存 L0/L1 向量与记录（`vectors.db`）+ 本地文件系统存 L2/L3 Markdown（"底层数据库保证据、上层文件保结构"）；Service（云服务化）= **TCVDB（腾讯云向量数据库）** + **COS（对象存储存 L2/L3，per-serviceId 路径隔离）** + Redis（分布式锁+任务队列）+ Shark（动态凭证下发）；另有 **MongoDB 后端（实验特性，默认关，mongot 原生 BM25）**；Proxy session store 用 Redis（TTL 1800s）；Analytics 用 ClickHouse。`storeBackend` 配置枚举即 `sqlite|tcvdb`；embedding 可换任意 OpenAI-compatible 远端（默认禁用则纯 BM25，BGE-M3 兼容开关 sendDimensions）(来源: https://raw.githubusercontent.com/TencentCloud/TencentDB-Agent-Memory/master/README.deployment.md 模式对照表；INSTALL.md MongoDB 节；MemoryProxy v3 文档 §3.1 Redis 注；openclaw.plugin.json storeBackend/embedding；main README.md 配置 Level3)。
- **LLM 依赖**：抽取/归纳/画像/技能沉淀全部依赖可配 OpenAI-compatible LLM；Docker 默认指向腾讯云 LKE + DeepSeek-V3.2，可 BYO 任意端点（TDAI_LLM_* / 分组 MEMORY_LLM_* + PROXY_UPSTREAM_*）；KS 每实例 llm-binding（proxy|byo 两模式，api_key 永不回显）(来源: main README.md docker 参数注释；INSTALL.md 两组 LLM；MemoryKnowledge v3 文档 §3.4)。
- **短期记忆（v1 内核，同仓复用）**：Mermaid 画布 + `refs/*.md` offload + node_id 检索；offload 有 local/backend/client/collect 四模式，backend/client 指向腾讯内部远端服务（该远端未开源，未验证）。

## 8 公开问题与成熟度

- **热度**：27,136 stars、2,600 forks（2026-09-22 抓仓库页内嵌数据 (来源: https://github.com/TencentCloud/TencentDB-Agent-Memory)；shields.io 亦报 27k (来源: https://img.shields.io/github/stars/TencentCloud/TencentDB-Agent-Memory.json)）；246 个未关闭 issue（来源: https://img.shields.io/github/issues/TencentCloud/TencentDB-Agent-Memory.json）；last commit "yesterday"（来源: https://img.shields.io/github/last-commit/TencentCloud/TencentDB-Agent-Memory.json）；曾登 Trendshift 榜（README 徽章，来源: main README.md）。
- **版本节奏**：v2.0.0 2026-08-03 → v2.0.1 2026-08-25 → v2.0.2-beta.1 2026-09-07 → beta.3 2026-09-21；v1.x 插件线并行，v1.0.3（2026-09-22，releases 页标 Latest）。自称 "Team Memory Beta is evolving quickly"（README 顶部）→ 团队线仍属**beta 期快速迭代** (来源: https://github.com/TencentCloud/TencentDB-Agent-Memory/releases；CHANGELOG.md；master README.md)。
- **Issue 热点（open 列表最近 25 条的聚类，来源: https://github.com/TencentCloud/TencentDB-Agent-Memory/issues）**：
  1. **静默失败/可靠性**（最扎眼）：health 报 ok 但 store 已 degraded（#1432）、StorePool 缓存 init 失败实例永不重试（#1433）、L1 抽取失败跳游标丢对话（#1395）、删除接口失败仍回 200（#1434）、Hermes 环境变量传不到 Gateway 致 L1 静默失败（#1386）、禁用 embedding 后中文短查询零命中且不降级（#1382）、sqlite 后端 skill 的 hybrid/embedding 模式"声明有效实现缺席"（#1363）。
  2. **模型配置弹性**：团队自定义模型地址/名称（#1462）、成员自主切换（#1461）、按 L1/L2/L3 分层配模型（#1396）、thinking 无法控制致响应截断（#1403）。
  3. **适配器长尾**：Pi/dsh/Hermes/Windows 各有 bug（#1424/#1422/#1411/#1391-1394 shepherding 系列、#1377 dsh headless 误判跳过注入、#1381、#1380、#1370）。
  4. **检索正确性/归属**：L1 把第三方名字安到用户头上（#1402）、asset get/list 漏元数据检索（#1469）。
- **文档自知的实现偏差文化（少见的坦诚）**：三份 API 文档专门设「已知实现偏差」表（注释声称 admin auth 实际未调用、信封 request_id 三服务不统一、错误码三套格式、endpoint 计数注释与实现不符等），按代码实况记录并提醒前端 (来源: MemoryProxy v3 文档 §4.2；MemoryKnowledge v3 文档 §4.2/§1.2；MemoryCore v3 文档 §3.1 版本类型注)。
- **官方已知限制**：Hermes/OpenClaw 接入强制 `x-task-id`、`x-conversation-id` 需静态手配（同 ID 会串会话），承诺下版解决 (来源: INSTALL.md "Known limitation" 两节)。
- **对比口径**：只与"聊天历史/标准 RAG"做维度对比（§1），无 mem0 等逐项评测（见 §1，未验证第三方对比）。

## 9 可借鉴点（→ 我们企业级概念词汇表）

1. **资产壳与内容体分离**（→ MemoryItem）：meta_asset 把 version/visibility/status/confidence/expires_at/usage_count/source_ref/content_ref 统一收在注册层，四类异构内容共用一套治理面；配合 `clear`（清内容、保壳与绑定/ACL）提供了"撤回但不毁约"的中间态——比"删或留"二值模型治理粒度好 (来源: MemoryCore v3 文档 §3.7.9、§3.4)。
2. **分层+异构存储的白盒证据链**（→ Lineage/审计）：L2/L3 用人类可读 Markdown 文件、L0/L1 用 DB；从画像逐层 drill-down 到原始对话是**确定性路径**且面板直接展示（截图里 L2 场景文件列表带日期）——审计者无需黑盒工具就能核对每条抽象的证据 (来源: master README.md White-Box 节；chat_memory.png 截图)。
3. **generation-log + prompt 版本化生效**（→ ChangeSet 的最小可用前身）：每次 L1/L2/L3 生成有 log 可按 memory_id 反查；抽取 prompt 本身可自定义、有版本、按 instance/team/agent 三层 apply/clear 并留操作日志——"哪版 prompt 对哪个范围生效、何时换的"是可答问题 (来源: MemoryCore v3 文档 §3.5、§3.6)。
4. **权限前置的装配语义**（→ RunGate/ContextManifest）：Fixed Binding + ACL 先收敛候选集再检索；knowledge 绑定默认 `injection_mode=tool`（只挂工具不注全文）；listing 有 char_budget 且超限自动降级 search——三种"注入强度"（fixed/tool/listing）值得抄成装配策略枚举 (来源: master README.md Technical §2；MemoryPanel §3.10；MemoryCore §3.2 listing)。
5. **private 默认 + 共享是显式动作**："Shared Experience, Not Shared Privacy"；含专供配装的 `agent` 可见性档位与 `restricted` 的三维 ACL（user/team_role/agent × 6 permission × allow/deny），比 mem0 式全局库+filter 的默认姿态更贴团队治理 (来源: master README.md 可见性表；MemoryCore §3.7.11)。
6. **Participation-Log 与 usage touch**：轻量追加式"谁/哪任务/哪 Agent 参与过"与 last_used_at/usage_count，给"僵尸记忆"退役评审供数据 (来源: MemoryCore §3.7.8、§3.7.9 list/touch-usage)。
7. **注入效果自评探针**（→ CopyInventory 的评测伴生件）：`/analyse` URL 标记让模型在末回复逐工具交代"帮没帮上忙"，双闸门（config+URL）、明示仅用于内部评测不碰生产——低成本建立"资产×效果"回路 (来源: INSTALL.md /analyse 节)。
8. **运维护栏三件套**：retention<3 天必须显式开危险开关、每日定点清理、`429/403/404` 不暴露存在性、user-key 只回显一次+常量时间比较、跨租户统一 404——小而对的默认安全设计 (来源: openclaw.plugin.json capture；MemoryCore §1.5；§3.7.2；MemoryKnowledge §1.4)。
9. **API 文档记录"代码实况与注释的偏差"并设维护约定**（接口变更须同 PR 更新文档）：治理文档文化的可抄样本 (来源: 三份 v3 文档卷首"维护约定"、MemoryProxy §4.2)。
10. **反代式零改动接入 + 客户端画像**：按 client 协议/系统提示格式做 Profile（CC markdown / CB xml / Pi label-line），配合 header preselect 兜住任意框架——分发策略值得学，但不建议照抄强耦合本身（见 §10-6）(来源: INSTALL.md Pi 节、agents/ 目录表)。
11. **对「企业级可审计」叙事现成的反面教材**：他们把 review/版本/可见性讲进了文案，但 API 层缺 Review/Publish 对象、缺访问审计、软删改物理删——**说明这类缺口在产品化早期是常态，值得我们在概念层先补齐**（逐条见 §10）。

## 10 不可取点

1. **鉴权边界外移到客户端**：内核数据面 delete/search 信任 `Bearer+service-id` 即管理员级，用户级 Owner 校验交给 Panel 转发前——换任何非官方客户端即绕过；企业级必须服务端强制 (来源: MemoryCore v3 文档 §3.1 引注、§3.4)。
2. **运维面裸奔的存量**：Proxy session 强制归档/缓存刷新、rate-limits、KS status-callback 无鉴权（注释与实现不一致且成文）；KS 整体内网信任；Gateway 不配 apiKey 时完全开放、仅启动 WARN (来源: MemoryProxy v3 文档 §1.3/§4.2；MemoryKnowledge v3 文档 §1.3；main README.md Gateway Security 节)。→ 我们 RunGate/接口应默认 require-auth + 显式降级。
3. **删除即毁证**：skill 从软删改物理删（全版本+storage+meta+ACL+绑定级联），`archived` 字段语义被挪用作"删除成功"；无 tombstone/回收站——与可审计撤回需求直接冲突 (来源: MemoryCore v3 文档 §3.2 delete)。
4. **契约漂移无门禁**：同一 version 字段 update 返回字符串、query 返回数字（文档自认"无法同时满足，前端按接口分别处理"）；三服务信封/错误码/分页回显互不一致；KS 端口 8421 vs 部署 8424 两套文档；工具执行失败出现"HTTP 500 但 code=0 成功信封"的特例 (来源: MemoryCore §3.1 注；三份文档卷首对照表；MemoryKnowledge §1.2 isError 特例)。→ 我们 ChangeSet/Review 门要卡 schema 契约，而非事后文档打补丁。
5. **"审核"停留在叙事**：README 三处提审核，但无评审单/审批队列/发布流对象；reviewer 仅是成员角色枚举；共享=改 visibility 一键完成——治理宣称与 API 能力差距是给我们概念（Review/Publish 作为一等对象）留白的好证据 (来源: master README.md vs 全部 v3/Panel API 文档无相关端点)。
6. **组装强绑 LLM 反代 + header 身份**：注入/捕获要求流量常驻经过 proxy，会话绑定靠 x-conversation-id 等 header，官方自认"缺 header 的续轮请求会跳过注入与记录"、Hermes/OpenClaw 必须手配静态 task/conversation id——单点+脆弱身份 (来源: INSTALL.md Known limitation 两节)。→ ContextManifest 应由记忆平台服务端按 run 生成，而非寄生在代理链路上。
7. **静默降级是默认风格**：召回超时直接跳过注入只打 warn、store 坏了 health 仍 ok、L1 失败游标照走（多号 issue 佐证）——可靠性叙事与实况相反；他们自己都排了 "durable outbox/at-least-once/dead-letter" 的 shepherding 重构票 (来源: openclaw.plugin.json recall.timeoutMs；issue #1432/#1433/#1395/#1391)。
8. **腾讯云私有组件耦合与迁移欠账**：service 形态绑 TCVDB/COS/Redis/Shark；skill 导出需 COS（错误码 50303）；v1 offload backend/client 模式指向未开源远端；sqlite↔Mongo 切换明确"不迁移、自己备份"，官方迁移工具在 TODO 里；跨实例 portable memory 未交付 (来源: README.deployment.md；MemoryCore v3 文档 §4.2 错误码表；INSTALL.md MongoDB 节；main README.md Roadmap)。
9. **默认零配置=无治理姿态**：admin key 落 `.admin-key` 明文文件、缺省回落 `default` 桶、auto-sync/健康端点无鉴权、analytics 默认全关——对个人 demo 合理，对企业是必须翻转的默认值 (来源: INSTALL.md Step 1；MemoryCore §1.4；INSTALL.md Analytics 节)。
10. **双产品线共仓的混乱外溢**：main（v1 插件）与 master/`feat/server_team`（v2 团队 hub）README 各讲各的，仓库真实 default branch 是个 feature 分支；ROADMAP 版本号与 CHANGELOG/releases 三处"current release"互不一致（README 称 v2.0.0、ROADMAP 称 v2.0.1-beta.1、CHANGELOG 顶 2.0.2-beta.1、releases Latest 标在 v1.0.3）——发布与分支治理反例 (来源: 本文 §8 各 URL 对照；repo 页 defaultBranch 数据)。
11. **人机会话凭证同一化**：user_key 既当 Panel 登录态又当 API Bearer（Panel 无服务端 session），泄露面与生命周期管理被压在同一串 `sk-mem-...` 上（2.0.2-beta 的 OAuth2 只是登录侧适配） (来源: MemoryPanel panel-api-doc §1.2；MemoryPanel/README.md 项目定位；CHANGELOG.md)。→ 我们应区分人类会话 token 与机器 key。

## 11 来源清单

实际访问（2026-09-22）：
- 仓库页（描述/star/fork/defaultBranch）: https://github.com/TencentCloud/TencentDB-Agent-Memory
- v2 团队线文档（master 分支 raw）:
  - https://raw.githubusercontent.com/TencentCloud/TencentDB-Agent-Memory/master/README.md ／ README_CN.md
  - .../master/INSTALL.md ／ ROADMAP.md ／ CHANGELOG.md ／ README.deployment.md
  - .../master/MemoryCore/README.md ／ openclaw.plugin.json ／ v3-api-memorycore-doc.md
  - .../master/MemoryKnowledge/v3-api-memoryknowledge-doc.md ／ openapi.yaml（仅读头部说明）
  - .../master/MemoryProxy/v3-api-memoryproxy-doc.md
  - .../master/MemoryPanel/panel-api-doc.md ／ README.md
  - .../master/agents/dsh/README.md
  - .../master/assets/images/{chat_memory,asset,wiki}.png（vision 转录）
- v1 插件线: https://raw.githubusercontent.com/TencentCloud/TencentDB-Agent-Memory/main/README.md
- 页面: https://github.com/TencentCloud/TencentDB-Agent-Memory/issues ／ /releases ／ /tree/master/MemoryCore
- 徽章 JSON: https://img.shields.io/github/stars|issues|last-commit/TencentCloud/TencentDB-Agent-Memory.json
- 已知限制: api.github.com 全程 IP 限流（未用）；web_search 未用（损坏）；demo 视频内容、offload 远端服务、TCVDB 内部实现、mem0 第三方对比 = 未验证。
