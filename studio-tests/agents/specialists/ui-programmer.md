# Agent 测试规范：ui-programmer

## Agent 摘要
领域：菜单界面、HUD、背包界面、对话框、UI 框架代码和数据绑定。
不负责：UX 流程和交互设计，包括 UI 动画时序（ux-designer）、视觉风格方向（art-director）。
未分配门禁 ID。

---

## 静态断言（结构性）

- [ ] `description:` 字段存在且针对领域（引用菜单 / HUD / UI 框架 / 数据绑定）
- [ ] `tools:` 列表包含 Read、Write、Edit、Bash、Glob、Grep
- [ ] 模型层级为 `inherit` — frontmatter 中 `model:` 精确为 `inherit`（层级说明：`.claude/docs/model-tiers.md`）
- [ ] Agent 定义未声称对 UX 流程设计或视觉艺术方向拥有决定权

---

## 测试用例

### 用例 1：领域内请求 — 输出恰当
**输入：** `design/ux/inventory-flow.md` 中的 UX 规范定义了五种槽位状态：default、hover、selected、empty-slot、locked-slot。请求："按照 `design/ux/inventory-flow.md` 中的 UX 规范实现背包界面。"
**预期行为：**
- 在产出任何代码前先阅读 UX 规范
- 使用项目配置的 UI 框架（UI Toolkit、UGUI、UMG 或 Godot Control 节点）实现
- 实现规范定义的每一个状态（default、hover、selected、empty-slot、locked-slot）— 不遗漏、不臆造
- 通过项目的数据模型将背包数据绑定到 UI 元素，而非硬编码值
- 按编码规范为公开 UI API 添加文档注释

### 用例 2：领域外请求 — 正确重定向
**输入：** "设计背包交互流程 — 玩家装备、丢弃或组合物品时会发生什么。"
**预期行为：**
- 不产出交互流程设计或用户流程图
- 明确说明 UX 流程设计属于 `ux-designer`
- 将请求重定向至 `ux-designer`
- 说明流程规范就绪后即可实现

### 用例 3：自定义动画协调
**输入：** "背包中的物品选中时需要自定义弹跳动画。"
**预期行为：**
- 认识到弹跳的手感（时长、缓动曲线、过冲）属于 UI 交互和视觉设计，由它实现但不负责设计
- 先检查正在实现的 UX 规范中是否有这些值；若缺失则不臆造
- 向规范负责人索取缺失值 — 交互时序找 `ux-designer`，视觉风格找 `art-director`
- 拿到值后，将动画绑定到选中状态，可跳过并遵循玩家的动效偏好

### 用例 4：UX 规范含糊 — 回传标记
**输入：** UX 规范写明"选中时显示物品详情"，但未定义选中空槽位时会发生什么。
**预期行为：**
- 识别规范中的含糊之处（空槽位选中状态未定义）
- 不为未定义状态做任意实现决策
- 将含糊之处回传 `ux-designer`，并附具体问题："选中空背包槽位时，详情面板应显示什么？"
- 它提供的任何选项（例如隐藏面板 / 显示占位符）作为问题的一部分交给 ux-designer — 在得到答复前不实现任何一项

### 用例 5：上下文传递 — 引擎 UI 工具包
**输入：** 提供的引擎上下文：项目使用 Godot 4.6 和 Control 节点 UI。请求："为背包实现可滚动的物品列表。"
**预期行为：**
- 使用 Godot 的 `ScrollContainer` + `VBoxContainer` + `ItemList`（或等效）模式，而非 Canvas 或 UGUI
- 不为 Godot 项目产出 Unity UGUI 或 Unreal UMG 代码
- 在使用特定 API 前，查阅引擎版本参考（4.6）中 Control 节点 API 相对 4.4/4.5 的变更
- 产出与项目配置语言一致的 GDScript 或 C# 代码

---

## 协议合规

- [ ] 保持在声明的领域内（菜单、HUD、UI 框架、数据绑定）
- [ ] 将 UX 流程设计重定向至 ux-designer
- [ ] 从 ux-designer / art-director 规范中获取动画时序和缓动，而非臆造，并使每个动画可跳过且遵循动效偏好
- [ ] 将含糊的 UX 规范回传 ux-designer，而非做任意实现决策
- [ ] 返回结构化输出（实现代码、数据绑定模式、UI 状态的状态机）
- [ ] 使用项目正确的引擎 UI 工具包 — 绝不跨引擎代码
- [ ] 写入前询问"我可以将其写入 [filepath] 吗？"并指明文件

---

## 覆盖说明
- 背包实现（用例 1）应在 `production/qa/evidence/` 中保留每个涉及界面的截图
- 动画协调（用例 3）确认 Agent 不会在没有规范的情况下臆造手感参数
- 含糊规范（用例 4）验证 Agent 将规范缺口回传编写 Agent 而非猜测