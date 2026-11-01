# Studio Tests — Claude 指令

本文件夹是 Claude Code Game Studios 技能/代理框架的质量保障层。它自成一体，与任何游戏项目相互独立。

## 关键文件

| 文件 | 用途 |
|------|---------|
| `catalog.yaml` | 全部 74 个技能和 49 个代理的总注册表。包含类别、spec 路径和最近测试跟踪字段。运行任何测试命令时先读此文件。 |
| `quality-rubric.md` | 按类别划分的通过/失败指标。运行 `/skill-test category` 时，读取与技能类别匹配的 `###` 小节。 |
| `skills/[category]/[name].md` | 技能的行为规范——5 个或更多测试用例 + 协议合规断言。 |
| `agents/[tier]/[name].md` | 代理的行为规范——5 个或更多测试用例 + 协议合规断言。 |
| `templates/skill-test-spec.md` | 编写新技能 spec 文件的模板。 |
| `templates/agent-test-spec.md` | 编写新代理 spec 文件的模板。 |
| `results/` | 由 `/skill-test spec` 在保存结果时写入。未加入 gitignore——将其加入 `.gitignore` 可使结果不进入提交。 |

## 路径约定

- 技能 spec：`studio-tests/skills/[category]/[name].md`
- 代理 spec：`studio-tests/agents/[tier]/[name].md`
- 注册表：`studio-tests/catalog.yaml`
- 指标：`studio-tests/quality-rubric.md`

`catalog.yaml` 中的 `spec:` 字段是每个技能/代理 spec 的权威路径。始终读取它，不要猜测路径。

## 技能类别

```
gate        → gate-check
review      → design-review, architecture-review, review-all-gdds
authoring   → design-system, quick-design, architecture-decision, art-bible,
              create-architecture, ux-design, ux-review
readiness   → story-readiness, story-done
pipeline    → create-epics, create-stories, dev-story, create-control-manifest,
              propagate-design-change, map-systems, vertical-slice
analysis    → consistency-check, balance-check, content-audit, code-review,
              tech-debt, scope-check, estimate, perf-profile, asset-audit,
              security-audit, test-evidence-review, test-flakiness
team        → team-combat, team-narrative, team-audio, team-level, team-ui,
              team-qa, team-release, team-polish, team-live-ops
sprint      → sprint-plan, sprint-status, milestone-review, retrospective,
              changelog, patch-notes
utility     → 其余所有技能
```

## 代理层级

```
directors      → creative-director, technical-director, producer, art-director
leads          → lead-programmer, narrative-director, audio-director, game-designer,
                 systems-designer, level-designer, qa-lead
specialists    → gameplay-programmer, engine-programmer, network-programmer,
                 ai-programmer, tools-programmer, ui-programmer, ux-designer,
                 technical-artist, sound-designer, performance-analyst, prototyper,
                 writer, world-builder
engine/godot   → godot-specialist, godot-gdscript-specialist, godot-csharp-specialist,
                 godot-shader-specialist, godot-gdextension-specialist
engine/unity   → unity-specialist, unity-ui-specialist, unity-shader-specialist,
                 unity-dots-specialist, unity-addressables-specialist
engine/unreal  → unreal-specialist, ue-gas-specialist, ue-replication-specialist,
                 ue-umg-specialist, ue-blueprint-specialist
operations     → devops-engineer, release-manager, live-ops-designer,
                 community-manager, analytics-engineer, economy-designer,
                 localization-lead
qa             → qa-tester, security-engineer, accessibility-specialist
```

层级是 spec 所在的目录。指标类别是代理在 `catalog.yaml` 中的 `category:`，两者可以不同：`security-engineer` 和 `accessibility-specialist` 位于 `qa/` 下，但按 `specialist` 评级。

## 测试技能或代理的工作流

1. 读取 `catalog.yaml` 获取 `spec:` 路径和 `category:`——技能查 `skills:` 条目，代理查 `agents:` 条目
2. 读取 `.claude/skills/[name]/SKILL.md` 及其 `references/` 文件夹中的每个文件，或读取 `.claude/agents/[name].md`
3. 读取项目 `CLAUDE.md` 及其通过 `@` 导入的文件：每个子代理都会收到这些文件，因此其中陈述的规则（例如 `coordination-rules.md` 中的重定向）视为技能或代理自身的规则
4. 读取 `spec:` 路径处的 spec
5. 评估 spec 的静态断言、每个用例的断言及其协议合规列表
6. 提议将结果写入 `results/` 并更新 `catalog.yaml`

## 夹具默认值

每个技能 spec 的用例上方都有 `## Fixture Defaults` 小节。用例的沙箱项目从以下 `project.yaml` 块开始：

```yaml
modes:
  rigor: minimal            # → workflow minimal, docs.density terse, qa.level minimal,
                            #   story_granularity coarse, review_mode solo, team.size individual
  automation: collaborative
```

- 用例夹具中指定的设置优先于该块，显式子开关（`modes.workflow`、`modes.review_mode`、`qa.level` 等）优先于 `rigor` 预设（`.claude/docs/config-resolution.md` § Defaults）。
- 用例若声明 `project.yaml` 未设置任何 `modes` 键，或不存在 `project.yaml`，则不使用该块运行：它测试未设置的默认值。
- 除非用例指定，否则不存在 `project.local.yaml`。
- 若用例的预期行为随某设置变化，则在其自身夹具中固定该设置并说明原因——例如 `modes.workflow: standard`（固定：未设置时解析为 minimal，会推荐下一次构建而非冲刺）。

固定该块是为了让框架默认值的变更不会悄悄改变用例所测试的内容。若某个 spec 的大多数用例需要不同的默认值，可在其自身小节中设置。

## 改进技能的工作流

使用 `/skill-improve [name]`。它处理完整循环：
测试 → 诊断 → 提出修复 → 重写 → 重测 → 保留或回滚。

## Spec 有效性说明

本文件夹中的 spec 断言的是**正确**行为，而非技能或代理当前的行为。断言失败意味着技能或代理有错，除非另有证明：修复它并重新运行，绝不要为了让断言通过而削弱断言。spec 也可能出错——当某个 spec 断言的行为本身不正确时，应将 spec 修正为正确行为，而非技能当前的行为。

## 本文件夹可删除

`.claude/` 中没有任何内容从此处导入，其他所有 CCGS 技能和代理在没有它时也能正常工作——但 `/skill-test` 和 `/skill-improve` 会按路径读取它。没有它时，`/skill-test static` 仍可工作，`/skill-test spec` 和 `/skill-test category` 停止工作，`/skill-test audit` 报告不存在注册表，`/skill-improve` 跳过其类别检查（详情见 `README.md` 中的表格）。