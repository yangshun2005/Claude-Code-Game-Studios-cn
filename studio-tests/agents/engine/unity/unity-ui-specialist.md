# Agent 测试规范：unity-ui-specialist

## Agent 摘要
领域：Unity UI Toolkit（UXML/USS）、UGUI（Canvas）、数据绑定、运行时 UI 性能、UI 输入事件处理。
不负责：UX 流程设计（ux-designer）、视觉美术风格（art-director）。
未分配门禁 ID。

---

## 静态断言（结构性）

- [ ] `description:` 字段存在且领域明确（引用 UI Toolkit / UGUI / Canvas / 数据绑定）
- [ ] `tools:` 列表包含 Read、Write、Edit、Bash、Glob、Grep
- [ ] 模型层级为 `sonnet` —— frontmatter 中 `model:` 精确为 `sonnet`（层级说明：`.claude/docs/model-tiers.md`）
- [ ] Agent 定义未声称对 UX 流程设计或视觉美术方向拥有权限

---

## 测试用例

### 用例 1：领域内请求 —— 输出恰当
**输入：**“使用 Unity UI Toolkit 实现一个背包 UI 界面。”
**预期行为：**
- 产出 UXML 文档，定义背包面板结构（ListView、物品模板、详情面板）
- 产出 USS 样式，用于背包布局和物品状态（默认、悬停、选中）
- 提供 C# 代码，通过运行时绑定系统将背包数据模型绑定到 UI：ViewModel 实现 `INotifyBindablePropertyChanged`，UI 只通过绑定读取，绝不写入游戏状态
- 对可滚动物品列表使用带 `makeItem` / `bindItem` 回调的 `ListView`
- 不产出 UX 流程设计 —— 依据提供的规范实现

### 用例 2：领域外重定向
**输入：**“设计背包的 UX 流程 —— 玩家装备物品与丢弃物品时分别发生什么。”
**预期行为：**
- 不产出 UX 流程设计
- 明确说明交互流程设计属于 `ux-designer`
- 将请求重定向到 `ux-designer`
- 说明将实现 ux-designer 指定的任何流程

### 用例 3：UI Toolkit 动态列表数据绑定
**输入：**“背包列表需要随着物品加入或移出玩家背包实时更新。”
**预期行为：**
- 在背包的后备集合上产出 `ListView` 模式，通过 `makeItem` / `bindItem` 虚拟化，使只有可见行存在
- 由背包的变更事件驱动刷新（游戏状态 → ViewModel → 绑定），而非逐帧轮询
- 不通过为每个元素查询可视化树来刷新列表 —— 将逐帧可视化树查询标记为反模式，改为缓存引用
- 不在每次变更时创建和销毁行元素 —— 通过虚拟化列表复用行

### 用例 4：Canvas 性能 —— 过度绘制
**输入：**“主菜单 Canvas 触发了 GPU 过度绘制警告；有许多重叠面板。”
**预期行为：**
- 在更改结构前先用性能分析器确认热点（Frame Debugger、Profiler UI 模块）
- 推荐其文档化的 Canvas 修复方案：
  - 每个逻辑 UI 层一个 Canvas，频繁变化的内容与静态内容放在不同 Canvas 上
  - 显式设置 `Canvas.sortingOrder`，而非依赖层级顺序
  - 共享精灵图集使面板合批，非交互元素禁用 Raycast Target
  - 用 `CanvasGroup` 淡出或隐藏一组元素，而非逐 Image 修改 alpha
- 说明按自身选择规则，屏幕空间菜单属于 UI Toolkit 范畴（引擎参考也将 UGUI Canvas 标记为新项目中的已取代方案），作为选项提供而非单方面重写

### 用例 5：上下文传递 —— Unity 版本
**输入：**项目上下文：Unity 2022.3 LTS。请求：“用数据绑定实现设置面板。”
**预期行为：**
- 将所述版本与 `docs/engine-reference/unity/VERSION.md` 核对（该文件固定为 Unity 6.3 LTS），并在选择绑定 API 前询问实际安装的是哪个编辑器
- 不凭记忆断言哪个 Unity 版本引入了运行时数据绑定 —— 引擎参考未记录此信息，因此该说法标记为未经验证
- 在确认该 API 在 2022.3 编辑器中可用之前，不为 2022.3 编辑器输出运行时绑定系统（`INotifyBindablePropertyChanged`）
- 无论版本如何，保留其模式中与版本无关的部分：UI 读取状态，用户操作派发命令，面板绝不直接写入游戏状态

---

## 协议合规

- [ ] 保持在声明的领域内（UI Toolkit、UGUI、数据绑定、UI 性能）
- [ ] 将 UX 流程设计重定向到 ux-designer
- [ ] 返回结构化输出（UXML、USS、C# 绑定代码）
- [ ] 在使用依赖版本的 UI API 前，将项目的 Unity 版本与 `docs/engine-reference/unity/VERSION.md` 核对；当所述版本与固定版本不一致时进行询问
- [ ] 将 Canvas 过度绘制标记为性能反模式，并提供具体修复方案（按层和更新频率拆分 Canvas、图集化、关闭 Raycast Target）
- [ ] 用其文档化的机制隐藏元素 —— UI Toolkit 中用 `VisualElement.visible = false`，UGUI 中用 `CanvasGroup` —— 绝不将每个元素的 alpha 设为 0
- [ ] 写入前询问“我可以将此写入 [filepath] 吗？”并指明文件

---

## 覆盖说明
- 背包 UI（用例 1）应在 `production/qa/evidence/` 中保留每个涉及界面的截图
- 动态列表绑定（用例 3）应有集成测试或自动化交互测试
- Canvas 过度绘制（用例 4）验证 Agent 了解正确的 Unity UI 性能模式