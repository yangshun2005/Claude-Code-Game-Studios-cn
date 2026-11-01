# 技能测试规范：/code-review

## 技能摘要

`/code-review [file(s)] [story-path]` 对源文件执行架构审查。它读取目标文件和项目编码规范（阶段 1），从 `project.yaml` 解析引擎专家（阶段 2），仅读取所引用 ADR 的 `## Decision` 和 `## Consequences` 章节来检查合规性（阶段 3），然后评估六项规范检查、架构与 SOLID 以及游戏特定关注点（阶段 4–6）。适用的引擎专家——以及针对 Logic/Integration 故事的 `qa-tester`——并行启动，每个专家的发现都记录文件/行号、证据以及 VERIFIED / UNVERIFIED 置信度（阶段 7）。报告（阶段 8）以裁决结尾，首个匹配者胜出：CHANGES REQUIRED、NOT ASSESSED（无可审查内容、引擎专家审查未运行，或某个报告章节——例如对无法读取的所引用 ADR 的 ADR Compliance——读作 NOT ASSESSED）、APPROVED WITH SUGGESTIONS、APPROVED。它仅授予 Bash 中的 `git log`。阶段 9 通过 `AskUserQuestion` 提供后续步骤。该技能为只读，不调用任何主管门禁。

---

## 静态断言（结构性）

由 `/skill-test spec` 对照 SKILL.md 检查——无需夹具。

- [ ] 具有必需的前置元数据字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题
- [ ] 包含裁决关键词：APPROVED、APPROVED WITH SUGGESTIONS、CHANGES REQUIRED、NOT ASSESSED
- [ ] 不要求"May I write"措辞（只读；声明不写入任何文件）
- [ ] 具有后续步骤交接（如何处理发现）

---

## 主管门禁检查

无。代码审查在任何审查模式下都不调用主管门禁。它确实会启动引擎专家和 `qa-tester` 作为审查者（阶段 7）；这些不是门禁。

---

## 夹具默认值

除非其自身夹具另有说明，每个用例的 `project.yaml` 都设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且没有 `project.local.yaml`。规则：`studio-tests/CLAUDE.md` § 夹具默认值。

---

## 测试用例

### 用例 1：正常路径——符合规范且遵循其 ADR 的文件

**夹具：**
- `project.yaml`：`engine.name: godot`、`specialists.code: godot-gdscript-specialist`、`specialists.shader: null`、`specialists.ui: null`
- `src/gameplay/health_component.gd` 满足阶段 4 的全部六项检查：
  - 每个公共方法和类都有文档注释（`##`）
  - 每个方法的圈复杂度低于 10 且少于 40 行
  - 依赖注入；无单例
  - 所有调优值从 `assets/data/` 加载
  - 依赖接口而非具体类
- 头部注释：`# Implements ADR-0004 (docs/architecture/adr-0004-health.md)`
- `docs/architecture/adr-0004-health.md` 具有 `## Decision` 和 `## Consequences` 章节，且代码遵循所选方案
- 专家未报告任何问题；没有可建议的内容

**输入：** `/code-review src/gameplay/health_component.gd`

**预期行为：**
1. 技能读取源文件和编码规范
2. 技能在文件头部找到 `ADR-0004` 引用
3. 技能 grep ADR 的 `^## ` 标题，然后仅读取 `## Decision` 和 `## Consequences` 范围
4. 通过 `Agent` 为 `.gd` 文件启动 `godot-gdscript-specialist`；`shader: null` 和 `ui: null` 视为未设置且不启动
5. 报告显示 ADR Compliance：COMPLIANT、Standards Compliance：6/6 通过、Engine Specialist Findings：CLEAN、Testability：`N/A — no story path given`，以及 Positive Observations 章节
6. 裁决为 APPROVED
7. 阶段 9 的 `AskUserQuestion` 提供 [A] 运行 `/story-done` 和 [B] 在此停止

**断言：**
- [ ] ADR 通过标题映射和有界的 `## Decision` 与 `## Consequences` 读取来读取，而非无界完整读取
- [ ] 为 `.gd` 文件启动 `godot-gdscript-specialist`；不启动名为 `null` 的代理
- [ ] Standards Compliance 读作 `6/6 passing`，ADR Compliance 读作 COMPLIANT
- [ ] Testability 读作 `N/A — no story path given`；无故事时不启动 `qa-tester`
- [ ] Positive Observations 章节存在
- [ ] 裁决为 APPROVED，阶段 9 提供 `/story-done` 或停止
- [ ] 不写入或编辑任何文件

---

### 用例 2：需要更改——缺少文档注释和单例使用，无 ADR

**夹具：**
- `project.yaml`：`engine.name: godot`、`specialists.code: godot-gdscript-specialist`
- `src/ui/inventory_ui.gd` 具有：
  - 2 个公共方法（`refresh_slots`、`sort_items`）缺少文档注释
  - 在第 42 和 87 行使用 `GameManager.instance`
  - 其他所有规范均满足
- 文件头部无 ADR 引用，且没有触及它的提交提及 ADR
- 未传入故事路径

**输入：** `/code-review src/ui/inventory_ui.gd`

**预期行为：**
1. 技能在文件头部和 `git log --oneline -- src/ui/inventory_ui.gd` 中搜索 `ADR-NNNN` 引用，未找到
2. 输出注明："No ADR references found — ADR compliance check skipped. For full ADR compliance review, provide the story path: `/code-review [files] [story-path]`。"
3. Standards Compliance 显示 4/6 通过，并列出失败项及其行号引用：两个未文档化的方法和第 42、87 行的单例
4. Required Changes 列出缺少的文档注释和单例，修复方式为依赖注入
5. 裁决为 CHANGES REQUIRED
6. 阶段 9 提供 [A] 修复问题并重新运行 `/code-review`、[B] 带注明例外仍运行 `/story-done`、[C] 在此停止

**断言：**
- [ ] "No ADR references found" 注释指明命令的 story-path 形式
- [ ] 缺少的文档注释以方法名（`refresh_slots`、`sort_items`）列出，而不仅是行号引用
- [ ] 单例使用以文件和行号（42、87）标记
- [ ] Standards Compliance 读作 `4/6 passing`
- [ ] 裁决为 CHANGES REQUIRED，阶段 9 提供三个 CHANGES REQUIRED 选项
- [ ] 技能不编辑文件

---

### 用例 3：架构违规——代码使用了其 ADR 拒绝的模式

**夹具：**
- `project.yaml`：`engine.name: godot`、`specialists.code: godot-gdscript-specialist`
- `src/core/save_system.gd` 头部：`# Implements ADR-0010 (docs/architecture/adr-0010-save-format.md)`
- `adr-0010-save-format.md` 的 `## Decision`：存档数据通过 `SaveService` 以 JSON 形式处理；明确拒绝游戏代码直接使用 `FileAccess` 写入
- `save_system.gd` 在第 58 行直接使用 `FileAccess.open(..., FileAccess.WRITE)` 写入存档数据
- 代码在其他方面满足全部六项规范检查

**输入：** `/code-review src/core/save_system.gd`

**预期行为：**
1. 技能仅读取 ADR-0010 的 Decision 和 Consequences 章节
2. 直接 `FileAccess` 写入被归类为 ARCHITECTURAL VIOLATION（BLOCKING）——ADR 明确拒绝的模式
3. ADR Compliance 读作 VIOLATION，列出 ADR-0010、结果及偏差及其严重程度
4. 违规出现在 Required Changes 下
5. 裁决为 CHANGES REQUIRED
6. 后续步骤：修复实现以符合 `docs/architecture/adr-0010-save-format.md`，或者——如果设计已合理变更——运行 `/architecture-decision` 修订现有 ADR，而非创建竞争性 ADR

**断言：**
- [ ] 偏差归类为 ARCHITECTURAL VIOLATION（BLOCKING），而非 ADR DRIFT 或 MINOR DEVIATION
- [ ] ADR Compliance 读作 VIOLATION 并指明 ADR-0010
- [ ] 违规列在 Required Changes 下
- [ ] 裁决为 CHANGES REQUIRED
- [ ] 输出说明应遵循现有 ADR 或通过 `/architecture-decision` 修订它，绝不编写竞争性 ADR

---

### 用例 4：边缘情况——指定路径下无源文件

**夹具：**
- `project.yaml`：`engine.name: godot`
- `src/networking/` 不存在

**输入：** `/code-review src/networking/`

**预期行为：**
1. 技能将其输入记录为 FOUND / ABSENT；目标源文件为 ABSENT
2. 每个必需输入均为 ABSENT，因此技能在生成审查章节前停止
3. 输出报告 `NOT ASSESSED — NO DATA` 作为整体裁决，指明 `src/networking/` 缺失
4. 不启动任何专家或 `qa-tester` 代理
5. 不填写任何审查章节，结果不呈现为 APPROVED

**断言：**
- [ ] 路径不存在时技能不崩溃
- [ ] 输出指明尝试的路径 `src/networking/`
- [ ] 裁决为 `NOT ASSESSED — NO DATA`，而非 APPROVED
- [ ] 无可审查内容时不启动任何专家代理

---

### 用例 5：未配置引擎——跳过的专家自我声明；无门禁

**夹具：**
- `project.yaml` 具有 `modes.review_mode: full` 且无 `engine.name`；`.claude/docs/technical-preferences.md` 读作 `[TO BE CONFIGURED]`
- `src/gameplay/loot_system.gd` 在第 31 行硬编码掉落率 `0.05`；其他一切满足规范

**输入：** `/code-review src/gameplay/loot_system.gd`

**预期行为：**
1. 阶段 2 发现未配置引擎，跳过引擎专家步骤
2. 输出记录 `Engine validation: NOT ASSESSED — no engine configured (engine.name unset in project.yaml)`
3. Engine Specialist Findings 读作 `N/A — no engine configured`；不启动任何引擎专家
4. Standards Compliance 列出"Configuration values loaded from data files"失败项及第 31 行，修复列在 Required Changes 下
5. 裁决为 CHANGES REQUIRED——已知缺陷优先于跳过的引擎审查，报告仍会指明后者
6. 任何审查模式下都不调用主管门禁
7. 不编辑任何文件

**断言：**
- [ ] 输出包含 `Engine validation: NOT ASSESSED — no engine configured` 行
- [ ] Engine Specialist Findings 读作 `N/A — no engine configured`，且不启动任何引擎专家代理
- [ ] 硬编码值在 Standards Compliance 下报告并附行号引用
- [ ] 裁决为 CHANGES REQUIRED，而非 NOT ASSESSED，且绝不 APPROVED
- [ ] 即使审查模式为 `full`，也不调用主管门禁
- [ ] 不进行代码编辑

---

### 用例 5b：未配置引擎，干净文件——NOT ASSESSED，绝不 APPROVED

**夹具：**
- `project.yaml` 无 `engine.name`；`.claude/docs/technical-preferences.md` 读作 `[TO BE CONFIGURED]`
- `src/gameplay/loot_table.gd` 满足阶段 4 的全部六项检查，且未引发任何架构、SOLID 或游戏特定关注点

**输入：** `/code-review src/gameplay/loot_table.gd`

**预期行为：**
1. 阶段 2 记录 `Engine validation: NOT ASSESSED — no engine configured`；不启动任何引擎专家
2. Required Changes 或 Suggestions 下无任何内容
3. 裁决为 NOT ASSESSED，指明未运行的引擎专家审查
4. 阶段 9 提供 NOT ASSESSED 选项，指明 `/setup-engine` 为使审查完整的方式

**断言：**
- [ ] 裁决为 NOT ASSESSED，而非 APPROVED——跳过引擎检查的审查未批准代码
- [ ] 裁决指明引擎专家审查为未运行的内容
- [ ] 阶段 9 不提供 `/story-done`；它指明 `/setup-engine`

### 用例 6：所引用 ADR 缺失——该章节的 NOT ASSESSED 影响裁决

**夹具：**
- `project.yaml`：`engine.name: godot`、`specialists.code: godot-gdscript-specialist`
- `src/gameplay/stamina.gd` 满足阶段 4 的全部六项检查，且未引发任何
  架构、SOLID 或游戏特定关注点；专家报告其干净
- 其头部读作 `# Implements ADR-0008 (docs/architecture/adr-0008-stamina.md)`，
  且该文件不存在

**输入：** `/code-review src/gameplay/stamina.gd`

**预期行为：**
1. 阶段 3 找到 `ADR-0008` 引用但无法读取 ADR
2. ADR Compliance 读作 `NOT ASSESSED — ADR-0008 could not be read`
3. Required Changes 或 Suggestions 下无任何内容
4. 裁决为 NOT ASSESSED，指明 ADR Compliance 章节和 ADR-0008

**断言：**
- [ ] ADR Compliance 读作 NOT ASSESSED 并指明 ADR-0008——而非 `NO ADRS FOUND`，而非 COMPLIANT
- [ ] 裁决为 NOT ASSESSED，绝不 APPROVED：无法评估的章节影响裁决
- [ ] 变体——文件完全未指明 ADR：ADR Compliance 读作 `NO ADRS FOUND`，裁决为 APPROVED（缺失引用不同于无法读取的引用）

---

## 协议合规性

- [ ] 审查前读取目标文件和编码规范（阶段 1）
- [ ] 从 `project.yaml` 读取 `specialists` 块，回退到 `technical-preferences.md`；`null` 专家被跳过，绝不启动
- [ ] 报告中的每个专家发现都带有文件/行号、证据以及 VERIFIED 或 `UNVERIFIED — specialist claim`
- [ ] 不编辑任何源文件（只读技能）
- [ ] 不调用任何主管门禁
- [ ] 裁决，首个匹配：CHANGES REQUIRED、NOT ASSESSED、APPROVED WITH SUGGESTIONS、APPROVED——当引擎专家审查未运行或任何报告章节读作 NOT ASSESSED 时，绝不批准
- [ ] `allowed-tools` 授予 `Bash(git log *)` 和限定范围的 `yaml-helper.sh resolve_config` 模式，而非裸 `Bash`——该技能为只读：它运行 `git log` 和配置解析器，别无其他
- [ ] 阶段 9 的提示和选项分支使用与阶段 8 报告相同的裁决词汇

---

## 覆盖说明

- 目录的批量审查未单独测试；相同的检查逐文件适用。
- `qa-tester` 可测试性审查（Logic 和 Integration 故事，故事路径作为最后一个参数传入）未单独设用例。
- 测试文件存在性检查属于 `/test-evidence-review` 的范畴。