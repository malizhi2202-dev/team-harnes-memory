# D-5 governance-check — 权限在线可验证：判定器、解释与影响面反查

审核: 待审（第1轮）

- 事实依据：`GAPS.md` §1.6、§2 行 14–16；backlog C4/C5（P0 未落地）
- 命题维度：权限与治理（⑥）联动 按需加载（服务端解析=「按权限加载」的执行体）
- 边界：不改原型；不写鉴权代码（本仓无服务）；矩阵/角色模型不动

## ① 头脑风暴

我们权限面「文档最强、运行时最弱」：8 角色×16 动作矩阵三处一致（§7/§5.0/p12）是**静态承诺**，外部扫描证明竞品全都把授权推给调用方——我们要做的增量不是新策略模型，而是**把已有模型变成可当场验证/可解释的东西**：

1. **判定器缺位**：没有任何交互/端点能回答「用户 X（经 harness Y、在 Run Z）此刻能不能读这条记忆？为什么？」——这正是我们项目本体「按权限加载」的运行时证明。backlog C4/C5 已在，但没有界面与契约形状。
2. **判定序不显**：TencentDB 把判定顺序写进代码头注（资源→owner→成员→visibility→角色默认→ACL→deny）；我们有四职责分离但没有**可展示的判定序**（fail-closed 应显示「卡在哪一步」）。
3. **影响面反查缺半边**：§3.5 有 code-impact（代码→影响面），没有「这条记忆/这个空间被谁在用」（资源→主体）与「这个主体能触达什么」（主体→资源集）两个方向的清单化。
4. **破坏性操作的 API 形状无兜底条款**：mem0 用请求形状做安全（filters 必填、无 filter 的 delete_all 直接 400、清全库须四类型通配）；我们 §3.5 命令清单是语义清单，没有「缺哪个参数就拒绝」的形状红线清单。

## ② 竞品分析

| 系统 | 事实 | 来源 |
|---|---|---|
| TencentDB（上游） | ACL：主体 user/team_role/agent × 动作 read/write/delete/assign/share/use × effect allow/deny（**一期实际 allow-only**）；判定序见代码头注；`acl/check(read)` 在线验证端点（召回前逐条过滤实用于 `tdai-l1-recall-injector.ts`）；403=归属一致性失败、404=不存在或不属于当前上下文（不暴露存在性）。**负例**：数据面删除/chat-memory clear 内核不查用户级权限（靠 Panel 转发前校验）、KS 内网信任、Proxy 三个 ops 实际无鉴权且注释谎称有 | `src/metadata/service/permission-checker.ts:1-12,38-43,154-159`；`v3-api-memorycore-doc.md:44-69,107,621,959-964`；`v3-api-memoryproxy-doc.md:32-40` |
| mem0 | 平台角色仅 READER/OWNER 两档；org/project 空间级而非条目级；**文档自认 filters 是筛选不是授权、同 key 可对任意 user_id 检索、RLS 须客户自建**；API key=租户选择器（一 key 一 project）；per-user key 继承创建者 scope | docs.mem0.ai organizations-projects.md；entity-scoped-memory.md；supabase cookbook |
| Letta | Admin/Editor/Analyst 三档 + 「agent 默认私有、共享是显式动作、分享 agent≠分享会话、工具审批与 agent 可见性分离」四句正交表述；`blocks.agents.list(block_id)` 反查「谁在用这块记忆」；`read_only` 整块不可拆主体；并发契约文档化（memory_rethink=last-writer-wins 反模式） | docs.letta.com teams/permissions；v1 shared-memory |
| Zep | RBAC（人/仪表盘）与 ABAC（挂 API key 与 UserGroup，allow/deny、off/report_only/enforce 三态、deny 优先）**分两套主体**；治理全部 Enterprise 付费 | help.getzep.com policy-based-access-control.md |
| Memos | authorship⊥visibility（作者身份与可见性两轴正交的清晰表述）；实例 admin 有审计权≠群成员身份（治理位与数据位分离） | usememos.com docs spaces v0.31.0 |
| Memobase/LangMem | 治理≈无：一个 access token / 权限完全外置到自写 auth handler（官方示例只示范 owner 单字段过滤） | 各来源 |

判决：条目级 ACL 在线验证 **TencentDB 一家有实现**（且其自身治理漏洞恰恰证明「有验证端点≠有治理」——端点之外的删除路径绕过它）；解释性、影响面反查、形状红线全部七家皆无体系化。**我们若做「判定+解释+反查+形状」四件套，即为此维度最强形态。**

## ③ 结论与建议

1. **权限判定器进 §3.5**（语义级，不设计实现）：`access/check(主体, 资源, 动作, 上下文)` 返回 allow/deny + **命中的判定步**（对应我们四职责序：空间成员→角色默认→条目 ACL→脱敏位→用途/风险位→fail-closed）。上下文含 harness/Run，回答「按权限加载」的当场证明。
2. **p12 屏补「验证器」卡**：给定 演示用户×演示记忆×动作 → 显示判定轨迹（前端演示数据即可，符合 CONTRACT 单数据集红线）；与既有矩阵视图互补：矩阵答「规则」，验证器答「个案」。
3. **§3.5 补「破坏性操作形状红线」清单**：批量删除必须带显式范围 filter（缺→400）、清空间级操作必须逐类显式声明（对齐 mem0 四通配做法+我们 9 步协议）、条件写 precondition（摘要不符→失败，对齐 Letta contentSha256）、只读/不可变位不可被后续流程改写。
4. **影响面反查双清单**：资源→主体（谁在用这条记忆/这个空间）与主体→资源集（他能触达什么），后者同时是**权限矩阵的可测试定义**（golden-set 授权用例输入，接 backlog D6 金集优先项）。
5. 词表统一：TencentDB「assign/share/use」三动作与我们矩阵动作的映射登记进 D 文件附录待 design 对表（不擅改矩阵）。

**建议改动清单（列表级）**
- `§3.5`：access/check + 两条反查端点语义 + 形状红线清单（约 6-8 行）。
- `_build/` p12 段：验证器卡（演示）。
- `1-requirement`：US 候选「任何一次加载/拒绝都能当场给出判定轨迹」；AC 用 fail-closed 反例措辞。
- backlog 映射：本议题是 C4/C5（P0）的界面/契约承接件；与 D6 金集互依赖登记 ROADMAP。
- 不做：真实 RBAC 实现、策略引擎选型（A-architect 域）。

## ④ GD 议题门（R16.7 法定减配 3 人）

待投票：高级产品经理（个案验证器是否管理员真需求）/ 前端架构师（判定轨迹展示与演示数据可行性、端点形状稳定性）/ 领域专家（四件套与议题 03/09 已确认原则的一致性、与 TencentDB 漏洞清单的对照是否成立）。

## ⑤ 人审门

未开启。GD 票齐后随总清单提交用户（晋级/留档）。
