# TOPICS — memory-systems-fusion（议题状态表）

- **Change ID**: `memory-systems-fusion`
- **阶段**: D-discovery（R16 · 议题发现前置）· 第 1 轮 · **零代码/零原型改动（R16.1）**
- **来源**: issue MALIZHI-9 `01a0c836-ed7f-7bd0-9915-82b2965a3e2c`；调度结论见 issue 评论 `01a0c83b-adfd-7999-be40-be39b57193f2`
- **上游门禁状态**: `enterprise-agent-memory-product` 至 T-04（脚本+浏览器验证全绿；`VALIDATION.md` §7 人工浏览器 UAT 待执行）；`enterprise-agent-memory-platform` 十议题全部「已确认（阶段性）」（第 76 轮，2026-09-21）
- **命题（8 对标维度）**: 分层 / 归属 / 按需融合 / 知识化发布 / 血缘 / 权限与治理 / 跨空间脱敏 / 撤回清理证据

## 事实扫描与外部调研（①②）产出

| 工件 | 说明 |
|---|---|
| `GAPS.md` | 现状缺口清单（已有/部分/完全没有三档判决 + 出处），子议题拆解的事实依据（R16.2） |
| `ROADMAP.md` | 下游路径与既有技术约束（当前无过审结论可并入，占位待 ⑤） |

## 子议题清单（第 1 轮拆解）

| # | 子议题 slug | 文件 | 一句话主张 | 审核 | 备注 |
|---|---|---|---|---|---|
| 1 | fusion-state-view | `D-fusion-state-view.md` | 给非破坏性融合补「读视图开关 + 双时态有效窗口 + 分层口径对表」三件表达层机制 | 审核: 待审（第1轮） | 对标 mem0/Zep/Letta |
| 2 | lineage-screen | `D-lineage-screen.md` | 补血缘专属屏与「记忆↔源消息/生成过程」反查端点契约（导航已有入口、无屏属内部缺陷） | 审核: 待审（第1轮） | 对标 TencentDB/mem0/Letta/Zep |
| 3 | harness-console | `D-harness-console.md` | 补「Harness 接入与绑定」控制面屏（连接方式/能力声明/降级/会话绑定指令） | 审核: 待审（第1轮） | 对标 TencentDB Proxy/Zep MCP/mem0 插件 |
| 4 | asset-telemetry | `D-asset-telemetry.md` | 知识资产补发布后度量（usage/last_used/expires）与装配口径；评估 SKILL.md 式发布形态 | 审核: 待审（第1轮） | 对标 TencentDB Asset/Memobase/Acontext/Letta |
| 5 | governance-check | `D-governance-check.md` | 权限治理从「文档矩阵」补到「在线可验证」：acl/check 式验证器 + 影响面反查 + 默认拒绝 API 形状 | 审核: 待审（第1轮） | 对标 TencentDB ACL/mem0/Letta/Memos |
| 6 | redaction-evidence | `D-redaction-evidence-hardening.md` | 差异化确认 + 证据硬化：脱敏门禁（只发派生层）、按计数级联删除、重算即撤销注入快照、反证据销毁 | 审核: 待审（第1轮） | 七家外部零脱敏＝领先面；硬化项见文件 |

## 流程状态（R16.3/R16.6）

- ①头脑风暴 ②竞品分析 ③结论建议 —— 6 份子议题文件均已完成，随步落盘。
- ④ GD 议题门 —— **本轮收尾评论已召集法定 3 人**（高级产品经理 / 前端架构师 / 领域专家，R16.7 减配），待投票回帖。
- ⑤ 人审门 —— 未开启；GD 票齐后以「晋级/留档」选择题提交用户，**停下等原话背书（R18.2②）**，禁止自行改写状态标。
- 过审结论并入 `ROADMAP.md`（R16.3）：目前为空（无过审项）。
- 轮次计数：D-discovery 第 1 轮 / ≤3 轮（R16.4）。本轮未启动第 2 轮（若 ⑤ 后有落盘新事实再评估）。

## 加载预算声明（R1.9）

- 整读（SPEC 级小文件）：product change 8 份 .md（均 ≤245 行）；`.specs/CONTEXT.md`(368) / `memory-platform-comparison-v2.md`(298) / `intent-os-mindmemos-adoption.md`(315) / `memory-platform-*.md` 各 ≤356 行；platform `README.md`（9KB）。
- 蒸馏稿（`_facts/*-digest.md` 共 460KB）**未整读**：按标题大纲 + 关键节精读（合计每份 ≤175 行）；议题原文（1.8MB）**仅 grep 命中行取节**（02:190-217 / 03:592-613 / 04:497-518）。
- `PRODUCT-DESIGN.html`（446KB）**未整读**：标题树 grep + 字段表定点行读（:1022-1210 / :4109 区域）。
- `refs/intent-os-platform.md`(205KB)、`refs/mindmemos.md`(110KB) 未整读（沿用其结论区既有蒸馏）。
