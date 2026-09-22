# D-2 lineage-screen — 血缘专属屏与「记忆↔源消息」反查契约

审核: 待审（第1轮）

- 事实依据：`GAPS.md` §1.5、§2 行 5–7
- 命题维度：血缘（联动：撤回证据、权限投影）
- 边界：不改原型；不裁决 §8 口径冲突

## ① 头脑风暴

血缘是我们十议题里模型最完整的维度之一（13 类关系、追加事件、影响分析、valid_time），但存在一个**可证伪的内部缺陷**和两个缺口：

1. **有入口、无屏**（本会话核验）：`_build/CONTRACT.md` §3 侧栏 19 条唯一真源路由里有 `#/governance/lineage`，而 13 屏标题树（p1–p13）**没有任何一屏承载它**——血缘只散落在 p5 运行详情与 p7 抽屉的片段里。用户视角：导航承诺了一个不存在的界面。§5.14 有血缘操作手册（文字），无对应可视化屏。
2. **正向有、反查弱**：我们的端点清单（§3.5）里有 code-impact（代码→影响面），但没有成文「给我一条记忆，回它全部来源消息/来源 Revision/生成过程」的反查契约；也没有「这条记忆被注入进过哪次运行」的消费侧回流记录。
3. **血缘承载权限未落地**：议题 07 的 Visibility Context 前置过滤在原则层已有，但「派生记忆的有效权限 = 其来源集合并集（或交集）」的算法表达没有——这正是 Zep 把血缘变成权限载体的做法。
4. 差异化机会（来自外部扫描的一致结论）：**mem0 与 Letta 都没有 memory→source-message 的一等外键**（mem0 只有 `history[].input` 旁路、Letta 只到 commit 级且 compaction 明说丢出处）。TencentDB 上游已做成 `source_message_ids[]` + generation-log——它是七家里唯一做全的，且**它就是我们 fork 的上游**，接口形态现成可参照。

## ② 竞品分析

| 系统 | 血缘做到哪 | 来源 |
|---|---|---|
| TencentDB（上游） | L1 记录带 `source_message_ids[]` 回指 L0 消息 + session/task/agent 归属；`MemoryGenerationLog`：layer/status/五归属/prompt{id,version,source,**sha256**}/anchor/input_refs/output_refs/model/latency/error；`MemoryGenerationRef`（`l1:<memory_id>`）支持 **memory_id+layer 反查生成日志**端点；注入行标 `[from <agent名>]`；资产 source_ref/content_ref。**缺**：「哪次会话被注入过」回流账 | `src/core/record/l1-writer.ts:64-106`；`src/core/memory-generation-log/types.ts:7-87`；`v3-api-memorycore-doc.md:739-763` |
| Zep | 「Everything traces back to episodes」：派生边经 `MENTIONS` 回指；云版 edge.episodes 列表、observation 关联证据 episodes；**metadata 沿血缘投影**：派生品的有效 metadata = 关联 episode metadata 去重并集，ABAC 按并集判定、过滤按逐 episode 判定（AND 语义单条全中） | graphiti README；help.getzep.com/episode-metadata-projection.md；memory-security.md |
| mem0 | `GET /v1/memories/{id}/history/`：每条含 **`input`=导致该变更的原始消息数组(role/content)** + old/new + event；合成记忆→源记忆有专门端点（`…/dream/memory/{id}/sources/`，Pro+）；但 search/get 返回体**无 message_ids 外键**，实用出处靠应用侧自填 metadata | docs.mem0.ai/api-reference/memory/history-memory.md；get-memory.md |
| Letta | git 即血缘：每次变更自动 commit、`versions.list(repo,{path})`/`versions.get(ref)` 历史快照、`precondition.contentSha256` 防覆盖；**只到 commit 级不到「哪句话」**，compaction 自陈「摘要可能省略原文与出处」 | docs.letta.com/agent-sdk/repositories/index.md；concepts/conversations/index.md |
| Memobase | 正向 Profile Delta（事件→改了哪些槽位）是一等；**反向无**（画像条目不回指来源）；且默认 blob 处理完即删＝源证据销毁默认开（负例） | docs.memobase.io features/event；readme「by default removes blobs once processed」 |
| LangMem | item 仅 value/key/namespace/时间戳，无任何来源引用结构 | docs.langchain.com/oss/python/langgraph/stores.md |

## ③ 结论与建议

1. **补屏方向确认**：`#/governance/lineage` 是导航对用户的既有承诺，补一屏（或把 §5.14 手册可视化为一屏）属「兑现承诺」而非新功能——这是六个子议题里最硬的一条（内部缺陷有 file:line 证据）。
2. **反查契约进接口清单**：建议 §3.5 增两条端点语义（不是新算法）：`memory → 来源链`（消息/Revision/生成事件，含 prompt 版本摘要）与 `memory → 消费记录`（被哪些 Run/ContextManifest 注入过）。后者同时服务撤回传播（D-6 的 revoke 判定）。
3. **`source_message_ids` 提为一等字段**：写入模型（MemoryEvent/MemoryRevision）登记源消息外键；外部七家五家没有＝真差异化，且上游 fork 已有落地形态可参照。
4. **血缘承载权限做留档观察**：Zep 的 metadata projection 与我们 Visibility Context 同向，但算法（并集 vs 逐条 AND）与我们 Q0–Q3 分级需要一次专门对表，本轮不建议直接抄。

**建议改动清单（列表级）**
- 原型 `_build/`：新增血缘屏（建议编号沿用导航路由 `#/governance/lineage`，屏标题「血缘与溯源」，含：正向来源树/反查消费列表/生成事件卡三段；演示数据沿用 CONTRACT 统一数据集）；§5.14 手册与屏互相引用。
- `PRODUCT-DESIGN §3.3`：MemoryEvent/MemoryRevision 字段行补 `source_message_ids`（一等外键）与 `generation_refs` 登记项。
- `§3.5 端点清单`：补 `lineage/trace(memory_id)` 与 `lineage/consumers(memory_id)` 两条语义（形状参照上游 generation-log 端点，命名用我们自己的词表）。
- `1-requirement`：US 候选「任意一条记忆可回答：从哪来、经哪些加工、去了哪里（被谁消费）」。
- 留档：Zep metadata projection ↔ Visibility Context 对表（转 platform 议题 07 补录轮候选，需人审背书才可动议题文档）。
- 不做：图谱可视化炫技屏（Our 关系是列表/树语义，非图形渲染需求）、引入图库依赖（CONTRACT 零外部依赖红线）。

## ④ GD 议题门（R16.7 法定减配 3 人）

待投票：高级产品经理（「有入口无屏」是否构成必须兑现的承诺缺陷）/ 前端架构师（新屏对 CONTRACT §3 三处导航一致性与 build/check 的连锁成本）/ 领域专家（反查契约与议题 07 已确认原则是否一致、有无重复造词）。

## ⑤ 人审门

未开启。GD 票齐后随总清单提交用户（晋级/留档）。
