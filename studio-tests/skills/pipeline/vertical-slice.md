# 技能测试规范：/vertical-slice

## 技能摘要

`/vertical-slice [--review full|lean|solo]` 是预制作阶段的验证构建：一个完整的 [开始 → 挑战 → 解决] 循环，接近生产质量，回答一个关于玩家体验*和*构建可行性的可证伪问题。它读取概念（`design/gdd/game-concept.md`，或在 `rigor: minimal` 时为 `design/game-brief.md`）、系统索引、架构和控制清单，定义验证问题和 3–5 分钟范围，请求将计划记录到 `production/session-state/active.md`，然后询问"我可以创建垂直切片目录 `prototypes/[concept-name]-vertical-slice/` 并开始实现吗？"。每个切片文件都以 `VERTICAL SLICE - NOT FOR PRODUCTION` 头部开头，使用该文件自身的注释语法。

在一次一问的复盘之后，它写入 `prototypes/[concept-name]-vertical-slice/REPORT.md`（来自 `.claude/docs/templates/vertical-slice-report.md`）和一行 `prototypes/index.md`，一次询问同时命名两者。判定为 **PROCEED / PIVOT / KILL**，外加 **NOT ASSESSED**——当没有人从头完整玩过循环时——排在 PROCEED 之上、PIVOT 和 KILL 之下，且绝不记录为 PROCEED。判定会输入 `/gate-check production`，后者对已构建切片中任何 Vertical Slice Validation 项为 NO 的情况判为 FAIL，因此此类 NO 之后不可用 PROCEED。解析后的 `modes.workflow` 决定 PROCEED 路线：`minimal` 下没有冲刺，因此速度日志输入简报的构建顺序，下一步是 `/create-stories` 然后 `/dev-story`；`standard` 和 `full` 下则是史诗、故事、冲刺和门禁列表。在 `full` 审查模式下，`creative-director` 通过 **CD-PLAYTEST** 审查结果（APPROVE / CONCERNS / REJECT，或 NOT ASSESSED）：APPROVE 保留建议；CONCERNS 交由用户（Revise / Accept / Discuss）；REJECT 意味着 PROCEED 不能成立——用户选择 PIVOT 或 KILL，报告记录总监的理由。PIVOT 写入 `PIVOT-NOTE.md`；KILL 运行五项健全性检查并追加到 `prototypes/GRAVEYARD.md`。

---

## 静态断言（结构性）

由 `/skill-test spec` 对照 SKILL.md 检查——无需 fixture。

- [ ] 具有必需 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题
- [ ] 包含判定关键词：PROCEED、PIVOT、KILL、NOT ASSESSED
- [ ] 在创建切片目录之前、以及在写入 REPORT.md、PIVOT-NOTE.md 和 GRAVEYARD.md 之前，包含"May I"措辞
- [ ] 每种判定都有下一步交接（PROCEED → `/create-epics`、`/create-stories`、`/sprint-plan`、`/gate-check production`；PIVOT → `/design-system`、`/architecture-decision`、`/vertical-slice`；KILL → `/brainstorm` 或 `/prototype`）
- [ ] 记录了 CD-PLAYTEST：在 `full` 中生成，在 `lean`/`solo` 中跳过并附注

---

## 总监门禁检查

- **Full 模式**：CD-PLAYTEST——`creative-director` 接收 REPORT.md 内容、验证问题以及支柱和核心幻想（来自 `design/gdd/game-concept.md`，或来自 `design/game-brief.md` 的推介和"他们的感受"一行）。APPROVE → 建议维持；CONCERNS → 与建议一同展示，由用户决定（Revise / Accept / Discuss）；REJECT → PROCEED 不能成立：用户选择 PIVOT 或 KILL，REPORT.md 记录总监的理由（PIVOT 或 KILL 成立）；NOT ASSESSED [缺少输入] → 既不支持也不推翻建议：补充输入并重新运行门禁，或记录 `CD-PLAYTEST: NOT ASSESSED — [input]`。对 REPORT.md 及其 `prototypes/index.md` 行的任何更改都先询问。当切片自身判定为 NOT ASSESSED 时跳过门禁："CD-PLAYTEST skipped — the slice has not been played yet."
- **Lean 模式**：跳过——"CD-PLAYTEST skipped — Lean mode."
- **Solo 模式**：跳过——"CD-PLAYTEST skipped — Solo mode."

---

## Fixture 默认值

每个用例的 `project.yaml` 都设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且没有 `project.local.yaml`，除非其自身 Fixture 另有说明。规则：`studio-tests/CLAUDE.md` § Fixture defaults。

---

## 测试用例

### 用例 1：正常路径——构建切片、无辅助游玩、PROCEED

**Fixture：**
- `project.yaml`：`engine.name: Godot`、`engine.language: GDScript`、`modes.review_mode: solo`、`modes.workflow: standard`（固定：未设置会解析为 minimal，从而将 PROCEED 路由到 `/create-stories` 和 `/dev-story` 而非步骤 7 的列表）
- `design/gdd/game-concept.md`（核心幻想、支柱）、`design/gdd/systems-index.md`、`docs/architecture/architecture.md` 和 `docs/architecture/control-manifest.md` 存在；`prototypes/` 中没有 `ember-trail-vertical-slice/`
- 复盘中玩家无辅助完成循环，40 秒内到达第一个有意义的动作，感受到核心幻想且未遇到乐趣阻碍；用户回答 PROCEED

**输入：** `/vertical-slice`

**预期行为：**
1. 读取概念、系统索引、架构和清单；陈述一个同时覆盖玩家体验和构建可行性的验证问题
2. 展示 3–5 分钟范围和构建计划；确认中命名 `production/session-state/active.md`，检查点写入其中
3. 询问"我可以创建垂直切片目录 `prototypes/ember-trail-vertical-slice/` 并开始实现吗？"；它写入的每个 GDScript 文件都以 `# VERTICAL SLICE - NOT FOR PRODUCTION` 开头
4. 与用户迭代直到循环可玩，然后请求从头游玩，并一次一个地提出六个复盘问题
5. 从 `.claude/docs/templates/vertical-slice-report.md` 起草报告，附逐日速度日志；询问"我可以将此报告写入 `prototypes/ember-trail-vertical-slice/REPORT.md` 并将其行添加到 `prototypes/index.md` 吗？"；写入两者
6. 注明"CD-PLAYTEST skipped — Solo mode."
7. PROCEED 路由到 `/create-epics layer:foundation`、`/create-epics layer:core`、`/create-stories`、`/sprint-plan` 和 `/gate-check production`

**断言：**
- [ ] 验证问题同时包含玩家体验和构建可行性两部分
- [ ] 在写入任何切片文件之前，范围已确认，且检查点询问命名 `active.md`
- [ ] 目录询问在任何切片文件之前；每个 GDScript 文件的头部是 `#` 注释，绝不是 `//`
- [ ] 复盘问题在从头游玩之后一次一个地提出
- [ ] REPORT.md 和 `prototypes/index.md` 行仅在一次同时命名两者的询问之后写入；该行记录 PROCEED
- [ ] 正式阶段步骤是 `/gate-check production`（预制作 → 制作），而非 `/gate-check pre-production`

---

### 用例 2：失败路径——已构建切片有 NO 则不能 PROCEED

**Fixture：**
- 同用例 1，但复盘中玩家需要开发者解释核心动作（问题 1：否）
- 用户仍回答 PROCEED，然后在被询问时选择 PIVOT

**输入：** `/vertical-slice`

**预期行为：**
1. 技能不记录 PROCEED：`/gate-check production` 对已构建切片中任何 Vertical Slice Validation 项为 NO 的情况判为 FAIL（`.claude/skills/gate-check/references/gate-production.md`）
2. 它要求用户选择 PIVOT 或 KILL，并命名问题 1 的答案为理由
3. 用户选择 PIVOT；REPORT.md 在其询问后写入，判定为 PIVOT 并命名该 NO
4. PIVOT 路线运行：两个结转问题，一次一个，然后"我可以将此写入 `prototypes/ember-trail-vertical-slice/PIVOT-NOTE.md` 吗？"

**断言：**
- [ ] 当验证项为 NO 时，PROCEED 不写入 REPORT.md 或 `prototypes/index.md`
- [ ] 由用户而非技能在 PIVOT 和 KILL 之间选择
- [ ] 报告命名失败的验证项
- [ ] PIVOT-NOTE.md 仅在其询问后写入，包含有效之处、失败之处以及下一个切片应证明什么

---

### 用例 3：模式变体——CD-PLAYTEST 在 full 中运行，在 lean 和 solo 中跳过

**Fixture：**
- 一个切片已复盘，建议为 PROCEED，REPORT.md 已写入
- 运行 A：`modes.review_mode: full`；CD-PLAYTEST 返回 APPROVE
- 运行 B：`modes.review_mode: lean`
- 运行 C：`modes.review_mode: full`，输入 `/vertical-slice --review solo`

**预期行为：**
1. 运行 A：REPORT.md 写入后，为 CD-PLAYTEST 生成 `creative-director`，传入 REPORT.md 内容、验证问题以及支柱和核心幻想；APPROVE 使 PROCEED 维持且 REPORT.md 不变
2. 运行 B：不生成；"CD-PLAYTEST skipped — Lean mode."
3. 运行 C：`--review solo` 覆盖解析后的 `full`；不生成；"CD-PLAYTEST skipped — Solo mode."

**断言：**
- [ ] 运行 A 仅在 REPORT.md 存在后生成 CD-PLAYTEST，传入三项输入
- [ ] 运行 A 的 APPROVE 保留建议且不触发 REPORT.md 更新询问
- [ ] 运行 B 和 C 不生成总监并打印其跳过注释
- [ ] 任何运行中都不生成其他总监门禁

---

### 用例 4：边缘情况——切片已构建但无人玩过

**Fixture：**
- 循环已可演示，但用户本次会话无法从头完整游玩
- `modes.review_mode: full`

**输入：** `/vertical-slice`

**预期行为：**
1. 阶段 5 游玩不发生，因此判定为 NOT ASSESSED，并附理由
2. REPORT.md 在其询问后记录速度日志和 NOT ASSESSED；`prototypes/index.md` 行记录 NOT ASSESSED
3. 不生成 CD-PLAYTEST："CD-PLAYTEST skipped — the slice has not been played yet."
4. 下一步：完成游玩并从 `active.md` 检查点复盘，然后询问后更新 REPORT.md；切片不会像已通过一样进入 `/gate-check production`

**断言：**
- [ ] 判定为 NOT ASSESSED——绝不是 PROCEED——并陈述其理由
- [ ] NOT ASSESSED 排在 PROCEED 之上、PIVOT 和 KILL 之下
- [ ] 未游玩的切片不生成 CD-PLAYTEST，并宣布跳过
- [ ] 不提供 PROCEED 的下一步

---

### 用例 5：总监门禁——CD-PLAYTEST 返回 REJECT、CONCERNS 或 NOT ASSESSED

**Fixture：**
- 一个切片已复盘，建议为 PROCEED，REPORT.md 已写入；`modes.review_mode: full`；`design/gdd/game-concept.md` 包含支柱
- 运行 A：CD-PLAYTEST 返回 REJECT——核心幻想不存在。被询问 PIVOT 或 KILL 时，用户选择 KILL；没有游玩会话显示出情感高点，且这是该概念的第三次切片尝试
- 运行 B：CD-PLAYTEST 返回 CONCERNS——解决节拍削弱了一个支柱；用户选择 `Accept with noted concerns`
- 运行 C：CD-PLAYTEST 返回 NOT ASSESSED——其收到的上下文中缺少支柱

**输入：** `/vertical-slice`

**预期行为：**
1. 运行 A：REJECT 被解读为门禁的裁决，而非切片判定。PROCEED 不能成立：技能要求用户选择 PIVOT 或 KILL；用户选择 KILL；在"我可以更新 `prototypes/[concept-name]-vertical-slice/REPORT.md` 及其 `prototypes/index.md` 行吗？"之后，两者记录 KILL，REPORT.md 记录总监的理由。阶段 8 的 KILL 路线运行：两项检查项适用，因此 KILL 健全，并询问"我可以将此追加到 `prototypes/GRAVEYARD.md` 吗？"
2. 运行 B：关注点与 PROCEED 一同展示，附 `Revise the recommendation` / `Accept with noted concerns` / `Discuss further`；选择 Accept 后，PROCEED 维持，并在更新询问后，REPORT.md 记录关注点
3. 运行 C：命名缺少的输入；用支柱重新运行门禁，或在更新询问后记录 `CD-PLAYTEST: NOT ASSESSED — [input]`；PROCEED 既不被其确认也不被推翻

**断言：**
- [ ] 总监的回答被解读为 APPROVE / CONCERNS / REJECT / NOT ASSESSED——绝不是其自身的 PROCEED / PIVOT / KILL
- [ ] 运行 A：REJECT 后不保留 PROCEED；用户选择 PIVOT 或 KILL，总监的理由在 REPORT.md 中
- [ ] 运行 A：GRAVEYARD.md 条目仅在其询问后追加，附具体终止理由
- [ ] 运行 B：由用户决定；建议仅在用户修订时改变
- [ ] 运行 C：NOT ASSESSED 不被视为 APPROVE
- [ ] 每次 REPORT.md / `prototypes/index.md` 更新都遵循同时命名两者的询问

---

### 用例 6：Minimal 工作流——PROCEED 路由到故事，速度输入简报

**Fixture：**
- `project.yaml`：`engine.name: Godot`、`engine.language: GDScript`、`modes.workflow: minimal`、`modes.review_mode: solo`（固定：此用例测试 `minimal` PROCEED 路线，与用例 1 相反）
- `design/game-brief.md` 包含推介、"他们的感受"一行和三个功能的构建顺序；没有 `design/gdd/game-concept.md`、没有冲刺计划、没有 `production/sprints/`
- 复盘中玩家无辅助完成循环，40 秒内到达第一个有意义的动作，感受到核心幻想且未遇到乐趣阻碍；用户回答 PROCEED

**输入：** `/vertical-slice`

**预期行为：**
1. 读取简报代替概念，并如用例 1 构建
2. 报告的逐日速度日志被描述为输入简报的构建顺序，而非冲刺规划（"在 `workflow: minimal` 下没有冲刺，输入简报的构建顺序"）
3. PROCEED 打印 minimal 路线：`/create-stories`（来自简报），然后对第一个故事运行 `/dev-story`。技能声明"此列表的其余部分是 `standard`/`full` 路径"
4. 不将 `/create-epics layer:foundation`、`/create-epics layer:core` 和 `/sprint-plan` 步骤作为下一步提供

**断言：**
- [ ] 工作流从 `project.yaml` 解析（`modes.workflow: minimal`），而非仅从 rigor 预设假定
- [ ] PROCEED 的下一步是 `/create-stories` 然后 `/dev-story`
- [ ] 在 `minimal` 下不推荐 `/create-epics` 和 `/sprint-plan` 作为下一步
- [ ] 速度日志与简报的构建顺序关联，不引用或创建冲刺计划
- [ ] REPORT.md 和 `prototypes/index.md` 行在一次同时命名两者的询问后仍记录 PROCEED

---

## 协议合规

- [ ] 在定义验证问题之前读取概念（或在 `minimal` 下读取简报）
- [ ] 构建前确认范围；在将计划记录到 `active.md` 之前、以及在创建 `prototypes/[concept-name]-vertical-slice/` 之前询问
- [ ] 将切片文件保留在 `prototypes/` 下，每个都以 `VERTICAL SLICE - NOT FOR PRODUCTION` 头部开头，使用其自身注释语法
- [ ] 在写入 REPORT.md（及其 `prototypes/index.md` 行）、PIVOT-NOTE.md、GRAVEYARD.md 以及任何门禁后报告更新之前询问
- [ ] 验证项为 NO 后绝不记录 PROCEED，也绝不为无人玩过的切片记录
- [ ] 在 CD-PLAYTEST 之前应用审查模式，跳过时打印跳过注释
- [ ] 以每种判定的下一步结束

---

## 覆盖说明

- 多轮构建循环、第 3 天沉没成本停止和逐日速度日志需要实际运行验证。
- 重新运行时读取先前的 `PIVOT-NOTE.md` 以框定新验证问题未测试。
- `rigor: minimal` 的 PROCEED 路线是用例 6。简报的推介和"他们的感受"一行在 CD-PLAYTEST 输入中代替支柱未测试，因为门禁在 `full` 之外被跳过。
- 健全性检查清单给出 0–1 项的 KILL（建议改为针对性 PIVOT）未测试。
- 联网游戏限制（本地切片无法验证网络手感）是建议性文本，未