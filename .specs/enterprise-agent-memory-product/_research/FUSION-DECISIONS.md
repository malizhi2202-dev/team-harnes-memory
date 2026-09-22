# FUSION-DECISIONS — 竞品调研 → PRODUCT-DESIGN.html v0.3 融合裁决表

- 日期：2026-09-22 · 状态：**已过审（2026-09-22 18:56 · 用户原话「通过」，随 PRODUCT-DESIGN v0.3 增量一起过人工审核门）**
- 输入：`_research/RECON-tencentdb-agent-memory.md · RECON-mem0.md · RECON-memos-memos.md · RECON-others.md`（各自 §可借鉴/不可取节）+ `LOCAL-FACTS.md`（禁止复活清单）
- 规则：借鉴点只有三种去向——**落入**（写明 HTML 落点）／**记候选**（进 §10.2 待确认，由未来架构 change 裁决）／**否决**（附理由）。已拍板否决项（第一版不建图库、DP 总开关、通用概率/事件溯源平台）不得借调研复活（LOCAL-FACTS §6 警告）。

## 一、落入（已改动文档）

| # | 借鉴点 | 来源 | 落点 |
|---|---|---|---|
| F1 | 生成溯源：抽取 prompt 版本+生效范围、来源消息区间、模型参数指纹、按 memory_id 反查生成日志 | TencentDB（generation-log + memory-prompt 版本化）RECON §9-2/3 | §6.6 p6「生成溯源」行；§5.12 步骤 5 |
| F2 | 合并建议语义化（ADD/UPDATE/SUPERSEDE/NOOP），但**降级为决策输入、不自我裁决** | mem0 四操作论文 + v3 撤除教训（RECON §3.2/§9）；TencentDB 去重 | §6.6 p6「机器合并建议（仅参考，不自我裁决）」行；§5.12；§9 Q13 |
| F3 | 双时间轴表达：现实线（何时为真）×系统线（何时得知/下架）；invalidated ≠ deleted；下架/删除证明/审计保留三分 | Zep bi-temporal（RECON-others A1）+ mem0 隐藏不删（RECON §9-③） | §6.7 p7「时间线与失效」面板；§5.7 步骤 2；Schema 留候选（C1） |
| F4 | superseded_by 关系化（取代是关系不是状态）| Zep 旧边失效 + backlog B12（既有采纳） | §6.7 面板 m-8807 行（演示口径，不落 Schema） |
| F5 | approved ≠ published 的"隐藏期"表达（发布前不可召回） | mem0 事件受理回执 + Google revision 计费教训 | §6.7 面板 m-9102 行（复用既有不变量口径，新增可视表达） |
| F6 | 竞品格局章（对照表/趋同 7/独有 5/缺口 6/引用纪律） | 四份 RECON | §1.5（新）；目录项；§9 Q13/Q14；§10.2/10.4/10.5；版本 v0.3+状态标 |
| F7 | 「行业零命中结论带日期与检索范围、竞品补齐即降级」维护约定 | 本次调研过程教训（口径漂移实例：MemOS 35.24%、mem0 双口径） | §10.5 新条目 |

## 二、记候选（进 §10.2，不动概念层）

| # | 候选 | 来源 | §10.2 行 |
|---|---|---|---|
| C1 | MemoryItem/Lineage 双时间轴字段语义（四戳、时点查询） | Zep | 「现实时间线（双时间轴）」 |
| C2 | 删除可逆窗口作为保留策略显式参数（对标 Google 48h） | Google/mem0/ChatGPT | 「删除的可逆窗口」 |
| C3 | 双速写入（个人空间直写 + 团队空间审核）缓解在线写摩擦 | Letta/Memobase | 「检索内核与在线写摩擦」 |
| C4 | 装配强度分档（fixed/tool/listing + 预算超限降级）作 ContextManifest 策略枚举 | TencentDB RECON §9-4 | 记于此表，未进 §10.2 ——若过审认为值得冻结，随 C1 一并裁决 |
| C5 | 三存储面分离（真值+history / 向量投影 / 实体投影）作 CopyInventory 最小可行结构 | mem0 RECON §9-① | 同上；物理结构属实现 change |
| C6 | 破坏性操作显式意图化（无过滤条件即报错；全量需多字段同时通配） | mem0 delete_all 改造 RECON §9-⑤ | 属 RunGate 接口细则，实现 change 输入 |

## 三、否决（附理由，防止日后翻案时失去上下文）

| # | 否决项 | 来源 | 理由 |
|---|---|---|---|
| V1 | 参数化/激活记忆（LoRA、KV-cache）进本平台对象模型 | MemOS 论文 | 论文=1.x 设计、2.0 仓库多为占位（Coming Soon）；实验未覆盖治理面；与「文档不承诺未验证机制」纪律冲突 |
| V2 | 全自动写图/整理者自审即应用 | Zep / Letta dreaming | 与已拍板的人工发布门、自动化永不自批（不变量池④）直接冲突；本次调研正是其反证（97.8% junk、矛盾并存） |
| V3 | 写入期概率置信度平台 | MemOS/Zep 置信分带 | 议题 2 已裁决「只做轻量分带」；mem0 #4573 佐证单一分数不可靠 |
| V4 | 竞品基准分数入正文 | 全部 | 引用纪律：厂商自报+托管/OSS 混称+争议未决（Zep vs mem0 原文两边已存 RECON-mem0 §8.2） |
| V5 | 反代注入寄生（LLM 代理链上做 ContextManifest） | TencentDB Proxy | 其官方自认 header 身份脆弱（issue #1377 等）；违反 §1.4 边界「不做执行侧代理」；Manifest 必须服务端按 Run 生成 |
| V6 | namespace 字符串约定式隔离 | 全组（趋同点 4） | 恰是我们要用 MemorySpace 类型化 schema 破的空位，不作采纳 |
| V7 | 「无产品同时覆盖四轴」旧断言原样保留 | mem0/TencentDB 均有 task/agent 轴 | 已校准为 §1.5 精确表述：四元组隔离有、**harness/执行器一等轴 + 交接语义无人做**；断言收窄不沿用 |

## 四、既有调研禁复活清单（本次核验确认未被违反）

A3 MMR/token-budget 公式、A12 BM25 参数、A10 双时间（MindMemOS 侧）、A11 skill 链、A13 schema learning——本次 RECON 未在任何落点引用 MindMemOS 被证伪的参数；Zep 的 bi-temporal 是**Zep 自身的源码级事实**（RECON-others A1），与被证伪的 MM TemporalEntity 无关，不属复活。

## 五、未处理（超出本轮 P-product 范围）

- 议题原文「参考资料」段的补挂接（LOCAL-FACTS §4.4）——归 S-align 或下轮文档 change。
- `tasks/plan.md`（Panel 前端重构）与 §1.5 缺口清单的对照吸收——属实现层，走 kernel backlog（.specs/memory-platform-backlog.md 的 B/C/D 组），禁入设计文档正文。
- docs/adr/memory-platform-backlog.md 旧拷贝漂移——另开 hygiene change 处理，本轮不动。
