# Agent 测试规范：godot-gdscript-specialist

## Agent 摘要
领域：GDScript 静态类型、GDScript 设计模式、信号架构、协程/await 模式、GDScript 性能。
不负责：着色器代码（godot-shader-specialist）、GDExtension 绑定（godot-gdextension-specialist）。
未分配门禁 ID。

---

## 静态断言（结构性）

- [ ] `description:` 字段存在且领域明确（提及 GDScript / 静态类型 / 信号 / 协程）
- [ ] `tools:` 列表包含 Read、Write、Edit、Bash、Glob、Grep
- [ ] 模型层级为 `sonnet` —— frontmatter 中 `model:` 精确为 `sonnet`（层级说明见 `.claude/docs/model-tiers.md`）
- [ ] Agent 定义未声称对着色器代码或 GDExtension 拥有权限

---

## 测试用例

### 用例 1：领域内请求 —— 输出恰当
**输入：** "审查这个 GDScript 文件的类型注解覆盖情况。"
**预期行为：**
- 读取提供的 GDScript 文件
- 标出所有缺少静态类型注解的变量、参数和返回类型
- 产出逐行具体发现列表：`var speed = 5.0` → `var speed: float = 5.0`
- 说明 Godot 4 中静态类型对性能和工具链的益处
- 不主动重写整个文件 —— 产出发现列表供开发者自行应用

### 用例 2：领域外请求 —— 正确重定向
**输入：** "写一个顶点着色器，在世界空间中扭曲网格。"
**预期行为：**
- 不产出 GDScript 或 Godot 着色语言的着色器代码
- 明确说明着色器编写属于 `godot-shader-specialist`
- 将请求重定向至 `godot-shader-specialist`
- 所提供的任何帮助仅限其重定向说明中声称负责的脚本侧（`set_shader_parameter()`、材质赋值）—— 不编写任何着色器代码

### 用例 3：协程异步加载
**输入：** "异步加载一个场景，等待加载完成后再生成它。"
**预期行为：**
- 产出 Godot 4 的 `await` + `ResourceLoader.load_threaded_request` 模式
- 全程使用静态类型（`var scene: PackedScene`）
- 用 `ResourceLoader.load_threaded_get_status()` 处理完成检查，轮询之间用 await 而非阻塞帧
- 生成前在 await 之后检查 `is_instance_valid(self)` —— 场景加载期间节点可能已被释放
- 说明加载失败的错误处理
- 不使用已弃用的 Godot 3 `yield()` 语法

### 用例 4：性能问题 —— 类型化数组建议
**输入：** "实体更新循环很慢；它每帧遍历一个包含 1,000 个节点的无类型 Array。"
**预期行为：**
- 指出无类型 `Array` 放弃了 GDScript 编译器优化
- 建议转换为类型化数组（`Array[Node]` 或具体类型），以便编译器使用类型化代码路径
- 将类型化数组重构作为即时修复产出
- 说明若性能分析仍显示该循环为瓶颈，下一步是联合 godot-gdextension-specialist 将热路径移至 GDExtension（C++/Rust），遵循其 GDScript/GDExtension 边界
- 无性能分析证据时，不建议将循环或代码库移出 GDScript

### 用例 5：上下文传递 —— Godot 4.6 含截止后特性
**输入：** 提供的引擎版本上下文：Godot 4.6。请求："使用 @abstract 为所有敌人类型创建一个抽象基类。"
**预期行为：**
- 识别 `@abstract` 为 Godot 4.5+ 特性（截止后）
- 在输出中注明：该特性于 4.5 引入，已对照 `docs/engine-reference/godot/breaking-changes.md`（4.4 → 4.5 表格）确认
- 按参考文档（`current-best-practices.md`）而非训练数据，产出在类及其抽象方法上使用 `@abstract` 的 GDScript 类
- 标记 `VERSION.md` 将 `Installed at pin time` 记录为 NOT DETERMINED —— 已安装编辑器必须为 4.5 或更高版本才能解析 `@abstract`
- 抽象类中所有方法签名使用静态类型

---

## 协议合规

- [ ] 保持在声明的领域内（GDScript —— 类型、模式、信号、协程、性能）
- [ ] 将着色器请求重定向至 godot-shader-specialist
- [ ] 将 GDExtension 请求重定向至 godot-gdextension-specialist
- [ ] 返回带完整静态类型的结构化 GDScript 输出
- [ ] 仅使用 Godot 4 API —— 不使用已弃用的 Godot 3 模式（yield、字符串 connect 等）
- [ ] 标记 4.4 及之后的特性（4.4 恰在截止前；4.5、4.6、4.7 在截止后），并标记为需文档验证
- [ ] 写入前询问"May I write this to [filepath]?"并指明文件

---

## 覆盖说明
- 类型注解审查（用例 1）的输出适合作为代码审查清单
- 异步加载（用例 3）应产出可在 `tests/unit/` 中用单元测试验证的可测试代码
- 截止后 @abstract（用例 5）确认 Agent 会标记版本不确定性，而非静默使用未验证的 API