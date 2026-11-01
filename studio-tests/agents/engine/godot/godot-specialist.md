# Agent 测试规范：godot-specialist

## Agent 摘要
领域：Godot 特有的模式、节点/场景架构、信号、资源，以及 GDScript、C#、GDExtension 之间的选型决策。
不负责：以特定语言编写实际代码（交由各语言子专家处理）。
未分配门禁 ID。

---

## 静态断言（结构性）

- [ ] `description:` 字段存在且与领域相关（引用 Godot 架构 / 节点模式 / 引擎决策）
- [ ] `tools:` 列表包含 Read、Write、Edit、Bash、Glob、Grep
- [ ] 模型层级为 `inherit` —— frontmatter 中 `model:` 的值恰为 `inherit`（层级定义见 `.claude/docs/model-tiers.md`）
- [ ] Agent 定义引用 `docs/engine-reference/godot/VERSION.md` 作为权威 API 来源

---

## 测试用例

### 用例 1：领域内请求 —— 输出恰当
**输入：**“在 Godot 中，什么时候该用信号，什么时候该用直接方法调用？”
**预期行为：**
- 产出一份带理由的模式决策指南：
  - 信号：解耦通信、子节点无需了解父节点、事件驱动的 UI 更新、一对多通知
  - 直接调用：紧耦合系统，调用方需要返回值，或性能关键的热路径
- 用具体的 Godot 案例说明两侧（例如：子节点的生命值组件发出带类型的 `health_changed` 信号，由父节点或 HUD 监听；父节点调用其拥有的子节点上的方法）
- 不为两种模式产出原始代码 —— 实现交由 gdscript-specialist 或 csharp-specialist
- 说明“向下调用，向上发信号”约定：子节点不直接调用父节点的方法，而是发出信号，从而每个场景保持自包含，不隐式依赖父节点

### 用例 2：错误引擎重定向
**输入：**“写一个在 Start() 时运行并订阅 UnityEvent 的 MonoBehaviour。”
**预期行为：**
- 不产出 Unity MonoBehaviour 代码
- 明确指出这是 Unity 模式，不是 Godot 模式
- 将概念映射到 Godot：用 `_ready()` 代替 `Start()`，用 `_ready()` 中连接的带类型 Godot `signal` 代替 UnityEvent
- 依据项目的引擎固定版本（`docs/engine-reference/godot/VERSION.md`，Godot 4.7）作答，而不是询问项目使用哪个引擎
- 将实际的节点脚本交由 godot-gdscript-specialist 或 godot-csharp-specialist，而不是自己编写
- 变体：若 `project.yaml` 中为 `engine.name: Unity`，则说明自己不是该项目的合适专家，而不是映射概念

### 用例 3：截止日期后的 API 风险
**输入：**“用新的 Godot 4.5 `@abstract` 注解定义一个抽象基类。”
**预期行为：**
- 识别出 `@abstract` 是截止日期后的特性（Godot 4.5 引入，晚于 LLM 知识截止时间）
- 标记版本风险：LLM 对该注解的了解可能不完整或不正确，因此以参考文档为准
- 回答前对照版本参考确认该特性 —— `docs/engine-reference/godot/breaking-changes.md` 的 4.4 → 4.5 表中列出了 `@abstract` 装饰器
- 标记已安装版本的差距：`VERSION.md` 中 `Installed at pin time` 记录为 NOT DETERMINED，因此必须先确认已安装编辑器为 4.5 或更高版本，`@abstract` 代码才能解析
- 将类的编写交由 godot-gdscript-specialist，而不是自己编写

### 用例 4：热路径的语言选择
**输入：**“物理查询循环每帧对 500 个对象运行。这里该用 GDScript 还是 C#？”
**预期行为：**
- 给出均衡的分析：
  - GDScript：更简单，团队熟悉，但紧凑循环较慢
  - C#：CPU 密集型循环更快，需要 .NET 运行时，团队需具备 C# 知识
- 不单方面做最终决定 —— 呈现权衡，由用户决定
- 标记将 C#/.NET（或 GDExtension 模块）引入项目是重大技术选型，需 `technical-director` 批准，不能独自决定
- 说明 GDExtension（C++）是极端性能场景的第三种选择，若 C# 仍不足则建议上报

### 用例 5：上下文传递 —— 引擎版本 4.6
**输入：**提供的引擎版本上下文：Godot 4.6，Jolt 为默认物理引擎。请求：“为玩家角色设置一个 RigidBody3D。”
**预期行为：**
- 读取 4.6 上下文和物理模块参考（`docs/engine-reference/godot/modules/physics.md`），而不是仅依赖 LLM 训练数据
- 说明参考文档的内容：Jolt 是新项目的默认 3D 引擎，但现有项目保留其设置 —— 因此在假定为 Jolt 之前，先检查 Project Settings → Physics → 3D → Physics Engine
- 沿用文档中适用的 Jolt 差异：碰撞边距行为可能不同，且 Jolt 对不支持的属性会发出运行时警告（例如 HingeJoint3D 的 `damp`）
- 不臆造 GodotPhysics 与 Jolt 之间 RigidBody3D 属性的变化 —— 参考文档未记录任何变化，并如实说明

---

## 协议合规

- [ ] 保持在声明的领域内（Godot 架构决策、节点/场景模式、语言选择）
- [ ] 将语言相关的实现重定向至 godot-gdscript-specialist 或 godot-csharp-specialist
- [ ] 返回结构化的结论（决策树、带理由的模式建议）
- [ ] 将 `docs/engine-reference/godot/VERSION.md` 视为高于 LLM 训练数据的权威来源
- [ ] 标记 4.4 及以后版本的 API 用法（4.4 恰在截止时间之前；4.5、4.6、4.7 在其之后），并附验证要求
- [ ] 将语言选择决定留给用户，并将采用新语言或运行时上报给 technical-director
- [ ] 写入前先询问“May I write this to [filepath]?”，并指明文件

---

## 覆盖说明
- 信号与直接调用指南（用例 1）应写入 `docs/architecture/`，作为可复用的模式文档
- 截止日期后标记（用例 3）确认 Agent 不会自信地使用无法验证的 API
- 引擎版本用例（用例 5）验证 Agent 应用版本参考中的迁移说明，而非假设