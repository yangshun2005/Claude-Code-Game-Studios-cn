# Agent 测试规范：ue-umg-specialist

## Agent 摘要
- **领域**：UMG 控件层级设计、数据绑定模式、CommonUI 输入路由与输入操作数据、控件样式（WidgetStyle 资产）、UI 优化（控件池化、ListView、失效）
- **不负责**：UX 流程与屏幕导航设计（ux-designer）、玩法逻辑（gameplay-programmer）、后端数据源（游戏代码）、服务器通信
- **门禁 ID**：无；UX 流程决策交由 ux-designer

---

## 静态断言（结构性）

- [ ] `description:` 字段存在且领域明确（提及 UMG、控件层级、CommonUI）
- [ ] `tools:` 读取 Read、Glob、Grep、Write、Edit、Bash —— 不含 `Agent` 工具
- [ ] 模型层级为 `sonnet` —— frontmatter 中 `model:` 精确读取为 `sonnet`（层级说明：`.claude/docs/model-tiers.md`）
- [ ] Agent 定义未声称对 UX 流程、导航架构或玩法数据逻辑拥有权限

---

## 测试用例

### 用例 1：域内请求 —— 带数据绑定的背包控件
**输入**："创建一个背包控件，显示物品槽网格。每个槽位应显示物品图标、数量和稀有度颜色。背包变化时需要更新。"
**预期行为**：
- 产出 UMG 控件结构：父级 WBP_Inventory 包含 UniformGridPanel 或 TileView，每个物品对应一个子级 WBP_InventorySlot 控件
- 描述数据绑定方式：由背包组件上的事件分发器触发刷新，或用基于 UObject 的物品数据（而非原始结构体）供给 TileView；它指出条目控件的列表条目接口（`IUserObjectListEntry`），并标注该接口未在 `docs/engine-reference/unreal/` 中验证（该文档未记录此接口）
- 说明稀有度颜色的驱动方式：WidgetStyle 资产或数据表查找，而非硬编码颜色值
- 输出包含控件层级、绑定模式和刷新触发机制

### 用例 2：域外请求 —— UX 流程设计
**输入**："为我们的背包系统设计完整的导航流程 —— 玩家如何打开它、切换到角色属性、再退出到暂停菜单。"
**预期行为**：
- 不产出导航流程或屏幕切换架构
- 指出 ux-designer（其交互设计协作方）是该流程的负责人，并表示流程确定后可在其分层架构（菜单层、`UCommonActivatableWidgetStack`）上构建屏幕
- 没有 UX 规范时不擅自做 UX 决策（返回按钮行为、切换动画、模态 vs. 全屏）—— 而是索要规范或将其列为待定问题

### 用例 3：领域边界 —— CommonUI 输入操作不匹配
**输入**："我们的背包控件不响应手柄 Back 按钮。我们用的是 CommonUI。"
**预期行为**：
- 识别可能原因：控件的 Back 绑定未指向项目 `CommonInputActionDataBase` 数据表的 Back 行，或控件不是其栈上已激活、已聚焦的屏幕
- 解释 CommonUI 输入路由模型：屏幕派生自 `UCommonActivatableWidget`，将 UI 操作绑定到数据表行；只有已聚焦、已激活的控件消费输入，未聚焦控件忽略输入
- 给出修复方案：核对 Back 绑定的数据表行，并确认控件已压入其可激活控件栈且处于激活状态
- 将其与硬件输入绑定问题（属于 Enhanced Input 范畴）区分开

### 用例 4：控件性能问题 —— 每帧大量控件实例
**输入**："我们的排行榜控件一次性创建 500 个独立的 WBP_LeaderboardRow 实例。打开排行榜时游戏卡顿 300ms。"
**预期行为**：
- 识别根因：单帧内实例化 500 个控件导致构建卡顿
- 建议改用带虚拟化的 ListView 或 TileView —— 只构建可见行
- 解释 ListView 的数据要求：每行数据是基于 UObject 的条目项，而非原始结构体；它指出行控件的列表条目接口（`IUserObjectListEntry`），并标注该接口未在 `docs/engine-reference/unreal/` 中验证
- 若 ListView 不适用，建议池化：预实例化固定数量的行，用新数据循环复用
- 输出为具体建议，指明应使用的具体 UMG 组件，而非笼统的"优化一下"

### 用例 5：上下文传递 —— CommonUI 已配置
**输入上下文**：项目使用 CommonUI；其 `CommonInputActionDataBase` 数据表定义了以下输入操作行：Confirm、Back、Pause、Secondary。
**输入**："给背包控件添加一个兼容 CommonUI 的'排序背包'按钮。"
**预期行为**：
- 使用现有的 Secondary 行（或若该屏幕已占用 Secondary，则建议新增 Sort 行）
- 不擅自发明新输入操作而不说明必须将其作为行添加到项目的 CommonUI 输入操作数据表
- 按钮使用 `UCommonButtonBase` 和 CommonUI 的输入路由 —— 而非非 CommonUI 绑定（例如事件图表中的原始按键或 `APlayerController::InputComponent`）
- 在建议中明确引用所提供的操作行

---

## 协议合规性

- [ ] 保持在声明的领域内（UMG 结构、数据绑定、CommonUI、控件性能）
- [ ] 将 UX 流程和导航设计请求转交 ux-designer，自身不做任何有约束力的 UX 决策
- [ ] 返回结构化发现（控件层级 + 绑定模式），而非自由形式的意见
- [ ] 使用上下文中现有的 CommonUI 输入操作行；不擅自发明新行而不标注必须将其添加到数据表
- [ ] 对大型集合，先推荐虚拟化列表（ListView/TileView），再考虑控件池化
- [ ] 写入前询问"我可以将其写入 [filepath] 吗？"并指明文件

---

## 覆盖说明
- 用例 3（CommonUI 输入路由）要求项目已配置 CommonUI；若项目不使用 CommonUI 则跳过该测试
- 用例 4（性能）是高影响失败模式 —— 300ms 卡顿会阻塞发布；优先测试此用例
- 用例 5 是 UI 流水线一致性方面最重要的上下文感知测试
- 无自动化运行器；手动审查或通过 `/skill-test` 审查