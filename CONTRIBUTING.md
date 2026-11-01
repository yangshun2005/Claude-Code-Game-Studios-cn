# 为 Claude Code Game Studios 做贡献

CCGS 是一个使用 Claude Code 进行独立游戏开发的协调框架。
欢迎贡献——缺陷修复、填补真实空白的新技能、Agent 改进以及 Hook 修复。不符合框架方向的 PR 将被关闭，且不作冗长解释。

## 什么样的 PR 是好 PR

- **缺陷修复**——某处坏了，这是修复方案
- **新技能**，解决尚未覆盖的工作流空白
- **改进**现有 Agent、技能或 Hook
- **文档更正**——错误信息、失效引用、过时步骤

以 PR 形式提交的功能请求将被关闭。请改为开 Issue。

**这个仓库不是什么：**
CCGS 是帮你构建游戏的系统，不是存放你用它所构建游戏的地方。GDD、ADR、PRD、游戏概念、关卡设计、叙事文档，或 CCGS 为你自己的项目生成的任何其他产出，都不会在此合并——请把它们留在你自己的仓库里。

## 不可妥协的技术规则

以下这些点一旦遗漏，你的 PR 就会被拒绝。

**技能文件**
- 技能位于 `.claude/skills/<name>/SKILL.md`——必须使用子目录格式。扁平的 `.md` 文件会被 Claude Code 静默忽略。
- SKILL.md 必须包含 YAML frontmatter：`name`、`description`、`argument-hint`、`allowed-tools` 和 `model`
- 模型层级：只读状态检查用 `haiku`，多文档综合与阶段门禁用 `opus`，其余一律用 `sonnet`

**Hook**
- 使用 `grep -E`——绝不用 `grep -P`（Perl 正则会在 Windows Git Bash 上出错）
- 为未安装 `jq` 或 `python` 的系统提供回退方案
- Hook 在每次会话启动时运行——必须快速退出，并在不适用时优雅退出（`exit 0`）

**Agent**
- 新 Agent 必须包含 **协作协议** 章节，说明该 Agent 如何提问并将决策权交给用户
- 未经用户明确授权，Agent 不得修改其文档所述领域之外的文件

**参考文档**
- 如果你的 PR 新增或更改了技能、Agent 或 Hook，请更新对应的参考文档（agent-roster、skills-reference、hooks-reference 或 rules-reference）。新增内容却不更新索引的 PR 将被退回。

## 协作原则

CCGS 不是自主系统。每个工作流都遵循：
**提问 → 选项 → 决策 → 草稿 → 批准 → 写入**

技能和 Agent 必须先问再行动。未经用户明确确认，不得写入任何文件。如果你的贡献中有 Agent 单方面做决策或写文件，它不会被合并。

## 测试你的更改

在 Claude Code 会话中运行并确认端到端可用。对于技能，调用该技能并验证输出与技能声称的功能一致。对于 Hook，触发相关事件并确认 Hook 正确触发且干净退出。

在 PR 描述中附上简短说明，描述你测试了什么以及输出是什么样的。

## 提交格式

使用 [Conventional Commits](https://www.conventionalcommits.org/)：

```
feat: add /retrospective skill for end-of-sprint reviews
fix: correct grep -P usage in session-start hook
docs: update skills-reference with new /qa-plan entry
```

类型：`feat`、`fix`、`docs`、`chore`、`refactor`、`test`

## PR 流程

- 你的 PR 会通过 CODEOWNERS 自动分配给维护者
- 评审按实际情况进行——这是一个单人维护的项目
- 如果你的 PR 开放数周没有反馈，发个提醒评论即可
- 合并的贡献者会在发布说明中署名

## 平台兼容性

CCGS 必须能在 Windows（Git Bash）、macOS 和 Linux 上运行。如果你的 Hook 或脚本使用了任何平台特定的东西，它将被拒绝。拿不准时，在 Windows 上测试。