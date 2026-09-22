# LOCAL-FACTS — 本地文档事实摘要（供竞品调研融合）

- **生成日期**: 2026-09-22 · **审核**: 已过审（2026-09-22 18:56 · 用户原话「通过」，随 PRODUCT-DESIGN v0.3） · 生成方式: 只读蒸馏本仓库既有文档，不修改任何源文件
- **用途**: 新竞品调研（TencentDB-Agent-Memory / mem0 / MemOS(memos) / 其他记忆系统）的增量基线——已有结论不重复，只补空白
- **口径与可信度警告**:
  - 本仓库**无源代码**。所有「已落地 ✅」标记均指**文档层面声称**，指向外部代码库（TencentDB-Agent-Memory monorepo：MemoryCore / MemoryKnowledge / MemoryPanel / MemoryProxy），本仓无法复核。
  - `.specs/CONTEXT.md`「文档可信度警告」明令：`MemoryCore/v3-api-memorycore-doc.md`、`MemoryPanel/panel-api-doc.md` 与实现已脱节（104 条路由未文档化），**判断能力有无一律以源码为准**；`.specs` 内所有「✅ 已落地」须附证据。
  - 「设计 vs 已落地」全文分标：〔设计〕=文档设想；〔声称落地〕=实现/改动清单类文档附文件级证据；两者冲突时以后者+日期新者为准。

---

## 1. 产品当前定义速览（≤40 行）

来源：`.specs/enterprise-agent-memory-product/CONCEPTS.md`、`CHANGE.md`、`VALIDATION.md`、`PRODUCT-DESIGN.html` §1/§2（change-id `enterprise-agent-memory-product`，2026-09-21，纯文档 change）。

- **定位**〔设计〕: Enterprise Agent Memory Platform = 外部 Harness（Codex / Claude Code / DSH 等）的**记忆、知识、血缘与治理平台**。Harness 负责执行；平台负责保留、理解、授权、复用、撤回、验证并审计工作证据。主链：`Harness/Connector → Evidence → MemorySpace/CodeScenario/Lineage → Triage → ChangeSet → Review → Publish → Retrieval/ContextManifest → ActionProposal → RunGate → Run → Verification/Audit`。
- **用户/角色**（8 类，每人附「不自动获得的能力」）: User / Team Owner / Project Admin / Memory·Knowledge Owner / Agent Admin / Governance·Security / Provider·Processor / Auditor·Legal。
- **核心对象模型**（四组）:
  - 身份组织: `User / Team / Project（唯一 owning Team）/ Agent（版本化）/ Harness`
  - 工作对象: `WorkContext / Task / Run / CodeScenario / Evidence`
  - 记忆知识: `MemorySpace（owner 仅 user|team|project|agent，无 public）/ MemoryItem / KnowledgeAsset（Wiki·Skill·RCA·Template·CodeContext）/ ChangeSet / ProcedureTemplate`
  - 安全副本: `Purpose / VisibilityContext / ContextManifest / CopyInventory / ExportAsset / ToolPolicy（硬边界）/ RunGate（动态准入）/ ActionProposal（默认不可执行）`
- **五维不可混淆**（PRODUCT-DESIGN §2.1）: Identity ≠ Relation ≠ Ownership ≠ Context ≠ Authorization；关联/可见 ≠ 可写/可执行。
- **核心不变量**（CONCEPTS §4，8 条）: ①owner≠visibility≠ACL≠execution authority ②发布/检索不授执行权 ③权限过滤先于召回/图分析/投影/模板渲染 ④requested≠executed≠verified、retained≠service-visible、unknown≠zero ⑤恢复/重建/迁移产生新副本新 revision，旧证明不继承 ⑥外部副本无法证明时停止高敏复用与「全量已清理」声明 ⑦技术阻断先于法务判断 ⑧补偿/回滚/替代 Provider 是新的受授权动作。
- **边界承诺**（§1.4）: 不做 Harness/编排运行时；不提供 public Memory；不用泛化 Asset 表替代专业资源页；原型不执行真实副作用；阶段性方案一律标「待确认」不冻结。
- **导航唯一**: 工作 / 知识与复用 / 输入与治理 / 组织与平台 四域；历史 final-v1、redesign-v2/v3 仅参考禁改。
- **未裁决口径冲突 4 类**（VALIDATION §8、PRODUCT-DESIGN §10.2，标「待确认」）: ①Team↔Project 基数（1:N 归属 vs redesign-v3 N:N 关联，本文档暂按 1:N）②Run/Attempt 嵌套层级（议题6 三层 vs CONCEPTS 两层，暂按 Task→Run）③ChangeSet 是否知识化统一变更对象 ④同名风险分级 R0–R3 / S0–S3 / E0–E4 / E0–E3 映射缺失。
- **与内核产品的关系**: 本平台设计以旧内核 TencentDB-Agent-Memory（L0–L3 分层、四轴隔离）为现状宿主（§3），设计层远比其宽（血缘/治理/报告为平台独有）。

---

## 2. PRODUCT-DESIGN.html 已承载的内容清单

来源：`.specs/enterprise-agent-memory-product/PRODUCT-DESIGN.html`（单文件 444KB / 4,353 行 / 零外部依赖 / 纯黑暗色可切换）与 `_build/CONTRACT.md`、`_build/sec*.html`。§5.0 ≡ §7.1 ≡ 原型 p12 三处权限矩阵逐格一致（16 模块 × 8 角色，● 可操作 / ○ 只可见 / — 不可见）。

| 章 | 内容 |
|---|---|
| §0 导读 | 文档定位、按角色阅读路径、「待确认/暂缓」约定、版本与来源 |
| §1 产品全景 | 1.1 一句话定位；1.2 目标用户与典型场景（S1 跨 Harness 续办 / S2 归属分流 / S3 经验知识化…）；1.3 能力地图（模块×核心能力×主要角色，与 §5/§7 逐字对应）；1.4 不做什么（边界即需求） |
| §2 核心概念 | 2.1 五个不能混淆的维度（Identity/Relation/Ownership/Context/Authorization + 被禁止的权限推导清单）；2.2 组织与工作对象；2.3 记忆与知识；2.4 安全与副本；2.5 执行与人工闸门；2.6 核心不变量 |
| §3 系统架构 | 3.1 部署拓扑；3.2 组件职责；3.3 数据模型（按域）；3.4 技术栈；3.5 接口一览（**候选契约，本 change 不实现**） |
| §4 核心时序 | 六条主链：4.1 Harness 接入与跨 Harness 交接；4.2 记忆自动归属与分诊（Triage→ChangeSet）；4.3 知识化与发布（Review→Publish→消费反馈）；4.4 按需融合与上下文投影（ContextManifest→Harness 投影）；4.5 跨空间复用与执行门禁（ActionProposal→RunGate→受约束 Run→Verification）；4.6 撤回与清理证明（PropagationStop→CopyInventory→分层处置→独立验证→分层报告） |
| §5 使用指南 | 18 个模块级操作手册（5.1 登录导航 / 5.2 工作台 / 5.3 项目 / 5.4 任务 / 5.5 运行 / 5.6 Agent / 5.7 记忆与空间策略 / 5.8 资源库 / 5.9 知识资产 / 5.10 代码场景 / 5.11 导入 / 5.12 分诊 / 5.13 变更发布 / 5.14 血缘 / 5.15 清理证据 / 5.16 报告中心 / 5.17 团队与成员 / 5.18 设置·权限矩阵·审计），每模块固定四段：入口路由 / 角色可做 / 产出与通知 / 异常态 |
| §6 界面原型 | 13 屏（6.1 登录 / 6.2 工作台 / 6.3 项目 / 6.4 任务 / 6.5 运行详情 / 6.6 分诊 / 6.7 记忆与空间策略 / 6.8 知识资产 / 6.9 Agent 配置与资源装配 / 6.10 清理证据 / 6.11 报告中心与投递 / 6.12 团队·权限矩阵·审计 / 6.13 状态与边界态画廊），复刻真实骨架、数据全为演示、侧栏 20 项 12 屏一致 |
| §7 角色权限矩阵 | 7.1 模块×角色矩阵；7.2 读表三前提；7.3 职责分离（SoD）硬约束 |
| §8 术语表 | 加粗术语权威定义（与散落各章简述不一致时以此表为准） |
| §9 FAQ | 12 问，全部围绕易误解处（如 Q4「已执行≠已删除」、Q7「Run 成功≠Task 完成」、Q9「撤回后报告仍有 unknown」） |
| §10 附录 | 10.1 状态与枚举（候选口径非最终 Schema）；10.2 口径冲突与待确认清单（4 类，见 §1 末）；10.3 验证基线与自检；10.4 相关文档索引；10.5 维护约定 |

验证状态（`VALIDATION.md`）: 静态自检 + Chromium 真实浏览器渲染全绿；**人工通读未做**；文档内含 70 处「待确认/暂缓」标记、73 处高风险动作 blocked。

---

## 3. 已落地/实现类文档的事实

（按文件一段：声称落地了什么 ↔ 与 enterprise-agent-memory-product 设计的关系）

### 3.1 `.specs/memory-platform-implementation.md` —「补全所有缺失」证据清单（2026-09-01~02）
〔声称落地〕按 comparison-v2 差距清单逐项补全外部代码库 MemoryCore：P0（Brain 12 域 + 写入策略四档 + Agent 创建挂载空间 + 服务端写路由 + space_id 落库 + 召回 Retention 层 + tokenBudget）/ P1（ExecutionBundle 编译 + Dreaming 7 类关系检测 + 巩固执行 + Feedback 回路 + 质量 8 态状态机 + Playbook + ExecutableScene + skill 版本）/ P2（写前脱敏 + 召回护栏注入过滤 + RBAC 审批门 + 审批五档 + 巩固监控 + recall@k/MRR/NDCG 评测 + archive 召回排除 + 确定性 tie-break）。测试 44→197 全绿。剩余如实列：Rerank 模型、三层可观测面、计划/任务/run 三分、外部基准 LoCoMo/PersonaMem、AgentEntity 多对多持久化。
**vs 设计**：写模型与算法细节比设计的议题文档更「硬」（有枚举与参数）；但设计的治理核心（血缘闭包、CopyInventory、撤回证明、报告投影、RunGate/ToolPolicy 分离、Purpose 维度）在此完全缺失——**设计超前**。⚠️ 其 Retention 混合分数源自 MindMemOS 参数表，该表后被 adoption 文首复核推翻（见 3.9），此实现的公式成分需按源码复核。

### 3.2 `.specs/memory-platform-changes.md` — 上述交付的 git 级核对（2026-09-02）
〔声称落地+高可信证据〕62 文件（21 实现+22 测试新增、6 接线、6 metadata 挂载、6 文档脚本），vitest 201 全绿；附录 B/C/D 给出宿主全貌：既有 L0-L3 流水线（l0-recorder/l1-writer/l1-extractor/scene/persona）、skill 体系、otel+clickhouse 可观测三平面、tcvdb/sqlite 双存储。附录 A：更早 HEAD「部署memory」已含 caller 鉴权 + public/user/project 三桶 scope（`memory-scope.ts`）。
**vs 设计**：与 3.1 同；额外价值=「现在真实有什么」的最可靠清单（本设计的议题文档均以此为现状基线）。

### 3.3 `.specs/design-memory-domain-write-model.md` — 域写入模型设计（阶段 A 已在 3.1 落地）
〔设计→部分声称落地〕12 域枚举（raw/episodic/semantic/preference/instruction/rule/procedural/task/artifact/persona/graph/archive，后 5 个新增）+ 每域 lifecycle/defaultPolicy/recallWeight/defaultRecall profile 表 + write_policy 四档（automatic/review_required/explicit_only/deny）+ 铁律「系统总结不可覆盖用户显式持久更改、可收紧不可放宽」+ MemorySpace 接口（ownerType 含 **task**）+ WriteRouter「模型只能请求、服务端裁决」。
**vs 设计**：与设计「持久更改 vs 记忆总结」心智一致；设计继承并扩展之（加 Purpose、血缘、Triage 白名单）。差异点：write-model 的 ownerType 有 task，设计 CONCEPTS 的 MemorySpace owner 只有 4 类（无 task）——小的口径漂移。

### 3.4 `.specs/design-memory-isolation-m1.md` — 隔离 M1 落地设计（M1a/M1b 声称完成）
〔设计+部分声称落地〕`meta_projects`/`meta_project_members` 两表 + 4 类核心资产加可空 `project_id`（零回归）+ scope 三桶**派生不落库**（public=visibility:team∧¬project；user=private∧¬project；project=挂 project_id）+ `/v3/meta/project/*` 8 action 契约 + 创建默认 private。后续 M2~M4（RBAC R1–R5 强制、proxy 多信号 project-anchor、前端四维度切换）的 ✅ 状态登记在 `.specs/CONTEXT.md`「实施状态」段（同样是文档声称）。
**vs 设计**：与设计的「默认私有、共享显式、注入永远白名单、一份记忆只挂一个 project」一致（CONTEXT 三铁律）；但 M1 的 scope 是**静态三元组推导**，设计要求的 VisibilityContext 含 Purpose/委托/策略版本/revision——**设计超前**；M1-M4 也不含撤回/副本/报告。

### 3.5 `.specs/memory-api-ui-plan.md` — 记忆能力三层接线计划（§5 声称已完成 E2E）
〔声称落地〕21 个 record 纯函数中除已自动接线者外全部补 `/v3/memory/*` 23 端点 + 面板代理 + Workbench `MemoryCapabilitiesPanel`（5 tab）+ AgentsPage 空间徽章 + ChatMemoryPage 反馈按钮；225 tests；端到端 curl 验证与落库验证（10 空间挂载）。
**vs 设计**：这版 UI 暴露的是**内核纯函数调试台**（review/gate、lifecycle/decide…），随后即被 3.6 文档自我否定（「不再暴露内核纯函数」）——**落后于** 3.6 与设计 §5 的「对象管理面」形态；与设计的治理视图（血缘/清理/报告）无重叠。

### 3.6 `.specs/memory-platform-capabilities-ui.md` — 「平台能力」交互稿（设计，未声称实现）
〔设计〕定位「把记忆当作可管理对象」5 tab：记忆浏览器 / 检索试调台（召回解释）/ 显式 remember·correct·forget / 治理审批（空间写策略+队列+审计账本）/ 评测观测（评测集+回归+RunTrace 回放）；每 tab 顶部「团队×Agent×空间」作用域筛选条；§7 逐条列现有 vs 需新建端点；§8 与 Zep/Letta/Mem0/AO/MM 能力逐点对照。
**vs 设计**：心智一致（先选 scope 再看对象、显式操作带来源审计、召回解释、门禁），可视为设计 §5.7/§5.12 的窄域先行版；覆盖仅记忆单域，无知识化/血缘/清理/报告——**设计超前**。

### 3.7 `docs/adr/memory-platform-backlog.md` 与 3.8 `.specs/memory-platform-backlog.md` — 同一基线两版本
〔设计/路线图〕5 组 47 条（A 关系与运行控制面 10 / B 记忆算法召回 / C 治理安全 / D 质量功能 / E 生态迁移）+ 四阶段优先级 + 总判断「**不缺实体，缺关系灵活性+运行控制面+记忆闭环+质量治理**」。`.specs` 版为权威（自称唯一对照基线），含 2026-09 逐文件复核增补：**新增 B11 Jaccard 近重复折叠 / B12 superseded_by 自引用取代关系 / B13 召回可见性单入口 P0 / B14 时间边只承诺左边界 / C11 配置必须有生效路径 / C12 禁止静默回退 / C13 遗忘必须有生产调用方 / C14 聚类必须带全部 scope 条件（P0）/ C15 审计与鉴权同源**；B1 MMR 公式与 B8 schema learning 加「被证伪」标注；C2 recall scope 隔离标 ✅ 已实施；最终结论加第 5 条「**治理的唯一有效形式是可执行门禁**」与第 6 条「**先建评测基线再改召回**」。`docs/adr` 版是 2026-09-01 旧拷贝，无上述更正——两文件有漂移，以 `.specs` 版为准。
**vs 设计**：backlog 管「内核能力演进」，设计管「治理与产品面」，互补不冲突；backlog 的 D 组（浏览器/统计/助手验证）与设计 §5 模块重叠但设计更完整。

### 3.9 `.specs/intent-os-mindmemos-adoption.md` 文首复核表（事实上的「已否决清单」）
〔复核结论，见 §4.2〕——它是判断 3.1 实现质量的关键上下文。

### 3.10 `.specs/CONTEXT.md` + `STATE.md` + `tasks/plan.md` + `tasks/todo.md`
- CONTEXT：项目共享上下文单一入口（TencentDB Agent Memory 产品定义、技术栈锁定、域语言、四层隔离、规则式 RBAC R1–R5、M2–M4 实施状态、对标同类产品一句话结论、2026-09 机械对账「后端零删除/前端 82 路径删除」、文档可信度警告、技术债清单）。**「市面无产品同时覆盖 team×project×user×harness 四轴」是竞品定位判断的现行基线**。
- STATE/CHANGELOG：最近一次 change = enterprise-agent-memory-product 纯文档重绘（§2）；其前一 change = health-sweep（产出 page-parity-audit + 更正若干文档）。
- `tasks/plan.md`/`tasks/todo.md`：「Agent Memory OS UI V2」5 阶段 25 任务 + C1–C8 契约门禁清单（Panel 前端重构计划）——**全部未勾选，属〔设计〕未落地**；其信息架构（Project/Agent/Task/Memory Center 五组导航）与设计的四域导航相近但更贴 Panel 现状。

### 3.11 `.specs/enterprise-agent-memory-platform/`（议题 01–10 原始文档 + README）+ `_facts/*-digest.md`（蒸馏稿）
〔设计，权威议题层〕十议题=产品核心。README 第 76 轮审计「阶段性综合路线图」给出 8 条跨议题能力骨架（来源版本血缘 / 权限用途门禁 / 派生与撤回 / 清理与保留 / 恢复与一致性 / 跨边界报告通知 / 外部副本与供应商 / 高影响安全默认值）；Schema/阈值/SLA/合同/法域**全部显式暂缓**。蒸馏稿结构统一（已确认结论/对象/状态枚举/角色权限/不变量/界面线索/暂缓项）。逐议题已拍板核心（全部〔设计〕层）：
- **议题1 跨 Harness 连续性**（59 条确认）：工作真源=Work Thread（≠Session/Run/Chat）；交接产物=Continuation Package（规范状态+按执行者投影，confirmed_fact/inference/hypothesis…分开）；现场恢复 L1 语义/L2 操作/L3 环境三层；Connector Contract V1 八项最小能力（Handshake/Event Append/Idempotency+Cursor/Capability/Resource Ref/Health+Lease/Probe/Structured Result）；权限=九要素交集；合并必须绑共同基线三方比较（禁 last-writer-wins/投票）；四类 Cursor 分离；「平台删除≠外部副本清除」。
- **议题2 记忆自动分层**（30 条）：⚠️ 未展开 L0–L3 全定义（口径校正：L1=原子记忆、L2=可恢复场景、L3=长期演进）；L1 存储=规范对象+追加事件+不可变快照+派生投影；9 态（observed/inferred/human_confirmed/current/stale/disputed/retracted/invalidated/unknown）；8 类冲突；M0/M1/M2 自动分级；撤回 9 步协议+10 类处置动作；置信度只做轻量分带不建概率平台。
- **议题3 记忆自动归属**（31 条）：单一 `canonical_owner_scope`+多关系；「候选+规则裁决」禁模型直写 owner；默认落点保守（无法判断→窄边界/L0/ownership_pending）；四种处置（自动落位/暂存/治理队列/隔离）；归属变化沿事件传播、检索重解析「不得先全库召回再 UI 隐藏」；历史存在≠当前可读。
- **议题4 记忆按需融合**：权限先过滤的混合检索+带来源轻量融合（constraint/fact/exception/suggestion/dispute 分开）+面向 Harness/Task 最小投影+上下文预算；记忆文本不得变成工具授权；「质量—预算—结果」三层指标；反馈不能自动改 ACL/敏感边界/工具权限。
- **议题5 记忆自动知识化**：统一「候选—审核—发布—复核—撤回」流程治理 Wiki/Skill/RCA/Template/CodeContext；**Published≠Actionable**（仍需 ToolPolicy/RunGate）；不可变版本+源变化事件触发影响评估（低/中/高=异步/暂停/同步阻断）；派生资产权限不得宽于来源。
- **议题6 代码场景与根因**（25 条）：事件日志+当前快照+结构化证据+可选 RCA 摘要（第一版不建图数据库）；`repo+revision` 为共同硬锚点；事实/假设/确认根因/促成因素分开；「修复有效≠根因完整、未复现≠不存在、测试通过≠生产风险消失」；失败路径一等数据；复用权限=六者交集；Patch 建议默认不可执行。
- **议题7 企业记忆血缘**（7-C1…C10）：血缘=结构化引用集合+追加事件+来源/影响视图（不建完整图库）；Assertion→验证→Current Relation 三段式（声明者/验证者/消费者分离）；血缘可见性与源内容/检索/执行权限分离，隐藏节点不得经名称/数量/错误/缓存/Embedding 侧信道泄露；血缘/关键词/向量/缓存/Manifest 共享版本化 Visibility Context、过滤先于召回；紧急撤回两阶段「权威状态先行、同步最小阻断、异步影响闭包」；导出=独立 Export Asset（E0–E3 分级、加密/水印/DLP 不可替代）。
- **议题8 反馈自进化**（8-C1…C8）：闭环「反馈事件→归因→候选→验证→受控审核→发布→灰度回滚」；任务成败不能直接证明记忆对错；阶段归因链可审计可重放可回滚优先于自动因果裁决；退休软式 `active→deprecating→restricted→archived`（不得因未使用直接删）；投毒四重检查，可疑反馈隔离不静默删；跨组织独立性 I0–I4 分级；灰度稳定分桶+硬护栏停止。
- **议题9 权限治理**（11 条）：授权模型「主体—资源—动作—用途—作用域—条件—风险—当前状态」，读取/检索/引用/投影/写/审核/发布/撤回/影响分析/审批/执行**分别授权**；继承只收紧、委托 ⊆ 被委托者、审批≠执行；R0–R3 按动作实例；Break-Glass 限时限域+事后复核不追认；撤回对运行中 Run「逐 Attempt 重检、取消≠副作用已停止」；N0–N3 四级通知；SLA 拆 TTA/TTC/TTI/TTR/TTP。
- **议题10 跨空间复用与脱敏**（21 轮全部完成，最后轮 2026-09）：第 1 轮拍板「**受控引用或受控派生，而非原文复制**」——8 种操作（reference/project/derive/promote/export/share/replicate/aggregate）、S0–S3 风险、派生资产独立版本/Owner/血缘、九项权限交集、目标范围不得扩大、跨空间可执行 Skill 默认不自动形成；第 4–21 轮为阶段性讨论节点（枚举多标「候选/不冻结」）：脱敏与重识别→Embedding 隐私→DP/匿名化→跨空间 ToolPolicy/RunGate→组合重识别→副本清理证明→CopyInventory→删除擦除验证（E0–E4 证据等级、不跨副本继承）→备份/法律保全（四轨契约）→Provider 证明合同审计（责任链不可一句话合并）→终端副本（不承诺未纳管终端全量发现）→恢复幂等（新 revision 不继承旧证明）→清理报告/客户通知/监管声明→多受众投影（事实内核不可变+字段五分类）→通知触发/时钟→多渠道投递（渠道接受≠接收≠已读）→跨租户/跨境/数据主体→外部未知补救联动。

**⚠️ digest 使用警告与不变量池**（新调研直接可用）：
- 五份 digest 里**竞品对比内容为零**（mem0/Letta/Zep 等全文 0 命中，已正则核验），且蒸馏时把议题原文各轮「参考资料」段**基本丢弃**——引用外部依据必须回 `.specs/enterprise-agent-memory-platform/0*.md` 的 `## 参考资料 / ### 本轮参考资料` 段（如 06:568、07:550、08:593）。
- 议题文档「已确认」普遍是**原则层/阶段性**（议题 1 状态即「已确认（原则层）」；议题 10 第 4–21 轮自标「候选、不冻结」；Schema/阈值/SLA/算法一律暂缓）。**引用时不得把设计机制当作已实现契约**，也不得当作行业已验证做法——后者正是竞品调研要回答的。
- **跨议题不变量池**（十条议题反复申明、彼此同构，可当竞品评估标尺）：①权限=多要素交集且派生不扩权（各议题分别为九/十/六方交集，`target_scope ⊆ effective_scope`）②状态分层禁止标签压缩（reference/retrieval/action 三态；接收≠发布≠可检索≠可执行；拒绝单一置信度/单一脱敏分）③撤回 fail-closed 且「失效快、恢复迟」，外部未知不得伪装 verified，禁 `catch→warn→返回成功` ④自动化永不自批：Published≠Actionable、知识文本≠权限、模型不能自我确认归因、外部系统不得写权威关系 ⑤不可变+追加+版本化贯穿证据/血缘/反馈/审计 ⑥不用单一总分，安全/权限指标是不可被业务成功率抵消的硬护栏 ⑦Session/Run≠工作真源、召回内容与 MCP resource 视为不可信数据、Manifest≠权限本身。

---

## 4. 既有竞品调研覆盖矩阵

### 4.1 `.specs/memory-platform-comparison-v2.md`（2026-09-01，源码/文档扫描版）
**调研对象**：Agent OS（= intent-os-platform，产品化平台）、MindMemOS（算法引擎）；TencentDB 作现状基线。**11 能力域**逐一「三方怎么做+差距」：①关系模型与执行编译 ②记忆空间与写入语义 ③检索/召回 ④记忆闭环（巩固/反馈/状态机）⑤资产演进（Playbook/Scene/Skill）⑥审批/安全/授权 ⑦观测监控 ⑧任务/计划/自动化 ⑨评测体系 ⑩MindMemOS 自认未做=机会窗口 ⑪扫描边角发现 12 条。
**已采纳并落地/排期**（结论→去向）：Brain 域+写入四档（→3.1 落地）、Agent 挂载空间、模型不得自选空间、Retention 层、确定性 tie-break、archive 召回排除、Dreaming 7 类、Feedback、质量 8 态、Playbook、ExecutableScene、skill 版本链、写前脱敏、召回护栏、RBAC 审批门、审批五档服务端解析、评测指标 recall@k/MRR/NDCG、A3 resolveExecutionBundle、A1 多对多（列 P1）。
**已识别空白/机会**（当时结论，仍有效）：Playbook 独立实体**双方都没做**（领先窗口）；MindMemOS 自认未做：Lite mode / Skills 治理 / **File system memory（本方已有 Wiki/CodeGraph=直接领先）** / 更多 agent 集成；MM skill 演进「算法对、工程半成品」（锁/校验/原子性缺失，别连坑抄）。

### 4.2 `.specs/intent-os-mindmemos-adoption.md`（2026-09-02，第二轮「读全部」+ 2026-09 复核更正）
**清单结构**：A1–A21 算法、R1–R10 实体关系、F1–F25 功能、五处实体关系断点（代码确证）、四层落地顺序（骨架→沉淀链→算法→功能）。
**已被复核「否决/降级」的结论**（新调研禁止复活这些说法）：
- ❌ A3 `priority=0.5·relevance+0.25·overlap+0.15·recency−0.10·cost`、半衰期 30d、MMR λ=0.70 —— MM 全仓零命中，**不是 MindMemOS 的做法**；MM 检索侧只有 rerank+top_k 截断。
- ❌ A12 BM25 `k1=1.5/b=0.75` —— stats 参数 13 个调用点无一传入，永远走 log_tf 兜底。
- ❌ A10 TemporalEntity 双时间 —— `validate_to` 无写入点，只有左边界。
- ❌ A11 skill 状态机 published 链路 —— `published_head` 恒 None，插件硬编码 `base_version_id:""`。
- ❌ A13 schema learning —— 零生产调用方（仅测试可达）。
- ⚠️ A7/A8 dreaming/feedback 机制存在但有**真实丢数据风险**（失败仍标 done、异常吞）与**跨用户越权聚类**（簇扩展只按 project_id）。
- ⚠️ A6 provider 契约 —— RoutingMemoryProvider 构建失败静默回退 internal（合规级问题）。
**复核后仍成立/新增采纳**：A1 RRF(K=60)、A2 有界生命周期加分（刻意小权重）、A4 recall@k/nDCG/MRR+harness、A5 embedding 缓存按 text_hash、A9 图扩展 hop 衰减；新增：Jaccard 近重复折叠（rerank 前）、superseded_by 自引用、召回可见性单路径、**门禁即治理**（AO 元结论）、intent-os 的 A14 Brain2.0 文件态记忆 / A15 case_library+critique_buffer / A16 temporal_edges / MM 的 A17 add_recall 写时召回 / A18 RAG 分块参数（6000B/600 重叠）/ A19 renforcement_count / A20 intent-protocol 内核隔离 / A21 A2A 对等互操作。
**铁律**：先建 recall@k golden set 评测基线，再动任何召回算法。

### 4.3 仅「点级引用」从未深调研的记忆产品（新调研的空白区）
- **Zep**：capabilities-ui 引其 Context Types / graph.search / Fact Invalidation / Governance / Debug mode（来源标注「本会话抓取」，无留存文档）。
- **Letta**：Memory blocks / Archival / Shared / Evals / HITL（同上，会话级引用）。
- **Mem0**：`add` API、记忆列表、身份轴（CONTEXT「身份派」一句带过）。
- **Claude Code / Cursor / Continue**（CONTEXT「文件系统派」：repo=边界、提交=共享/local=个人）；**LangMem**（身份派并列提及）。
- **MemOS（memos 系）**：**本仓库零覆盖**（注意与已调研的 MindMemOS 是两个不同项目，勿混淆结论）。

### 4.4 议题 01–10（设计层）的外部资料形态——「标准密集、产品稀疏」
经对 `.specs/enterprise-agent-memory-platform/0*.md` 全部 URL 引用与 `_facts/*-digest.md` 的核验：
- **记忆产品级调研几乎为零**：议题 2 来源仅链接 mem0/Graphiti(getzep)/Letta 三仓库 + 本方 README；议题 4 加 Pinecone 语义搜索、LlamaIndex Query Engine；议题 5 加 Notion AI、Atlassian Rovo、GitHub Copilot 仓库指令、Google SRE Postmortem。**均为「来源清单」式链接或阶段 2 机制表，未对各产品做过源码/参数级核验**（与 §4.1/4.2 对 AO/MM 的深扫完全不对称）。
- 议题 1 各深化「阶段 2 官方同类产品/既有模式调研」实际采用：Git merge/rerere（合并绑基线、复用历史解法）、GitHub Merge Queue / Protected Branches（组合验证、门禁与写权分离）、K8s Server-Side Apply / resourceVersion（多 applier 字段冲突、存储版本与业务 revision 分离）、Jira Linked Work Items（关系类型不压缩）、Event Sourcing（追加事件）；每条附「对本平台启发+局限」，且明言 404 页面不作依据。
- 议题 6–10 密集引用**标准与合规规范**：W3C PROV-O（血缘，且明确「因果边≠provenance 边」）、SLSA、CDEvents/CloudEvents、OpenTelemetry signals、NIST（800-88 介质擦除、AI RMF「不授权用风险分数确认根因」）、GDPR/eur-lex、ICO、OpenFGA/OPA、SPIFFE（工作负载身份）、MCP/A2A（只作为**接入适配协议**，明确不替代平台对象与权限）、BEIR/MTEB（检索基准）、dowhy（因果推断）、OCI 镜像规范（内容寻址）。
- **对调研的含义**：设计的治理机制（撤回证明/擦除验证/DP/组合重识别）已对标到**标准与工程模式层**，但**从未对标到「任何真实记忆产品怎么实现」**——竞品调研在这里最有增量。
- 议题原文「参考资料」段还引用过（digest 未保留，需回原文，锚点如 06:568、07:550、08:593）：OpenLineage、Microsoft Purview、Databricks Unity Catalog 与 Clean Rooms、Backstage Catalog Descriptor、CRDT、HTTP Cache-Control、**NIST SP 800 系列密集引用**（实测计数：800-53×39、800-61×24、800-88×15 介质擦除、800-57×17、800-188×7〔NIST 无此编号，疑为源文档对 800-88 系列的误写，引用前核原文〕、800-226×6、800-207 零信任、800-63、800-122、800-162 等）、GDPR/eur-lex、ICO Anonymisation、Five Safes/ONS disclosure control、CIS Controls v8、Amazon S3 Object Lock / AWS Backup Vault Lock（不可变保留）、Dwork 等 DP 论文、Kubernetes RBAC、GitHub Actions/Secret Scanning、Nielsen Norman Progressive Disclosure 与 Material Design Progress Indicators（均附「不替代权限治理/不等于业务完成」限定）。**若竞品把这些标准做成了产品能力（Purview 式血缘目录、Clean Rooms 式跨租户计算、S3 Object Lock 式不可变保留），才构成增量信息。**
- **外部名在议题层的三种身份（终版归纳，来自 digest 全文穷举核验）**：①**已采纳的抽象原则**——MCP/A2A 须显式 Protocol Adapter 接入且映射五值分档（议题1 深20）、MCP resource/工具返回不可作可信控制指令（议题4 深5）、DP/加密/水印/DLP/MDM/DRM/终端擦除「互不替代且各不证明匿名」（议题7 C10、议题10 第4/5/6/14 轮）、semver≠runtime 兼容（议题5 深15）、Git 类写操作不自动获权（议题6）；②**明确否决**——第一版不建完整图数据库（议题6/7）、不建通用概率/事件溯源平台（议题2 深6）、不建全局「已匿名」或「脱敏通过」分数、DP 不作平台总开关（议题10 第4/6轮）、不采用单条记忆胜出/最后写入者裁决（议题4）；③**待调研选型（设计主动留白，竞品调研可填充但须标「我方未决」）**——向量/关键词/图/缓存/索引引擎与「Registry 品牌选型」、RRF vs 学习排序 vs 规则排序、CDC/Outbox/消息总线拓扑、OpenAI Responses API 等具体 Harness 接入、PDP/PEP 与 Tool Policy 策略语言、IAM/KMS/HSM/WORM 实现、DLP/DRM/CASB/水印、k-anonymity 与隐私预算参数。
- ⚠️ **grep 假阳性警告**（复核时勿当竞品）：`Cursor`＝幂等游标、`S3`＝风险层 S0–S3、`opa/dify/CEL`＝adapter/modify/cancelled 子串、`Provider`＝泛化再处理者、`Harness`＝内部对象词；digest 文件内 URL 数为 0，所有「来源」是路径+行号锚点式引用。
- **历史调研指针**（更早批次，结论多被 comparison-v2 之后的复核覆盖，读时带日期意识）：`.specs/final-v1/PLATFORM-POSITIONING.md`（定位决策：可被外部 Harness 接入的 Memory Platform + 平台自有治理 Agent，非二选一）、`.specs/redesign-v2/01-REFERENCE-ARCHITECTURES.md`（intent-os/MindMemOS/my-tencentDB 三者参考架构采纳矩阵，含「AO 文档/代码冲突不用宣传图作证据」的口径）。

### 4.5 新调研「禁止重复/禁止复活」清单（合并 4.1–4.4 的最终裁决）
已裁决，不再讨论（除非带新证据）：AO/MM 的 11 域能力差距（→backlog 47 条）；A3 MMR/token-budget 公式、BM25 参数、双时间、skill 全状态机、schema learning——**均被源码级证伪**；「不缺实体缺四块」总判断；Playbook 独立实体=行业空白（AO/MM 均无）；MCP/A2A 是传输适配非平台对象；四轴隔离无产品全覆盖（截至 2026-08 的点级判断，允许用新产品证据推翻）。

---

## 5. refs/ 已有材料清单

| 文件 | 是什么 | 关键内容与结论 | 未覆盖 |
|---|---|---|---|
| `.specs/refs/intent-os-platform.md`（1,946 行/205KB） | 对 **intent-os-platform（Agent OS，Rust）**的源码+1012 份文档静态分析报告；每条结论标【代码已落地/文档声称/两者冲突】 | §10 记忆与知识 15 小节最深：双写入路径（α 工具面 memory_commit / β MemoryHookBus 11+1 事件）、三层存储（Brain2.0 文件态+15 表 pgvector+embedding_cache）、RRF(k=60)+有界生命周期加分、superseded_by 取代=关系非状态、MemoryProvider port 5 后端 7 能力位六维隔离、🔴§10.12 α/β/γ 三个互不重叠读写世界致同一条记忆可见性不一致而单测全绿、🔴§10.10 其内置 TencentDB 适配器 wire 契约已失效（/v1/messages、tdai_memory_search 终点不存在，store() 把六维隔离降为一维）。元结论（§15.3）：执行内核工程水准远高于文档治理；唯一生效治理=可执行门禁；接口声明与实现极易脱钩；召回判定必须收敛单函数。§16 借鉴 18 项 / §17 不借鉴 22 项 | UI/页面、自动抽取 prompt、冲突消解 NLI、多级审批共享 ACL、计费、中文 tokenizer |
| `.specs/refs/mindmemos.md`（1,003 行/110KB） | 对开源竞品 **MindMemOS（Python/MIT，checkout c1befcb）**的源码通读报告 | 定位「记忆操作层」：Entity-Property-Timeline 建模、9 类记忆 HTTP API、git-like Skill 版本库、Qdrant 11 collection+Neo4j+Kafka；成熟度分层表（写入/检索/dreaming 工程级，skill 6 态只写 2 态，token-budget MMR 不存在，schema 自演化未落地）；隔离=扁平 project_id 判「不值得学」；🔴dreaming 失败语义净丢记忆、🔴聚类扩展跨用户越权；§6.3 逐条证伪本仓库旧评估（4+ 条与代码不符，「公式级内容不可作实现依据」）；借鉴 Top3：近似重复折叠、两阶段 dreaming 工程护栏、写入预算分层+保头保尾压缩 | UI/交互、权限共享（其本身无）、L0-L3 分层先例、巩固调度、评测口径（README 指标无定义） |
| `.specs/refs/page-parity-audit.md`（383 行/40KB） | **自有项目**新旧版（my-tencentDB-Agent-memory → TencentDB-Agent-Memory）页面/功能机械对账，**非竞品** | 后端零删除（MemoryCore 路由 173→254、Panel 67→82、meta 白名单 55→105）；丢失全在前端 82 路径；确定丢失 3 项（AnalyticsPage 整页误删、默认 Agent 模板 UI、L1 刷新按钮）；实测缺陷（/admin/users 缺 AdminOnlyGuard、3 处重复路由、23 死模块、24 i18n 缺键）；附复现命令 | — |

---

## 6. 给新竞品调研的增量问题清单

（基于 §3 现状 + §4 覆盖矩阵的 gap；优先级按「设计已定但未获外部佐证」>「已否决结论需要真实现代」>「空白域」。提问与取证时以 §3.11 末「跨议题不变量池」为评估标尺；证据标准沿用 refs 报告的教训——**每条结论标【源码已落地/文档声称/营销宣传】三档，凡公式/参数必须给出可运行出处**。禁止重复 §4.1/4.2/4.5 已闭环的结论，除非要求**新证据**。）

1. **四轴隔离验证**：CONTEXT 断言「市面无产品同时覆盖 team×project×user×harness 四轴」——在 mem0（org/project/user/agent/run 键空间）、MemOS（memcube？）、Zep/Letta 企业版逐一核验其隔离键的真实实现（API 参数?服务端强制?），确认或推翻该断言；特别注意**harness/执行器维度是否被任何产品作为一等轴**。
2. **写入策略/持久 vs 总结分界的行业形态**：write_policy 四档（automatic/review_required/explicit_only/deny）与「模型不得自选空间、服务端裁决」在竞品中是否有对应物（mem0 的 manual/auto 双模式？Letta 的 agent-controlled vs user-controlled memory？）——找**可执行证据**（API schema、源码），不接受宣传页措辞。
3. **Retention 层的真实业界参数**：被证伪的 A3 公式留下空洞——竞品实际如何做 token 预算选材（预算计算、中文 tokenizer 成本、MMR/近重复折叠在 rerank 前后位置、只挑不改写原则）？各家给出可跑配置项名。
4. **离线巩固的生产者闭环**：AO 遗忘闭环零调用方、MM dreaming 有丢数据+越权两大缺陷（refs 两份报告实证）——竞品（含 mem0 平台、MemOS、Zep 的 memory update/invalidation）的巩固/遗忘是否有**默认开启的调度器+失败语义+范围守卫**？逐家找失败路径处理证据。
5. **冲突消解与取代语义**：superseded_by 自引用（B12）已采纳但无外部先例佐证——竞品的冲突检测（NLI?LLM?规则?）、「旧事实失效不删除」的 UI/API 表达（Zep Fact Invalidation 仅点级引用，需深挖成事实）。
6. **血缘与撤回证明（设计独有域）**：CopyInventory/PropagationStop/verified≠requested 这套在竞品中是否存在任何对应物（GDPR 删除工单、级联删除、embedding/index 副本清单）？若全行业空白，这正是企业叙事差异化证据，需拿各家「delete 到底删了什么」的文档/源码证据。
7. **Purpose 与执行门禁**：设计要求的 Purpose+RunGate+ToolPolicy 三层（ActionProposal 默认 blocked）——竞品除 AO 审批五档外，谁有「读取/复用/导出/执行分别授权」的实现（尤其 agent 执行动作前对记忆用途的重检）？
8. **跨 Harness 工作现场连续性**（议题1）：竞品如何取得/回放 Codex/Claude Code/DSH 会话证据（Connector 协议、幂等 cursor、gap 记录）？有无第二家做「交接包/最小上下文投影」？——验证设计的 ContextManifest 形态是否领先。
9. **知识化管道**（议题5）：candidate→ChangeSet→Review→Publish→反馈回缩（Update/Narrow/Retire）在竞品中的最近似物（Zep knowledge graph 发布流?Letta shared blocks 审批?企业知识库产品?）；重点问**审核对象是否版本化、发布失败是否可回滚**。
10. **多受众报告与投递证据**（议题10）：清理/合规报告按内/客/监管分层投影+delivery attempt+supersedes/revokes——竞品与合规工具（Vanta 式）有没有可借鉴的既成形态？或确认空白。
11. **评测对标表**：各竞品**自己宣称**的 LoCoMo/PersonaMem/MemoryAgentBench 分数逐家收集并核口径（B 组已定「先建基线」；需要一张各家指标定义对照表，标注哪些无定义如 MM）；scope leakage 指标谁做。
12. **UI 形态对账**：竞品控制台（mem0 dashboard、Zep、Letta、Langfuse 类）的「记忆对象管理面」截图/文档级事实：先选空间再列表？召回解释（score 组成）给不给用户看？纠正/遗忘入口形态？——capabilities-ui 的 5 tab 设计只有 Zep/Letta 点级引用，需成体系证据。
13. **TencentDB-Agent-Memory 定位澄清**：若调研目标是本项目的公开版（若已开源），需回答公开版本与 `.specs/memory-platform-changes.md` 声称的内部实现（21 模块、/v3/memory 23 端点、L0-L3）之间的差距清单；若未公开，则以其为「我方」基线不做竞品处理——先确认再动手。
14. **Provider 可插拔与静默回退**：C12「禁止静默回退」来自 AO 教训——竞品的多后端（qdrant/pgvector/neo4j/tcvdb）路由是否有能力位协商+契约测试+失败显式化？（AO 的 7 能力位手写无契约测试是反面教材；正面样本待找。）
15. **中文场景工程参数**：AO 4chars/token 英文硬编码低估 3–4 倍（refs 报告实证）——各家 tokenizer/token 预算/分块参数对中文的真实处理（分块默认值、embedding 模型中文表现、CJK 分词）；这是我们部署环境硬需求且两份竞品报告都没覆盖。

---

## 附：文档谱系速览（谁是谁的事实源）

- 权威议题层：`.specs/enterprise-agent-memory-platform/`（01–10 + README 第 76 轮路线图）→ 蒸馏进 `_facts/*-digest.md` → 汇入设计层。
- 设计层（2026-09-21，本目录）：`CHANGE/REQUIREMENT/DESIGN/CONCEPTS/ARCHITECTURE/UI-DESIGN/TASK/VALIDATION` + `PRODUCT-DESIGN.html`（§2 清单）。
- 内核实现/路线图层（2026-08~09）：`memory-platform-{implementation,changes,backlog,capabilities-ui}.md`、`design-memory-{domain-write-model,isolation-m1}.md`、`memory-api-ui-plan.md`、`CONTEXT.md`、`docs/adr/memory-platform-backlog.md`（旧拷贝）。
- 竞品调研层：`memory-platform-comparison-v2.md` → `intent-os-mindmemos-adoption.md` → `refs/{intent-os-platform,mindmemos}.md`（深度复核，含证伪）+ `refs/page-parity-audit.md`（自有对账）→ 汇总 `.specs/redesign/08-SCAN-FINDINGS.md`。
- 前端重构计划（未落地）：`tasks/plan.md` + `tasks/todo.md`（Agent Memory OS UI V2）。
