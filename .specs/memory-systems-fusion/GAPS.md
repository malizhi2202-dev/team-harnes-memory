# GAPS — 现状缺口清单（D-discovery ①② 事实扫描产出）

子议题拆解的事实依据（R16.2）。三档判决：**已有**（我们模型/原型已覆盖，可只对标不改动）／**部分**（概念已有、缺屏/缺字段/缺契约/缺实现）／**没有**（外部有、我们文档+原型均无）。每个外部能力都有来源；无法证实的标「待确认」。**本文件不裁决，只列事实。**

## 0. 事实基线（我们先有什么）

- 原型：`.specs/enterprise-agent-memory-product/PRODUCT-DESIGN.html`（446KB，§0–§10 + 13 屏 p1–p13，`_build/` 构建、check 全绿；人工浏览器 UAT 待执行 = `VALIDATION.md` §7）。
- 平台议题：`.specs/enterprise-agent-memory-platform/` 十议题「已确认（阶段性）」第 76 轮；Schema/阈值/算法/SLA 系统性留白。
- 代码落地：主仓（本仓）为文档仓；可执行参考实现 = TencentDB-Agent-Memory fork（工作区外，见 §4 前注）。backlog P0 未落地项：B13 召回可见性**单入口**（代码约束：单函数+枚举门禁，`backlog:70`）、D1 Memory Quality Eval（recall@k/nDCG golden set，`backlog:97` 与 `:189` 优先序第 6 条「先建评测基线再改召回」）、C1 审计账本、C4/C5 服务端 RBAC 解析/默认拒绝、C7 召回前脱敏、C9 软删+证据；界面承接属 D3 召回解释与调试台（P1，`:99`）与 A5 配置来源解释（P2，`:45`）——**B13 不是界面项**（GD-1 高级产品经理勘误）。
- 元规则：`.specs/LESSONS.md` L-003（「已落地」标记不可靠，需 path:line）、L-007（API 文档≠能力）、L-008（外部公式须回源核实——MindMemOS MMR/半衰期公式曾零命中）。

## 1. 分维度判决（8 维）

### 1.1 分层
| 项 | 判定 | 依据 |
|---|---|---|
| 我们 | MemoryItem L0–L3 + 生命周期**候选态**（§10.1＝9 个领域行各自的候选口径，该节自标「非最终 Schema」）+ ChangeSet→Review→Publish：模型已有；**分层口径冲突未裁决**（`VALIDATION.md` §8） | CONCEPTS/原型 §2.3、:4182 |
| mem0 | 论文两阶段（提取 m=10→更新 s=10）→ 现行 **单遍 ADD-only 累积**（2026-04-14 起），update/delete 降为显式操作；三库分离。论文≠现状，须按版本引用 | docs.mem0.ai/core-concepts/how-it-works.md（2026-09-29 抓取） |
| TencentDB | L0→L1→L2→L3，**L1 实际 7 类两族**（chat 3 类 + work 4 类），上游 API 文档仍写 3 类＝文档落后代码；升层触发参数化（warm-up 1→2→4→8→N、L2 downward-only 90s/1h、L3 并发=1） | 上游 ref `v2.0.2-beta.1`：`MemoryCore/src/core/record/l1-writer.ts:71-106`、`MemoryCore/src/utils/pipeline-manager.ts:53-64,796-861`；`MemoryCore/v3-api-memorycore-doc.md:105` |
| Zep | episode→fact→community 三层 + 云版 6 类 context type；LangMem 认知三分 × **hot-path/background 写入时机二分**；Memobase profile+event 两层+buffer | help.getzep.com/context-types.md；langmem conceptual_guide.md |
| 缺口判决 | **部分**：分层模型已有；「写入时机（对话内 vs 会话后）作为一等设计维度」（LangMem）与「升层触发参数化口径」我们议题有概念、无表达到原型/字段级 | — |

### 1.2 归属
| 项 | 判定 | 依据 |
|---|---|---|
| 我们 | MemorySpace 四类并列（user/team/project/agent），无 public；`belongs_to/participates_in/references` 不自动继承权限（原型 :1023 行区）＝**空间层级+不继承不变量**双强 | 议题 01/03/09；PRODUCT-DESIGN §3.3 |
| 外部普遍形态 | 扁平 ID/单层：mem0 四平行 ID（user/agent/app/run）无层级、OSS 无 app；Zep group_id=命名空间（2026-09 才修并发跨组泄漏，官方明说跨组自行合并）；Memobase project→user 两级；LangMem namespace tuple 自理；Memos Space 单层属籍；Letta agent repo/org repo 两仓层；TencentDB 数据面按 team+agent+user 三元组划隔，但三字段**类型层 optional、缺省回落 `DEFAULT_ISOLATION_ID`/`''` 桶、历史行 `__legacy__` 回填**——「缺省回落默认桶」本身才是负例（GD-1 领域专家口径修正，「强制」措辞收敛） | 各报告来源清单 |
| 缺口判决 | **已有（领先）**。可补强只有一处写法：LangMem「namespace 模板变量运行时注入」＝归属与身份解耦的落地范式，供接口层参考（不建议改模型） | docs.langchain.com/oss/python/langgraph/stores.md |

### 1.3 按需融合
| 项 | 判定 | 依据 |
|---|---|---|
| 我们 | 议题 02/04/10：冲突八分类、追加 Revision 不覆盖、按权限投影加载、分层预算/降级序——**原则已确认，读视图/状态字段/参数全部留白** | 议题原文；`memory-platform-backlog.md` §三 |
| mem0 | 条目自带 `lifecycle_state`/`replaced_by`/`synthesized`；Dream 非破坏 Supersede/Merge/Synthesis + **读视图三挡**（默认含 superseded／`latest_only`／`include_merged`）；`score_breakdown{semantic,bm25,entity}` 可解释融合 | dream.md；get-memory.md；search-memories.md |
| Zep | 双时态四字段（created/valid/invalid/expired），矛盾→旧边写 `invalid_at` 不删；point-in-time 免费获得 | facts.md；how-graph-creation-works.md |
| TencentDB | 批量 LLM 冲突检测 store/update/merge/skip + 多目标 target_ids + merged_priority/timestamps 并集；冲突「以新覆盖旧」、无裁决队列（**本项目对其的改进清单 B6 登记同一缺口——是我们的建议，非对方自认**，GD-1 勘误）；召回 budget 5s 超时/字符预算/RRF k=60/借入≤2 | 上游 ref v2.0.2-beta.1：`MemoryCore/src/core/prompts/l1-dedup.ts:35-134`；`MemoryCore/src/core/hooks/auto-recall.ts`；`.specs/memory-platform-backlog.md` B6 |
| 缺口判决 | **部分**：融合状态的可判定表达（读视图开关＋条目回链）＝没有；双时态**词对已有**（07:87-92 `valid_time/recorded_at`、sec3.html:350 `valid_from/valid_until`），缺的是上移到条目/Revision 层＝部分；预算化召回＝议题有、屏与参数无＝部分 | — |

### 1.4 知识化发布
| 项 | 判定 | 依据 |
|---|---|---|
| 我们 | ChangeSet→Review→Publish 状态机 + 五类 KnowledgeAsset 各有不可替代消费者（§3.3 :1116 行区）+ 报告/导出/审批绑定＝**评审门禁领先**（外部无一家有发布前门禁） | 议题 05；原型 §2.4/§6.8 |
| 我们缺 | **两类不同性质的缺口**（GD-1 修正）：①到期复核已有承载——p8「复核到期」列（protoB.html:486/3115、演示值 :3130）＋不变量「到期资产不得获得新的高风险执行资格」（sec4.html:127/§5:1418），但 §3.3 KnowledgeAsset 字段行（:1114）未登记 `review_due`＝**文档↔原型漂移**，不是能力没有；②使用遥测真没有——`usage_count/last_used_at` 全库零命中 | 本会话 grep＋GD-1 双票回源 |
| TencentDB | Asset 统一登记含 version/confidence/expires_at/usage_count/last_used_at；发布=visibility+ACL+装配（knowledge 固定 injection_mode=tool），撤回=unbind；touch-usage 回写；Agent 模板冷启动克隆资产；「点开资产看它从哪来/哪个版本/分给谁/最近是否被使用」 | types.ts:531-553；panel-api-doc.md:1061-1128；panel proxy.ts:224-296；README_CN.md:226 |
| Memobase | `context()` 把记忆编译成带 token 预算/主题过滤/时间窗的发布物＝「画像即产品」出口形状；团队新方向 Acontext：记忆蒸馏为 **SKILL.md（Markdown 技能文件）可导出跨框架复用** | docs.memobase.io/features/context.md；Acontext README |
| Letta | 发布=向 org 共享 git 仓库提交 + `skills/<name>/SKILL.md` 随 attach 生效（无评审门禁、无供应链审查＝负例） | shared-memory/index.md |
| 缺口判决 | **部分**：发布后使用度量（计数/最近使用/回写）没有；到期复核**已有承载**（屏＋§5 条文）缺的是字段行登记＝漂移；装配口径部分有（p9 已有 Agent 配置与资源装配，缺 injection_mode/priority 粒度与「发布三要素」表述）；SKILL.md 式发布形态＝趋势观察项 | — |

### 1.5 血缘
| 项 | 判定 | 依据 |
|---|---|---|
| 我们 | 议题 07 十三类血缘关系 + 追加事件 + `valid_time/recorded_at` 成对（07:87-92）；§3.5 已有 `get_lineage`（来源闭包＋影响闭包，分层最小投影，:1251）与 `analyze_impact`（:1279）——**模型强**；原型 13 屏**无血缘专属屏**（p5/p7 内嵌片段），侧栏 `#/governance/lineage` 路由存在但无对应屏；**同类悬挂路由共 5 条**（GD-1 复核），血缘是其中唯一被两议题依赖的＝优先修它（初稿把端点写成「code-impact」系 R6.1 违规，已勘误） | CONTRACT §3；grep 核验＋GD-1 三票 |
| 外部最佳 | TencentDB `MemoryGenerationLog`（layer/input_refs/output_refs/prompt_sha256/model/latency）+ L1 `source_message_ids[]` + `generation_ref` 反查端点；mem0 `history[].input`（role/content 原文数组）+ `replaced_by` + Dream sources 端点；Letta git commit/`versions.list(ref)` 仓库级；Zep episode `MENTIONS` 回指 + edge.episodes 列表 + **metadata 沿血缘投影=权限载体**（Enterprise 独有） | 各来源清单 |
| 缺口判决 | **部分**：来源/影响反查已有（`get_lineage`），缺 **消费闭包层**（被哪次 Run/ContextManifest 注入过）与专属屏；「血缘承载权限（metadata projection）」议题 07 有 Visibility Context 前置过滤、无投影算法＝部分 | — |

### 1.6 权限与治理
| 项 | 判定 | 依据 |
|---|---|---|
| 我们 | 8 角色 × 16 动作矩阵三处一致（§7/§5.0/p12）+ 四职责分离 + fail-closed + 证据卡——**文档强**；实现缺（C1 账本/C4/C5 服务端解析未落地，backlog 原文） | backlog；CONTRACT |
| 外部参照 | TencentDB：6 动作×allow/deny ACL + `acl/check` 在线验证端点 + 判定序「资源→owner→成员→visibility→角色→ACL→deny」+ 403/404 语义；负例：数据面删除 trusting Bearer 不查属主、Proxy ops 无鉴权、KS 内网信任。mem0 自认 filters≠授权需客户自建 RLS。Letta Admin/Editor/Analyst + 「分享 agent≠分享会话」。Memos authorship⊥visibility、关系不携带权限。Zep RBAC+ABAC 双主体但全在 Enterprise | permission-checker.ts；v3-api 文档；各报告 |
| 缺口判决 | **没有（交互层）**：「现在就试一下」式权限验证器（acl/check 的产品化）与「这条记忆被谁在用」影响面反查屏；模型层已有，不动 | — |

### 1.7 跨空间脱敏
| 项 | 判定 | 依据 |
|---|---|---|
| 我们 | 议题 10：Q0–Q3 分级、脱敏三段序、双 Owner、9 步撤回、E0–E4 证据——模型最完整；p6 有脱敏预览表；实现缺（C7） | 议题 10；UI-DESIGN |
| 外部 | **七家全部没有**：mem0 仅自然语言 Exclude + Direct Import 逐字旁路（实测 404 否证 PII/加密文档）；Letta 只 redact 凭据；Zep 只有按数据类拒读 ABAC；LangMem/Memobase/Memos 未见；TencentDB 上游只有日志/SQL 参数脱敏，记忆内容层无（其 fork 的 redaction.ts 头注直陈此缺口，属[本地]非上游能力） | 各报告否证清单 |
| 缺口判决 | **已有（独家领先）**。行动=不追赶、补硬化项与不变量核查（见 D-6） | — |

### 1.8 撤回清理证据
| 项 | 判定 | 依据 |
|---|---|---|
| 我们 | Copy Inventory 五桶 + 六态证据 + 9 步协议 + p10/p11 屏（§5.15 清理证据 [锚点 s5-16]/§5.16 报告中心 [s5-17]——初稿误标 §5.12/§5.13 已勘误）——**设计最全**；实现缺（C9）；未成文三语义：依赖计数删除（Zep；GD-1 限定：计数到零＝进清理候选走 9 步第 5 步处置计划，**不自动物理删**；撤销来源影响只能以**追加再评估事件**，07:13/187）、逐条机读证据响应（TencentDB clear 形态，停在语义级）、注入快照「重算即撤销」操作入口（我们有 Manifest 重算原则、无端点级契约） | 议题 10；本会话核对＋GD-1 两票 |
| 外部负例 | mem0 删除异步最终一致「删后几秒仍可见」、无 trash；Zep 删 episode 不撤销它造成的失效、build_communities 重建先清空；Memobase **默认处理完即删原始 blob**（证据销毁默认开）；TencentDB Redis session 不清靠 TTL 自然过期 | 各报告 |
| 缺口判决 | **部分**：三种硬化语义没有成文；反证据销毁条款没有 | — |

## 2. 聚合判决表（外部能力 → 三档）

| 外部能力（出处） | 判决 | 去向 |
|---|---|---|
| 读视图开关 latest_only/include_merged（mem0 dream.md） | 没有（且**默认值属待裁决产品缺口**：既有不变量只排 dispute/隔离/撤回——sec4.html:168、sec5.html:412——superseded 从未定档；GD-1 产品经理：召回轴默认须定成「仅当前有效」，不得用「与 mem0 一致」偷渡） | D-1 |
| 条目级非破坏状态字段可判定（mem0 get-memory.md） | 部分（生命周期列与 superseded 写法**屏上已有**——protoB.html:395/501/588/609、protoC.html:50；缺条目级回链/合成件/按态筛，演示值全 active＝表达不完整而非无能力） | D-1 |
| 双时态有效窗口 created/valid/invalid/expired（Zep facts.md） | 部分（词对已有：07:87-92 `valid_time/recorded_at`、sec3.html:350 `valid_from/valid_until`；缺上移条目/Revision 层；**禁造第三套词**，GD-1 领域专家） | D-1 |
| 写入时机 hot-path/background 一等维度（LangMem） | 部分（议题有「会话后沉淀」，无表达到策略字段） | D-1 |
| 血缘专属屏 + 生成溯源反查端点（TencentDB generation-log；mem0 history） | 部分（来源/影响闭包已有 get_lineage；缺消费闭包层与屏；导航同类悬挂 5 条中优先这条） | D-2 |
| `source_message_ids` 来源消息投影（TencentDB `MemoryCore/src/core/record/l1-writer.ts:71-106`@v2.0.2-beta.1） | 没有（此投影未登记进字段行，外部七家五家无）——**定性**：captured_from 类型化关系的只读投影，真源＝带 07 §3 必填字段的关系记录（GD-1 领域专家，「一等外键」措辞废弃） | D-2 |
| 血缘承载权限 metadata projection（Zep） | 部分 | D-2/D-5 |
| Harness 接入零代码（改 base URL）+ 8 客户端（TencentDB INSTALL_CN）；MCP+IdP+全局 kill switch（Zep 2026-08）；插件矩阵含 deepseek-plugin（mem0 2026-08） | 部分→没有（p9 已有 Connector 能力声明+状态表 `PRODUCT-DESIGN.html:3341`；缺实例级接入屏：注册/租约/撤销/降级；议题 01 契约未落地。GD-1 横切口径：写「哪一层没有＋相邻已有写法在哪」） | D-3 |
| 会话内绑定指令 mem: 6 条 + 归属就地重挂（TencentDB help.ts:12-51） | 没有（运行级 ContextBinding 已在 p5 显示；缺会话级绑定清单与就地改挂入口） | D-3 |
| 每轮资产反思 /analyse 双闸门自评（TencentDB INSTALL_CN:330-406） | 没有（议题 08 有反馈闭环概念、无轮级归因） | D-3/D-4 |
| 资产 usage_count/last_used_at + touch-usage（TencentDB） | 没有（真缺，本会话 grep 零命中）；到期复核已有＝`review_due`（屏+§5 在用），缺字段行登记＝漂移，勿再造第三个到期词（GD-1） | D-4 |
| 发布三要素 visibility+ACL+装配 / 撤回=unbind（TencentDB panel-api-doc:1061-1128） | 部分（p9 有装配；发布语义未成三要素） | D-4 |
| Agent 模板冷启动克隆（TencentDB proxy.ts:224-296） | 没有 | D-4 |
| context() 预算化发布物（Memobase）；SKILL.md 式跨框架复用（Acontext/Letta skills） | 没有（趋势项，含供应链负例） | D-4 |
| acl/check 在线验证 + 判定序可解释（TencentDB permission-checker.ts） | 没有（产品化验证器） | D-5 |
| 影响面反查「谁在用这块记忆」blocks.agents.list（Letta）；CodeGraph impact（TencentDB） | 部分（§3.5 已有 `analyze_impact`/`get_lineage` 的**对象→影响**方向；缺资源→主体、主体→资源集两个主体方向清单；初稿「code-impact」名系误写已勘误） | D-5 |
| 破坏性操作 API 形状兜底：无 filter 报错/四通配才清库/precondition sha256/immutable（mem0；Letta contentSha256） | 部分（fail-closed 原则有，接口形状未成文到 §3.5） | D-5 |
| 按证据依赖计数级联删除（Zep deleting-data） | 没有（未成文；GD-1 限定：计数到零＝进清理候选走 9 步第 5 步处置计划，不自动物理删；撤销来源影响只能追加再评估事件，07:13/187、10:333/1089） | D-6 |
| 逐条机读清理响应 cleared/reason/retryable/attempts（TencentDB clear） | 没有 | D-6 |
| 注入快照重算即撤销 refresh-cache clearBefore（TencentDB proxy ops） | 没有（原则有、契约无） | D-6 |
| 只发派生层/raw 不出域默认策略（TencentDB KS 内网信任、Redis 不清＝负例；Memos share 链接最小切片+TTL+可撤＝正例） | 部分 | D-6 |
| 脱敏内容层能力本身 | 已有（独家） | D-6 确认 |
| Spaces「作者身份⊥可见性」「关系不携带权限」（Memos） | 已有（同义不变量已在 §3.3） | 无（不立题） |

## 3. 内部未裁决/待确认（不新开题，登记）

1. `VALIDATION.md` §8 四组口径冲突（Team↔Project 基数、L0/L3 分层口径、ChangeSet 定位、R/S 与 E 映射）——属 product change 债务，融合改动**不得**在其上叠新口径。
2. TencentDB 基准数字自报（WideSearch +51.52%／PersonaMem 48→76%）；mem0 基准两处不一致（92.5/94.4 PyPI vs 91.6/93.4 changelog）且为托管平台非 OSS；Zep 论文口径过时（对比对象 MemGPT/RAG，未对 mem0）。→ 一律不得引用为能力证据（R6.2）。
3. 待确认：mem0 per-key scope 能力字段；Letta PII 脱敏是否存在、统一删除审计；LangMem store TTL；Memobase profile 条目是否内部保存来源 event id；TencentDB 上游是否有 rerank。（各家「未见」≠「无」，按否证方式引用，标来源与日期。）
4. Letta V1 memory blocks/archival 全部标 legacy、0.27.10 删 block 管道，引用须注明版本；其「分享 agent≠分享会话」四句话可作正例引用。

## 4. 前注（取证口径，GD-1 领域专家三条已采纳重写）

TencentDB-Agent-Memory 证据取自本地克隆 `/home/malizhi/project/TencentDB-Agent-Memory`（用户 fork；上游 `TencentCloud/TencentDB-Agent-Memory`，CHANGELOG 最新 2.0.2-beta.1/2026-09-07）。**上游/本地判定以 commit 作者为据、可复现**（放弃初稿「[本地] 标记」说法——该字样只存在于本文件，fork 文件里并没有）：上游提交为 `220af62 chrishuan feat: release v2.0.2-beta.1`；fork 自有提交如 `3b28b4c malizhi(2026-09-08) 代码恢复` 等 19 次，其引入的文件（redaction.ts、Project/AgentSpace 实体、meta_audit_logs、WriteApproval 等）**不算上游产品能力**。引用路径一律带模块前缀（`MemoryCore/src/...`）并标上游 ref `v2.0.2-beta.1`（本文件与 D-1/2/3/5 内路径已按此改写；裸 `src/...` 前缀不可解析，属取证缺陷已修）。上游 README 称其定位「先落地、后开源」的本地记忆插件，勿与本项目文档仓混同。
