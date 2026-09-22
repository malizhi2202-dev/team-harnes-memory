# D-6 redaction-evidence-hardening — 差异化确认：跨空间脱敏与清理证据硬化

审核: 待审（第1轮 · GD 通过 3/3 ✅，待 ⑤ 人审）

- 事实依据：`GAPS.md` §1.7、§1.8、§2 行 17–20；platform 议题 10（Q0–Q3/9 步/E0–E4/Copy Inventory）
- 命题维度：跨空间脱敏（⑦）+ 撤回清理证据（⑧）
- 边界：不改原型；不新增脱敏「功能」（我们已是全场唯一有模型者），只做**不变量硬化与证据语义补齐**

## ① 头脑风暴

本议题的结论方向与前五个相反：**不是补缺口，是护城河加固**。扫描给了强证据：七个外部产品记忆内容层脱敏全部缺位（mem0 只有自然语言「Exclude SSNs」提示+Direct Import 逐字旁路、Letta 只修凭据回显、Zep 只有按数据类拒读、TencentDB 上游只有日志/SQL 参数脱敏——其 fork 里新增 redaction.ts 的头注直陈「记忆内容没有脱敏，密码/PII 会原样沉淀进 L1 并随 recall 回注模型，构成泄密+注入双重风险」，LangMem/Memobase/Memos 未见）。同时撤回/清理证据面有四种外部语义我们未成文：

1. **按证据依赖计数删除**（Zep）：一条派生事实由多来源支撑时，删一个来源不删事实，**所有支撑消失才进入清理候选**——团队多人贡献场景的正确方向；两处限定（GD-1 领域专家）：①计数到零 **≠自动物理删除**，应＝进入清理候选并走 9 步第 5 步处置计划（10:333「撤回与物理删除分离，默认优先暂停/收窄/撤回/替代/归档」、10:1089「阻断完成不冒充删除完成」），初稿「所有支撑消失才删」单读会被实现成自动删，已改；②「撤销该来源造成的影响」只能以**追加式再评估事件**表达（07:13 追加式血缘事件、07:187 血缘声明与当前关系分离、07:13 历史版本与已完成 Run 不被静默改写），不得原地改写旧结论。
2. **重算即撤销**（TencentDB Proxy ops）：撤回共享后调用 `refresh-cache(clearBefore)` 强制重算注入快照，旧快照不再复用——我们有「Manifest 重算、旧 Run Snapshot 不重写」原则，缺「已注入快照的传播失效」操作语义（撤销不追及已发生的注入是合规空洞）。
3. **逐条机读清理证据**（TencentDB clear 响应）：逐条返回 `cleared/…/reason/retryable/attempts`，单条失败不整体报错——我们六态证据是给人看的卡，没有定义给机器读的逐条契约。
4. **证据销毁反例清单**（负面知识）：Memobase 默认处理完即删原始 blob（源证据默认销毁！）；TencentDB Redis session 不清、靠 1800s TTL 自然过期；Zep 删 episode 不撤销它造成的失效、build_communities 重建先清空旧社区；mem0 删除异步最终一致「删后数秒仍可见」+project 删除 irreversible 无回收站。→ 我们的「清理完成证据」必须显式排除这四类形态，并规定**原文留存策略默认不销毁**（对照 Memobase）。
5. 「只发派生层/raw 不出域」：TencentDB 分享只是 visibility+grant 无出域清洗（其文档自曝）、Memos 分享链接=最小切片+TTL+可撤销（我们的 Export 已有等价更强形态）→ 把「跨空间/跨组织复用的缺省发布形态=派生层」写成不变量，raw 出域为显式例外（走审批+脱敏+证据），而不是反过来。

## ② 竞品分析（来源）

| 点 | 事实 | 来源 |
|---|---|---|
| Zep 计数删除 | 删 episode 时边/节点仅当「无其他关联 episode」才删；user 实体节点永不删；已知残留：episode 曾让事实失效，删 episode 后失效态不回滚 | help.getzep.com/deleting-data-from-the-graph.md |
| Zep 排除优先于删除 | 「不想删就先改 episode metadata + 策略排除」（可回滚撤回） | memory-security.md；episode-metadata-projection.md |
| TencentDB 重算撤销 | `proxy/session/refresh-cache`（可带 clearBefore）重算缓存；**但该 ops 面实际无鉴权=必须反例**（硬化时要求带治理位） | v3-api-memoryproxy-doc.md:32-40,76-124 |
| TencentDB 逐条证据 | chat-memory/clear 逐条返回 cleared/l0_deleted/l1_deleted/profile_deleted/reason/retryable/attempts；instance/destroy 返回 cleaned{...} 明细；**但内核不查用户级删除权限=反例** | v3-api-memorycore-doc.md:615-652,1031-1041 |
| mem0 | delete 异步+最终一致告警；expiration_date=到期从检索隐藏而非删除（与 D-4 到期语义同构，可引为「隐藏≠删除」正例）；DELETE entities 级联异步；无 trash/restore | docs.mem0.ai delete-memories.md；core-concepts/memory-operations/delete.md；platform/faqs.md |
| Memobase | 默认 blob 处理完即删；「永久删除 profile 与 memories，先取消 pending job」；无删除留痕接口 | readme.md；api-reference/users/delete_user.md |
| Letta | 删除多为物理永久（archives passages「从数据库和向量存储永久移除」；repositories 删后无 restore 方法）；git 历史只覆盖 MemFS 仓 | v1 archives；agent-sdk/repositories |
| 脱敏横向 | 七家内容层脱敏全部缺位/未见（逐家否证与「未见≠无」声明见 GAPS §1.7/§3，此处不重复） | 汇总自各报告 |

## ③ 结论与建议

1. **差异化判定（本议题核心结论）**：跨空间脱敏上我们是唯一有完整模型者，七家外部零实现——**维持领先，不加新功能，只加不变量**。建议把 §2.6/§2.7 的既有原则升格为可核查条款：①原文不出域（默认）；②出域=派生层+脱敏版本入血缘（`source_redaction_version` 字段已在 §3.3，把它的「必须非空」写进不变量）；③隐藏≠删除（到期/撤回的检索退出与物理清理两态分离，引 mem0 expiration 正例）。
2. **清理证据三语义补录——其中「传播撤销」升为首要用户承诺**（GD-1 高级产品经理：「独家领先」只有变成用户可感知的承诺才成立）：进 §5.15 清理证据 [锚点 s5-16]/§5.16 报告中心 [s5-17] 手册与 §3.5 端点语义——初稿引「§5.12/§5.13」系错位（彼二节为分诊/变更与发布），GD-1 指出已勘误；不动 9 步协议编号。三条：依赖计数删除（对齐 Zep 并避开其**文档记载的既有行为**——删来源 episode 后失效不回滚——我们不照抄也不称对方 bug，GD-1 领域专家措辞修正）；**传播撤销（首要）**：撤回共享后新的运行不再注入它，此前已注入的快照**可列出、可撤、有逐条回执**（快照重算端点+强制治理位，反例=TencentDB 无鉴权 ops）；逐条机读响应——**停在语义级**（逐条返回状态＋单条失败不整体报错＋存在可重试位＋输出不得只有布尔值）：JSON 字段名与形状不进 PRODUCT-DESIGN，命名交 2-design（GD-1 约束，合 §3.5:1233 候选契约声明）。
3. **反证据销毁条款**：清理完成证据必须含「原文留存策略=保留/销毁(需审批+理由)」显式字段；禁止任何默认销毁（点名 Memobase 形态为不可接受）。
4. p10 屏登记（建议级）：证据卡加「逐条重试位(retryable/attempts)」与「撤销传播链」两行演示元素——细节交 2a-ui-design，本清单只到列表级。

**建议改动清单（列表级）**
- `§2.6/§2.7`：脱敏三不变量 + 反销毁条款（新文字，不动结构）。
- `§3.5/§5.12/§5.13`：重算撤销、依赖计数、逐条证据形状三条语义。
- `_build/` p10 段：两行演示元素（缓行，UI 门定）。
- `1-requirement`：**US 第一位＝首要用户承诺**（GD-1 高级产品经理条件）「撤回一条记忆后：新的运行不再注入它；此前已注入的快照可列出、可撤、有逐条回执；剩什么、谁批的处置计划全程可见」。
- 不做：新脱敏算法、KPI/SLA 数字（十议题留白纪律不破）。

## ④ GD 议题门（R16.7 法定减配 3 人）

三票（第 1 轮）：全部 ✅（高级产品经理条件「传播撤销提第一位＋US 首位」＝已落 ③.2/清单；领域专家条件「计数到零=清理候选非自动删＋追加再评估＋『既有行为』措辞」＝已落 ①.1/③.2；前端架构师条件「JSON 形状停语义级」＝已落 ③.2）。**本议题 GD 通过（3/3）**，待 ⑤。

## ⑤ 人审门

未开启。GD 票齐后随总清单提交用户（晋级/留档）。
