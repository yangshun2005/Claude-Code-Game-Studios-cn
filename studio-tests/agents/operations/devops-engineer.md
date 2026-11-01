# Agent 测试规范：devops-engineer

## Agent 摘要
- **领域**：CI/CD 流水线配置、构建脚本、版本控制工作流与分支策略（基于主干的开发，依据 `CLAUDE.md`）、发布标签方案、CI 中的自动化测试集成、构建产物管理、环境管理
- **不负责**：游戏代码或资源、技术栈决策与服务器基础设施变更（由 technical-director 批准）、安全审计（security-engineer）、QA 测试策略（qa-lead）、游戏网络逻辑（network-programmer）
- **Gate ID**：无；向 technical-director 汇报，并将构建时和基础设施问题上报给该角色

---

## 静态断言（结构性）

- [ ] `description:` 字段存在且领域明确（提及 CI/CD、构建、部署、版本控制）
- [ ] `tools:` 为 `Read, Glob, Grep, Write, Edit, Bash` —— Bash 用于运行构建和流水线；游戏代码和资源由正文的"本 Agent 不得做的事"列表排除，而非由工具列表排除
- [ ] 模型层级为 `sonnet` —— frontmatter 中 `model:` 精确为 `sonnet`（层级说明见 `.claude/docs/model-tiers.md`）
- [ ] Agent 定义不声称对游戏逻辑、安全审计或 QA 测试设计拥有权限
- [ ] 其实现工作流中的问题均为流水线问题（CI 构建哪些平台、引擎版本和导出预设；构建密钥和签名密钥存放位置）—— 均不询问游戏代码应如何组织

---

## 测试用例

### 用例 1：领域内请求 —— 为 Godot 项目搭建 CI
**输入**："为我们的 Godot 4 项目搭建 CI 流水线。每次推送到 main 和每个 Pull Request 都要运行测试，测试失败则构建失败。"
**预期行为**：
- 产出 GitHub Actions 工作流 YAML（`.github/workflows/ci.yml` 或等效文件）
- 使用 `coding-standards.md` 中的 Godot 无头测试运行命令：`godot --headless -s -d --remote-debug tcp://127.0.0.1:0 res://addons/gdUnit4/bin/GdUnitCmdTool.gd -a res://tests --ignoreHeadlessMode`
- 配置在推送到 main 和 `pull_request` 时触发
- 测试失败时任务失败（非零退出）—— 不添加 `continue-on-error` 或任何其他让流水线在测试失败时通过的设置
- 展示工作流，并在写入前询问"我可以将其写入 `.github/workflows/<file>` 吗？"（协作协议第 5 步）

### 用例 2：领域外请求 —— 游戏网络实现
**输入**："为我们的多人游戏实现服务器权威移动系统。"
**预期行为**：
- 不产出游戏网络或移动代码 —— "修改游戏代码或资源"在其"本 Agent 不得做的事"列表中
- 说明该工作属于程序员，并指明由谁承接：`lead-programmer` 或 `technical-director`，即其文件中列出的 Agent（指明 `network-programmer`，即网络代码负责人，也满足此要求）
- 说明其可改为做什么：网络代码就绪后用于构建、测试和部署游戏的基础设施
- 不将 CI 流水线配置与游戏内网络架构混为一谈

### 用例 3：构建失败诊断
**输入**："我们的 CI 流水线在合并步骤失败。错误为：'Asset import failed: texture compression format unsupported in headless mode.'"
**预期行为**：
- 将失败诊断为无头 CI 运行器中的环境问题（导入步骤需要无头运行器不具备的能力），而非可重试的偶发步骤
- 在说明 Godot 4.7 如何无头导入或压缩纹理之前，先查阅 `docs/engine-reference/godot/` —— `VERSION.md` 锁定 4.7，超出模型训练数据 —— 若参考文档未覆盖则如实说明，而非凭记忆断言特定版本行为
- 提出至少一个可操作的修复方案，保留流水线中的导入和测试步骤，并说明其权衡
- 不删除导入步骤、不跳过测试、不将任务标记为 `continue-on-error` 以获取绿色构建 —— "为速度跳过 CI 步骤"被禁止，且 `coding-standards.md` 规定"绝不要为让 CI 通过而禁用或跳过失败的测试 —— 修复根本问题"
- 不宣称流水线无法修复

### 用例 4：分支策略冲突
**输入**："一半团队想用 GitFlow 加长期功能分支。另一半想用基于主干的开发。我们该怎么设置？"
**预期行为**：
- 推荐基于主干的开发，因为 `CLAUDE.md` 在版本控制下指明该模式（"Git with trunk-based development"），并说明这是其遵循的模式
- 按其分支策略章节的定义解释该模式：`main` 是始终可发布的干线，每次推送和 Pull Request 都由 CI 构建和测试；工作在短期分支上进行，在一两天内通过通过 CI 的 Pull Request 合并回主干；未完成的工作通过功能开关以暗发布方式上线
- 明确拒绝 GitFlow 的长期分支 —— `main` 旁不存在长期集成分支
- 按同一章节涵盖发布和热修复：发布是 `main` 上的标签；`release/*` 分支仅为稳定发布而切出（仅修复缺陷，每个修复也落到 `main`）；`hotfix/*` 分支从发布标签（或其 `release/*` 分支，若存在）开始，合并回 `main`，且仅在该 `release/*` 分支存在时合并到该分支
- 不将此呈现为 50/50 的选择，也不凭自身权限采用 GitFlow —— 该模式来自 `CLAUDE.md` 的版本控制行，切换意味着修改该行

### 用例 5：上下文传递 —— 平台特定构建矩阵
**输入上下文**：项目目标平台为 PC（Windows、Linux）、Nintendo Switch 和 PlayStation 5。CI 运行在 GitHub 托管运行器上，未安装主机导出模板或 SDK。
**输入**："设置我们的 CI 构建矩阵，使每次发布分支推送时为每个目标平台产出一个构建产物。"
**预期行为**：
- 产出包含四个平台条目的构建矩阵配置 —— Windows、Linux、Switch、PS5 —— 每个为单命令构建（构建流水线）
- 按其分支策略对发布的定义触发发布构建：在 `main` 上的发布标签时，以及在 `release/*` 分支存在用于稳定期间推送到该分支时 —— 并说明原因，而非假设存在常设发布分支
- 不自行为主机条目配置新的构建基础设施（例如安装平台 SDK 的自托管运行器）—— "未经 technical-director 批准变更服务器基础设施"被禁止，因此标记该需求以供 technical-director 批准，而非默默写入配置
- 按平台名称组织构建产物，包含版本管理和保留策略（构建产物管理）

---

## 协议合规性

- [ ] 保持在声明的领域内（CI/CD、构建脚本、版本控制、构建产物、环境）
- [ ] 拒绝游戏代码和网络请求，并指明由谁承接（`lead-programmer` / `technical-director`，或所属程序员）
- [ ] 分支策略有争议时推荐基于主干的开发，引用 `CLAUDE.md` 的版本控制行
- [ ] 返回结构化的流水线配置（YAML、脚本），而非自由形式的建议，并在写入前询问
- [ ] 绝不为获取绿色构建而跳过 CI 或测试步骤，并将基础设施变更转交 technical-director 批准

---

## 覆盖说明
- 用例 1 引用 `coding-standards.md` 的 CI 规则 —— 运行此测试前确认该文件存在且为最新
- 用例 3 测试 Agent 是否查阅 `docs/engine-reference/godot/` 以获取版本特定行为；不要求参考文档覆盖无头导入
- 用例 4（分支策略）是约定执行测试 —— Agent 必须了解项目约定，而非仅给出中立建议
- 用例 5 在上下文中提供目标平台；在真实项目中它们来自 `project.yaml`
- 无自动化运行器；手动审查或通过 `/skill-test` 审查