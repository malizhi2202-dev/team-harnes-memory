# D-1 fusion-state-view — 融合状态的可判定表达（读视图·双时态·分层口径）

审核: 待审（第1轮）

- 事实依据：`GAPS.md` §1.1/§1.3、§2 行 1–4
- 命题维度：分层 + 按需融合
- 边界：不改原型（R16.1）；不借外部阈值数字进我们的 Schema（十议题系统性留白，采纳参数须另题走设计阶段）

## ① 头脑风暴

我们的融合语义在原则层已经很硬（冲突八分类、追加 Revision 不覆盖、9 态、ChangeSet 唯一审核对象），但**从原则到「用户在屏上可判定」缺三段**：

1. **条目状态不显**。原型里 `superseded` 只存在于血缘关系枚举与 §10.1 术语表；一条记忆被新事实取代后，用户在工作台/详情上看不到「它现在处于什么态、被谁取代、是不是合成件」。模型有态，屏上无态。
2. **读视图不显**。融合之后「默认看到什么」是产品决策：mem0 用三个读挡位（默认含 superseded／latest_only／include_merged）把选择权做成查询参数+UI 开关。我们没有任何「视图挡位」概念，投影加载（议题 04）只写了按权限裁剪，没写按状态裁剪。
3. **时间轴不显**。我们的 `occurred_at/created_at` 是单时点；Zep 把「事实何时成立/失效（valid/invalid_at）」与「系统何时知道（created/expired_at）」拆成双时态四字段，于是「当时为什么是那个答案」可回答、point-in-time 免费获得。血缘表里已有一个 `valid_time`（议题 07），但融合态层没有。
4. **写入时机不成维度**。LangMem 把 hot-path（对话内主动写）vs background（会话后反思沉淀）列为一等设计轴；我们议题里有「会话后沉淀」，但从未把「这条记忆是什么时候、由哪条通道写进来的」做成可筛选/可展示的属性。
5. **分层口径对表**（附带）：TencentDB 的 L1 实际 7 类两族而上游 API 文档仍写 3 类——外部「文档落后实现」活标本；我们 `VALIDATION.md` §8 已有 L0/L3 分层口径冲突未裁决。本议题**不裁决口径**，只建议融合态表达设计必须以「§8 裁决结果」为输入，防止在冲突口径上盖楼。

备选形态：a) 只在 p7 详情抽屉加状态徽标+来源事件列表（最小）；b) 全局读视图开关（当前态/全部态/仅已确认）+ 双时态字段入 §2.3 不变量（完整）；c) 引入外部五态枚举直接替换 9 态（否决：破坏既有已确认议题）。

## ② 竞品分析

| 系统 | 事实 | 来源（2026-09-22/29 抓取） |
|---|---|---|
| mem0 | 条目自带 `lifecycle_state`/`replaced_by`/`synthesized`/`structured_attributes`/`expiration_date`，五态直接可读；Dream 三动作 Supersede/Merge/Synthesis **全部非破坏**（merged 记录保留、默认隐藏，`include_merged=true` 可取）；读三挡位：默认 active+superseded／`latest_only`／`include_merged`；官方原话「Dream 无任何破坏性操作，每次变更可复核」 | docs.mem0.ai/api-reference/memory/get-memory.md；platform/features/dream.md |
| Zep | 事实失效不删除：新 episode 与旧边比对后由模型裁决，把 `invalid_at` 写在旧边上；双时态四字段 created/valid/invalid/expired（「何时知道它失效」与「事实何时失效」分离）；矛盾不留并列 | help.getzep.com/facts.md；how-graph-creation-works.md |
| TencentDB | 批量 LLM 冲突检测动作 `store|update|merge|skip`，多目标 `target_ids`、跨类 `merged_type`、`merged_priority`、`merged_timestamps`（新旧时间戳并集）；但冲突语义是「静默以新覆盖旧」，**无待裁决队列/无 diff/无置信**——其团队自己的 backlog 也把该缺口列为未解 | 上游 `src/core/prompts/l1-dedup.ts:35-70,97-134`；`docs/adr/memory-platform-backlog.md:61`（fork 内对标件） |
| Memobase | 反面：融合=画像覆盖为当前态，无失效时间轴，前后不可追溯（同事实变更无法归因） | docs.memobase.io（全文 grep 无 invalid/TTL 时间轴） |
| LangMem | collection 型入库调和 insert/update/delete-or-invalidate，官方承认「过抽→精度掉，欠抽→召回掉」需 eval 调优；hot-path vs background 写入二分带延迟/召回/算力权衡表 | github langmem conceptual_guide.md |

结论性观察：**三家（mem0/Zep/TencentDB）都收敛到「写侧累积+状态降级，读侧视图开关」的三态分离**；差别在 mem0/Zep 把状态挂在条目上（可判定），TencentDB 只在决策瞬间用（不可追溯）。我们的 9 态+追加 Revision 与 mem0 语义同构，**缺的只是把它表达出来**。

## ③ 结论与建议

判断：不新增融合算法、不引入外部枚举；把**已有模型能力翻译成可判定表达**，三项最小增量：

1. **条目状态显性化**：记忆详情呈现「当前态/被谁取代(replaced_by 语义=我们的 supersedes 关系)/是否合成件/到期隐藏」，全部复用既有 9 态与血缘关系，不改枚举。
2. **读视图挡位**：召回投影与列表提供「仅当前有效｜含被取代｜含已合并」三挡（映射 mem0 三读法），并明确默认挡=含被取代（与 mem0 一致，保历史可见性优先于洁净感）。
3. **双时态字段进数据模型讨论**：MemoryRevision 层增加「事实生效/失效区间」与「系统知悉时间」的分离表述（对齐议题 07 已有 valid_time，避免两套时间语义）；此项**只进 REQUIREMENT 候选，字段设计待 `VALIDATION.md` §8 口径裁决后**。
4. 附带登记：写入通道（对话内/会话后）作为记忆的可筛选属性列入候选；分层口径以 §8 裁决为前置。

**建议改动清单（列表级，不改原型）**
- `1-requirement`：US 候选「融合状态可见可筛」3 条（对应上 1/2/4）+ AC 以可判定输出措辞（哪个屏、哪一挡、看到什么）。
- `2a-ui-design`/原型 `_build/sec5*.html`（p7 记忆与空间策略屏区）：建议增「状态徽标 + 读视图挡位控件」两组元素；血缘详情联动部分移交 D-2，不在此重复。
- `PRODUCT-DESIGN §2.3/§10.1`：术语表补「读视图挡位」「双时态（讨论项）」条目占位；§10.1 九态枚举不动。
- platform 议题 04（按需加载）：建议以「补录轮」追加「按状态裁剪视图」一句到已确认原则下——**属修订已确认议题，需人审背书才可动**（此处仅登记）。
- 不做：外部阈值/参数导入、五态替换 9 态、融合算法变更。

## ④ GD 议题门（R16.7 法定减配 3 人）

待投票：高级产品经理（读视图挡位是否用户可感价值）/ 前端架构师（p7 屏元素与投影 API 形状可行性）/ 领域专家（双时态与九态是否冲突）。状态：审核: 待审（第1轮）。

## ⑤ 人审门

未开启。GD 票齐后随总清单以「晋级/留档」选择题提交用户，等原话背书（R18.2②）。
