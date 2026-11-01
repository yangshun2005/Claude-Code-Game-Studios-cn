# Claude Code 游戏工作室 -- 游戏工作室 Agent 架构

通过 49 个协同工作的 Claude Code 子 Agent 管理独立游戏开发。
每个 Agent 负责特定领域，强制执行关注点分离与质量要求。

## 技术栈

- **引擎**：[选择：Godot 4 / Unity / Unreal Engine 5]
- **语言**：[选择：GDScript / C# / C++ / Blueprint]
- **版本控制**：Git，采用主干开发
- **构建系统**：[选定引擎后指定]
- **资源管线**：[选定引擎后指定]

> **注意**：Godot、Unity 和 Unreal 均有对应的引擎专家 Agent 及
> 专属子专家。请使用与你的引擎匹配的那一组。

## 项目结构

@.claude/docs/directory-structure.md

## 引擎版本参考

<!-- ENGINE-REFERENCE-IMPORT: 下一行与引擎相关。/setup-engine
     会将其改写为 @docs/engine-reference/<engine>/VERSION.md（针对所选引擎），
     这样 Unity 或 Unreal 项目就不会每次会话都加载 Godot 参考。
     默认指向 Godot（模板的示例引擎）；需要固定版本的技能会按需
     读取 docs/engine-reference/<engine>/VERSION.md，不受此导入影响。 -->
@docs/engine-reference/godot/VERSION.md


## 技术偏好

仓库根目录的 `project.yaml` 是主要配置存储——引擎、专家、命名、平台、性能、模式。
技能通过 `resolve_config` 解析它（见 `.claude/docs/config-resolution.md`）。

`.claude/docs/technical-preferences.md` 是**旧版回退**，当 `project.yaml` 中缺少某个键时按需读取。
它不在此处导入：在 `/setup-engine` 运行之前，它几乎全是 `[待配置]` 占位符；
运行之后，真实值保存在 `project.yaml` 中。

## 协作规则

@.claude/docs/coordination-rules.md

## 协作协议

**用户驱动的协作，而非自主执行。**
每项任务遵循：**提问 -> 选项 -> 决策 -> 草稿 -> 批准**

- Agent 在使用 Write/Edit 工具前必须询问“我可以将此写入 [文件路径] 吗？”
  （豁免：追加 `active.md`、Agent 记忆、子 Agent 在其编排技能指定的路径下新建文件；
  见 `.claude/rules/agent-memory.md`）
- Agent 在请求批准前必须展示草稿或摘要
- 多文件变更需对整个变更集获得明确批准
- 未经用户指示不得提交
- 被跳过或忽略的问题不算回答：不要采取任何行动，用纯文本重新提问并等待
- 从本文件所在文件夹之外启动？先警告：Hook、拒绝规则和先询问模式仅在 Claude Code 从此处启动时加载

完整协议与示例见 `docs/COLLABORATIVE-DESIGN-PRINCIPLE.md`。

> **首次会话？** 如果项目尚未配置引擎且没有游戏概念，
> 运行 `/start` 开始引导式入门流程。

## 编码规范

@.claude/docs/coding-standards.md

## 上下文管理

按需阅读 `.claude/docs/context-management.md`——它是参考文档，不是会话上下文。
其中两条约定是关键性的，且被仓库其他地方按名称引用，因此在此重述以免丢失：

- **`production/session-state/active.md` 是会话检查点。** 该文件才是记忆，而非对话。
  在任何压缩、崩溃或 `/clear` 之后，首先读取它。
- **`.claude/scripts/` 中的辅助脚本只输出观察结果，绝不输出裁决。**
  一个打分或评判的脚本最终会与它看不到的模式相矛盾，或将其覆盖。
  （被 `artifact-check.sh` 和 `adr-dep-graph.sh` 引用。）