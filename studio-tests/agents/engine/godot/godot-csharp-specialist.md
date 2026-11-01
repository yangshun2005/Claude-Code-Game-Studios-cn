# Agent 测试规范：godot-csharp-specialist

## Agent 摘要
领域：Godot 4 中的 C# 模式、应用于 Godot 的 .NET 惯用法、`[Export]` 特性用法、信号委托以及 async/await 模式。
不负责：GDScript 代码（gdscript-specialist）、GDExtension C/C++ 绑定（gdextension-specialist）。
未分配门禁 ID。

---

## 静态断言（结构性）

- [ ] `description:` 字段存在且领域明确（引用 Godot 4 中的 C# / .NET 模式 / 信号委托）
- [ ] `tools:` 列表包含 Read、Write、Edit、Bash、Glob、Grep
- [ ] 模型层级为 `sonnet` — frontmatter 中 `model:` 精确为 `sonnet`（层级说明见 `.claude/docs/model-tiers.md`）
- [ ] Agent 定义未声称对 GDScript 或 GDExtension 代码拥有权限

---

## 测试用例

### 用例 1：域内请求 — 输出恰当
**输入：** "为敌人生命值创建一个导出属性，带校验，将其限制在 1 到 1000 之间。"
**预期行为：**
- 生成带 `[Export]` 特性的 C# 属性
- 使用后备字段配合属性 getter/setter，在 setter 中限制取值
- 不使用无校验的裸 `[Export]` 公共字段
- 遵循 Godot 4 C# 命名约定（属性用 PascalCase，字段私有并加下划线前缀）
- 按编码规范为属性添加 XML 文档注释

### 用例 2：域外请求 — 正确重定向
**输入：** "用 GDScript 重写这个敌人生命值系统。"
**预期行为：**
- 不生成 GDScript 代码
- 明确说明 GDScript 编写属于 `godot-gdscript-specialist`
- 将请求重定向到 `godot-gdscript-specialist`
- 其提供的任何交接内容仅描述现有 C# 系统的 API（信号和公共方法），供 godot-gdscript-specialist 移植 — 不含任何 GDScript

### 用例 3：异步等待信号
**输入：** "用 C# async 等待动画播放完毕后再切换游戏状态。"
**预期行为：**
- 生成正确的 `async Task` 模式，使用 `ToSignal()` 等待 Godot 信号
- 使用 `await ToSignal(animationPlayer, AnimationPlayer.SignalName.AnimationFinished)`
- 不使用 `Thread.Sleep()` 或 `Task.Delay()` 作为轮询替代
- 说明调用方法必须为 `async`，且即发即弃的 `async void` 仅可用于事件处理器
- 在 `await` 之后、切换状态之前检查 `IsInstanceValid(this)` — 等待期间节点可能已被释放
- 若动画可能永不结束（被中断、被替换），将其作为未指定的边界情况提出，交由用户决定；其添加的任何超时使用 `GetTree().CreateTimer()`，绝不用 `Task.Delay()`

### 用例 4：线程模型冲突
**输入：** "这段 C# 代码从后台 Task 线程访问 Godot Node 以更新其位置。"
**预期行为：**
- 标记为竞态风险：Godot 节点非线程安全，只能从主线程访问
- 不批准也不实现多线程访问节点的模式
- 提供正确模式：使用 `CallDeferred()`、`Callable.From().CallDeferred()`，或通过线程安全队列编组回主线程
- 解释为何代码能编译却不安全：`Task` 主体运行在 .NET 线程池线程上，C# 中没有任何机制阻止它触碰 Node — 主线程规则属于 Godot 的场景树

### 用例 5：上下文传递 — Godot 4.7 API 正确性
**输入：** 引擎版本上下文：Godot 4.7。请求："使用新的类型化信号委托模式连接信号。"
**预期行为：**
- 生成使用 Godot 4 C# 引入的类型化委托模式的 C# 信号连接（对类型化信号使用 `+=` 运算符）
- 检查 4.7 上下文，确认 4.4、4.5、4.6 或 4.7 中信号委托 API 无破坏性变更
- 不使用旧的基于字符串的 `Connect("signal_name", callable)` 模式（在 Godot 4 C# 中已弃用）
- 生成的代码兼容 VERSION.md 中记录的、项目锁定的 4.7 版本

---

## 协议合规

- [ ] 保持在声明领域内（Godot 4 中的 C# — 模式、导出、信号、异步）
- [ ] 将 GDScript 请求重定向到 godot-gdscript-specialist
- [ ] 将 GDExtension 请求重定向到 godot-gdextension-specialist
- [ ] 返回遵循 Godot 4 约定的 C# 代码（而非 Unity MonoBehaviour 模式）
- [ ] 将多线程访问 Godot 节点标记为不安全，并提供正确模式
- [ ] 使用类型化信号委托 — 而非已弃用的基于字符串的 Connect() 调用
- [ ] 生成代码前检查引擎版本参考以确认 API 变更
- [ ] 写入前询问"May I write this to [filepath]?"并指明文件

---

## 覆盖说明
- 带校验的导出属性（用例 1）应有单元测试验证限制行为
- 线程冲突（用例 4）属安全关键：Agent 必须无需提示即可识别并修复
- 异步信号（用例 3）验证 Agent 在 Godot 单线程约束下正确应用 .NET 惯用法