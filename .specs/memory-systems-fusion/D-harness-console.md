# D-3 harness-console — Harness 接入控制面与会话绑定可见性

审核: 待审（第1轮）

- 事实依据：`GAPS.md` §2 行 8–10；`.specs/CONTEXT.md`（四轴含 harness）、backlog M5、platform 议题 01（Connector Contract V1 八条能力）
- 命题维度：归属×按需加载的「入口侧」（跨 harness 是我们项目描述的本体：「跨harness的团队记忆系统…按权限加载」）
- 边界：不改原型；不设计协议（协议是议题 01 已确认+设计阶段的事）

## ① 头脑风暴

我们的差异化命题就是「跨 harness」，但**13 屏里没有任何一屏回答「我的记忆接了哪些 harness、以什么身份接的、此刻注入了什么」**：

1. p9 是「Agent 配置与资源装配」——配置面有了，**接入运行面没有**：harness 实例的注册/租约/能力声明/降级状态（`harness_id` 字段在 §3.3 已有，屏上不可见）。
2. 「按权限加载」发生在每次运行的注入瞬间，用户对「这一轮到底加载了什么、为什么是这些、超时降级了什么」不可见——正是 backlog **B13（单次召回可见性入口，P0）**与议题 05「召回解释」的界面缺口。
3. 会话与空间的**绑定关系**（当前会话挂在哪个 user/team/project/agent 空间、能否就地改挂）没有任何交互表达；外部已有一手形态可参照（见②）。
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

1. 立项方向：「**Harness 接入与运行观测**」屏（建议路由 `#/harness`，与 D-2 血缘屏分职：那张看一条记忆的来路，这张看一个 harness 的接入态）。三段：实例清单（注册/能力声明/租约/降级态）、会话绑定卡（当前挂在哪个空间 + 就地重挂入口，参照 mem:session-reset 语义）、最近注入快照（复用 B13 数据：这一轮注入了什么/预算/超时降级/来源标注，即 `/analyse` 资产反思的**面板化静态版**——让模型自评回写是后置增强，留档不立项）。
2. 模型结论（登记）：**MemoryEvent 已有 `harness_id` 字段——确认「记忆携带采集来源 harness」不变量已覆盖差异化点**，外部无一家有此字段；本议题只需把这个既有能力照亮到界面，不新增字段。
3. 接入形态结论（登记）：Connector Contract V1（议题 01）+ MCP 适配已确认；反代零代码形态是**接入通道的一种**而非前提，屏设计必须通道无关（代理/插件/MCP 三种都只体现为「实例+租约+能力声明」行）。
4. 预算化召回的界面语义（自 B13/议题 04）在本屏承接：注入快照即「按需融合」的用户可见出口。

**建议改动清单（列表级）**
- 原型 `_build/`：新增 `#/harness` 屏（三段式如上，演示数据进统一数据集）；p9 保留为「配置/装配」，新增一列「运行时接入状态」链接到新屏。
- `§3.3`：不动字段（harness_id 已在）；`§5` 操作手册补「接入一个新 harness 的 8 步」节（§5.x 编号顺延）。
- `1-requirement`：US 候选「管理员能看到：每个 harness 实例的接入态、每轮注入了什么、当前会话挂在哪个空间并可改挂」。
- backlog 映射：本屏是 B13（P0）的界面承接件；ROADMAP 中登记依赖。
- 不做：真实代理网关实现、`mem:` 指令语法照搬（会话内指令属产品交互大改，超出融合范围，留档）。

## ④ GD 议题门（R16.7 法定减配 3 人）

待投票：高级产品经理（接入屏是否 v1 必要 vs 后台化）/ 前端架构师（新路由对 CONTRACT §3 连锁、注入快照渲染的数据形状）/ 领域专家（与议题 01 已确认契约、B13 的重复度）。

## ⑤ 人审门

未开启。GD 票齐后随总清单提交用户（晋级/留档）。
