

> **[中文化] Claude-Code-Game-Studios**
>
> 此项目是 [Donchitos/Claude-Code-Game-Studios](https://github.com/Donchitos/Claude-Code-Game-Studios) 的中文翻译版本。
> - 原项目 Stars: 31393
> - 主语言: Shell
> - 许可证: MIT
> - 翻译日期: 2026-11-01
> - 原始 README: [README_en.md](README_en.md)
>
> 说明：下方 README 正文为**原项目文档的中文翻译**，力求忠于原文；本页顶部的「中文快速上手」为**面向中文用户的补充导读**（非原文内容），目的是让你不用读完整篇 README 就能先把项目跑起来。
>
> 如有翻译不准确之处，欢迎提 Issue 或 PR。

---


## 🚀 中文快速上手（补充导读，非原文）

### 这是什么 / 解决什么问题

Claude Code Game Studios 是一套面向 Claude Code 的配置模板，把单次 AI 会话组织成一支有层级的「游戏工作室」：49 个专职 agent（分导演、部门主管、专员三层）、74 个斜杠命令技能、15 个自动化校验 hook、14 条按路径生效的编码规则、62 个文档模板。它解决的是「一个人用 AI 做游戏时缺乏结构」的问题——没有设计评审、没有 QA、没人拦你写硬编码和意大利面代码。它本身不直接产出游戏，而是给会话加上流程与质量关卡（README 中专门有一节讨论「产出游戏还是文档」，但材料在此处被截断，具体结论原文未说明）。

### 快速上手（最短路径）

1. 克隆仓库到本地：`git clone https://github.com/Donchitos/Claude-Code-Game-Studios`（仓库地址按项目信息推断，原文未给出克隆命令）。
2. 在 Claude Code 中打开该项目目录。
3. 输入 `/start` 开始引导流程。
4. 用 `/setup-engine` 选择引擎（Godot 4 / Unity / Unreal Engine 5）。
5. 之后按需使用 `/brainstorm`、`/design-system`、`/create-epics`、`/create-stories`、`/dev-story`、`/story-done` 等命令推进。

原文未提供明确的安装/启动命令，请参考下方 README 正文。

### 环境要求

- **操作系统**：原文未说明（「Platform Support」一节在材料中被截断）。
- **硬件**：原文未说明。
- **依赖**：需要 Claude Code（徽章标注 "built for Claude Code"）。
- **版本要求**：原文未说明。
- **模型**：三个导演 agent 要求 Opus，`lead-programmer` 及 15 个专员指定 Sonnet，其余 30 个继承你当前会话的模型。
- **引擎**：模板覆盖 Godot 4、Unity、Unreal Engine 5 三套 agent 集，按项目选用其一。
- 国内网络环境可能需要更换镜像源或加速下载（材料未涉及具体源，仅作通用提示）。

### 常见坑与注意事项

- **模型成本**：部分 agent 明确要求 Opus / Sonnet，若你的会话模型较低，这些角色可能无法按预期工作。
- **引擎需二选一**：三套引擎 agent 集需通过 `/setup-engine` 选定匹配项目的那一套，选错会导致专员不对口。
- **它是流程模板，不是游戏生成器**：产出以设计文档、架构、故事、评审等流程产物为主，别期待一键出可玩游戏。
- **升级有专门文档**：仓库含 `UPGRADING.md`，升级前应查阅，避免覆盖自定义配置（具体升级步骤原文未说明）。

---



---

<p align="center">
  <h1 align="center">Claude Code Game Studios</h1>
  <p align="center">
    将单个 Claude Code 会话变成完整的游戏开发工作室。
    <br />
    49 个智能体。74 项技能。一支协同工作的 AI 团队。
  </p>
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="MIT License"></a>
  <a href=".claude/agents"><img src="https://img.shields.io/badge/agents-49-blueviolet" alt="49 Agents"></a>
  <a href=".claude/skills"><img src="https://img.shields.io/badge/skills-74-green" alt="74 Skills"></a>
  <a href=".claude/hooks"><img src="https://img.shields.io/badge/hooks-15-orange" alt="15 Hooks"></a>
  <a href=".claude/rules"><img src="https://img.shields.io/badge/rules-14-red" alt="14 Rules"></a>
  <a href="https://code.claude.com/docs"><img src="https://img.shields.io/badge/built%20for-Claude%20Code-f5f5f5?logo=anthropic" alt="Built for Claude Code"></a>
  <a href="https://www.buymeacoffee.com/donchitos3"><img src="https://img.shields.io/badge/Buy%20Me%20a%20Coffee-Support%20this%20project-FFDD00?logo=buymeacoffee&logoColor=black" alt="Buy Me a Coffee"></a>
  <a href="https://github.com/sponsors/Donchitos"><img src="https://img.shields.io/badge/GitHub%20Sponsors-Support%20this%20project-ea4aaa?logo=githubsponsors&logoColor=white" alt="GitHub Sponsors"></a>
</p>

---

## 为什么需要它

用 AI 单枪匹马做游戏很强大——但单个聊天会话毫无结构。没人阻止你硬编码魔法数字、跳过设计文档或写出意大利面条式代码。没有 QA 环节，没有设计评审，没人问"这真的符合游戏愿景吗？"

**Claude Code Game Studios** 通过为你的 AI 会话赋予真实工作室的结构来解决这个问题。你得到的不是单个通用助手，而是 49 个按工作室层级组织的专业智能体——守护愿景的总监、负责各自领域的部门主管，以及执行具体工作的专家。每个智能体都有明确的职责、升级路径和质量关卡。

结果：你仍然做出每一个决定，但现在你有一支团队，它会提出正确的问题、及早发现错误，并让项目从最初头脑风暴到发布始终保持有序。

---

## 目录

- [包含内容](#包含内容)
- [工作室层级](#工作室层级)
- [斜杠命令](#斜杠命令)
- [它产出游戏还是文档？](#它产出游戏还是文档)
- [快速开始](#快速开始)
- [配置](#配置)
- [升级](#升级)
- [项目结构](#项目结构)
- [工作原理](#工作原理)
- [设计理念](#设计理念)
- [自定义](#自定义)
- [平台支持](#平台支持)
- [社区](#社区)
- [支持本项目](#支持本项目)
- [许可证](#许可证)

---

## 包含内容

| 类别 | 数量 | 说明 |
|------|-------|-------------|
| **智能体** | 49 | 覆盖设计、编程、美术、音频、叙事、QA 和制作的专业子智能体 |
| **技能** | 74 | 覆盖每个工作流阶段的斜杠命令（`/start`、`/design-system`、`/create-epics`、`/create-stories`、`/dev-story`、`/story-done` 等） |
| **Hook** | 15 | 在提交、推送、资源变更、设计文档读取、写入和注册表、会话生命周期、智能体审计追踪和缺口检测时自动校验 |
| **规则** | 14 | 按路径生效的编码标准，在编辑玩法、引擎、AI、UI、网络代码、场景和资源文件等时强制执行 |
| **模板** | 62 | 用于 GDD、UX 规格、ADR、冲刺计划、HUD 设计、无障碍等的文档模板 |

## 工作室层级

智能体按三个层级组织，与真实工作室的运作方式一致：

```
第 1 层 — 总监
  creative-director    technical-director    producer

第 2 层 — 部门主管
  game-designer        lead-programmer       art-director
  audio-director       narrative-director    qa-lead
  release-manager      localization-lead

第 3 层 — 专家
  gameplay-programmer  engine-programmer     ai-programmer
  network-programmer   tools-programmer      ui-programmer
  systems-designer     level-designer        economy-designer
  technical-artist     sound-designer        writer
  world-builder        ux-designer           prototyper
  performance-analyst  devops-engineer       analytics-engineer
  security-engineer    qa-tester             accessibility-specialist
  live-ops-designer    community-manager
```

每个智能体在其 frontmatter 中声明模型：三位总监请求
Opus，`lead-programmer` 和十五位专家指定 Sonnet，其余三十位
**继承你会话的模型**。请将层级视为角色资历，而非
计费方案——工作室的大部分运行在你已在使用的模型上。

### 引擎专家

模板包含三大主流引擎的智能体集。使用与你的项目匹配的那一套：

| 引擎 | 主管智能体 | 子专家 |
|--------|-----------|-----------------|
| **Godot 4** | `godot-specialist` | GDScript、着色器、GDExtension |
| **Unity** | `unity-specialist` | DOTS/ECS、着色器/VFX、Addressables、UI Toolkit |
| **Unreal Engine 5** | `unreal-specialist` | GAS、蓝图、复制、UMG/CommonUI |

## 斜杠命令

在 Claude Code 中输入 `/` 访问全部 74 项技能：

**入门与导航**
`/start` `/help` `/project-stage-detect` `/setup-engine` `/adopt` `/settings`

**游戏设计**
`/brainstorm` `/map-systems` `/design-system` `/quick-design` `/review-all-gdds` `/propagate-design-change`

**美术与资源**
`/art-bible` `/asset-spec` `/asset-audit`

**UX 与界面设计**
`/ux-design` `/ux-review`

**架构**
`/create-architecture` `/architecture-decision` `/architecture-review` `/create-control-manifest`

**故事与冲刺**
`/create-epics` `/create-stories` `/dev-story` `/sprint-plan` `/sprint-status` `/story-readiness` `/story-done` `/estimate`

**评审与分析**
`/design-review` `/code-review` `/balance-check` `/content-audit` `/scope-check` `/perf-profile` `/tech-debt` `/gate-check` `/consistency-check` `/security-audit`

**QA 与测试**
`/qa-plan` `/smoke-check` `/soak-test` `/regression-suite` `/test-setup` `/test-helpers` `/test-evidence-review` `/test-flakiness` `/skill-test` `/skill-improve`

**制作**
`/milestone-review` `/retrospective` `/bug-report` `/bug-triage` `/reverse-document` `/playtest-report`

**发布**
`/release-checklist` `/launch-checklist` `/changelog` `/patch-notes` `/hotfix` `/day-one-patch`

**创意与内容**
`/prototype` `/vertical-slice` `/onboard` `/localize`

**团队编排**（在单个功能上协调多个智能体）
`/team-combat` `/team-narrative` `/team-ui` `/team-release` `/team-polish` `/team-audio` `/team-level` `/team-live-ops` `/team-qa`

## 它产出游戏还是文档？

这是个合理的问题，答案是一项测量而非示例项目。
四个游戏基于同一份简报在不同 `modes.rigor` 级别下构建，统计了每个游戏
**在写出第一行游戏代码之前**所写的文档数量：

| `modes.rigor` | 第一行游戏代码前的文档数 | 时间 |
|---|---:|---:|
| `minimal` | **1**（一页简报） | 0 分钟 |
| `standard` | 30 | 58 分钟 |

它们都产出了可运行、经过测试的游戏。随后两位盲审评审员和一次人工试玩
将 `standard` 构建排在**最后**——多出的 29 份文档换来的是可追溯性，而非更好的游戏。

### 它会运行游戏并观察它

上述测量关乎成本。另一半是东西是否能用，而对游戏而言这意味着必须有人
*看*它——通过的测试对画到屏幕外的菜单毫无意义。因此，任何改变玩家所见内容的故事，
在游戏被启动、观察、并将截图保存在 `production/qa/evidence/` 之前都不算完成。
解析检查不等于运行。
这在每个 rigor 级别都成立，包括 `minimal`——那里自动化测试被豁免，但观察不豁免。

游戏本身刻意**不**随附于此：这是一个供你克隆以启动自己项目的模板，
别人仓库里的游戏是杂物，不是证据。

## 快速开始

### 前置条件

- [Git](https://git-scm.com/)
- [Claude Code](https://code.claude.com/docs/en/setup) — 推荐使用原生安装程序（`npm install -g @anthropic-ai/claude-code` 也可用，需 Node.js 22+）
- [Python 3](https://www.python.org/) — 每项技能和 Hook 都会读取 `project.yaml`；没有它，每个设置都会静默回退到默认值
- Bash — 运行每个 Hook；macOS 和 Linux 内置，Windows 上随 [Git for Windows](https://git-scm.com/downloads/win) 提供
- **推荐**：[jq](https://jqlang.github.io/jq/)（用于 Hook 校验）

缺少可选工具时 Hook 会优雅降级——你只是失去校验——但有一个刻意的例外：当没有 Python 来校验时，暂存数据 JSON 的提交会被阻止。

### 设置

1. **克隆或用作模板**：
   ```bash
   git clone https://github.com/Donchitos/Claude-Code-Game-Studios.git my-game
   cd my-game
   ```

2. **在项目根文件夹中打开 Claude Code**（即包含
   `CLAUDE.md` 和 `project.yaml` 的那个）并启动会话：
   ```bash
   claude
   ```
   在 `src/` 等子文件夹中启动会跳过项目的 Hook、拒绝
   规则和先询问权限模式。

3. **运行 `/start`**——系统会询问你处于哪个阶段（毫无头绪、模糊概念、
   清晰设计、已有工作）并引导你进入正确的工作流。不做任何假设。

   或者如果你已知道自己需要什么，直接跳到特定技能：
   - `/brainstorm` — 从零探索游戏创意
   - `/setup-engine godot 4.7` — 如果你已知引擎，直接配置
   - `/project-stage-detect` — 分析现有项目

## 配置

仓库根目录的 `project.yaml` 是工作室在你的项目上如何运作的唯一事实来源——引擎、命名约定，以及一个 `modes` 块，控制每项技能强制执行多少流程。`/start` 会为你写入它；你很少需要手动编辑。

最关键的问题是 **`modes.rigor`**——`minimal`、`standard`
或 `full`。它是六个独立旋钮之上的单一入口（`workflow`——
需要哪些 GDD 章节、`docs.density`、`qa.level`、
`story_granularity`、`review_mode`——多少总监智能体评审你的工作，
以及 `team.size`），所以 `/start` 只问一次而非六次：

- **`minimal`**（默认）——游戏 Jam 速度。无需 GDD，简洁文档，最少 QA 证据，单人评审。`/start` → `/setup-engine` → `/brainstorm`（一页简报）→ `/create-stories` → `/dev-story` ↔ `/story-done`。
- **`standard`**——5 个必需 GDD 章节，均衡文档深度，标准 QA 证据，精简评审。
- **`full`**——全部 8 个 GDD 章节，详尽文档，每种故事类型都有完整 QA 证据，完整总监评审。

`minimal` 是默认值，因为更重的层级要花更多成本才能达到可运行
代码，却不产出更好的结果，且当新人接手项目时毫无回报。当项目成长时，
一次 `/settings` 调用即可提升——`/help` 和 `/gate-check` 在发现迹象时会建议你这么做。

两个逃生舱口避免单一设置成为钝器。
**`system_overrides`** 让单个系统保持比项目其余部分更高的标准——你的战斗系统得到完整待遇，而其他一切保持轻量。**`testing.strict`** 按测试类型（逻辑、集成、
视觉、UI、配置）设置缺失证据是阻止故事还是仅警告。

六个底层旋钮中的任何一个仍可单独设置，如果你想要混合
（例如 `workflow: full` 配 `docs.density: terse`——全面但紧凑）。
其中两个——`review_mode`（`full` / `lean` / `solo`）和 `team.size`——也是
**个人的**：rigor 设置团队级默认值，但你可以在 gitignored 的 `project.local.yaml` 中
按开发者覆盖它们，而无需触碰共享配置。`modes.automation`（AI 行动前暂停询问的频率）位于
rigor 之外，同样可个人覆盖。

使用 `/settings` 技能查看或更改任何设置：

```
/settings                              # 查看生效配置
/settings modes.review_mode            # 查看单个设置
/settings modes.rigor=full             # 写入 project.yaml（团队级）
/settings --local modes.review_mode=solo   # 写入 project.local.yaml（仅你）
```

完整 Schema 和每个设置的行为：`.claude/docs/effects-map.md`。

## 升级

已在使用此模板的旧版本？参见 [UPGRADING.md](UPGRADING.md)
获取分步迁移说明、版本间变更明细，以及哪些文件可安全覆盖、哪些需要手动合并。

## 项目结构

```
CLAUDE.md                           # 主配置
.claude/
  settings.json                     # Hook、权限、安全规则
  agents/                           # 49 个智能体定义（markdown + YAML frontmatter）
  skills/                           # 74 个斜杠命令（每技能一个子目录）
  hooks/                            # 18 个脚本（bash）——15 个事件 Hook、两个它们
                                    #   引用的辅助脚本（yaml-helper.sh、update-check.sh），以及一个
                                    #   需自行接入的可选诊断
  rules/                            # 14 个按路径生效的编码标准
  statusline.sh                     # 状态栏脚本（上下文百分比、模型、阶段、史诗面包屑）
  docs/
    workflow-catalog.yaml           # 7 阶段流水线定义（由 /help 读取）
    templates/                      # 62 个文档模板（+ 每章节指南）
src/                                # 游戏源代码（Godot）。代码根目录是
                                    #   引擎相关的：Unity 仅编译
                                    #   Assets/，Unreal 从 Source/<Module>/ 构建
assets/                             # 美术、音频、VFX、着色器、数据文件
design/                             # GDD、叙事文档、关卡设计
docs/                               # 技术文档和 ADR
tests/                              # 测试套件（单元、集成、性能、试玩）
tools/                              # 构建和流水线工具
prototypes/                         # 一次性原型（与 src/ 隔离）
production/                         # 冲刺计划、里程碑、发布追踪
studio-tests/                       # 技能和智能体自身的 QA——见下文
```

### 测试你的自定义

CCGS 是一个供你编辑的模板。`studio-tests/` 是你
检查编辑过或新写的技能是否仍然可靠的方式——包含全部
74 项技能和 49 个智能体的目录、按类别的质量评分标准、行为规格，以及
编写你自己规格的模板。

```
/skill-test static [name]     # 结构 linter——7 项合规检查
/skill-test category [name]   # 类别特定质量评分标准
/skill-test spec [name]       # 行为规格断言
/skill-test audit             # 技能和智能体的覆盖率报告
/skill-improve [name]         # 测试-修复-重测循环；分数变化时保留或回退
```

它测试的是*框架*，绝不是你的游戏——游戏测试位于 `tests/`。

## 工作原理

### 智能体协调

智能体遵循结构化的委托模型：

1. **垂直委托**——总监委托给主管，主管委托给专家
2. **横向咨询**——同层智能体可相互咨询，但不能做出有约束力的跨领域决定
3. **冲突解决**——分歧上报至共同父级（设计类为 `creative-director`，技术类为 `technical-director`）
4. **变更传播**——跨部门变更由 `producer` 协调
5. **领域边界**——智能体未经明确委托不修改其领域外的文件

### 协作而非自主

这**不是**自动驾驶系统。每个智能体遵循严格的协作协议：

1. **询问**——智能体在提出方案前先提问
2. **呈现选项**——智能体展示 2-4 个选项及利弊
3. **你决定**——用户始终做出决定
4. **草拟**——智能体在定稿前展示工作
5. **批准**——未经你签字确认，不会写入任何内容

你始终掌控。智能体提供结构和专业知识，而非自主权。

这就是在项目根目录启动的终端会话开箱即用的行为。`guided` 和 `autonomous` 自动化设置，以及 Claude Code 自身的
自动模式，