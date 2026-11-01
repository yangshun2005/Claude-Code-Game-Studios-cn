# 技能测试规范：/team-release

## 技能摘要

通过 7 阶段流水线编排发布团队，从发布候选版本到部署及发布后监控。协调 release-manager、qa-lead、devops-engineer、producer、security-engineer（在线/多人/玩家数据场景必需）、network-programmer（多人场景必需）、analytics-engineer、localization-lead、performance-analyst 和 community-manager。阶段 0 解析 `review_mode`、`automation`、`team.size`、`workflow` 和 `project.stage`。`/gate-check release` 是发布前必须通过的就绪门禁，因此当 `project.stage` 不是 `Release` 时，技能在阶段 1 之前停止，不生成任何代理，提示"请先运行 `/gate-check release`"，并提供显式覆盖选项，该覆盖会记录（`Stage override: …`）在 go/no-go 记录和报告中——技能本身从不设置 `project.stage`。技能在阶段 1 之前宣布活跃集——默认的 `individual` 仅生成 release-manager。每个代理写入 `production/releases/` 下自己的命名文件（补丁说明写入 `docs/patch-notes/`）。阶段 3 代理并行运行。以 go/no-go 决策结束；若 producer 判定 NO-GO 则跳过部署（阶段 6），即使 GO 也仍需在每个自动化模式下通过显式 `AskUserQuestion` 批准。以发布后监控计划收尾。

---

## 静态断言（结构性）

- [ ] 具有必需的前置元数据字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题
- [ ] 包含裁决关键词：COMPLETE、BLOCKED
- [ ] 文件写入协议区分子代理（有界例外：命名路径、`production/`、`docs/` 或 `tests/` 下的新构建产物、门控阶段——无逐次写入提示）与子技能（正常协议——写入前询问），并要求对标签、推送、构建和商店页面变更进行显式确认
- [ ] 文件写入协议章节声明编排器不直接写入文件
- [ ] 为每个代理指定目标文件，确保没有两个并行代理共享同一文件：`production/releases/release-plan-[version].md`、`release-checklist-[version].md`、`qa-gate-[version].md`、`build-[version].md`、`security-signoff-[version].md`、`netcode-signoff-[version].md`、`localization-signoff-[version].md`、`performance-signoff-[version].md`、`analytics-signoff-[version].md`、`go-no-go-[version].md`、`announcement-[version].md`、`release-report-[version].md`，以及 `docs/patch-notes/[version].md`
- [ ] 存在错误恢复协议章节：验证命名构建产物在磁盘上、立即暴露 BLOCKED、始终生成部分报告，完整流程（暴露/评估/提供选项/部分报告）见 `.claude/docs/error-recovery-protocol.md`
- [ ] 阶段 0 解析 `project.stage`，阶段 1 之前的阶段检查在其不是 `Release` 时停止（"请先运行 `/gate-check release`"），不生成任何代理；唯一通过方式是显式 `AskUserQuestion` 覆盖，在每个 `automation` 模式下都会询问，并记录为 go/no-go 记录和最终报告中的 `Stage override:` 行
- [ ] 具有引用发布后监控和 `/retrospective` 的下一步交接；技能本身不写入 `project.stage` 或 `production/stage.txt`
- [ ] qa-lead 在发布门禁处按已解析 `workflow` 的阈值检查未关闭缺陷：S1 在所有层级，S2 和 S3 在 `full` 层级也检查
- [ ] "阶段门"是流水线自身列出的决策点，无论 `automation` 模式如何；没有任何条款说总监在阶段门生成
- [ ] 阶段 2 基于主干：发布候选版本是 `main` 上的一个提交，仅当发布需要稳定化（冻结/认证）时才切出 `release/*` 分支，且每个修复也落到 `main`
- [ ] `/patch-notes` 由 release-manager 在阶段 6 运行，community-manager 根据其输出撰写公告
- [ ] 在需要用户批准才能继续的阶段转换处使用 `AskUserQuestion`
- [ ] 阶段 0 解析 `team.size`，技能要求在阶段 1 之前进行一行活跃集宣布
- [ ] 明确声明阶段 3 代理（qa-lead、devops-engineer，以及可选的 security-engineer、network-programmer）并行运行
- [ ] 阶段 6（部署）以阶段 5 的 GO 决策为条件，且无论 `modes.automation` 如何都需显式 `AskUserQuestion` 批准
- [ ] security-engineer 被描述为以 `platform.online` / `platform.multiplayer` 和玩家数据为条件——并非总是生成，缺失键会被询问而非读作 `false`

---

## 夹具默认值

除非其自身夹具另有说明，每个用例的 `project.yaml` 都设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且无 `project.local.yaml`。规则：`studio-tests/CLAUDE.md` § 夹具默认值。

---

## 测试用例

### 用例 1：正常路径（单人）——所有阶段完成，版本已部署

**夹具：**
- 已解析配置块：`review_mode: full`、`team.size: studio`、`automation: collaborative`
- `project.yaml` 中的 `project.stage` 为 `Release`（`/gate-check release` 已通过）
- 里程碑验收标准全部满足（producer 可确认）
- `project.yaml` 设置 `platform.online: false` 和 `platform.multiplayer: false`；游戏不收集玩家数据
- 此发布不计划冻结或平台认证
- `main` 上所有 CI 构建干净
- 无未关闭的 S1/S2 缺陷；所有字符串已翻译

**输入：** `/team-release v1.0.0`

**预期行为：**
1. 阶段检查通过（`project.stage: Release`）；在任何生成之前，一行宣布 `team.size: studio` 的活跃集
2. 阶段 1：通过 Agent 生成 `producer`；确认所有里程碑验收标准满足；识别任何延期范围；写入 `production/releases/release-plan-v1.0.0.md`；AskUserQuestion：用户在阶段 2 之前批准
3. 阶段 2：通过 Agent 生成 `release-manager`；在 `main` 上提升版本号，该提交即发布候选版本；未找到 Polish 阶段的发布清单，因此调用 `/release-checklist`（`production/releases/release-checklist-v1.0.0.md`）；不切 `release/*` 分支，因为无需稳定化；AskUserQuestion：用户在阶段 3 之前批准
4. 阶段 3（并行）：同时向 `qa-lead`（回归套件、关键路径签核、`production/releases/qa-gate-v1.0.0.md`）和 `devops-engineer`（构建产物、CI 验证、`production/releases/build-v1.0.0.md`）发出 Agent 调用；security-engineer 不生成（`platform.online: false`，无玩家数据），network-programmer 不生成（`platform.multiplayer: false`），阶段 3 输出逐一命名每个跳过及其原因
5. 阶段 4：localization-lead 验证所有字符串已翻译（`production/releases/localization-signoff-v1.0.0.md`）；performance-analyst 对发布构建进行基准测试（`performance-signoff-v1.0.0.md`）；`analytics-engineer` 验证遥测在发布构建上正确触发（`analytics-signoff-v1.0.0.md`）——每个代理写入自己的文件
6. 阶段 5：通过 Agent 生成 `producer`；收集 qa-lead、release-manager、devops-engineer 和 technical-director 的签核；无未关闭的阻塞问题；producer 宣布 GO（`production/releases/go-no-go-v1.0.0.md`）
7. 阶段 6：`AskUserQuestion` "Producer 裁决为 GO。执行阶段 6——打标签、部署到预发布、部署到生产？" 选项 [A] 是，部署 / [B] 仅预发布——暂缓生产 / [C] 在此停止；用户选择 [A]
8. 阶段 6：release-manager + devops-engineer 为发布打标签，通过 `/changelog` 定稿变更日志，部署到预发布，然后生产；同时 release-manager 通过 `/patch-notes v1.0.0` 定稿补丁说明（`docs/patch-notes/v1.0.0.md`），`community-manager` 根据其撰写发布公告（`production/releases/announcement-v1.0.0.md`）
9. 阶段 7：release-manager 生成发布报告（`production/releases/release-report-v1.0.0.md`）；producer 更新里程碑跟踪；qa-lead 监控回归；community-manager 发布；analytics-engineer 确认实时仪表盘健康
10. 下一步：监控仪表盘 48 小时；若发生重大问题则 `/retrospective`
11. 裁决：COMPLETE——发布已执行并部署

**断言：**
- [ ] 在生成第一个代理之前出现命名 `team.size: studio` 的活跃集行
- [ ] 阶段 2 不切 `release/*` 分支：发布候选版本是 `main` 上的版本提升提交
- [ ] 阶段 3 qa-lead 和 devops-engineer 的 Agent 调用同时发出，而非顺序发出
- [ ] 当 `platform.online` 和 `platform.multiplayer` 为 false 且无玩家数据时，security-engineer 不生成，且阶段 3 输出按名称说明（例如 `security-engineer: not spawned — platform.online: false, platform.multiplayer: false, no player data`）——跳过从不静默
- [ ] 每个代理被告知其自己的命名输出文件——并行的阶段 3 和阶段 4 代理从不共享——编排器在将阶段视为完成之前确认文件存在
- [ ] 阶段 5 producer 在宣布 GO 之前收集 qa-lead、release-manager、devops-engineer 和 technical-director 的签核
- [ ] 阶段 6 部署仅在用户在阶段 6 `AskUserQuestion` 中选择 [A] 后才开始
- [ ] `/changelog` 由 release-manager 在阶段 6 调用（非直接写入）
- [ ] `/patch-notes v1.0.0` 由 release-manager 在阶段 6 调用，community-manager 根据其输出撰写公告——从不要求 community-manager 运行 `/patch-notes`
- [ ] 阶段 7 监控计划包含 48 小时发布后监控承诺
- [ ] 阶段检查处无停止也无覆盖问题，因为 `project.stage` 读作 `Release`；技能既不写入 `project.stage` 也不写入 `production/stage.txt`，且不虚构发布后阶段值
- [ ] 最终输出中出现裁决：COMPLETE

---

### 用例 2：Go/No-Go：NO——阶段 3 发现 S1 缺陷，跳过部署

**夹具：**
- 已解析配置块：`team.size: small`、`automation: collaborative`、`project.stage: Release`
- v0.9.0 发布候选版本是 `main` 上的一个提交
- qa-lead 在阶段 3 回归测试中发现一个此前未报告的主菜单 S1 崩溃
- devops-engineer 构建干净，产物就绪

**输入：** `/team-release v0.9.0`

**预期行为：**
1. 阶段 1–2 正常完成；识别发布候选版本提交
2. 阶段 3（并行）：devops-engineer 返回干净的构建签核；qa-lead 返回并识别出 S1 缺陷且回归套件失败；qa-lead 宣布质量门：未通过
3. 在阶段 3 → 4 转换处，编排器在对话中写入 qa-lead 的完整结果——"QA-LEAD：发现 S1 缺陷——[崩溃描述]。质量门：未通过。"——并通过 `AskUserQuestion` 捕获决策
4. 阶段 5：通过 Agent 生成 `producer`；producer 收到 qa-lead 的未通过裁决；producer 宣布 NO-GO，理由："S1 缺陷 [ID] 未关闭且未解决。发布不安全。"
5. 编排器暴露"PRODUCER：NO-GO——[理由]"并使用 `AskUserQuestion` 提供选项：修复阻塞项并重跑受影响阶段、延期发布、或有记录理由地覆盖 NO-GO
6. 阶段 6（部署）完全跳过——不打标签、不部署到预发布、不部署到生产
7. 阶段 6 的两个复制步骤都不运行——release-manager 不运行 `/patch-notes`，community-manager 不生成（无部署可公告）
8. 技能以部分报告结束，总结已完成（阶段 1–5）和已跳过（阶段 6）的内容及原因
9. 裁决：BLOCKED——发布未部署

**断言：**
- [ ] qa-lead 的 S1 缺陷发现在阶段 3 完成后立即暴露给用户——不压制到阶段 5
- [ ] producer 的 NO-GO 决策明确引用 S1 缺陷和质量门结果
- [ ] 当 producer 宣布 NO-GO 时，阶段 6 部署完全跳过
- [ ] NO-GO 时不生成补丁说明或发布公告——不运行 `/patch-notes`，不生成 community-manager
- [ ] 部分报告清楚说明哪些阶段完成、哪些跳过及原因
- [ ] 因 NO-GO 跳过部署时裁决为 BLOCKED（非 COMPLETE）
- [ ] AskUserQuestion 向用户提供解决选项（修复并重跑/延期/有理由覆盖）
- [ ] 覆盖路径（若选择）要求书面理由，并在任何阶段 6 操作之前将其作为"Override Justification"字段嵌入记录

---

### 用例 3：在线游戏安全审计——security-engineer 在阶段 3 生成

**夹具：**
- 已解析配置块：`review_mode: full`、`team.size: studio`、`automation: collaborative`、`project.stage: Release`
- `project.yaml` 设置 `platform.multiplayer: true` 和 `platform.online: true`；游戏存储玩家账户数据
- v2.1.0 的发布候选版本存在
- qa-lead 和 devops-engineer 均返回干净签核

**输入：** `/team-release v2.1.0`

**预期行为：**
1. 阶段 1–2 正常完成
2. 阶段 3（并行）：编排器从 `project.yaml` 读取 `platform.multiplayer: true` 和 `platform.online: true` 并注意到存储的玩家数据；同时向 `qa-lead`、`devops-engineer`、`security-engineer` 和 `network-programmer` 发出 Agent 调用
3. security-engineer 进行发布前安全审计：审查认证流程、反作弊存在、数据隐私合规；返回签核
4. network-programmer 验证延迟补偿、重连处理和负载下带宽；返回签核
5. 所有四个阶段 3 代理完成；其结果在阶段 4 开始前收集
6. 阶段 5：producer 在做出 go/no-go 决策前收集 qa-lead、release-manager、devops-engineer、security-engineer、network-programmer 和 technical-director 的签核
7. 其余阶段正常进行至 COMPLETE

**断言：**
- [ ] 当游戏具有在线功能、多人或玩家数据时，security-engineer 在阶段 3 生成——不跳过
- [ ] 当游戏具有多人功能时，network-programmer 在阶段 3 生成
- [ ] 所有四个阶段 3 Agent 调用（qa-lead、devops-engineer、security-engineer、network-programmer）同时发出
- [ ] security-engineer 审计覆盖认证、反作弊和数据隐私合规
- [ ] 阶段 5 producer 签核收集包括 security-engineer 和 network-programmer，与 qa-lead、release-manager、devops-engineer 和 technical-director 并列
- [ ] 阶段 6 部署在 security-engineer 签核之前不开始
- [ ] 技能不将有玩家数据的游戏的 security-engineer 视为可选

---

### 用例 4：本地化遗漏——未翻译字符串进入 go/no-go 决策

**夹具：**
- 已解析配置块：`team.size: studio`、`automation: collaborative`、`project.stage: Release`
- `project.yaml` 设置 `platform.online: false` 和 `platform.multiplayer: false`；无玩家数据
- v1.2.0 的发布候选版本存在
- 阶段 3（qa-lead、devops-engineer）完成且签核干净
- 阶段 4：localization-lead 在法语区域（受支持语言）发现 47 个未翻译字符串，并将其作为 BLOCKED/CONCERNS 项返回
- producer 判定该遗漏阻塞发布并宣布 NO-GO

**输入：** `/team-release v1.2.0`

**预期行为：**
1. 阶段 1–3 完成且签核干净
2. 阶段 4：翻译检查委托给 localization-lead（在 `studio` 层级活跃）；其签核写入 `production/releases/localization-signoff-v1.2.0.md`，编排器确认