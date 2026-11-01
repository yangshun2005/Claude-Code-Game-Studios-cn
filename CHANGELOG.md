# 更新日志

Claude Code Game Studios 的新变化，面向使用它的人编写。

分步升级说明见 [UPGRADING.md](UPGRADING.md)；如果你的现有项目仍在使用旧配置文件，见 [docs/migration-guide-v1.1.md](docs/migration-guide-v1.1.md)。

---

## [1.2.0] — 2026-10-19

**故事技能围绕共享辅助脚本重建，读取守卫阻止整文件读取大型设计文档，引擎参考升级到 Godot 4.7 和 Unreal Engine 5.8，且不再有技能预先批准所有 shell 命令。** 这是一个次要版本。[UPGRADING.md](UPGRADING.md#v113--v120) 列出了 `settings.json` 的合并项、新增文件以及重命名的 `studio-tests/` 文件夹。

### 新增

- **会话启动时会提示有新版本发布**，一行显示发布页面和关闭提示的方法；不会下载或更改任何内容。该 Hook 只读取项目外的每用户缓存。后台任务每天最多向 GitHub 查询一次最新发布来刷新缓存，因此会话启动从不等待，也不会发送任何项目信息。用 `/settings --local updates.check=false` 关闭，或在 `project.yaml` 中用 `updates.check: false` 为团队关闭；设置了 `CI` 时始终关闭。
- **大型设计文档的读取守卫。** 新的 `validate-read.sh` Hook 拒绝整文件读取 `design/`、`docs/architecture/` 或 `production/session-state/` 下超过 40 KB 的文档，并返回其按行排列的标题，以便重试时只读一节；范围读取始终通过。协调规则 6 和大多数读取 GDD、ADR 或美术圣经的技能现在按节读取。`features.read_guard: off`（或 `false`）可关闭，`features.read_guard_kb` 设置大小阈值（#136、#140，由 @mamonbleau 建议）。
- **证据知道其代码何时变更。** `/dev-story` 在故事保留的截图旁写入 `stamp.txt`，记录它们证明的文件。`/story-done` 和 `/test-evidence-review` 会列出其中自那以后变更的文件（"重新截图"），或对旧证据显示"无标记"。它们只报告，从不因此阻塞（#135，由 @jiangshoujie 提议）。
- **故事列出其文件。** 新的故事模板 `.claude/docs/templates/story.md` 包含 `## Files` 节：`/create-stories` 列出计划文件，`/dev-story` 用其更改的文件替换，`/story-done` 对照列表检查范围。其范围检查过去查找一个没有故事拥有的列表，因此无法运行。
- **辅助脚本回答技能手工推算的问题。** `.claude/scripts/` 中的新脚本读取故事事实、ADR 状态、Markdown 节到下一个标题、项目阶段、会话检查点和配置写入。重建的技能和 `/settings` 使用它们，因此同一问题每次得到相同答案，长节不会被固定大小的 grep 截断。还有更多脚本随附供你运行：bug 计数、代码标记、冲刺算术、技术债评分、不稳定测试率、资产检查、内容计数、测试运行和测试脚手架。每个脚本输出观察结果而非结论，并有 `--help`。除需要 Python 3 的 `agent-check.sh` 和 `compat-inventory.sh` 外，其余只需 bash 和 awk。`resolve_config` 还为技能作者回答另外六个问题（`gate_level`、`engine_values`、`specialists`、`review_gate`、`input_methods`、`perf_budgets`）。
- **注册表检查。** `registry.sh` 列出、查找并验证三个注册表，并发放下一个 ID 为历史最高值加一，因此退役的 ID 永不复用。它读取 1.1.3 的 `/architecture-decision` 写入的未加引号的 `status: superseded_by:` 行，并逐一命名以供引用。新的 `validate-registry.sh` Hook 在写入实体或 TR-ID 注册表会删除或重新编号条目、或设置头部未列出的状态时发出警告，写入注册表的技能随后会验证它。
- **只警告、从不阻塞的写入规则。** `validate-write.sh` 标记会替换技能从不覆盖的文件（现有 `EPIC.md`、`project.godot`、引擎参考文档）、设置列表外的配置值、越过原型边界或破坏本地化冻结的写入。`validate-agent-artifact.sh` 在子代理停止时未写入其简报指定的文件时告知你。
- **`/sprint-plan` 检查冲刺目标可被展示。** 它列出展示目标所需的内容（运行所在的场景、启动它的设置、所选故事提到但无人构建的任何内容），并在新的 Goal Prerequisites 节和 Risks 中报告任何缺口，不阻塞（基于 #76 中 @FreedomPortal 的想法）。
- **引擎场景和资产文件的规则**，错误的手工编辑可能损坏它们。新的路径作用域 `.claude/rules/scene-files.md` 涵盖 `.tscn`、`.tres`、`.unity`、`.prefab` 和 `Assets/**/*.asset`：尽可能使用编辑器，绝不虚构 Godot `uid://`，在编辑器可达时绝不手工编辑 Unity YAML。`settings.json` 拒绝 `Edit(**/*.uasset)` 和 `Edit(**/*.umap)`，这也涵盖写入。
- **代理从不向上游提交。** 协调规则 7：任何代理都不在引擎、插件或供应商仓库中提交、开启或评论 Issue 或 Pull Request，即使被要求。它写草稿供你提交，`/bug-report` 说明它只起草。
- **`/skill-test static` 检查代理和仅 Claude Code 的功能**：描述过于接近以致选错代理、代理被要求询问用户、Claude Code 忽略的设置，以及技能、代理、Hook 或设置依赖仅 Claude Code 提供之物的每一处。这些从不改变结论。`/skill-test` 和 `/skill-improve` 现在说明通过证明什么：它们读取文件，从不运行技能。
- **`.claude/docs/templates/` 中的新模板**，用于技能写入的布局，从 bug 报告和认证清单到 `/help` 输出和 `sprint-status.yaml`。`/create-stories` 从 `story.md` 写入；其他技能尚未使用新布局。GDD 模板现在正是 `/design-system` 写入的骨架，其指导移至 `guidance/game-design-document-guide.md`。

### 变更

- **六个技能按 Anthropic 的技能编写指南重建。** `/create-stories`、`/story-readiness`、`/create-epics`、`/dev-story`、`/story-done` 和 `/start` 更短：主路径在 `SKILL.md`，罕见分支在 `references/`（`/start` 是单个 `SKILL.md`），固定步骤在辅助脚本中。`/dev-story` 和 `/story-done` 用更少、命名的文件替换 1.1.3 的编号阶段文件，`/brainstorm` 仅在 `standard` 和 `full` 时加载其完整概念流程。
  - `/start` 根据你的描述确定严谨度，并与阶段一起显示在一张卡片上，只有一个"保存 / 不保存"问题。
  - `/create-stories` 在起草前报告每个管辖 ADR 的状态，并阻塞其 ADR 依赖于尚未 Accepted 的 ADR 的故事；`/create-epics` 标记唯一 ADR 为 Proposed 的需求，因此该阻塞仍然有效。
  - `/dev-story` 将每个代码故事的更改文件交给引擎专家，其简报排除一次性辅助函数、对不可能发生的错误的处理，以及为使测试通过而特判的值。
  - `/story-done` 的报告带有**Reviewed by** 行，永不为空：`lead-programmer`、`/code-review (you said you ran it; not verified here)` 或 `self, no independent review`（#135）。
- **引擎参考升级到 Godot 4.7 和 Unreal Engine 5.8。** Godot 文档涵盖 4.7（最新补丁 4.7.2）：AreaLight3D、HDR 输出、Asset Store、VirtualJoystick、Jolt 变更和类型化返回覆盖。Unreal 5.8（最新热修复 5.8.3）记录在破坏性变更、已弃用 API、模块和插件中。Unity 保持 6.3 LTS，即最新 LTS，并将 6.4 至 6.6 和 6.7 beta 列为 API 未确认的更新版本。时间线带有供应商来源的日期；Unreal 的对象池示例和 GAS 标签调用已更正。当 5.8 编辑器与 Epic 的发行说明不一致时，Unreal 参考会说明：MegaLights 的工具提示仍称其为 Experimental，并说需要硬件光线追踪且跳过方向光，Iris 插件仍带有 Beta 标志。仍在使用 Godot 4.6 或 Unreal 5.7？保留你当前的引擎文件夹。
- **CI 模板。** `/test-setup` 写入 `actions/checkout@v7` 和 `actions/upload-artifact@v7`（自托管运行器需要 2.327.1 或更新），并指定 Godot 4.7.2 和 gdUnit4 6.2。在 Godot 4.7 上，`/setup-engine` 在新项目首次导入以 139 退出时再次运行导入，这是导入完成后退出时的崩溃。
- **工作树从你当前的提交开始，并获得你的本地配置。** `settings.json` 设置 `worktree.baseRef: "head"`，因此工作树（原型设计者的、隔离子代理的、`claude --worktree`）从你的提交开始，而非远程的默认分支。新的 `.worktreeinclude` 将 `project.local.yaml`、`CLAUDE.local.md` 和 `.claude/settings.local.json` 复制进去，`.gitignore` 忽略 `.claude/worktrees/`。原型设计者命名其工作树和分支，列出它看不到的未提交文件并请你提交它需要的文件，在项目使用 LFS 时拉取 Git LFS 文件，并说明它无法运行哪些检查。
- **无作用的 Hook 停止运行。** `post-compact.sh` 已移除：Claude Code 丢弃其输出，而 `session-start.sh` 已恢复上下文。Stop Hook 在后台运行，因此不再阻塞回合，`notify.sh` 仅对权限和空闲提示触发。
- **代理以问题结束回合。** 49 个代理被要求"等待 yes"，而由技能启动的代理无法做到；它们现在以草稿和问题结束回合，由调用者向你提出。`/balance-check`、`/tech-debt` 和 `/bug-report` 作为子代理运行时也如此。
- **`/settings` 拒绝 `true` 或 `false` 之外的裸 `testing.strict`** 而非写入，并建议最接近的已知键，在写入未知键前询问（#135）。
- **`/prototype` 在所有地方上限为一天**；`/team-release` 可启动 `/release-checklist`；`/launch-checklist` 仅在你输入时运行。
- **技能和代理规范文件夹现为 `studio-tests/`**，从 `CCGS Skill Testing Framework/` 重命名。每个技能规范固定其用例假定的设置，`/settings` 有一个规范。UPGRADING 说明如何移动你自己的规范；`/skill-test` 和 `/skill-improve` 在旧文件夹仍存在时说明。
- **`/create-stories` 提议估算。** 每个故事获得一个大小（`S`、`M` 或 `L`），你可在故事写入前更改。`/story-readiness` 在 `standard` 和 `full` 时仍使无估算的故事失败，而新故事现在有估算。
- **`yaml-helper.sh` 直接回答六个只读查找**，从项目根运行（`bash .claude/hooks/yaml-helper.sh resolve_setting <key>` 及另外五个）；`/settings` 和 `/perf-profile` 使用它们。
- **`.gitattributes` 在每次检出时保持 awk 程序和 `.worktreeinclude` 为 LF**，包括 Windows 上的 `core.autocrlf=true`。

### 修复

- **`project.yaml` 中的重复键被合并并命名。** 第二个 `modes:` 块在 schema 检查通过时抹掉了第一个。块现在被合并，会话启动、`/settings` 和 `resolve_config` 会命名重复的键。
- **保存的评审仅在其读取的内容均未变更时复用。** `/architecture-review` 在引擎升级或 TR 注册表、`architecture.md` 或预算变更后复用了上次报告。它现在检查它读取的每个文件，命名被删除的 GDD 或 ADR，且从不依赖更窄的报告。`/design-review` 记录其模式和层级。
- **`/design-review` 检查注册表值出现的每一处**，因此长 GDD 表格中重述的与注册表矛盾的值（敌人的生命值、掉落率）会成为带行号的阻塞项。
- **`/story-readiness` 要求与 `story_granularity` 写入的一样多的验收标准**（coarse 3/2/1、balanced 2/2/1、fine 1），而非对以 `fine` 或 `balanced` 写入的 Logic 故事要求三个。
- **需求 ID 和 ADR。** `/create-architecture` 保留已注册的 TR-ID 并将新的标记为临时，因此重跑不再重新编号故事引用的内容。`/architecture-review` 现在可以 Edit，因此其对 `tr-registry.yaml` 的追加不再重写整个文件。`/architecture-decision` 写入 ADR Layer 行；没有它，每个 ADR 都被计为关键，在 `standard` 时将建议项变为阻塞项。被取代的状态按 YAML 要求加引号写入。
- **提交检查**跳过 `design/gdd/` 子文件夹（如 `archive/`）中的 GDD 副本（#133），检查提交所取的副本，并读取带反斜杠的 PowerShell `git add` 路径。`gdd-structure-check.sh` 将仅含 `[To be designed]` 的节命名为占位符。
- **会话启动。** 杂散的 `.claude/.claude/` 不再使 Hook 写入 `.claude/production/`，指向 `.claude/` 本身的 `CLAUDE_PROJECT_DIR` 也不会。检查点预览会说明它是在另一分支上写入的。"Open bugs"只计未 Closed 或 Verified Fixed 的 bug。稀疏设计文档通知只计 `design/gdd/` 顶层的 GDD，与提交检查一致，并命名它跳过的子文件夹（如 `archive/`）中的副本。
- **轮换 `active.md` 保留活动检查点**；过去当文件来自模板时，它会将其移入会话日志。
- **缺失的先决条件命名创建它们的技能**：无故事的 `/dev-story` 和 `/story-done` 指向 `/create-stories`，`/create-epics` 在 `standard` 和 `full` 时无 `architecture.md` 则停止。
- **`/adopt` 的"帮我找到它们"选项可用**：它按标题搜索，显示每个文档应属的位置，并在用 `git mv` 移动前询问。在 `workflow: minimal` 时，代码已包含游戏的项目被送往 `/reverse-document design [code root]`，而非 `/brainstorm`。
- **`/estimate` 测量它要求的大小和分支**，来自报告为近似值的 Grep 计数。
- **部分检查说明它跳过了什么。** 聚焦的 `/review-all-gdds` 标记其结论（"仅一致性"），在 Technical Setup 门不计为完整评审；`/smoke-check quick` 说明未检查覆盖率，Polish 和 Release 门不计它。Pre-Production 门仅按其模式检查的内容计 `/architecture-review` 报告：`single-gdd` 或 `consistency` 报告不再通过 Foundation-gap 和引擎审计项，1.2.0 之前无 `Mode:` 行的报告不计任何项。
- **阶段从不越过无法运行的检查。** 有疑虑且必需检查未运行的关卡返回 CONCERNS，接受疑虑会移动阶段。`/gate-check` 现在在该检查运行前不提供覆盖，其报告会命名它。
- **`/sprint-plan` 在写入计划前询问**，即使对其 QA 计划问题已答"是，继续"，过去会将其视为写入批准。
- **Godot 不再导入 `production/`。** 在 Godot 项目中，`production/qa/evidence/` 中的证据截图及 Movie Maker 在旁边写入的 `.wav` 被作为游戏资源导入并带 `.import` 文件，默认导出将它们打包进游戏。新的 `production/.gdignore` 使 Godot 跳过该文件夹。
- **试玩记录计为测试证据。** `/story-done`、`/test-evidence-review` 和 `/qa-plan` 过去在 `production/session-logs/` 中查找它们，而该目录被 gitignore。它们现在读取 `production/qa/playtests/`，`/playtest-report` 在那里写入。
- **`/help` 和 `/gate-check` 能发现故事缺失。** 两个 `EPIC.md` 文件被计为 Create Stories 步骤所需的两个故事文件，因此该步骤在任何故事存在前就显示为完成。
- **ADR 循环仅命名循环上的 ADR。** `adr-dep-graph.sh` 还列出了依赖循环的每个 ADR，因此建议是切断合法依赖。它现在还会打印构建顺序，并命名仍含模板文本的 Depends On 行，而非将其读为无依赖。
- **`/architecture-review single-gdd` 仅写入该系统。** 其"写入全部三个文件"用单个系统的行替换了项目级可追溯性索引，并可能提议退役其他系统的需求。无 ADR 的评审现在报告 NOT ASSESSED