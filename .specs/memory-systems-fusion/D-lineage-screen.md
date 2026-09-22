# D-2 lineage-screen — 血缘专属屏与「记忆↔源消息」反查契约

审核: 待审（第1轮）

- 事实依据：`GAPS.md` §1.5、§2 行 5–7
- 命题维度：血缘（联动：撤回证据、权限投影）
- 边界：不改原型；不裁决 §8 口径冲突

## ① 头脑风暴

血缘是我们十议题里模型最完整的维度之一（13 类关系、追加事件、影响分析、valid_time），但存在一个**可证伪的内部缺陷**和两个缺口：

1. **有入口、无屏（论据重心已按 GD-1 修正）**：`_build/CONTRACT.md` §3 侧栏 19 条唯一真源路由里有 `#/governance/lineage`，而 13 屏标题树（p1–p13）**没有任何一屏承载它**（GD-1 两票复核为真；`sec5.html:360` 手册还在指示用户去该屏追溯写入结果）。**同类悬挂共 5 条路由**（`#/library`、`#/scenario/…`、`#/ingestion/import`、`#/changes`、`#/settings` 亦无屏）——本议题不是「兑现唯一承诺」，而是**系统性「导航-屏覆盖缺口」中优先修血缘这一条**。优先理由（价值/依赖论据，非承诺论据）：①§5.0 已把血缘当**带 8 角色可见性的模块**（`sec5.html:69`）；②它是 D-4 消费列与 D-6 撤销传播的**共同数据源**；③p5/p7 内嵌片段承担不了角色可见性的演示。其余 4 条悬挂的处置在 ⑤ 单列（不得让「补一屏」被读成该类缺陷已结）。
2. **正向有、消费侧回流缺**（GD-1 前端架构师票勘误，已回源核对）：来源/影响反查 §3.5 **已有成文条目**——`get_lineage`（查询来源闭包与影响闭包，按 existence/metadata/summary/content/impact/export/admin 分层返回最小必要投影，PRODUCT-DESIGN.html:1251）与 `analyze_impact`（:1279）。初稿所写「code-impact」为引用错误（R6.1 违规：该名字在原型零命中，已勘误）。真缺口收敛为两条：`get_lineage` **没有消费闭包层**（这条记忆被哪个 Run/ContextManifest 注入过）；`source_message_ids` 这层**来源消息投影/索引**在字段行不可见（定性见③.3——它是 `captured_from` 关系的只读投影，不是新外键）。
3. **血缘承载权限未落地**：议题 07 的 Visibility Context 前置过滤在原则层已有，但「派生记忆的有效权限 = 其来源集合并集（或交集）」的算法表达没有——这正是 Zep 把血缘变成权限载体的做法。
4. 差异化机会（来自外部扫描的一致结论）：**mem0 与 Letta 都没有 memory→source-message 的出处回链表达**（mem0 只有 `history[].input` 旁路、Letta 只到 commit 级且 compaction 明说丢出处；措辞按 GD-1 领域专家统一，「一等外键」废用）。TencentDB 上游已做成 `source_message_ids[]` + generation-log——它是七家里唯一做全的，且**它就是我们 fork 的上游**，接口形态现成可参照。

## ② 竞品分析

| 系统 | 血缘做到哪 | 来源 |
|---|---|---|
| TencentDB（上游） | L1 记录带 `source_message_ids[]` 回指 L0 消息 + session/task/agent 归属；`MemoryGenerationLog`：layer/status/五归属/prompt{id,version,source,**sha256**}/anchor/input_refs/output_refs/model/latency/error；`MemoryGenerationRef`（`l1:<memory_id>`）支持 **memory_id+layer 反查生成日志**端点；注入行标 `[from <agent名>]`；资产 source_ref/content_ref。**缺**：「哪次会话被注入过」回流账 | `MemoryCore/src/core/record/l1-writer.ts:71-106`、`MemoryCore/src/core/memory-generation-log/types.ts:7-87`、`MemoryCore/v3-api-memorycore-doc.md:739-763`（均标上游 ref `v2.0.2-beta.1`，GD-1 取证口径③） |
| Zep | 「Everything traces back to episodes」：派生边经 `MENTIONS` 回指；云版 edge.episodes 列表、observation 关联证据 episodes；**metadata 沿血缘投影**：派生品的有效 metadata = 关联 episode metadata 去重并集，ABAC 按并集判定、过滤按逐 episode 判定（AND 语义单条全中） | graphiti README；help.getzep.com/episode-metadata-projection.md；memory-security.md |
| mem0 | `GET /v1/memories/{id}/history/`：每条含 **`input`=导致该变更的原始消息数组(role/content)** + old/new + event；合成记忆→源记忆有专门端点（`…/dream/memory/{id}/sources/`，Pro+）；但 search/get 返回体**无 message_ids 外键**，实用出处靠应用侧自填 metadata | docs.mem0.ai/api-reference/memory/history-memory.md；get-memory.md |
| Letta | git 即血缘：每次变更自动 commit、`versions.list(repo,{path})`/`versions.get(ref)` 历史快照、`precondition.contentSha256` 防覆盖；**只到 commit 级不到「哪句话」**，compaction 自陈「摘要可能省略原文与出处」 | docs.letta.com/agent-sdk/repositories/index.md；concepts/conversations/index.md |
| Memobase | 正向 Profile Delta（事件→改了哪些槽位）是一等；**反向无**（画像条目不回指来源）；且默认 blob 处理完即删＝源证据销毁默认开（负例） | docs.memobase.io features/event；readme「by default removes blobs once processed」 |
| LangMem | item 仅 value/key/namespace/时间戳，无任何来源引用结构 | docs.langchain.com/oss/python/langgraph/stores.md |

## ③ 结论与建议

1. **补屏方向确认（论据已换轴，GD-1 双票条件）**：不是「兑现导航承诺」，而是价值/依赖——血缘是 D-4 消费列与 D-6 撤销传播的共同数据源，且 §5.0 已把它当带角色可见性的模块；补一屏（或把 §5.14 手册可视化）属兑现该模块的运行时形态。**系统性判定一并登记**：同类「有入口无屏」共 5 条路由（见①.1），本议题只优先修血缘这条，其余 4 条的去留在 ⑤ 单列，防止「补一屏＝该类缺陷已结」的误读。
2. **不建第二套血缘查询，收敛为 `get_lineage` 增层**（语义级）：给 `get_lineage` 增「消费闭包」层（被哪些 Run/ContextManifest 注入过，沿用其按分层最小投影的既有语义），同时服务撤回传播（D-6 的 revoke 判定）；来源链（消息/Revision/生成事件含 prompt 摘要）以 `get_lineage` 为唯一真源，**不再新立并列 trace 端点**，避免下游在同一位置定义两套血缘查询。**权限红线进 AC**（GD-1 领域专家条件③＋高级产品经理条件②）：07 §5「血缘不得成为绕过 ACL 的反向索引」＋§7 侧信道——消费列表**必须分列三态：无消费记录／无权查看（隐藏占位，07 §5「隐藏节点与抽象占位」）／追溯失败**，「缺口、无权与超时不得显示为无影响」，否则血缘屏自己变成新的假绿来源。
3. **`source_message_ids` 定性修正（GD-1 领域专家条件①）**：它是 `captured_from` 类型化血缘关系的**只读投影/查询索引**，真源仍是携带 07 §3 必填字段（source_version/derivation_policy_version/redaction_policy_version/authorization_context/valid_time/recorded_at/status）的血缘关系记录——「一等外键」一词废弃（裸 id 数组承载不了任何必填字段，与 07 §2「关系不得简化为 source_id」相抵）。表达层价值不变：外部七家五家连这个投影都没有＝真差异化。
4. **血缘承载权限做留档观察**：Zep 的 metadata projection 与我们 Visibility Context 同向，但算法（并集 vs 逐条 AND）与我们 Q0–Q3 分级需要一次专门对表，本轮不建议直接抄。

**建议改动清单（列表级）**
- 原型 `_build/`：新增血缘屏（建议编号沿用导航路由 `#/governance/lineage`，屏标题「血缘与溯源」，含：正向来源树/反查消费列表/生成事件卡三段；演示数据沿用 CONTRACT 统一数据集）；§5.14 手册与屏互相引用。
- `PRODUCT-DESIGN §3.3`：MemoryEvent/MemoryRevision 字段行补「`source_message_ids`（= `captured_from` 关系的只读投影）」登记项；生成过程表达用 07 既有词「**追加式血缘事件**＋`derived_from`」，**不引入 `generation_refs` 新术语**（GD-1 领域专家条件②，已废止初稿造词）。
- `§3.5 端点清单`：`get_lineage` 增「消费闭包」一层的语义行（七层投影词表不动，+1 层）；**不新立端点名**——§3.5 每行都是候选契约（:1233 声明路径/方法/错误码/幂等键未固化），命名与形状留 2-design。
- `1-requirement`：US 候选「任意一条记忆可回答：从哪来、经哪些加工、去了哪里（被谁消费）」+ AC 三态分列（无消费记录/无权查看-隐藏占位/追溯失败，见③.2）；**AC 明确本屏只承接 `#/governance/lineage` 一条路由，其余 4 条同类悬挂不在本 change 范围**（GD-1 领域专家条件④：防「补一屏＝该类缺陷已结」误读）。
- 留档：Zep metadata projection ↔ Visibility Context 对表（转 platform 议题 07 补录轮候选，需人审背书才可动议题文档）。
- 不做：图谱可视化炫技屏（Our 关系是列表/树语义，非图形渲染需求）、引入图库依赖（CONTRACT 零外部依赖红线）。

## ④ GD 议题门（R16.7 法定减配 3 人）

三票（第 1 轮）：前端架构师 ✅／高级产品经理 ✅（条件：论据换轴+三态分列——已落 ①.1/③.2）／领域专家 ❌（四条件：`captured_from` 投影定性／废 `generation_refs`／权限 AC 三态／悬挂系统性——已落 ①.2/③.3/③.1/清单）。
状态：待领域专家按「只看这几行」复审转 ✅ 后，本议题 GD 通过、进 ⑤。

## ⑤ 人审门

未开启。GD 票齐后随总清单提交用户（晋级/留档）。
