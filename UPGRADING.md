# 升级 Claude Code Game Studios

本指南介绍如何将现有游戏项目仓库从模板的一个版本升级到下一个版本。

**确定当前版本** —— 按顺序使用以下第一个适用项：

- **克隆或合并过模板：** `git describe --tags --abbrev=0` 会打印历史中最新的发布标签，例如 `v1.1.2`。
- **`project.yaml` 中的 `framework.version`：** 该文件最后一次写入的版本。若升级时保留了你自己的 `project.yaml`，该值可能滞后；1.1 之前的项目没有 `project.yaml`。
- **`CHANGELOG.md`：** `grep -m1 '^## \[' CHANGELOG.md` 会打印其中列出的最新发布版本。只要该文件仍是模板的，这就是你的版本。

---

## 目录

- [升级策略](#升级策略)
- [v1.1.3 → v1.2.0](#v113--v120)
- [v1.1.2 → v1.1.3](#v112--v113)
- [v1.1.1 → v1.1.2](#v111--v112)
- [v1.1.0 → v1.1.1](#v110--v111)
- [v1.0 → v1.1](#v10--v11)
- [v1.0.0-beta → v1.0](#v100-beta--v10)
- [v0.4.x → v1.0](#v04x--v10)
- [v0.4.0 → v0.4.1](#v040--v041)
- [v0.3.0 → v0.4.0](#v030--v040)
- [v0.2.0 → v0.3.0](#v020--v030)
- [v0.1.0 → v0.2.0](#v010--v020)

---

## 升级策略

有四种方式获取模板更新。根据仓库的设置选择。

### 策略 A —— Git 远程合并（推荐）

适用场景：你克隆了模板并在其上添加了自己的提交 —— 仓库与模板共享历史。

```bash
# Add the template as a remote (one-time setup; skip this if `git remote -v` already lists template)
git remote add template https://github.com/Donchitos/Claude-Code-Game-Studios.git

# Fetch the new version and its release tag
git fetch --tags template main

# Merge into your branch
git merge template/main
```

Git 只会在模板*和*你都修改过的文件中标记冲突。逐个解决 —— 你的游戏内容保留，结构性改进一并带入。然后提交合并。

**提示：** 最可能冲突的文件是 `CLAUDE.md`、`project.yaml` 和 `.claude/docs/technical-preferences.md`，因为你已填入引擎和项目设置。保留你的内容，接受结构性变更。

**如果 git 报错 `fatal: refusing to merge unrelated histories`**，说明你的仓库不是从模板克隆而来（zip 下载、从零 `git init`）。**不要**用 `--allow-unrelated-histories` 强行合并 —— 没有共同祖先时，git 会把*每个*模板文件都标记为冲突，你需要手动解决数百个文件。改用策略 A2。

### 策略 A2 —— 选择性检出（无共享历史）

适用场景：仓库与模板无共同历史，但你使用 git。

> **先检查是否编辑过任何框架文件。** 下面的检出会**替换** `.claude/` 下的所有内容。*你新增的*文件会保留，但你对框架自带文件的编辑会被覆盖 —— 代理定义和总监门禁是最常被定制的。
>
> ```bash
> git log --oneline -- .claude    # commits here mean you have customisations
> ```
>
> 如果列出了初始导入之外的任何提交，请执行下面的恢复步骤，不要跳过。

```bash
# One-time setup; skip it if `git remote -v` already lists template
git remote add template https://github.com/Donchitos/Claude-Code-Game-Studios.git
git fetch template main

# Take the framework-owned paths wholesale from the new version.
# This overwrites/adds template files but never deletes files you added.
git checkout template/main -- .claude UPGRADING.md CHANGELOG.md docs/migration-guide-v1.1.md

# The checkout is STAGED, not committed — so nothing is lost yet, and this
# lists every framework file it changed:
git diff --cached --stat -- .claude

# Restore any file whose local edits you want to keep. This is one command per
# file, deliberately: each is a decision between your version and the new one.
git checkout HEAD -- .claude/docs/technical-preferences.md   # v1.0 config; v1.1 keeps config in project.yaml
git checkout HEAD -- <any other file you customised>

git status   # review what changed before committing
```

> **恢复文件会完整保留你的版本 —— 包括新版本在该文件中的改动。** 对于只做了轻微编辑的文件，通常更好的做法是采用新版本并在其上重新应用你的改动，而不是整体保留旧版。`git diff HEAD template/main -- <file>` 显示你将放弃什么。

对于 `CLAUDE.md`，不要检出 —— 手动 diff 并合并，保留你的引擎/项目内容：

```bash
git diff HEAD template/main -- CLAUDE.md
```

`project.yaml` 取决于你的来源版本，两种情况需要相反的操作：

- **从 v1.0 升级**（本节讨论的情况）：你没有 `project.yaml` —— v1.0 中不存在该文件。无需手动合并，上面的检出也刻意不获取它。改为从旧文件构建，这正是转换器的用途：

  ```bash
  bash .claude/scripts/migrate-v1-config.sh --dry-run   # preview
  bash .claude/scripts/migrate-v1-config.sh             # writes project.yaml
  ```

  如果跳过，`detect-gaps.sh` 会在下次会话启动时提醒你 —— 但请在此处完成，不要留到以后。

- **从 v1.1 或更高版本升级：** 你已有保存设置的 `project.yaml`。像 `CLAUDE.md` 一样处理 —— 手动 diff 并合并，绝不检出，否则会覆盖你自己的配置：

  ```bash
  git diff HEAD template/main -- project.yaml
  ```

---

### 策略 B —— 拣选特定提交

适用场景：你只需要某个特定功能（例如只要新技能，不要完整更新）。

```bash
# One-time setup; skip it if `git remote -v` already lists template
git remote add template https://github.com/Donchitos/Claude-Code-Game-Studios.git
git fetch template main

# Cherry-pick the specific commit(s) you want
git cherry-pick <commit-sha>
```

各版本的提交 SHA 列在下面的版本章节中。

---

### 策略 C —— 手动复制文件

适用场景：你没有用 git 设置模板（只下载了 zip）。

1. 在仓库旁下载或克隆新版本。
2. 直接复制 **"可安全覆盖"** 下列出的文件 —— 但列表中标记为你的文件除外，例如 `.claude/docs/technical-preferences.md`。
3. 对于 **"谨慎合并"** 下的文件，并排打开两个版本，手动合并结构性变更，同时保留你的内容。

---

## v1.1.3 → v1.2.0

**发布日期：** 2026-10-19
**提交范围：** `v1.1.3..v1.2.0`
**关键主题：** 故事技能围绕共享辅助脚本重建；大型设计文档的读取守卫；Godot 4.7 和 Unreal Engine 5.8 参考；技能只预批准其运行的命令

### 变更内容

| 类别 | 变更 |
|----------|---------|
| **配置辅助脚本** | `yaml-helper.sh` 从项目根目录直接执行六种只读查询（`bash .claude/hooks/yaml-helper.sh resolve_setting <key>` 等五种），`/settings` 和 `/perf-profile` 以该方式调用 |
| **Hook** | 移除 `post-compact.sh`：其输出从未到达 Claude。`pre-compact.sh` 只记录压缩。`session-stop.sh` 在后台运行且不打印任何内容。`notify.sh` 仅在权限和空闲提示时触发 |
| **新 Hook** | `validate-read.sh` 拒绝整文件读取超过 40 KB 的设计文档，并返回其标题，使重试只读取一节（`features.read_guard: off` 关闭，`features.read_guard_kb` 设置大小）。`validate-write.sh` 和 `validate-registry.sh` 在写入会违反技能声明的规则时警告但不阻止（技能从不覆盖的文件、原型边界、仅追加的注册表）。`validate-agent-artifact.sh` 在子代理停止但未写入其任务说明指定的文件时警告 |
| **更新提醒** | 有新版本时，会话启动打印一行。它每天最多在后台询问 GitHub 一次，从不等待；`project.yaml` 或 `project.local.yaml` 中的 `updates.check: false` 可关闭，CI 运行从不询问 |
| **工作树** | `settings.json` 将 `worktree.baseRef` 设为 `"head"`，使工作树（原型器的、隔离子代理的、`claude --worktree`）从你当前的提交开始，而非远程默认分支。新的 `.worktreeinclude` 将 `project.local.yaml`、`CLAUDE.local.md` 和 `.claude/settings.local.json` 复制到每个工作树。原型器会列出其工作树看不到的未提交文件，并说明哪些检查无法运行 |
| **规则** | 新增 `.claude/rules/scene-files.md`：引擎场景和资源文件（`.tscn`、`.tres`、`.unity`、`.prefab`、`Assets/**/*.asset`）尽可能在编辑器中修改 |
| **模板** | `.claude/docs/templates/` 中的新布局：缺陷报告、发布和认证检查清单、补丁说明、启动路径等。`/create-stories` 从 `story.md` 写入故事；其他技能尚未使用新布局 |
| **权限** | `settings.json`：移除三条 Bash `.env` 拒绝规则；`Read(**/.env*)` 和 `Edit(**/.env*)` 已覆盖。没有技能再预批准裸 `Bash`：34 个技能现在只预批准其文本运行的命令（框架脚本、`git log` 等）。技能未指定的命令（如你的 `commands.test`）会先询问；批准一次，或将 `Bash(<your command> *)` 添加到 `.claude/settings.local.json` |
| **Git** | `.gitignore` 忽略 `.claude/worktrees/` |
| **文档** | `settings-local-template.md` 使用当前权限模式名称和更窄的示例规则 |
| **技能测试** | `CCGS Skill Testing Framework/` 重命名为 `studio-tests/`。`/skill-test`、`/skill-improve` 及其 `catalog.yaml` 中的 `spec:` 路径使用新名称；旧文件夹仍存在时，这两个技能会说明 |
| **技能** | `/create-stories`、`/story-readiness`、`/create-epics`、`/dev-story`、`/story-done` 和 `/start` 重建：更短，少用分支移到各技能的 `references/` 文件夹（`/start` 仍为单个 `SKILL.md`），固定步骤交给 `.claude/scripts/` 中的脚本。`/dev-story` 和 `/story-done` 用更少的命名阶段文件替换 1.1.3 的编号阶段文件。`/start` 根据你的描述推断严格度，在一张卡片上显示并只问一个保存问题，而非直接询问。`/brainstorm` 的完整概念路径移到 `references/full-concept.md` |
| **故事** | 新故事模板 `.claude/docs/templates/story.md`，含 `## Files` 节：`/create-stories` 列出计划文件，`/dev-story` 用实际修改的文件替换列表，`/story-done` 据此检查范围。`/create-stories` 为每个故事提议估算（`S`、`M` 或 `L`），写入前可修改 |
| **证据** | `/dev-story` 在故事截图旁写入 `stamp.txt`。`/story-done` 列出证据捕获后发生变更的故事文件；仅报告，从不阻止 |
| **门禁** | Technical Setup → Pre-Production 门禁读取每个 `/architecture-review` 报告的 `Mode:` 行，仅按该模式检查的内容计入报告。1.1.3 写入的报告没有 `Mode:` 行，不计入，因此升级后、该门禁前运行一次 `/architecture-review` |
| **引擎参考** | `docs/engine-reference/` 现覆盖 Godot 4.7（原 4.6）和 Unreal Engine 5.8（原 5.7）。Unity 保持 6.3 LTS（最新 LTS），并记录 6.4 至 6.6 及 6.7 beta。发布时间线使用厂商来源日期 |
| **CI 模板** | `/test-setup` 写入 `actions/checkout@v7` 和 `actions/upload-artifact@v7`（原 `@v4`），指定 Godot 4.7.2 和 gdUnit4 6.2，`game-ci/unity-test-runner@v4` 现解析为 v4.4.0 |

---

### 文件：可安全覆盖

**要覆盖的现有文件（无用户内容）及要新增的文件：**
```
.claude/hooks/*.sh                           ← take the whole folder: 13 hooks
                                               changed, yaml-helper.sh among
                                               them, and 5 are new. Take
                                               .claude/scripts/ in the same
                                               step: the hooks call its helpers
.claude/scripts/**                           ← 25 new helper scripts, 7 changed
.claude/statusline.sh
.claude/skills/**                            ← if you edited a skill, see
                                               Merge Carefully
.claude/agents/*.md                          ← all 49 changed; if you edited
                                               one, see Merge Carefully
.claude/docs/**                              ← EXCEPT technical-preferences.md,
                                               which is yours and unchanged in
                                               1.2.0
.claude/rules/scene-files.md                 ← new
docs/WORKFLOW-GUIDE.md
docs/engine-reference/<engine>/              ← unless your project stays on
                                               Godot 4.6 or Unreal 5.7; see
                                               Merge Carefully
README.md, CHANGELOG.md, UPGRADING.md        ← keep your README if it is your
                                               game's
studio-tests/                                ← renamed from CCGS Skill Testing
                                               Framework/; if you wrote specs
                                               there, see Merge Carefully
production/.gdignore                         ← new: Godot skips production/
```

在 Godot 项目中，添加 `production/.gdignore` 后，删除 Godot 在 `production/` 下生成的 .import 文件（`find production -name '*.import' -delete`）：Godot 曾将那里的证据截图作为游戏资源导入，默认导出会将它们打包进游戏。

使用策略 A2 时，还要检出本版本在 `.claude` 之外变更的内容（你的 README 除外）：`git checkout template/main -- studio-tests production/.gdignore .worktreeinclude docs/WORKFLOW-GUIDE.md`。

整体采用 `.claude/hooks/`，而不只是新文件：1.1.3 的 `yaml-helper.sh` 对新查询（`resolve_setting modes.rigor` 等）返回其用法行，导致 `/settings`、`/perf-profile` 和更新通知打印的命令失效，重建的技能失去审查和门禁行。

---

### 文件：谨慎合并

**策略 A：** 合并已在 git 能处理的地方完成这些变更，即你从未编辑过的每个文件。逐项检查而非重新执行（重复添加的 hook 条目会运行两次）。以下步骤适用于冲突的文件，以及策略 A2 和 C。

**`.claude/settings.json`** —— 若未编辑，用自带文件覆盖。若已编辑：

- 删除 `hooks` 下的 `"PostCompact"` 条目，并删除 `.claude/hooks/post-compact.sh`。两者都做：指向已删除脚本的条目会在每次压缩时失败。
- 在 `"Stop"` 下，为 `session-stop.sh` hook 添加 `"async": true`。
- 在 `"Notification"` 下，将 `"matcher": ""` 改为 `"matcher": "permission_prompt|idle_prompt"`。
- 从 `deny` 中移除 `Bash(*>.env*)`、`Bash(cat *.env*)` 和 `Bash(type *.env*)`。保留 `Read(**/.env*)` 和 `Edit(**/.env*)`（覆盖 Bash 文件命令和重定向到 `.env` 文件），并保留两条 PowerShell `.env` 规则。
- 将 `Edit(**/*.uasset)` 和 `Edit(**/*.umap)` 添加到 `deny`。它们阻止 Unreal 二进制资源被作为文本编辑或覆盖（会损坏资源）；`Edit(...)` 规则也覆盖写入。
- 添加四个新 hook，从自带文件复制每个条目：在 `"PreToolUse"` 下，添加运行 `validate-registry.sh` 和 `validate-write.sh` 的 `"Write|Edit"` 条目，以及运行 `validate-read.sh` 的 `"Read"` 条目；在 `"SubagentStop"` 下，在 `log-agent-stop.sh` 之后添加 `validate-agent-artifact.sh`。先采用 `.claude/hooks/` 和 `.claude/scripts/`（可安全覆盖）：指向缺失脚本的条目每次触发都会失败，四个 hook 调用 `.claude/scripts/` 中的辅助脚本，缺少它们时每次 Read 和 Write 都会打印 "guard skipped"。
- 在顶层添加 `"worktree": { "baseRef": "head" }