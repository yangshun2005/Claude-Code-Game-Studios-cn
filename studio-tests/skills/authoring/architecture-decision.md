# Skill 测试规范：/architecture-decision

## Skill 摘要

`/architecture-decision` 编写一份架构决策记录（ADR）。它有三种入口模式：常规编写（`/architecture-decision [title]`）、`retrofit <path>`（为现有 ADR 补充缺失章节，不改动其余内容）、`accept <ADR-id>`（唯一能将 ADR 从 `Proposed` 变为 `Accepted` 的路径）。当 retrofit 询问缺失的 Status 且用户回答 `Accepted` 时，它写入 `Proposed`，并将该 ADR 交给验收模式处理。

常规编写先加载引擎上下文（`docs/engine-reference/[engine]/VERSION.md`，并对 MEDIUM/HIGH 风险领域给出知识缺口警告），选取下一个 ADR 编号，检查 `docs/registry/architecture.yaml` 中的约束性立场，然后以确认/调整式 `AskUserQuestion` 呈现其假设（Status 始终为 `Proposed`，从不询问）。随后一次性按模板起草完整 ADR，由主引擎专家验证（步骤 5.5），仅在 `full` 审查模式下运行 TD-ADR 门禁（步骤 5.6），运行始终会报告的 GDD 同步检查，并在写入 `docs/architecture/adr-[NNNN]-[slug].md` 前询问一次——"May I write it?"。注册表更新需单独批准。结尾输出提示用户在新会话中运行 `/architecture-review`，并在决策确定后运行 `/architecture-decision accept ADR-NNNN`。编写过程从不设置 `Accepted`，也从不解除故事的阻塞。

---

## 静态断言（结构性）

由 `/skill-test spec` 对照 SKILL.md 检查——无需 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题
- [ ] 包含状态/判定关键词：`Proposed`、`Accepted`，以及 TD-ADR 结果 `CONCERNS` / `REJECT`
- [ ] 包含 "May I write" 协作协议语言（整份 ADR 写入批准和注册表批准）
- [ ] 结尾有下一步交接（`/architecture-review`、`accept ADR-NNNN`）
- [ ] 记录了门禁行为：先引擎专家验证，再在 full 模式下运行 TD-ADR；lean/solo 模式下跳过 TD-ADR 并给出具名说明
- [ ] 记录了编写过程始终产出 `Proposed`，只有验收模式设置 `Accepted`——retrofit 的 `Accepted` 回答经由它路由
- [ ] 提及来自 `docs/engine-reference/` 的引擎版本戳（Engine Compatibility 表）

---

## Director 门禁检查

审查模式来自解析后的 `review_mode`（`project.yaml` 中的 `modes.review_mode`）；`--review full|lean|solo` 可对单次运行覆盖它。

在 `full` 模式下：草稿生成后，主引擎专家验证它（步骤 5.5），然后 `technical-director` 以门禁 TD-ADR 启动（步骤 5.6）。两者按此顺序运行，不并行。CONCERNS 时逐字显示关注点，用户选择 `Revise flagged items` / `Accept and proceed` / `Discuss further`；REJECT 时在写入批准前修订草稿。任何判定都不改变 ADR 的 Status。

在 `lean` 模式下：跳过 TD-ADR（它不是 PHASE-GATE）。输出注明 "TD-ADR skipped — Lean mode."。引擎专家验证仍会运行。

在 `solo` 模式下：跳过 TD-ADR。输出注明 "TD-ADR skipped — Solo mode."。引擎专家验证仍会运行。

在所有模式下，写入的 ADR 均为 `Status: Proposed`。

---

## Fixture 默认值

除非其自身 Fixture 另有说明，每个用例的 `project.yaml` 都设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且无 `project.local.yaml`。规则：`studio-tests/CLAUDE.md` § Fixture defaults。

---

## 测试用例

### 用例 1：正常路径——新建渲染 ADR，full 模式，TD-ADR 批准

**Fixture：**
- `project.yaml` 具有 `engine.name: Godot` 和 `modes.review_mode: full`
- `docs/engine-reference/godot/VERSION.md` 存在，并将锁定版本标记为 HIGH 风险；`breaking-changes.md` 列出一项渲染变更
- `docs/architecture/` 包含 `adr-0001-*.md` 和 `adr-0002-*.md`；均不涉及渲染
- `docs/registry/architecture.yaml` 存在，无涉及渲染的立场
- `design/gdd/visual-effects.md` 存在；用户确认它是驱动该决策的 GDD，且其名称与 ADR 的 Key Interfaces 匹配
- 引擎专家仅发现次要备注；TD-ADR 返回 APPROVE

**输入：** `/architecture-decision rendering-approach`

**预期行为：**
1. 步骤 1 先读取 `VERSION.md`，并显示渲染领域的 ENGINE KNOWLEDGE GAP WARNING
2. 步骤 2 扫描 `docs/architecture/` 并分配 ADR-0003
3. 步骤 3a 在任何现有 ADR 之前读取注册表
4. 步骤 4 以确认/调整式 `AskUserQuestion` 呈现假设（问题、2–3 个备选方案、GDD 关联、依赖、`Status: Proposed`）；用户确认前不生成 ADR
5. 步骤 5 起草完整模板，包括 Engine Compatibility 表（引擎、领域、知识风险、查阅的参考资料）
6. `godot-specialist` 验证草稿；次要备注进入 Risks 章节
7. `technical-director` 随后以门禁 TD-ADR 启动并返回 APPROVE
8. GDD 同步检查打印 `GDD sync: checked 1 referenced GDD(s), no naming inconsistencies.`——从不静默
9. "ADR draft is complete. May I write it?"——同意后写入 `docs/architecture/adr-0003-rendering-approach.md`，`Status: Proposed`
10. 显示注册表候选项，并单独询问 "May I update `docs/registry/architecture.yaml` with these [N] new stances?"
11. 结尾输出包含固定提示：在新会话中运行 `/architecture-review`，以及 "Run `/architecture-decision accept ADR-0003` when the decision is settled"

**断言：**
- [ ] 在任何其他步骤之前读取 `VERSION.md`，并对 HIGH 风险领域显示知识缺口警告
- [ ] 生成 ADR 前通过 `AskUserQuestion` 确认假设；Status 显示为 `Proposed`，从不询问
- [ ] 引擎专家验证在 TD-ADR 之前运行（顺序执行，非并行）
- [ ] 一次 "May I write" 批准覆盖整份 ADR；文件为 `docs/architecture/adr-0003-rendering-approach.md`
- [ ] 即使 TD-ADR 返回 APPROVE，写入的 ADR 仍为 `Status: Proposed`
- [ ] GDD 同步检查以一行报告 `visual-effects.md` 的干净结果
- [ ] 注册表更新有独立的批准提示，与 ADR 写入分开
- [ ] 结尾输出包含新会话 `/architecture-review` 提示和 `accept ADR-0003` 命令

---

### 用例 2：失败路径——TD-ADR 返回 CONCERNS

**Fixture：**
- 同用例 1，但 TD-ADR 返回 CONCERNS："Alternatives do not address the mobile renderer"
- 修订后仍有一点未解决（移动端目标渲染器）

**输入：** `/architecture-decision rendering-approach`

**预期行为：**
1. TD-ADR 以草稿、引擎版本、领域和相关 ADR 启动
2. 逐字向用户显示 CONCERNS，并附 `director-gates.md` 中的标准选项：`Revise flagged items` / `Accept and proceed` / `Discuss further`
3. 用户选择 `Revise flagged items`；仅此时才修订 Decision 或 Alternatives 章节（步骤 5.6）
4. 未解决点作为独立的 `AskUserQuestion` 呈现，选项来自审查加上 "Different approach — I'll describe it"
5. 仍会询问写入批准；在此之前不写入任何内容
6. ADR 以 `Status: Proposed` 写入

**断言：**
- [ ] TD-ADR 收到 ADR 草稿/路径、引擎版本和领域作为上下文
- [ ] 逐字显示 CONCERNS，用户选择 Revise / Accept / Discuss——在该选择前不修订草稿
- [ ] 每个未解决决策都是独立的 `AskUserQuestion`，带自由文本逃生选项
- [ ] ADR 仅在 "May I write" 批准后写入，`Status: Proposed`

---

### 用例 3：Lean 和 Solo 模式——跳过 TD-ADR；无法运行的检查会说明

**Fixture：**
- 场景 (a)：`project.yaml` 具有 `modes.review_mode: lean` 和 `engine.name: Godot`
- 场景 (b)：同一项目，以 `--review solo` 调用
- 场景 (c)：在未配置引擎的项目上以 `--review solo` 调用——`engine.name` 未设置，且 `.claude/docs/technical-preferences.md` 仍为 `[TO BE CONFIGURED]`；在步骤 1 的提示处用户不运行 `/setup-engine` 继续
- `design/gdd/` 无 GDD，因此 ADR 的 GDD Requirements Addressed 不命名任何 GDD

**输入：** (a) `/architecture-decision save-format` (b) 和 (c) `/architecture-decision save-format --review solo`

**预期行为：**
1. (a) 和 (b)：引擎专家验证（步骤 5.5）运行
2. (a) 输出注明 "TD-ADR skipped — Lean mode." 并继续 GDD 同步检查
3. (b) 和 (c)：输出注明 "TD-ADR skipped — Solo mode." 并继续 GDD 同步检查
4. (c) 不启动引擎专家，输出记录 `Engine validation: NOT ASSESSED — no engine configured (engine.name unset in project.yaml)`
5. GDD 同步检查打印 `GDD sync: NOT ASSESSED — [the ADR names no GDDs]`，而非干净的一行结果
6. 写入文件前仍会询问 "May I write it?"
7. ADR 以 `Status: Proposed` 写入

**断言：**
- [ ] lean 或 solo 模式下不启动 `technical-director`
- [ ] 跳过说明指明门禁和模式（"TD-ADR skipped — Lean mode." / "— Solo mode."）
- [ ] `--review solo` 参数对本次运行覆盖解析后的 `review_mode`
- [ ] (a) 和 (b)：引擎专家验证仍会运行
- [ ] (c) 未配置引擎时，输出说明引擎验证为 NOT ASSESSED，而非静默通过
- [ ] 未命名 GDD 时，GDD 同步检查报告 NOT ASSESSED，而非干净结果
- [ ] 写入文件前询问 "May I write"，Status 为 `Proposed`

---

### 用例 4：边界情况——提议的决策与已注册立场矛盾

**Fixture：**
- `docs/registry/architecture.yaml` 记录一项接口契约：`damage_delivery → signal pattern (ADR-0003)`
- ADR-0003 为 Accepted
- 用户在调用 skill 时、步骤 1 之前陈述提议：决策为通过直接方法调用传递伤害

**输入：** `/architecture-decision damage-pipeline`，同一消息中附该描述

**预期行为：**
1. 步骤 3a 在打开任何现有 ADR 之前读取注册表，并将相关立场作为锁定约束呈现在 "Existing Architectural Stances (must not contradict)" 下
2. 仍在步骤 3a——紧接立场之后、步骤 4 的假设提示之前——skill 暴露冲突："⚠️ Conflict: This ADR proposes [X], but ADR-0003 established that [Y]…"
3. 提供三个选项：(1) 与现有立场对齐，(2) 以显式替换取代 ADR-0003，(3) 解释为何此情况是例外
4. 冲突解决或被接受为有意例外之前，不进入步骤 4
5. 若用户选择取代：在注册表步骤将旧条目设为 `status: "superseded_by: ADR-[NNNN]"`（带引号）并添加新条目——现有条目不作其他修改

**断言：**
- [ ] 在任何现有 ADR 文件之前读取注册表
- [ ] 冲突警告指明 ADR-0003，并恰好提供上述三个选项
- [ ] 冲突解决前不出现确认/调整假设提示（步骤 4）
- [ ] 取代时，旧注册表条目标记为 `superseded_by`，不编辑或删除

---

### 用例 5：验收模式——Status 仅通过 `accept` 变为 Accepted

**Fixture：**
- `docs/architecture/adr-0005-physics-layers.md` 存在，`Status: Proposed`
- 场景 (a)：其 `## ADR Dependencies` 列出 `Depends On: ADR-0002`，且 ADR-0002 为 `Proposed`
- 场景 (b)：其唯一依赖是 ADR-0001，为 `Accepted`；`production/epics/*/story-*.md` 下三个文件命名 `ADR-0005` 且为 Blocked——两个为 `/create-stories` 写入的头部形式（`> **Status**: Blocked`），一个为 `**Status:** Blocked`
- `modes.automation` 为 `autonomous`

**输入：** `/architecture-decision accept ADR-0005`

**预期行为：**
1. skill glob `docs/architecture/adr-0005-*.md` 并读取唯一匹配
2. (a) 拒绝并指明阻塞项：ADR-0005 依赖 ADR-0002，后者仍为 Proposed；不编辑任何文件
3. (b) 询问前先找到要解除阻塞的故事，然后询问 "Accept ADR-0005 — [title]? …" 并附数量（"3 stories become Ready"），选项 `[A] Yes — accept it` / `[B] Not yet — leave it Proposed`
4. (b) 即使 automation 为 `autonomous`，提示仍会触发
5. (b) 同意后：将 `## Status` 编辑为 `Accepted`，保留现有 `## Date` 不变（仅在缺失时添加），将三个故事从 `Blocked` 移至 `Ready`，并报告 ADR 及每个移动的故事

**断言：**
- [ ] (a) 依赖未 `Accepted` 时拒绝验收，指明 ADR-0002；不写入任何内容
- [ ] (b) 确认提示说明将有多少故事变为 Ready
- [ ] (b) 确认提示在 `autonomous` automation 模式下出现
- [ ] (b) 仅在用户确认后 Status 变为 `Accepted`，三个故事移至 `Ready`
- [ ] 仅在 `production/epics/` 下匹配故事
- [ ] (b) 找到全部三个故事——Blocked grep 匹配 `> **Status**: Blocked` 和 `**Status:** Blocked`，而不仅是字面 `Status: Blocked`

---

### 用例 5b：验收模式——仅将 ADR 命名为后继的故事被排除

**Fixture：**
- `docs/architecture/adr-0005-physics-layers.md` 存在，`Status: Proposed`，其 `## ADR Dependencies` 章节唯一依赖 ADR-0001 为 `Accepted`
- `production/epics/` 下三个故事为 `> **Status**: Blocked`：
  - `story-001.md`：因 ADR-0005 为 Proposed 而阻塞
  - `story-002.md`：阻塞信息为 "BLOCKED: ADR-0003 is Superseded — point the story at ADR-0005 (edit its ADR field …) before implementing"；ADR-0005 仅作为 ADR-0003 的后继被命名
  - `story-003.md`：因无关原因阻塞，未命名 ADR-0005
- 用户回答 `[A] Yes — accept it`

**输入：** `/architecture-decision accept ADR-0005`

**预期行为：**
1. 依赖检查通过
2. Blocked 故事 grep 返回全部三个文件；仅 `story-001.md` 将 ADR-0005 命名为阻塞原因。`story-002.md` 被排除，因为它仅将 ADR-0005 命名为应指向的后继：它仍受 ADR-0003 管辖，接受 ADR-0005 不会重新指向它
3. 确认提示说 "1 story becomes Ready"，而非 2 或 3
4. 同意后：`## Status` 变为 `Accepted`；仅 `story-001.md` 从 `Blocked` 移至 `Ready`；报告列出 ADR 和该一个故事

**断言：**
- [ ] 提示中要解除阻塞的故事数为 1
- [ ] `story-002.md` 保持 `Blocked`，未编辑，且不在移动报告内
- [ ] `story-003.md` 保持 `Blocked`，未编辑
- [ ] `story-001.md` 仅在用户确认后变为 `Ready`

---

### 用例 6：Retrofit——缺失 Status 回答 `Accepted`

**Fixture：**
- `docs/architecture/adr-0004-input-mapping.md` 存在，含 Context、Decision、Consequences、Engine Compatibility 和 GDD Requirements Addressed，但无 `## Status` 和 `## ADR Dependencies`
- ADR-0001 为 `Accepted`
- `production/epics/*/story-*.md` 下一个文件命名 `ADR-