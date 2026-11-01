# Agent 测试规范：unity-specialist

## Agent 摘要
领域：Unity 专属架构模式、MonoBehaviour 与 DOTS 的取舍、子系统选型（Addressables、New Input System、UI Toolkit、Cinemachine 等）。
不负责：语言层面的深入细节（交由 unity-dots-specialist、unity-ui-specialist 等处理）。
未分配门禁 ID。

---

## 静态断言（结构性）

- [ ] `description:` 字段存在且与领域相关（提及 Unity 模式 / MonoBehaviour / 子系统决策）
- [ ] `tools:` 列表包含 Read、Write、Edit、Bash、Glob、Grep
- [ ] 模型层级为 `inherit` — frontmatter 中 `model:` 的值恰为 `inherit`（层级说明见 `.claude/docs/model-tiers.md`）
- [ ] Agent 定义中承认子专家路由表（DOTS、UI、Shader、Addressables）

---

## 测试用例

### 用例 1：领域内请求 — 输出恰当
**输入：**“存储敌人配置数据应该用 MonoBehaviour 还是 ScriptableObject？”
**预期行为：**
- 产出一份模式决策树，涵盖：
  - MonoBehaviour：用于运行时行为，需挂载到 GameObject，具有 Update() 生命周期
  - ScriptableObject：用于纯数据/配置，以资源形式存在，可跨实例共享，不依赖场景
- 推荐用 ScriptableObject 存储敌人配置数据（无状态、可复用、对设计师友好）
- 指出 MonoBehaviour 可引用该 ScriptableObject 供运行时使用
- 给出 ScriptableObject 类定义的具体示例（不产出完整代码 — 实现交由 engine-programmer 或 gameplay-programmer）

### 用例 2：错误引擎重定向
**输入：**“为这个敌人系统搭建带信号的 Node 场景树。”
**预期行为：**
- 不产出 Godot 的 Node/信号代码
- 识别出节点树和信号是 Godot 概念
- 从 `project.yaml` 的 `engine.name` 确认项目引擎，而非猜测；若非 Unity，说明自己不是该项目的对口专家，而不是做概念转换
- 若 `engine.name` 未设置，说明未配置引擎并询问项目使用哪种引擎 — 不假定为 Unity
- 在 Unity 项目中做概念映射：节点树 → 由组合的 MonoBehaviour 构成的 GameObject 层级；信号 → C# 事件或 UnityEvent（与其自身规则一致：用事件而非 `SendMessage()` / `Find()`）
- 不把请求转交给 Godot 专家 — 其 Agent 授权仅覆盖四个 Unity 子专家

### 用例 3：Unity 版本 API 标记
**输入：**“用新的 Unity 6 GPU Resident Drawer 做批量渲染。”
**预期行为：**
- 回答前先查 `docs/engine-reference/unity/VERSION.md` 和 `docs/engine-reference/unity/modules/rendering.md`，而非依赖训练数据（版本文件显示 Unity 6 超出模型知识截止时间）
- 说明参考文档所述：GPU Resident Drawer 是 Unity 6+ 特性，在 URP Asset 中启用（Rendering > GPU Resident Drawer），因此需要 SRP 项目
- 标记已安装版本的差距：当版本文件中 `Installed at pin time` 未确定时，先询问安装了哪个编辑器版本再推荐该特性
- 不因参考文档锁定 Unity 6 就假定已安装编辑器为 Unity 6
- 提出 URP Asset 的修改方案并在编辑前询问；超出该设置范围的渲染管线定制交由 `unity-shader-specialist`

### 用例 4：DOTS 与 MonoBehaviour 冲突
**输入：**“战斗系统用 MonoBehaviour 做状态管理，但我们想加入基于 DOTS 的投射物系统。两者能共存吗？”
**预期行为：**
- 识别为混合架构场景
- 按 `docs/engine-reference/unity/plugins/dots-entities.md` 所述，在架构层面解释共存机制：authoring MonoBehaviour 经 `Baker<T>` 烘焙为实体，投射物数据存于 `IComponentData`，由 `ISystem` 处理
- 指出混用两种模式的性能与复杂度权衡
- 建议将架构决策上报给 `lead-programmer` 或 `technical-director`
- DOTS 侧实现细节交由 `unity-dots-specialist`

### 用例 5：上下文传递 — Unity 版本
**输入：**提供的项目上下文：Unity 6.3 LTS，目标为键盘/鼠标和手柄。请求：“为这个项目配置新的 Input System。”
**预期行为：**
- 应用 `docs/engine-reference/unity/modules/input.md` 中的 Unity 6.3 LTS 上下文：Input System 包（`com.unity.inputsystem`），Active Input Handling 设为新系统
- 不产出旧版 Input Manager 代码（`Input.GetKeyDown()`、`Input.GetAxis()`）— 引擎参考将其列为已弃用
- 在 `.inputactions` 资源中定义动作，使用 Player Input 组件或生成的 C# 类，优先用动作回调（`performed`、`canceled`）而非在 `Update()` 中轮询
- 设置键盘+鼠标和手柄控制方案并支持自动切换
- UI 导航和输入提示相关工作交由 `unity-ui-specialist`
- 将添加该包视为需 `technical-director` 签核的依赖决策，写入任何设置或资源前先询问

---

## 协议合规

- [ ] 保持在声明的领域内（Unity 架构决策、模式选型、子系统路由）
- [ ] 将 Godot 模式视为错误引擎：检查 `engine.name`，用 Unity 等价物作答，绝不产出 Godot 代码
- [ ] 将 DOTS 实现重定向至 unity-dots-specialist
- [ ] 将 UI 实现重定向至 unity-ui-specialist
- [ ] 建议版本相关 API 前先查 `docs/engine-reference/unity/`（锁定版本和 `Installed at pin time`），未记录时确认已安装版本
- [ ] 返回结构化的模式决策指南，而非自由形式的意见

---

## 覆盖说明
- MonoBehaviour 与 ScriptableObject（用例 1）若形成项目级决策，应记录为 ADR
- 版本标记（用例 3）确认 Agent 会检查锁定版本和已安装的 Unity 版本，而非假定为最新版
- DOTS 混合（用例 4）验证 Agent 会上报架构冲突，而非单方面解决