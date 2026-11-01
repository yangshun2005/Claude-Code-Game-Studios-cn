# Studio Tests

**Claude Code Game Studios** 框架的质量保障基础设施。
测试的是技能和智能体本身——而非用它们构建的任何游戏。

**它随模板一起发布，且本就该如此。** CCGS 是一个你应当定制的模板——修改技能、添加自己的技能、调整智能体。这个文件夹用来检查你所做的改动是否仍然站得住脚：`catalog.yaml` 跟踪全部 74 个技能和 49 个智能体，`quality-rubric.md` 定义各类别的通过/失败指标，`templates/` 提供你新写内容的规范格式。由 `/skill-test` 和 `/skill-improve` 驱动。

> **若删除它，`/skill-test` 四种模式中的两种将失效。**
>
> | 模式 | 没有此文件夹时 |
> |------|---------------------|
> | `/skill-test static` | **正常** — 7 项结构检查只读取技能自身的文件 |
> | `/skill-test audit` | **降级** — 报告不存在 catalog |
> | `/skill-test spec` | **失效** — 行为规范存放于此 |
> | `/skill-test category` | **失效** — 从此处读取 `quality-rubric.md` |
> | `/skill-improve` | **降级** — 其测试-修复-重测循环失去类别通过判定 |
>
> `/skill-test` 和 `/skill-improve` 按路径读取此文件夹，因此删除它是实实在在的代价，而非免费的清理——上表列出了实际成本。

---

## 内容概览

```
studio-tests/
├── README.md              ← 你在这里
├── CLAUDE.md              ← 告诉 Claude 如何使用此框架
├── catalog.yaml           ← 主注册表：全部 74 个技能 + 49 个智能体，覆盖率跟踪
├── quality-rubric.md      ← 供 /skill-test category 使用的类别专属通过/失败指标
│
├── skills/                ← 全部 74 个技能的行为规范文件
│   ├── gate/              ← gate 类别规范
│   ├── review/            ← review 类别规范
│   ├── authoring/         ← authoring 类别规范
│   ├── readiness/         ← readiness 类别规范
│   ├── pipeline/          ← pipeline 类别规范
│   ├── analysis/          ← analysis 类别规范
│   ├── team/              ← team 类别规范
│   ├── sprint/            ← sprint 类别规范
│   └── utility/           ← utility 类别规范
│
├── agents/                ← 智能体的行为规范文件（每个智能体一个）
│   ├── directors/         ← creative-director、technical-director、producer、art-director
│   ├── leads/             ← lead-programmer、narrative-director、audio-director、game-designer、systems-designer、level-designer、qa-lead
│   ├── specialists/       ← gameplay/engine/network/AI/tools/UI 程序员、technical-artist、sound-designer、ux-designer、performance-analyst、prototyper、writer、world-builder
│   ├── engine/            ← 引擎专属专家，按引擎拆分：
│   │   ├── godot/         ← Godot 专属专家
│   │   ├── unity/         ← Unity 专属专家
│   │   └── unreal/        ← Unreal 专属专家
│   ├── operations/        ← devops、release、live-ops、community、analytics、economy、localization
│   └── qa/                ← qa-tester、security-engineer、accessibility-specialist
│
├── templates/             ← 用于编写新规范的规范文件模板
│   ├── skill-test-spec.md ← 技能行为规范模板
│   └── agent-test-spec.md ← 智能体行为规范模板
│
└── results/               ← 测试运行输出（由 /skill-test spec 写入；未加入 gitignore——将其加入 .gitignore 可避免提交）
```

---

## 使用方法

所有测试由框架中已有的两个技能驱动：

### 检查结构合规性

```
/skill-test static [skill-name]     # 检查单个技能（7 项检查）
/skill-test static all              # 检查全部 74 个技能
```

### 运行行为规范测试

```
/skill-test spec gate-check         # 按书面规范评估技能
/skill-test spec design-review
/skill-test spec creative-director  # 智能体也可：评估 agents/directors/creative-director.md
```

### 按类别指标检查

```
/skill-test category gate-check     # 按类别指标评估单个技能
/skill-test category art-director   # 或按智能体类别评估单个智能体
/skill-test category all            # 对所有已分类的技能和智能体运行指标检查
```

### 查看完整覆盖率

```
/skill-test audit                   # 技能 + 智能体：是否有规范、上次测试、结果
```

### 改进未通过的技能

```
/skill-improve gate-check           # 测试 → 诊断 → 提出修复 → 重测循环
```

---

## 技能类别

| 类别 | 技能 | 关键指标 |
|----------|--------|-------------|
| `gate` | gate-check | 审查模式读取、完整/精简/单人总监小组、不自动推进 |
| `review` | design-review、architecture-review、review-all-gdds | 只读、8 节检查、判定正确 |
| `authoring` | design-system、quick-design、art-bible、create-architecture、… | 逐节 May-I-write、骨架优先 |
| `readiness` | story-readiness、story-done | 暴露阻塞项、完整模式下的总监门禁 |
| `pipeline` | create-epics、create-stories、dev-story、map-systems、… | 上游依赖检查、交接路径清晰 |
| `analysis` | consistency-check、balance-check、code-review、tech-debt、… | 只读报告、判定关键词、不写入 |
| `team` | team-combat、team-narrative、team-audio、… | 所有必需智能体已生成、暴露阻塞项 |
| `sprint` | sprint-plan、sprint-status、milestone-review、… | 读取冲刺数据、存在状态关键词 |
| `utility` | start、adopt、hotfix、localize、setup-engine、… | 通过静态检查 |

---

## 智能体层级

| 层级（目录） | 智能体 |
|------|--------|
| `directors` | creative-director、technical-director、producer、art-director |
| `leads` | lead-programmer、narrative-director、audio-director、game-designer、systems-designer、level-designer、qa-lead |
| `specialists` | gameplay-programmer、engine-programmer、network-programmer、ai-programmer、tools-programmer、ui-programmer、ux-designer、technical-artist、sound-designer、performance-analyst、prototyper、writer、world-builder |
| `engine/godot` | godot-specialist、godot-gdscript-specialist、godot-csharp-specialist、godot-shader-specialist、godot-gdextension-specialist |
| `engine/unity` | unity-specialist、unity-ui-specialist、unity-shader-specialist、unity-dots-specialist、unity-addressables-specialist |
| `engine/unreal` | unreal-specialist、ue-gas-specialist、ue-replication-specialist、ue-umg-specialist、ue-blueprint-specialist |
| `operations` | devops-engineer、release-manager、live-ops-designer、community-manager、analytics-engineer、economy-designer、localization-lead |
| `qa` | qa-tester、security-engineer、accessibility-specialist |

---

## 更新 catalog

`catalog.yaml` 跟踪每个技能和智能体的测试覆盖率。运行测试后：

- `/skill-test spec [name]` 会提示更新 `last_spec` 和 `last_spec_result`
- `/skill-test category [name]` 会提示更新 `last_category` 和 `last_category_result`
- `last_static` 和 `last_static_result` 需手动更新或通过 `/skill-improve` 更新

---

## 编写新规范

1. 在 `templates/skill-test-spec.md` 找到规范模板
2. 复制到 `skills/[category]/[skill-name].md`
3. 更新 `catalog.yaml` 中的 `spec:` 字段，指向新文件
4. 运行 `/skill-test spec [skill-name]` 验证

对于智能体，将 `templates/agent-test-spec.md` 复制到 `agents/[tier]/[agent-name].md`，
在 `catalog.yaml` 的 `agents:` 下为其条目设置 `spec:` 字段，然后运行
`/skill-test spec [agent-name]`。

---

## 移除此框架

主项目中没有任何 Hook 或导入依赖此文件夹，但 `/skill-test`
和 `/skill-improve` 按路径读取它。移除方法：

```bash
rm -rf studio-tests
```

之后 `/skill-test static` 照常工作，`spec` 和 `category` 停止
工作，`audit` 报告不存在 catalog，`/skill-improve` 跳过其
类别检查——本文件顶部的表格列出了每种模式的代价。