# GAPS — 现状缺口清单（D-discovery ①② 事实扫描产出）

子议题拆解的事实依据（R16.2）。三档判决：**已有**（我们模型/原型已覆盖，可只对标不改动）／**部分**（概念已有、缺屏/缺字段/缺契约/缺实现）／**没有**（外部有、我们文档+原型均无）。每个外部能力都有来源；无法证实的标「待确认」。**本文件不裁决，只列事实。**

## 0. 事实基线（我们先有什么）

- 原型：`.specs/enterprise-agent-memory-product/PRODUCT-DESIGN.html`（446KB，§0–§10 + 13 屏 p1–p13，`_build/` 构建、check 全绿；人工浏览器 UAT 待执行 = `VALIDATION.md` §7）。
- 平台议题：`.specs/enterprise-agent-memory-platform/` 十议题「已确认（阶段性）」第 76 轮；Schema/阈值/算法/SLA 系统性留白。
- 代码落地：主仓（本仓）为文档仓；可执行参考实现 = TencentDB-Agent-Memory fork（工作区外，见 §4 前注）。backlog P0 未落地项：B13 单次召回可见性、C1 审计账本、C4/C5 服务端 RBAC 解析/默认拒绝、C7 召回前脱敏、C9 软删+证据、golden-set 评测。
- 元规则：`.specs/LESSONS.md` L-003（「已落地」标记不可靠，需 path:line）、L-007（API 文档≠能力）、L-008（外部公式须回源核实——MindMemOS MMR/半衰期公式曾零命中）。

## 1. 分维度判决（8 维）

### 1.1 分层
| 项 | 判定 | 依据 |
|---|---|---|
| 我们 | MemoryItem L0–L3 + 9 态 + ChangeSet→Review→Publish：模型已有；**分层口径冲突未裁决**（`VALIDATION.md` §8） | CONCEPTS/原型 §2.3 |
| mem0 | 论文两阶段（提取 m=10→更新 s=10）→ 现行 **单遍 ADD-only 累积**（2026-04-14 起），update/delete 降为显式操作；三库分离。论文≠现状，须按版本引用 | docs.mem0.ai/core-concepts/how-it-works.md（2026-09-29 抓取） |
| TencentDB | L0→L1→L2→L3，**L1 实际 7 类两族**（chat 3 类 + work 4 类），上游 API 文档仍写 3 类＝文档落后代码；升层触发参数化（warm-up 1→2→4→8→N、L2 downward-only 90s/1h、L3 并发=1） | 本地克隆 `src/core/record/l1-writer.ts:40`、`src/utils/pipeline-manager.ts:53-64,796-861`；`v3-api-memorycore-doc.md:105` |
| Zep | episode→fact→community 三层 + 云版 6 类 context type；LangMem 认知三分 × **hot-path/background 写入时机二分**；Memobase profile+event 两层+buffer | help.getzep.com/context-types.md；langmem conceptual_guide.md |
| 缺口判决 | **部分**：分层模型已有；「写入时机（对话内 vs 会话后）作为一等设计维度」（LangMem）与「升层触发参数化口径」我们议题有概念、无表达到原型/字段级 | — |

### 1.2 归属
| 项 | 判定 | 依据 |
|---|---|---|
| 我们 | MemorySpace 四类并列（user/team/project/agent），无 public；`belongs_to/participates_in/references` 不自动继承权限（原型 :1023 行区）＝**空间层级+不继承不变量**双强 | 议题 01/03/09；PRODUCT-DESIGN §3.3 |
| 外部普遍形态 | 扁平 ID/单层：mem0 四平行 ID（user/agent/app/run）无层级、OSS 无 app；Zep group_id=命名空间（2026-09 才修并发跨组泄漏，官方明说跨组自行合并）；Memobase project→user 两级；LangMem namespace tuple 自理；Memos Space 单层属籍；Letta agent repo/org repo 两仓层；TencentDB team+agent+user 三元组强制、缺省回落 `default` 桶（负例） | 各报告来源清单 |
| 缺口判决 | **已有（领先）**。可补强只有一处写法：LangMem「namespace 模板变量运行时注入」＝归属与身份解耦的落地范式，供接口层参考（不建议改模型） | docs.langchain.com/oss/python/langgraph/stores.md |

### 1.3 按需融合
| 项 | 判定 | 依据 |
|---|---|---|
| 我们 | 议题 02/04/10：冲突八分类、追加 Revision 不覆盖、按权限投影加载、分层预算/降级序——**原则已确认，读视图/状态字段/参数全部留白** | 议题原文；`memory-platform-backlog.md` §三 |
| mem0 | 条目自带 `lifecycle_state`/`replaced_by`/`synthesized`；Dream 非破坏 Supersede/Merge/Synthesis + **读视图三挡**（默认含 superseded／`latest_only`／`include_merged`）；`score_breakdown{semantic,bm25,entity}` 可解释融合 | dream.md；get-memory.md；search-memories.md |
| Zep | 双时态四字段（created/valid/invalid/expired），矛盾→旧边写 `invalid_at` 不删；point-in-time 免费获得 | facts.md；how-graph-creation-works.md |
| TencentDB | 批量 LLM 冲突检测 store/update/merge/skip + 多目标 target_ids + merged_priority/timestamps 并集；冲突「以新覆盖旧」、无裁决队列（其自家 backlog 亦列为缺口）；召回 budget 5s 超时/字符预算/RRF k=60/借入≤2 | l1-dedup.ts:35-134；auto-recall.ts |
| 缺口判决 | **部分→没有**：融合状态的可判定表达（条目状态字段+读视图开关）与双时态有效窗口＝**没有**；预算化召回＝议题有、屏与参数无＝部分 | — |

### 1.4 知识化发布
| 项 | 判定 | 依据 |
|---|---|---|
| 我们 | ChangeSet→Review→Publish 状态机 + 五类 KnowledgeAsset 各有不可替代消费者（§3.3 :1116 行区）+ 报告/导出/审批绑定＝**评审门禁领先**（外部无一家有发布前门禁） | 议题 05；原型 §2.4/§6.8 |
| 我们缺 | KnowledgeAsset 字段行（:1116 区）无 `usage_count`/`last_used_at`/`expires_at`（全文件 grep：`expires_at` 仅出现在 Delegation :1028 与 ExportAsset :1188；usage 计数零命中）＝发布后度量与到期治理没有 | 本会话 grep |
| TencentDB | Asset 统一登记含 version/confidence/expires_at/usage_count/last_used_at；发布=visibility+ACL+装配（knowledge 固定 injection_mode=tool），撤回=unbind；touch-usage 回写；Agent 模板冷启动克隆资产；「点开资产看它从哪来/哪个版本/分给谁/最近是否被使用」 | types.ts:531-553；panel-api-doc.md:1061-1128；panel proxy.ts:224-296；README_CN.md:226 |
| Memobase | `context()` 把记忆编译成带 token 预算/主题过滤/时间窗的发布物＝「画像即产品」出口形状；团队新方向 Acontext：记忆蒸馏为 **SKILL.md（Markdown 技能文件）可导出跨框架复用** | docs.memobase.io/features/context.md；Acontext README |
| Letta | 发布=向 org 共享 git 仓库提交 + `skills/<name>/SKILL.md` 随 attach 生效（无评审门禁、无供应链审查＝负例） | shared-memory/index.md |
| 缺口判决 | **没有**：发布后使用度量/到期复审字段与屏内展示；**部分**：装配口径（p9 已有 Agent 配置与资源装配，缺「发布三要素」「模板冷启动」概念）；SKILL.md 式发布形态＝趋势观察项 | — |

### 1.5 血缘
| 项 | 判定 | 依据 |
|---|---|---|
| 我们 | 议题 07 十三类血缘关系 + 追加事件 + valid_time 字段 + 影响分析（code-impact 端点 §3.5）——**模型强**；原型 13 屏**无血缘专属屏**（p5/p7 内嵌片段），侧栏 `#/governance/lineage` 路由存在但无对应屏（本会话核验：CONTRACT §3 路由表 vs p1–p13 标题树）＝内部缺陷；「记忆↔源消息」反查端点契约未成文 | CONTRACT §3；grep 核验 |
| 外部最佳 | TencentDB `MemoryGenerationLog`（layer/input_refs/output_refs/prompt_sha256/model/latency）+ L1 `source_message_ids[]` + `generation_ref` 反查端点；mem0 `history[].input`（role/content 原文数组）+ `replaced_by` + Dream sources 端点；Letta git commit/`versions.list(ref)` 仓库级；Zep episode `MENTIONS` 回指 + edge.episodes 列表 + **metadata 沿血缘投影=权限载体**（Enterprise 独有） | 各来源清单 |
| 缺口判决 | **部分**：反查端点契约与专属屏没有；「血缘承载权限（metadata projection）」议题 07 有 Visibility Context 前置过滤、无投影算法＝部分 | — |

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
| 我们 | Copy Inventory 五桶 + 六态证据 + 9 步协议 + p10/p11 屏（§5.12/§5.13）——**设计最全**；实现缺（C9）；未成文三语义：依赖计数才删（Zep）、逐条机读证据响应（TencentDB clear 形态）、注入快照「重算即撤销」操作入口（我们有 Manifest 重算原则、无端点级契约） | 议题 10；本会话核对 |
| 外部负例 | mem0 删除异步最终一致「删后几秒仍可见」、无 trash；Zep 删 episode 不撤销它造成的失效、build_communities 重建先清空；Memobase **默认处理完即删原始 blob**（证据销毁默认开）；TencentDB Redis session 不清靠 TTL 自然过期 | 各报告 |
| 缺口判决 | **部分**：三种硬化语义没有成文；反证据销毁条款没有 | — |

## 2. 聚合判决表（外部能力 → 三档）

| 外部能力（出处） | 判决 | 去向 |
|---|---|---|
| 读视图开关 latest_only/include_merged（mem0 dream.md） | 没有 | D-1 |
| 条目级非破坏状态字段可判定（mem0 get-memory.md） | 部分（superseded 枚举已有 :864/§10.1，无条目屏表达） | D-1 |
| 双时态有效窗口 created/valid/invalid/expired（Zep facts.md） | 没有（仅血缘表有 valid_time） | D-1 |
| 写入时机 hot-path/background 一等维度（LangMem） | 部分（议题有「会话后沉淀」，无表达到策略字段） | D-1 |
| 血缘专属屏 + 生成溯源反查端点（TencentDB generation-log；mem0 history） | 没有（屏与契约均缺；导航入口已在） | D-2 |
| source_message_ids 一等外键（TencentDB l1-writer.ts:64-106） | 没有（外部均无，仅 mem0 history.input 旁路）＝差异化项 | D-2 |
| 血缘承载权限 metadata projection（Zep） | 部分 | D-2/D-5 |
| Harness 接入零代码（改 base URL）+ 8 客户端（TencentDB INSTALL_CN）；MCP+IdP+全局 kill switch（Zep 2026-08）；插件矩阵含 deepseek-plugin（mem0 2026-08） | 没有（原型无接入屏；议题 01 契约未落地） | D-3 |
| 会话内绑定指令 mem: 6 条 + 归属就地重挂（TencentDB help.ts:12-51） | 没有 | D-3 |
| 每轮资产反思 /analyse 双闸门自评（TencentDB INSTALL_CN:330-406） | 没有（议题 08 有反馈闭环概念、无轮级归因） | D-3/D-4 |
| 资产 usage_count/last_used_at/expires_at + touch-usage（TencentDB） | 没有 | D-4 |
| 发布三要素 visibility+ACL+装配 / 撤回=unbind（TencentDB panel-api-doc:1061-1128） | 部分（p9 有装配；发布语义未成三要素） | D-4 |
| Agent 模板冷启动克隆（TencentDB proxy.ts:224-296） | 没有 | D-4 |
| context() 预算化发布物（Memobase）；SKILL.md 式跨框架复用（Acontext/Letta skills） | 没有（趋势项，含供应链负例） | D-4 |
| acl/check 在线验证 + 判定序可解释（TencentDB permission-checker.ts） | 没有（产品化验证器） | D-5 |
| 影响面反查「谁在用这块记忆」blocks.agents.list（Letta）；CodeGraph impact（TencentDB） | 部分（§3.5 有 code-impact 端点，无记忆→使用者反查） | D-5 |
| 破坏性操作 API 形状兜底：无 filter 报错/四通配才清库/precondition sha256/immutable（mem0；Letta contentSha256） | 部分（fail-closed 原则有，接口形状未成文到 §3.5） | D-5 |
| 按证据依赖计数级联删除（Zep deleting-data） | 没有 | D-6 |
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

## 4. 前注（取证口径）

TencentDB-Agent-Memory 证据取自本地克隆 `/home/malizhi/project/TencentDB-Agent-Memory`（用户 fork，上游 `TencentCloud/TencentDB-Agent-Memory`，CHANGELOG 最新 2.0.2-beta.1/2026-09-07）。fork 内带[本地]标记的文件（redaction.ts、Project/AgentSpace 实体、meta_audit_logs、WriteApproval 等）**不算上游产品能力**；上文引用 TencentDB 处均为上游核对件（permission-checker.ts、v3-api 三份文档、l1-writer/l1-dedup/auto-recall/pipeline-manager/proxy 等，raw 200 比对）。上游 README 称其定位「先落地、后开源」的本地记忆插件，勿与本项目文档仓混同。
