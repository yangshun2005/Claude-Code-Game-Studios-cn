# Agent 测试规范：unity-dots-specialist

## Agent 摘要
领域：ECS 架构（IComponentData、ISystem、SystemAPI）、Jobs 系统（IJob、IJobEntity、Burst）、Burst 编译器约束、DOTS 游戏玩法系统、混合渲染器。
不负责：MonoBehaviour 游戏玩法代码（gameplay-programmer）、UI 实现（unity-ui-specialist）。
未分配门禁 ID。

---

## 静态断言（结构性）

- [ ] `description:` 字段存在且领域明确（引用 ECS / Jobs / Burst / IComponentData）
- [ ] `tools:` 列表包含 Read、Write、Edit、Bash、Glob、Grep
- [ ] 模型层级为 `sonnet` —— frontmatter 中 `model:` 精确为 `sonnet`（层级说明：`.claude/docs/model-tiers.md`）
- [ ] Agent 定义未声称对 MonoBehaviour 游戏玩法或 UI 系统拥有权限

---

## 测试用例

### 用例 1：领域内请求 —— 输出恰当
**输入：** "把玩家移动系统转换为 ECS。"
**预期行为：**
- 产出：
  - `PlayerMovementData : IComponentData` 结构体，包含 velocity、speed 和 input vector 字段
  - `PlayerMovementSystem : ISystem`，其 `OnUpdate()` 使用 `SystemAPI.Query<>` 或 `IJobEntity`
  - 通过 `Baker<T>` 子类从 authoring MonoBehaviour 烘焙玩家初始状态（模式见 `docs/engine-reference/unity/plugins/dots-entities.md`）
- 使用 `RefRW<LocalTransform>` 更新位置（而非已弃用的 `Translation`）
- 为 job 标记 `[BurstCompile]`，并说明哪些内容必须为非托管类型才能兼容 Burst
- 每个系统只负责一件事：移动系统从组件（如 `PlayerInputData`）读取输入，由独立的输入系统写入，而非自行轮询输入；在创建或修改该输入侧之前先询问

### 用例 2：MonoBehaviour 反驳
**输入：** "玩家移动就用 MonoBehaviour 吧——更简单。"
**预期行为：**
- 认可简单性论点
- 解释 DOTS 的取舍：前期配置更多，但 ECS/Burst 方案能提供项目 ADR 或需求中记录的性能特性
- 若项目已承诺使用 DOTS，则不实现 MonoBehaviour 版本
- 若无此承诺，将 MonoBehaviour 与 DOTS 的决策上报给 `unity-specialist`（其指定的 Unity 整体架构联系人），技术冲突无法解决时升级至 `technical-director`
- 不单方面做出 MonoBehaviour 与 DOTS 的决策

### 用例 3：Burst 不兼容的托管内存
**输入：** "这个 Burst job 访问 `List<EnemyData>` 来查找最近的敌人。"
**预期行为：**
- 指出 `List<T>` 是托管类型，与 Burst 编译不兼容
- 不批准带托管内存访问的 Burst job
- 提供正确的替代方案：`NativeArray<EnemyData>`、`NativeList<EnemyData>` 或 `NativeHashMap<>`，视用例而定
- 说明原生容器必须释放，分配器按生命周期选择（帧级用 `Allocator.TempJob`，长期存活用 `Allocator.Persistent`），已知大小时预分配容量
- 使用非托管原生容器产出修正后的 job

### 用例 4：混合访问 —— DOTS 系统需要 MonoBehaviour 数据
**输入：** "DOTS 移动系统需要读取由 MonoBehaviour CameraController 管理的相机 transform。"
**预期行为：**
- 识别这是混合访问场景
- 不将 `Transform` 或 MonoBehaviour 本身放入组件——组件不持有对托管对象的引用
- 以非托管值（`Unity.Mathematics` 类型）在 `IComponentData` 中跨边界传递相机数据，在 MonoBehaviour 侧一个明确的交接点写入、由系统读取——不按实体逐个获取
- 不在 Burst job 内访问 MonoBehaviour——将 Burst 代码中的托管访问标记为编译错误

### 用例 5：上下文传递 —— 性能目标
**输入：** 上下文中的技术偏好：60fps 目标，每帧 CPU 脚本预算最多 2ms。请求："为 10,000 个敌人实体设计 ECS chunk 布局。"
**预期行为：**
- 在设计理由中明确引用 2ms CPU 预算
- 为缓存效率设计 `IComponentData` chunk 布局：
  - 按系统访问模式而非游戏概念拆分组件——不要"上帝组件"
  - 保持热组件小巧，少用数据放入独立组件
  - 用标签组件做过滤，避免会碎片化 archetype 的 `ISharedComponentData`，对切换状态使用 `IEnableableComponent` 而非结构性变更
- 说明预算将如何验证——Entities Profiler 的系统耗时和按 archetype 的内存——而非将未测量的耗时当作事实呈现
- 不设计明显超出所述 2ms 预算的布局而不加提示

---

## 协议合规

- [ ] 保持在声明的领域内（ECS、Jobs、Burst、DOTS 游戏玩法系统）
- [ ] 将仅限 MonoBehaviour 的游戏玩法转交 gameplay-programmer
- [ ] 返回结构化输出（IComponentData 结构体、ISystem 实现、`Baker<T>` authoring 类）
- [ ] 将 Burst job 中的托管内存访问标记为编译错误，并提供非托管替代方案
- [ ] 当 DOTS 系统需要与 MonoBehaviour 系统交互时，提供混合访问模式
- [ ] 依据给定的性能预算设计 chunk 布局，并说明验证所用的性能分析步骤
- [ ] 写入前询问"我可以将其写入 [filepath] 吗？"并指明文件

---

## 覆盖说明
- ECS 转换（用例 1）必须包含使用 ECS 测试框架（`World`、`EntityManager`）的单元测试，位于 `Assets/Tests/EditMode/` 下
- Burst 不兼容（用例 3）是安全关键——Agent 必须在代码编写前捕获此问题
- Chunk 布局（用例 5）验证 Agent 依据所述预算进行设计，并说明如何测量，而非断言未测量的数字