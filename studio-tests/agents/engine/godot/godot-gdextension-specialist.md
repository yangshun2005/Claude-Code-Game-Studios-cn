# Agent 测试规范：godot-gdextension-specialist

## Agent 摘要
领域：GDExtension API、godot-cpp C++ 绑定、godot-rust 绑定、原生库集成、原生性能优化。
不负责：GDScript 代码（gdscript-specialist）、着色器代码（godot-shader-specialist）。
未分配门禁 ID。

---

## 静态断言（结构性）

- [ ] `description:` 字段存在且领域明确（提及 GDExtension / godot-cpp / 原生绑定）
- [ ] `tools:` 列表包含 Read、Write、Edit、Bash、Glob、Grep
- [ ] 模型层级为 `sonnet` —— frontmatter 中 `model:` 的值恰为 `sonnet`（层级说明见 `.claude/docs/model-tiers.md`）
- [ ] Agent 定义未声明对 GDScript 或着色器编写拥有权限

---

## 测试用例

### 用例 1：领域内请求 —— 输出恰当
**输入：**“通过 GDExtension 将一个 C++ 刚体物理仿真库暴露给 GDScript。”
**预期行为：**
- 使用 godot-cpp 给出 GDExtension 绑定模式：
  - 类继承自 `godot::Object` 或合适的 Godot 基类
  - `GDCLASS` 宏注册
  - `_bind_methods()` 实现，将物理 API 暴露给 GDScript
  - 模块入口点：`register_types.cpp` 中的 `initialize_module` 在 `MODULE_INITIALIZATION_LEVEL_SCENE` 层级用 `ClassDB::register_class<T>()` 注册该类，由 manifest 的 `entry_symbol` 指定的导出初始化函数调用
- 说明所需的 `.gdextension` manifest 文件格式
- 不产出 GDScript 使用代码（那属于 gdscript-specialist）

### 用例 2：领域外重定向
**输入：**“编写调用用例 1 中物理仿真的 GDScript。”
**预期行为：**
- 不产出 GDScript 代码
- 明确说明 GDScript 编写属于 `godot-gdscript-specialist`
- 重定向到 `godot-gdscript-specialist`
- 其提供的任何交接内容仅为扩展的绑定 API 表面（方法名、参数与返回类型）—— 而非调用它的 GDScript

### 用例 3：ABI 兼容性风险 —— 次版本更新
**输入：**“我们正从 Godot 4.5 升级到 4.6。现有 GDExtension 还能用吗？”
**预期行为：**
- 说明兼容方向：为 4.5 构建的扩展应能在 4.6 中加载，反之则不行 —— 因此它可能继续可用，但必须重新测试，并针对 4.6 重新构建才能使用更新的 API
- 指引查阅 4.5→4.6 迁移指南中的 GDExtension API 变更
- 建议针对 4.6 的 godot-cpp 头文件重新构建，而非把“应能加载”当作已测试
- 指出 `.gdextension` manifest 可能需要更新 `compatibility_minimum` 版本
- 以清单形式列出重新构建步骤：为每个目标平台重新构建 debug 和 release（`scons ... target=template_debug` / `template_release`，或 `cargo build` / `cargo build --release`），更新 `compatibility_minimum`，然后在 4.6 编辑器中重新测试后再发布

### 用例 4：内存管理 —— Godot 对象的 RAII
**输入：**“在 C++ GDExtension 代码中创建的 Godot 对象，其生命周期应如何管理？”
**预期行为：**
- 给出 GDExtension 中 Godot 对象基于 RAII 的生命周期模式：
  - `Ref<T>` 用于引用计数对象（Ref 离开作用域时自动释放）
  - `memnew()` / `memdelete()` 用于非引用计数对象
  - 警告：不要对 Godot 对象使用 `new`/`delete` —— 会导致未定义行为
- 说明对象所有权规则：谁负责释放加入场景树的节点
- 其具体示例（例如在 C++ 中创建的 `CollisionShape3D`）同时应用两条规则：节点用 `memnew()` 创建并通过 `add_child()` 交给场景树，之后由场景树释放（不调用 `memdelete()`）；其形状资源由 `Ref<>` 持有；`memdelete()` 仅用于从未进入场景树的节点

### 用例 5：上下文传递 —— Godot 4.6 GDExtension API 检查
**输入：**引擎版本上下文：Godot 4.6（从 4.5 升级）。请求：“检查 4.5 到 4.6 之间是否有 GDExtension API 变更。”
**预期行为：**
- 引用 VERSION.md 已验证来源列表中的 4.5→4.6 迁移指南
- 读取 `docs/engine-reference/godot/breaking-changes.md` 中的 4.5 → 4.6 表格，并明确说明其中没有 GDExtension 专属行，同时提示需对照官方更新日志核实
- 仍列出影响原生代码的 4.6 变更：`Quaternion` 现在初始化为单位四元数（此前为零）
- 指出 Windows 上 D3D12 成为默认（4.6 变更）对 GDExtension 渲染代码可能相关
- 提供升级后需验证事项的清单

---

## 协议合规性

- [ ] 保持在声明的领域内（GDExtension、godot-cpp、godot-rust、原生绑定）
- [ ] 将 GDScript 编写重定向到 godot-gdscript-specialist
- [ ] 将着色器编写重定向到 godot-shader-specialist
- [ ] 返回结构化输出（绑定模式、RAII 示例、ABI 清单）
- [ ] 说明兼容性仅向前（旧构建 → 新 Godot），升级后需重新测试 —— 绝不把“应能加载”当作已测试
- [ ] 使用 Godot 专属内存管理（`memnew`/`memdelete`、`Ref<T>`），而非原生 C++ new/delete
- [ ] 在确认兼容性前，查阅引擎版本参考中的 GDExtension API 变更
- [ ] 写入前询问“我可以将其写入 [filepath] 吗？”并指明文件

---

## 覆盖说明
- 绑定模式（用例 1）应包含冒烟测试，验证扩展可加载且方法可从 GDScript 调用
- ABI 风险（用例 3）是关键升级路径 —— Agent 不得批准发布未经测试的扩展二进制
- 内存管理（用例 4）验证 Agent 应用 Godot 专属模式，而非通用 C++ RAII