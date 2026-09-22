# D-4 asset-telemetry — 知识资产发布后度量与到期治理

审核: 待审（第1轮 · GD 通过 3/3（GD-2 复审改判架构票），待 ⑤ 人审）

- 事实依据：`GAPS.md` §1.4/§1.5（已同步勘误）、§2 行 11–13；本会话 grep + GD-1 回源复核（`PRODUCT-DESIGN.html:1418/:3115/:3130/:3026/:3222/:3246`）
- 命题维度：知识化发布（④）联动 血缘（⑤）、撤回（⑧）
- 边界：不改原型；字段进模型须走 requirement/design，不在本清单定 schema

## ① 头脑风暴

我们的发布链路是七家里唯一带评审门禁的（ChangeSet→Review→Publish），但链路止步于「发布成功」：

1. **到期轴：原型与条文在用、字段行没登记**（GD-1 修正后判决；屏名勘误 GD-2 修正）：初稿「永久资产反而没有到期＝语义倒挂」**不成立，撤回**——p8 列表已有「复核到期」列（:3115 表头、:3130 演示值 `2026-10-10`），§5 已写「`review_due ≠ invalid`，但到期资产不得获得新的高风险执行资格」（:1418），p7 条目行有 `review_due` 徽标（:3026），**p8 段内另有 `review_due` 卡位（:3222）与状态口径（:3246「到期需复核≠内容错误」）**——初稿把 :3222/:3246 写成「p7/p10」系屏名错（行号无误；两行实测均在 p8「知识资产」段，段界 :3073–:3251，GD-2 复核实测，本会话独立复核一致）。改对后对本议题更有利：**p8 自己就在用 `review_due`，正是「漂移非缺能力」的直接证据**。**真缺口＝文档↔原型漂移**：§3.3 KnowledgeAsset 字段行（:1114-1116）未登记 `review_due`。
2. **使用遥测真没有**：「被谁用过、多久没用」（使用计数/最近使用时间，对应外部 `usage_count/last_used_at` 语义）全文件零命中（本会话 grep，GD-1 同判）——「发布→使用→复审」闭环断在使用段。
3. **没有冷启动口径**：新成员/新 Agent 进空间时默认装什么，我们只字未提（外部已有产品化形态，见②）。
4. 「发布=三件事」可借鉴性：TencentDB 把发布拆为 visibility+ACL grant+绑定装配，撤回=unbind——与我们模型（Publish 改 effective_scope）等价，但它的**绑定带 injection_mode（direct/summary/tool/reference）+priority**，比我们 p9 的「装配」粒度更细，值得对表。

## ② 竞品分析

| 系统 | 事实 | 来源 |
|---|---|---|
| TencentDB（上游） | Asset 统一登记含 `version/visibility/status/confidence/expires_at/usage_count/last_used_at`；`touch-usage` 回写；面板口径「点开一条资产，关心的不只是它写了什么，还有它从哪来、哪个版本、**分给了谁、最近是否被使用**」；发布=改 visibility+ACL grant+绑定 Agent（knowledge 固定 injection_mode=tool），撤回=unbind；**Agent 模板冷启动**：无模板→建 default-agent+导入预置 Skill，有模板→cloneTemplateAssets（幂等靠 name 唯一约束 42201 跳过）；绑定带 injection_mode∈direct/summary/tool/reference+priority | `src/metadata/types.ts:26,531-553,37`；`v3-api-memorycore-doc.md:917-949`；`panel-api-doc.md:1061-1128`；`README_CN.md:226`；`MemoryPanel/src/panel/http/routes/meta/proxy.ts:224-296,443-491` |
| Zep | Observations＝「跨多事实、有证据支撑」的稳定模式层（付费档）；社区摘要「增量更新+定期重建（重建先清旧）」——知识产物需要**版本化重建**而非只 patch | help.getzep.com/context-types.md；graphiti communities.md |
| Memobase | `context()` 是发布物 API：token 预算/主题过滤/时间窗/自定义模板的「画像即产品」出口；无到期概念——画像永远当前态（负例） | docs.memobase.io/features/context.md |
| Letta/Acontext | 发布载体=git 提交+`skills/<name>/SKILL.md` 随 attach 生效（**无评审、无供应链审查=负例**）；Memobase 新方向 Acontext：记忆蒸馏成 Markdown SKILL.md 跨框架复用（ZIP 导出/get_skill）——与我们 Skill/Template 形态正面重叠的趋势信号 | docs.letta.com shared-memory；Acontext README（2025-11-23 起） |
| mem0 | Custom Instructions/Schema 驱动 Export 等平台专有「出口配置」；无使用度量（Platform/OSS 均无 usage 字段——待确认项见 GAPS §3） | docs.mem0.ai platform/features/memory-export.md |
| Memos | 分享链接＝**TTL+可撤销+最小切片**：「到期」长在出口而非资产上——与我们现状同构（出口有 `expires_at/revoked_at` :1188、资产字段行没登记），D-6 交叉 | usememos.com/docs（v0.31.0）；PRODUCT-DESIGN :1028/:1188 |

## ③ 结论与建议

1. **修漂移**：§3.3 KnowledgeAsset 字段行补 `review_due` 登记，措辞与 :1418/:3246 既有用法逐字对齐（到期需复核≠内容错误）；到期治理语义**不新造**。
2. **补真缺口**：使用计数/最近使用两字段（对应 TencentDB `usage_count/last_used_at/touch-usage` 形态）；p8 随之**只加这两列**——到期列已存在，勿重复造。列数 9→11 布局风险照 GD-1 处理：沿用 `table.tbl.compact`/`.tbl-wrap`（`.app-body` 有 `overflow:auto`，1080px 可行），不需新样式。
3. **到期≠删除**：维持 :1418 既有口径（到期→退出高风险执行资格、转复审），本议题补登记与遥测，不动这条语义。
4. **冷启动模板**：方向候选（新成员/新 Agent 默认装配清单），但我们的版本=「一组 ChangeSet+审批记录」而非文件拷贝——保住外部没有的发布前门禁底线；仅留档对标，不与 SKILL.md 文件形态混同。
5. **趋势登记（不改）**：Acontext 式 SKILL.md 出口形态；Letta「attach 即执行」供应链缺位作我们门禁论据的反例。

**建议改动清单（列表级）**
- `PRODUCT-DESIGN §3.3`：KnowledgeAsset 行补 `review_due`（修漂移）＋「使用计数/最近使用」两字段候选。
- `_build/` p8 段：使用/最近使用**两列**＋行内「转复审」操作（演示数据；到期列不动）。**显示纪律（GD-1 高级产品经理条件②）**：本仓无服务，遥测未落地前两列一律显示「未接入」而非 `0`——`0` 与「发布后从未被用」不可区分＝假数据，踩 CONTRACT 演示数据标识红线（LESSONS L-003/L-007 点过的形态）。
- `1-requirement`：US 候选「知识资产可回答：被谁用、多久没用；到期复核链路与 `review_due` 字段登记对齐」；AC 含「到期→退出高风险执行资格但不改内容状态」既有语义回归（:1418）。
- ROADMAP 依赖登记：详情「最近被哪些 Run 消费」深链依赖 D-2 `get_lineage` 消费闭包先行。
- 不做：真实遥测采集（本仓无服务）、Agent 模板实现、SKILL.md 导出定稿、第二套到期机制。

## ④ GD 议题门（R16.7 法定减配 3 人）

三票（第 1 轮）：高级产品经理 ✅（附带条件：三字段收敛两字段+复用已有 `review_due`；遥测未落地显「未接入」——**均已落本文件**；票面原话「附带条件进 ⑤ 前落到文件即可，不必重开 ④」）／领域专家 ✅（改文：到期语义出处修正——初稿「9 态里已有的 stale/expired」误引**已改**：`expired` 是议题 10 清理候选态（`10-跨空间复用与脱敏.md:1084` retained/expired），与 §10.1 Knowledge 行态（stale/withdrawn/superseded）分属两层；补登记须同时声明 **`review_due`（复核到期）／`expires_at`（出口到期）／`expired`（清理候选态）三词分工**）／前端架构师 ❌→**GD-2 复审改判 ✅**（唯一条件「缺口改写为 §3.3↔原型漂移＋三列收敛两列」按其给词整段落盘 :13/:32/:38/:39/:42，复审并顺带交付屏名勘误 p10→p8——已修）。**本议题 GD 通过（3/3 ✅）**，待 ⑤。

## ⑤ 人审门

未开启。GD 票齐后随总清单提交用户（晋级/留档）。
