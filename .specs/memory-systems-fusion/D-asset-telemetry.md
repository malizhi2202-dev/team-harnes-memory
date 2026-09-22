# D-4 asset-telemetry — 知识资产发布后度量与到期治理

审核: 待审（第1轮）

- 事实依据：`GAPS.md` §1.4、§2 行 11–13；本会话 grep 核验（KnowledgeAsset 字段行无 usage/expires；`expires_at` 仅 Delegation :1028 与 ExportAsset :1188 出现）
- 命题维度：知识化发布（④）联动 血缘（⑤）、撤回（⑧）
- 边界：不改原型；字段进模型须走 requirement/design，不在本清单定 schema

## ① 头脑风暴

我们的发布链路是七家里唯一带评审门禁的（ChangeSet→Review→Publish），但链路止步于「发布成功」：

1. **发布后无人观测**：p8 知识资产屏有版本/置信状态/级联撤回影响，**没有「被谁用过、多久没用、什么时候该复审」**。五类资产各写各的，缺统一的「使用遥测」概念。
2. **没有到期轴**：ExportAsset 有 `expires_at/revoked_at`、Delegation 有 `expires_at`——**一次性出口都有 TTL，永久资产反而没有**，语义倒挂。
3. **没有冷启动口径**：新成员/新 Agent 进空间时默认装什么，我们只字未提（外部已有产品化形态，见②）。
4. 「发布=三件事」可借鉴性：TencentDB 把发布拆为 visibility+ACL grant+绑定装配，撤回=unbind——与我们模型（Publish 改 effective_scope）等价，但它的**绑定带 injection_mode（direct/summary/tool/reference）+priority**，比我们 p9 的「装配」粒度更细，值得对表。

## ② 竞品分析

| 系统 | 事实 | 来源 |
|---|---|---|
| TencentDB（上游） | Asset 统一登记含 `version/visibility/status/confidence/expires_at/usage_count/last_used_at`；`touch-usage` 回写；面板口径「点开一条资产，关心的不只是它写了什么，还有它从哪来、哪个版本、**分给了谁、最近是否被使用**」；发布=改 visibility+ACL grant+绑定 Agent（knowledge 固定 injection_mode=tool），撤回=unbind；**Agent 模板冷启动**：无模板→建 default-agent+导入预置 Skill，有模板→cloneTemplateAssets；绑定带 injection_mode∈direct/summary/tool/reference+priority | `src/metadata/types.ts:26,531-553,37`；`v3-api-memorycore-doc.md:917-949`；`panel-api-doc.md:1061-1128`；`README_CN.md:226`；`MemoryPanel/src/panel/http/routes/meta/proxy.ts:224-296,443-491` |
| Zep | Observations＝「跨多事实、有证据支撑」的稳定模式层（付费档），社区摘要「增量更新+定期重建（重建先清旧）」——知识产物需要**版本化重建**而非只 patch | help.getzep.com/context-types.md；graphiti communities.md |
| Memobase | `context()` 是发布物 API：带 token 预算/主题过滤/时间窗/自定义模板的「画像即产品」出口；`expires`? 无——画像永远当前态（负例：无到期概念） | docs.memobase.io/features/context.md |
| Letta/Acontext | 发布载体=git 仓库提交+`skills/<name>/SKILL.md` 随 attach 生效（**无评审、无供应链审查=负例**）；Memobase 团队新方向 Acontext：记忆蒸馏成 **Markdown SKILL.md 跨框架复用**（get_skill/list_skills/ZIP 导出）——与我们的 Skill/Template 资产形态正面重叠，是趋势信号 | docs.letta.com shared-memory；Acontext README（2025-11-23 起） |
| mem0 | Custom Instructions/Schema 驱动 Export 等平台专有「出口配置」，无使用度量（Platform/OSS 均无 usage 字段，待确认项已在 GAPS §3） | docs.mem0.ai platform/features/memory-export.md |
| Memos | 分享链接＝**TTL + 可撤销 + 最小切片**——「到期」在外部产品里长在*出口*上而非资产上，与我们现状同构（它的出口有 TTL、资产没有），D-6 交叉引用 | usememos.com/docs（v0.31.0） |

## ③ 结论与建议

1. **补三字段（模型登记，schema 定稿走 design）**：KnowledgeAsset 增「使用计数 / 最近使用时间 / 到期（或到期复审）时间」；与 ExportAsset 已有 expires 语义统一词汇（避免两套 TTL 词）。
2. **到期≠删除**：到期触发的动作是**退出候选集**（不再被装配/召回引用），不是物理删除——必须落进「到期→转复审候选」的治理位（对齐我们 9 态里已有的 stale/expired 语义，不新增态）。
3. **p8 屏增量**：列表加「使用/最近使用/到期」三列与排序；详情加「最近被哪些 Run 消费」链接（数据源=D-2 消费反查，两议题互为依赖，登记进 ROADMAP）。
4. **冷启动模板**：方向性采纳为候选（新成员/新 Agent 的空间默认装配清单），但我们的版本是「模板=一组 ChangeSet+审批记录」而非文件拷贝——保住「发布前门禁」这条外部没有的底线；仅留档对标 TencentDB 形态，不与 SKILL.md 文件形态混同。
5. **趋势登记（不改）**：Acontext 式 SKILL.md 跨框架发布＝我们 Skill 资产的出口格式候选；供应链审查缺位（Letta attach 即执行）作为反例进入我们「发布前门禁」论据。

**建议改动清单（列表级）**
- `PRODUCT-DESIGN §3.3`：KnowledgeAsset 行补 usage/last_used/expires 登记项；`§2.4` 状态机图注补「到期→复审候选」箭头（不新增态）。
- `_build/` p8 段：三列+到期徽标+「转复审」行内操作（演示数据）。
- `1-requirement`：US 候选「知识资产可回答：被谁用、多久没用、何时该复审/到期；到期不删除只退出候选」。
- ROADMAP 依赖登记：p8 消费列依赖 D-2 反查端点。
- 不做：真实遥测采集（本仓无服务）、Agent 模板实现、SKILL.md 导出格式定稿。

## ④ GD 议题门（R16.7 法定减配 3 人）

待投票：高级产品经理（遥测三列是否演示级最小可信集）/ 前端架构师（p8 列扩张对 1080px 布局与统一数据集的影响）/ 领域专家（到期语义与 9 态 stale/expired、ExportAsset expires 的词表统一）。

## ⑤ 人审门

未开启。GD 票齐后随总清单提交用户（晋级/留档）。
