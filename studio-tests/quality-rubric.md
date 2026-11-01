# 技能质量评分标准

由 `/skill-test category [name|all]` 用于在结构合规性之外评估技能。
每个类别定义 4–5 项针对该技能职责的二元 PASS/FAIL 指标。

当技能的书面指令明确满足标准时，该指标为 PASS。
当指令缺失、含糊或矛盾时，该指标为 FAIL。
当指令部分满足标准时，该指标为 WARN。
当完全无法评估时，该指标为 NOT ASSESSED——评分标准中没有该类别对应的章节，或技能/代理文件无法读取。

---

## 技能类别

### `gate`

**技能**：gate-check

Gate 技能控制阶段转换。它们必须在不自动推进阶段的前提下强制保证正确性，并且必须遵循三种审查模式。

| 指标 | PASS 标准 |
|---|---|
| **G1 — 读取审查模式** | 技能在决定生成哪些主管之前，从其解析后的配置块中读取 `review_mode`（以及 `workflow`）——`modes.review_mode`、旧版 `production/review-mode.txt`，否则取 `modes.rigor` 值 |
| **G2 — 主管小组规模** | 除非为 `solo`，PHASE-GATE 小组按 `modes.workflow` 设定的规模并行运行——`minimal` 仅 PR，`standard` 为 TD + PR，`full` 为全部 4 个（CD、TD、PR、AD）——且输出需列出较窄小组遗漏的视角 |
| **G3 — Lean 模式：仅 PHASE-GATE** | 在 `lean` 模式下，仅运行 `*-PHASE-GATE` 门；内联门（CD-PILLARS、TD-ARCHITECTURE 等）跳过 |
| **G4 — Solo 模式：无主管** | 在 `solo` 模式下，不生成任何主管门；每个均标注为“已跳过——Solo 模式” |
| **G5 — 不自动推进** | 技能绝不在未经用户通过“May I”问题明确确认的情况下写入 `project.yaml` 中的 `project.stage`，或旧版 `production/stage.txt` |

---

### `review`

**技能**：design-review、architecture-review、review-all-gdds

审查技能读取文档并生成结构化结论。它们主要只读，且不得在分析阶段触发主管门。

| 指标 | PASS 标准 |
|---|---|
| **R1 — 强制只读** | 技能未经用户明确批准不得修改被审查文档；任何写入操作（审查日志、索引更新）均须经“May I write”把关 |
| **R2 — 8 节检查** | 技能显式评估全部 8 个必需的 GDD 章节（或等效的架构章节） |
| **R3 — 正确的结论词汇** | 结论恰为以下之一：APPROVED / NEEDS REVISION / MAJOR REVISION NEEDED / NOT ASSESSED（design-review、ux-review）或 PASS / CONCERNS / FAIL / NOT ASSESSED（architecture-review、review-all-gdds） |
| **R4 — 分析期间无主管门** | 技能不在其分析阶段生成主管门；当技能的范围和风险需要时，分析后的主管审查是可接受的 |
| **R5 — 结构化发现** | 输出在最终结论之前包含逐节状态表或检查清单 |

> **例外：**
> - `design-review`：在 allowed-tools 中包含 `Write, Edit`，以支持可选的“立即修订”路径（所有写入均须经用户批准）以及写入审查日志。R1 满足，因为被审查文档绝不会被静默修改。

---

### `authoring`

**技能**：design-system、quick-design、architecture-decision、ux-design、ux-review、art-bible、create-architecture

编写技能协作创建或更新设计文档。完整 GDD/UX 编写技能使用逐节循环；轻量编写技能使用适合其较小范围的单稿模式。

| 指标 | PASS 标准 |
|---|---|
| **A1 — 逐节循环** | 完整编写技能（design-system、ux-design、art-bible）一次编写一节，在继续下一节之前呈现内容以供批准。轻量技能（quick-design、architecture-decision、create-architecture）可先起草完整文档再请求批准——对于实现范围约一周以内的文档，单稿是可接受的。 |
| **A2 — 逐节 May-I-write** | 完整编写技能在每次写入章节前询问“May I write this to [filepath]?”。轻量技能对完整文档询问一次。 |
| **A3 — 改造模式** | 技能检测目标文件是否已存在，并提供更新特定章节而非覆盖整个文档的选项。始终创建新文件的轻量技能（quick-design）豁免。 |
| **A4 — 主管门处于正确层级** | 若为此技能定义了主管门（例如 CD-GDD-ALIGN、TD-ADR），则仅在 `full` 中运行；在 `lean` 和 `solo` 中跳过并附具名说明（`[GATE-ID] skipped — Lean mode` / `— Solo mode`） |
| **A5 — 骨架优先** | 完整编写技能在填充内容之前创建包含所有章节标题的文件骨架，以在会话中断时保留进度。轻量技能豁免。 |

> **完整编写技能**（必须通过全部 5 项指标）：`design-system`、`ux-design`、`art-bible`
> **轻量编写技能**（A1、A2、A5 使用单稿模式；仅新建文件的技能 A3 豁免）：`quick-design`、`architecture-decision`、`create-architecture`
> **审查模式技能**（按审查指标评估）：`ux-review`

---

### `readiness`

**技能**：story-readiness、story-done

就绪技能在实现之前或之后验证故事。它们必须生成多维结论，并与主管门模式正确集成。

| 指标 | PASS 标准 |
|---|---|
| **RD1 — 多维检查** | 技能检查 ≥3 个独立维度（例如设计、架构、范围、DoD）并分别报告每个维度 |
| **RD2 — 四级结论** | 结论层级明确定义，NOT ASSESSED 位于通过值与失败值之间：READY > NOT ASSESSED > NEEDS WORK > BLOCKED（story-readiness）；COMPLETE > COMPLETE WITH NOTES > NOT ASSESSED > BLOCKED（story-done） |
| **RD3 — BLOCKED 需要外部操作** | BLOCKED 结论仅保留给故事作者无法单独修复的问题（例如提议中的 ADR、无法解决的依赖） |
| **RD4 — 主管门处于正确模式** | QL-STORY-READY 或 LP-CODE-REVIEW 门在 `full` 模式下生成，在 `lean`/`solo` 下跳过并附具名跳过消息 |
| **RD5 — 下一故事交接** | 完成后，技能从活动冲刺中呈现下一个 READY 故事。若无冲刺计划，`/story-done` 按构建顺序命名下一个故事，`/story-readiness` 说明未找到冲刺文件 |

---

### `pipeline`

**技能**：create-epics、create-stories、dev-story、create-control-manifest、propagate-design-change、map-systems、vertical-slice

流水线技能生成其他技能消费的构建产物。它们必须以正确的 Schema 写入文件、遵循层/优先级排序，并在写入前把关。

| 指标 | PASS 标准 |
|---|---|
| **P1 — 正确的输出 Schema** | 每个生成的文件遵循项目模板（EPIC.md、故事 frontmatter 等）；技能引用模板路径 |
| **P2 — 层/优先级排序** | 生成史诗或故事的技能遵循层排序（基础 → 核心 → 功能 → 表现）和优先级字段 |
| **P3 — 每个构建产物前 May-I-write** | 技能在创建输出文件前询问“May I write …?”——每个文件一个问题，或一次批准并列出变更集中的每个文件；绝不写入批准未列名的文件 |
| **P4 — 主管门处于正确层级** | 范围内的门（PR-EPIC、QL-STORY-READY、LP-CODE-REVIEW 等）在 `full` 中运行，在 `lean`/`solo` 中跳过并附具名跳过 |
| **P5 — 先读后写** | 技能在生成构建产物前读取相关 GDD/ADR/清单以确保对齐 |

---

### `analysis`

**技能**：consistency-check、balance-check、content-audit、code-review、tech-debt、
scope-check、estimate、perf-profile、asset-audit、security-audit、test-evidence-review、test-flakiness

分析技能扫描项目并呈现发现。它们在分析期间只读，且必须在建议任何文件写入前询问。

| 指标 | PASS 标准 |
|---|---|
| **AN1 — 只读扫描** | 分析阶段不改变任何内容：扫描本身期间无 Write 或 Edit，也无变更性 Bash（只读命令如 `git log` 或文件大小测量是可以的） |
| **AN2 — 结构化发现表** | 输出包含发现表或检查清单（不仅是散文），每项发现附严重性/优先级 |
| **AN3 — 不自动写入** | 任何建议的文件写入（例如技术债务登记、修复补丁）均须经“May I write”把关 |
| **AN4 — 分析期间无主管门** | 分析技能不生成主管门；它们生成发现供人工审查 |

---

### `team`

**技能**：team-combat、team-narrative、team-audio、team-level、team-ui、team-qa、
team-release、team-polish、team-live-ops

团队技能为某个部门编排多个专家代理。它们必须生成正确的代理、并行运行独立代理，并立即呈现阻塞。

| 指标 | PASS 标准 |
|---|---|
| **T1 — 具名代理列表** | 技能明确命名其生成的代理及顺序 |
| **T2 — 独立处并行** | 输入互不依赖的代理并行生成（单条消息、多个 `Agent` 调用） |
| **T3 — BLOCKED 呈现** | 若任何生成的代理返回 BLOCKED 或失败，技能立即呈现并停止依赖工作——绝不静默跳过 |
| **T4 — 继续前收集所有结论** | 依赖阶段等待所有并行代理完成后再继续 |
| **T5 — 无参数时用法错误** | 若缺少必需参数（例如功能名称），技能要么输出用法提示并停止，要么推断参数（例如发布版本、活动冲刺）并与用户确认——两种情况均在生成任何代理之前 |

---

### `sprint`

**技能**：sprint-plan、sprint-status、milestone-review、retrospective、changelog、patch-notes

冲刺技能读取生产状态并生成报告或规划构建产物。它们在特定模式阈值下具有 PR-SPRINT 或 PR-MILESTONE 门。

| 指标 | PASS 标准 |
|---|---|
| **SP1 — 读取冲刺/里程碑状态** | 技能在生成输出前读取 `production/sprints/` 或 `production/milestones/` |
| **SP2 — 正确的冲刺门** | PR-SPRINT（用于规划）或 PR-MILESTONE（用于里程碑审查）门在 `full` 模式下运行，在 `lean`/`solo` 中跳过 |
| **SP3 — 结构化输出** | 输出使用一致的结构（速度表、风险列表、行动项）而非自由散文 |
| **SP4 — 不自动提交** | 技能绝不在未经“May I write”的情况下写入冲刺文件或里程碑记录 |

---

### `utility`

**技能**：start、help、brainstorm、onboard、adopt、hotfix、prototype、localize、
launch-checklist、release-checklist、smoke-check、soak-test、test-setup、test-helpers、
regression-suite、qa-plan、bug-triage、bug-report、playtest-report、asset-spec、
reverse-document、project-stage-detect、setup-engine、skill-test、skill-improve、
day-one-patch，以及上述类别之外的任何其他技能

实用技能通过 7 项标准静态检查。若它们恰好生成主管门，则门模式逻辑也必须正确。

| 指标 | PASS 标准 |
|---|---|
| **U1 — 通过全部 7 项静态检查** | `/skill-test static [name]` 返回 COMPLIANT 且 0 个 FAIL |
| **U2 — 门模式正确（如适用）** | 若技能生成任何主管门，则读取审查模式并正确应用 full/lean/solo 逻辑 |

---

## 代理类别

由 `/skill-test category [agent-name]` 用于依据类别指标评估 `.claude/agents/` 中的代理定义。

没有类别固定模型。每个代理的层级是其自身 frontmatter 中的 `model:`，其在 `agents/` 中的规范断言该别名；参见 `.claude/docs/model-tiers.md`。

### `director`

**代理**：creative-director、technical-director、art-director、producer

| 指标 | PASS 标准 |
|---|---|
| **D1 — 正确的结论词汇** | 返回 `[GATE-ID]: [word]` 作为首行，该词取自所调用门的 **Verdicts** 行——或当门指定的输入未提供时返回 `NOT ASSESSED` 并命名缺失输入 |
| **D2 — 尊重领域边界** | 不在其声明领域之外做出约束性决策 |
| **D3 — 冲突升级** | 当两个部门冲突时，升级至正确的上级（creative-director 或 technical-director）而非单方面决定 |

### `lead`

**代理**：lead-programmer、qa-lead、narrative-director、audio-director、game-designer、
systems-designer、level-designer

| 指标 | PASS 标准 |
|---|---|
| **L1 — 门结论或发现** | 为门调用的 lead（lead-programmer 的 LP-、qa-lead 的 QL-、narrative-director 的 ND- 门）返回该门的 **Verdicts** 词汇——或 `NOT ASSESSED` 并命名缺失输入；无门的 lead 返回发现并将结论留给调用技能 |
| **L2 — 升级至共享上级** | 领域外冲突升级至 creative-director（设计）或 technical-director（技术） |

### `specialist`

**代理**：gameplay-programmer、ai-programmer、technical-artist、sound-designer、
engine-programmer、tools-programmer、network-programmer、security-engineer、
accessibility-specialist、ux-designer、ui-programmer、performance-analyst、prototyper、
writer、world-builder

| 指标 | PASS 标准 |
|---|---|
| **S1 — 留在领域内** | 明确将自身范围限定于其声明领域；领域外请求予以转交 |
| **S2 — 无约束性跨领域决策** | 不单方面决定属于其他专家的事项 |
| **S3 — 正确转交** | 领域外请求重定向至正确代理，而非静默拒绝 |

### `engine`

**代理**：godot-specialist、godot-gdscript-specialist、godot-csharp-specialist、
godot-shader-specialist、godot-gdextension-specialist、unity-specialist、unity-ui-specialist、
unity-shader-specialist、unity-dots-specialist、unity-addressables-specialist、
unreal-specialist、ue-blueprint-specialist、ue-gas-specialist、ue-umg-specialist、
ue-replication-specialist

| 指标 | PASS 标准 |
|---|---|
| **E1 — 版本感知** | 在建议 API 调用前引用 `docs/engine-reference/` 中的引擎版本；标记截止后风险 |
| **E2 — 文件路由** | 将文件类型路由至正确的子专家（例如 `.gdshader` → godot-shader-specialist，而非 godot-gdscript-specialist） |
| **E3 — 引擎特定模式** | 强制引擎特定惯用法（例如 GDScript 静态类型、C# 特性导出、Blueprint 函数库） |

### `qa`

**代理**：qa-tester

| 指标 | PASS 标准 |
|---|---|
| **Q1 — 产出构建产物而非代码** | 主要输出为测试用例、缺陷报告或覆盖率缺口——而非实现代码 |
| **Q2 — 证据格式** | 测试用例遵循项目的测试证据格式（按 coding-standards.md 的单元/集成/视觉/UI） |
| **Q3 — 无范围蔓延** | 不提议新功能；标记缺口供人工决定 |

### `operations`

**代理**：devops-engineer、release-manager、live-ops-designer、community-manager、
analytics-engineer、economy-designer、localization-lead

| 指标 | PASS 标准 |
|---|---|
| **O1 — 领域归属清晰** | 代理描述清楚说明其拥有什么（流水线、发布、经济等） |
| **O2 — 转交实现** | 不编写游戏逻辑或引擎代码；委托给适当的专家 |
| **O3 — 工具集匹配角色** | frontmatter 中的 `tools:` 匹配角色的运营（而非编码）性质 |