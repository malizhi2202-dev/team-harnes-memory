# D-3 harness-console — Harness 接入控制面与会话绑定可见性

审核: 待审（第1轮）

- 事实依据：`GAPS.md` §2 行 8–10；`.specs/CONTEXT.md`（四轴含 harness）、backlog M5、platform 议题 01（Connector Contract V1 八条能力）
- 命题维度：归属×按需加载的「入口侧」（跨 harness 是我们项目描述的本体：「跨harness的团队记忆系统…按权限加载」）
- 边界：不改原型；不设计协议（协议是议题 01 已确认+设计阶段的事）

## ① 头脑风暴

我们的差异化命题就是「跨 harness」，但**13 屏里没有任何一屏回答「我的记忆接了哪些 harness、以什么身份接的、此刻注入了什么」**：

1. p9 是「Agent 配置与资源装配」——配置面有了，**p9 已有「工具/Connector 能力声明＋当前状态（可用/blocked）」表**（`PRODUCT-DESIGN.html:3341`，GD-1 横切修正），缺的是**实例级接入运行面**：一个 harness 多实例的注册/租约/撤销/降级状态（`harness_id` 等字段在 §3.3 已有，屏上到不了实例粒度）。
2. 「按权限加载」发生在每次运行的注入瞬间，用户对「这一轮到底加载了什么、为什么是这些、超时降级了什么」不可见——界面承接是 backlog `.specs/memory-platform-backlog.md:99` 的 **D3「召回解释与调试台」（P1：rank/score/scope/lineage/reason）**与 `:45` A5 配置来源解释（P2）；**B13 是实现前置而非需求来源**（`:70` B13=召回可见性**单入口**＝单函数+枚举门禁的代码约束——GD-1 高级产品经理改锚，初稿把屏的 v1 必要性押在「承接 P0 项 B13」属错锚，已改）。
3. 会话与空间的**绑定关系**：运行级已照亮（p5 运行详情显示 `ContextBinding`，GD-1 横切），**会话级不可见不可操作**——「当前会话挂在哪个 user/team/project/agent 空间、能否就地改挂」没有任何交互表达；外部已有一手形态可参照（见②）。
4. 采集通道状态（哪些 harness 正在回写、回写了什么）同样不可见——「接入-采集-沉淀」链路只有后半截有屏。

## ② 竞品分析

| 系统 | 接入形态事实 | 来源 |
|---|---|---|
| TencentDB（上游） | **零代码接入＝反向代理**：改 base URL `ANTHROPIC_BASE_URL=…:8096/claude-code/<spaceId>` + Bearer key，「协议不变——不需要插件、Hook 或 MCP Server」；8 类客户端各一份文档（Claude Code/CodeBuddy/WorkBuddy/Codex/**DeepSeek Harness**/OpenCode/Hermes/OpenClaw/Header 通用预选）；Proxy 链路 auth→sessionInit（选 team/agent/task）→injection→forward，每轮 capture 回报；**`mem:` 会话内指令 6 条**：session-reset（就地重挂归属）/sync（刷新注入）/create-skill/create-task(+confirm/cancel 预览)/update-task(非本人创建拒绝)/help；`/analyse` 标记：system prompt 尾部追加 `<asset_reflection>` 让模型对**本轮真正用过的资产**逐项自评是否起作用/为何没命中（双闸门、明示不建议默认开）；降级：控制面不可达时只查当前 agent | `INSTALL_CN.md:135-139,256-266,268-270,330-406`；`MemoryProxy/src/mem-command/commands/help.ts:12-51`；`tdai-l1-recall-injector.ts:11-43` |
| mem0 | 插件矩阵按 harness 分发（Claude Code/Cursor/Codex/OpenCode/deepseek-plugin…），记忆只映射 user_id/app_id，**不带「来源 harness/会话身份」**；openmemory 从 MCP 记忆服务器转向跨 harness 会话搬运 CLI（搬 transcript 非记忆） | docs.mem0.ai changelog/highlights.md；OpenMemory README |
| Zep | Memory MCP Server GA：**一个人一份记忆跨 harness**（Claude/ChatGPT/Claude Code/Codex/Cursor），IdP OAuth2.1+PKCE 登录、写操作全局 kill switch、token ~5min | blog.getzep.com/agent-memory-mcp/（2026-08-19） |
| Letta | agent 跨机器（云 repo/本地 checkout 同步）解决「同一 agent」不解决「不同 harness 不同身份」 | letta.com/blog/context-repositories/ |
| Memos | 每集成发一把限权令牌 + MCP 内建——接入身份与用户身份分离的通用做法 | usememos.com/docs/advanced/pat；README v0.27.0 MCP |

扫描共识：**七家没有一家把「记忆条目携带来源 harness/会话身份」做成一等公民**；TencentDB 最接近（sessionInit+三元组+`[from agent]` 标注）但其归属键是 team/agent/user/task，harness 只是通道不是身份。我们的四轴（含 harness）在模型层领先，缺的是**把模型翻译成控制面屏幕**。

## ③ 结论与建议

1. 立项方向：「**Harness 接入与运行观测**」屏（建议路由 `#/harness`，与 D-2 血缘屏分职：那张看一条记忆的来路，这张看一个 harness 的接入态）。**v1 只留两段（GD-1 高级产品经理切段条件②）**：①**实例清单**（注册/能力声明/租约/降级态——`harness_id/instance_id/capability_profile/lease_id/revoked_at` 已在 `_build/sec3.html:249` 字段行，含租约过期与撤销呈现）；②**会话↔空间绑定与就地改挂**（今天确实无处回答；参照 mem:session-reset 语义）。原第三段「最近注入快照」**降级为「跳到本次运行的 p5」一行链接**——p5 运行详情已显示 `Harness/Agent/责任链/ContextBinding`、p7 已有 Context Manifest 运行投影行，两段式避免「一屏两址」与双处维护同一事实。注入快照的完整面板化留给 D3（P1）承接，不混进 v1。
2. **空态与不确定态（GD-1 高级产品经理条件③）**：一个 harness 都没接时，屏面写「接入向导」而非空表；实例行呈现「**未探测 ≠ 可用**」（`sec3.html` 既有不变量）。
2. 模型结论（登记）：**MemoryEvent 已有 `harness_id` 字段——确认「记忆携带采集来源 harness」不变量已覆盖差异化点**，外部无一家有此字段；本议题只需把这个既有能力照亮到界面，不新增字段。
3. 接入形态结论（登记）：Connector Contract V1（议题 01）+ MCP 适配已确认；反代零代码形态是**接入通道的一种**而非前提，屏设计必须通道无关（代理/插件/MCP 三种都只体现为「实例+租约+能力声明」行）。
4. 预算化召回的界面语义（自 D3/议题 04）由本屏承接跳转：注入快照详情即「按需融合」的用户可见出口（v1 经 p5 链接承接，不复制数据）。

**建议改动清单（列表级）**
- 原型 `_build/`：新增 `#/harness` 屏（三段式如上，演示数据进统一数据集）；p9 保留为「配置/装配」，新增一列「运行时接入状态」链接到新屏。**新增路由的四处连锁成本（GD-1 前端架构师登记，付费前不得选此路径）**：① `CONTRACT §3` 19→20 路由；② 侧栏 blocks 须逐字一致（`check.sh:81` 硬编码 `len(blocks)==12 and set(anchors)=={20}`，加屏后计数同变，改一条路由＝改全部屏）；③ 新条目须在 §3 图标表登记 sprite id（现 18 id 无 plug/harness；可复用 `i-cpu`/`i-flow`，或 `template-sprite.html` 新增 symbol——后者不破零外部依赖红线）；④ 若把 Harness 登记为**新模块**，§7/§5.0/p12 三处同步 +1 行（16×8→17×8）。**v1 退路（GD-1 给出、我采纳进清单）**：若不愿付 ①③④，三段内容退到 **p9 内新增页签**承载，路由决策留给 2a-ui-design——零导航、零矩阵、零 sprite 变更。
- `§3.3`：不动字段（harness_id 已在）；`§5` 操作手册补「接入一个新 harness 的 8 步」节——**只追加、永不改号**（GD-1：新节用 `s5-20`、若新屏用 `p14`；`01-topbar-nav.html` doc-nav 逐条硬列 `#s5-1..#s5-19`/`#p1..#p13`，`check.sh` 第 6/11 项是「目录锚点不得断链」「目录顺序＝文档顺序」两道机器闸，改号会同时打断本 change 内 D-2 引 §5.14、D-6 引 §5.15/§5.16 的交叉引用）。
- `1-requirement`：US 候选「管理员能看到：每个 harness 实例的接入态、每轮注入了什么、当前会话挂在哪个空间并可改挂」。
- backlog 映射（GD-1 改锚）：本屏界面需求来源＝**D3 召回解释与调试台（P1）**与 A5 配置来源解释（P2）；**B13（P0）是实现前置**（召回只允许一条被解释的路径），不是本屏的需求来源。依赖登记 ROADMAP §3。
- 不做：真实代理网关实现、`mem:` 指令语法照搬（会话内指令属产品交互大改，超出融合范围，留档）。

## ④ GD 议题门（R16.7 法定减配 3 人）

三票（第 1 轮）：领域专家 ✅（并加强：租约/撤销时间已在字段层，「模型已有、缺屏」比他稿自述更强）／前端架构师 ❌（两条件：只追加永不改号＋四处路由成本含 p9 页签退路——**已落清单**）／高级产品经理 ❌（三条件：改锚 D3/A5＋v1 切两段＋空态/未探测——**已落 ①.2/③.1/③.2/清单**）。两位 ❌ 均书面声明条件落地即转 ✅；状态：待两票复核确认。

## ⑤ 人审门

未开启。GD 票齐后随总清单提交用户（晋级/留档）。
