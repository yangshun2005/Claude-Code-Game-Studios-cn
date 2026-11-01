# Agent 测试规范：unreal-specialist

## Agent 摘要
- **领域**：Unreal Engine 模式与架构——Blueprint 与 C++ 的取舍、UE 子系统（GAS、Enhanced Input、Niagara）、UE 项目结构、插件集成，以及引擎级配置
- **不负责**：美术风格与视觉方向（art-director）、服务器基础设施与部署（devops-engineer）、UI/UX 流程设计（ux-designer）
- **Gate ID**：无；将 Gate 裁决交由 technical-director

---

## 静态断言（结构性）

- [ ] `description:` 字段存在且领域明确（提及 Unreal Engine）
- [ ] `tools:` 读取 Read、Glob、Grep、Write、Edit、Bash 和 `Agent`，且其正文要求它只启动四个子专家（ue-gas、ue-blueprint、ue-replication、ue-umg），因为子 Agent 的 `tools` 中的类型列表会被忽略
- [ ] 模型层级为 `inherit`——frontmatter 的 `model:` 恰好为 `inherit`（层级说明：`.claude/docs/model-tiers.md`）
- [ ] Agent 定义不主张其声明领域之外的权限（不涉及美术，不涉及服务器基础设施）

---

## 测试用例

### 用例 1：领域内请求——Blueprint 与 C++ 的决策标准
**输入**："我们的连招攻击系统应该用 Blueprint 还是 C++ 实现？"
**预期行为**：
- 应用其既定默认：连招系统的框架用 C++，内容和原型用 Blueprint
- 将战斗逻辑交由 GAS 处理（连招攻击作为 Gameplay Ability，连招状态作为 Gameplay Tag，蒙太奇流程通过 Ability Task），并指定 ue-gas-specialist 负责 GAS 设计，而非自行实现
- 建议将设计师可调的值放在 Blueprint（用 `EditAnywhere` / `BlueprintReadWrite` 暴露，连招变体用纯数据 Blueprint，设计师覆盖行为处用 `BlueprintNativeEvent`）
- 引用其约 20 节点/函数的阈值，作为 Blueprint 连招逻辑应迁入 C++ 的临界点
- 在不知晓项目上下文时不给出最终裁决——上下文缺失时提出澄清问题
- 以带权衡的类结构提案呈现拆分方案，而非自由发挥的意见

### 用例 2：领域外请求——Unity C# 代码
**输入**："给我写一个 C# MonoBehaviour，处理玩家生命值，并在死亡时触发 Unity 事件。"
**预期行为**：
- 不产出 Unity C# 代码
- 回答前先从 `project.yaml` 的 `engine.name` 确认项目引擎：若配置为 Unreal，说明项目基于 Unreal Engine 5 构建；若未设置，则询问项目使用哪个引擎，而非假设
- 若配置为 Unreal，将请求映射到其自身标准规定的 Unreal 等价物：使用 GAS 时，生命值作为 Attribute Set 属性，仅通过 Gameplay Effect 变更；否则用 C++ `UActorComponent` 向 Blueprint 暴露死亡事件
- 不转交 unity-specialist——该 Agent 仅服务 Unity 项目，而本 Agent 的 `Agent` 授权仅覆盖其四个 Unreal 子专家

### 用例 3：领域边界——UE5.4 API 需求
**输入**："我需要使用 UE5.4 引入的新 Motion Matching API。"
**预期行为**：
- 指出 UE5.4 超出模型训练覆盖范围（`docs/engine-reference/unreal/VERSION.md` 的 Knowledge Gap Warning 将覆盖范围定为约 5.3，并将 5.4 列为 HIGH 风险版本）
- 在信任任何 API 建议前先查阅 `docs/engine-reference/unreal/`
- 说明 `docs/engine-reference/unreal/` 下无文件记录 Motion Matching，并将其提供的任何类名或节点名标注为未针对锁定版本验证
- 不在无警示的情况下静默产出过时或错误的 API 签名

### 用例 4：冲突——核心系统中的 Blueprint 面条代码
**输入**："我们的复制逻辑完全在一个深度嵌套的 Blueprint 事件图中，300+ 节点且没有函数。已经难以维护了。"
**预期行为**：
- 将其识别为 Blueprint 架构问题，而非次要风格问题
- 建议将核心复制逻辑迁移到 C++ ActorComponent 或 GameplayAbility 系统
- 指出所需的协调：复制架构的变更必须涉及 lead-programmer
- 不在未向用户说明重构范围的情况下单方面宣布"迁移到 C++"
- 产出具体的迁移建议，而非含糊的提议

### 用例 5：上下文传递——版本适配的 API 建议
**输入上下文**：项目引擎参考文件标明 Unreal Engine 5.3。
**输入**："如何为新角色设置 Enhanced Input 动作？"
**预期行为**：
- 使用 UE5.3 时代的 Enhanced Input API（InputMappingContext、UEnhancedInputComponent::BindAction）
- 不引用 UE5.3 之后引入的 API，除非标注其可能不可用
- 在回答中引用项目声明的引擎版本
- 提供具体、锚定版本的代码或 Blueprint 节点名

---

## 协议合规

- [ ] 保持在声明领域内（Unreal 模式、Blueprint/C++、UE 子系统）
- [ ] 对 Unity 或其他引擎的请求以 Unreal 等价物回答，绝不给出错误引擎的代码
- [ ] 在实现前提出类结构、数据流和权衡（实现工作流第 3 步），而非自由发挥的意见
- [ ] 在产出 API 建议前明确标注版本不确定性，并引用 `docs/engine-reference/unreal/`
- [ ] 架构级重构与 lead-programmer 协调，而非单方面决定
- [ ] 写入前询问"我可以将其写入 [filepath] 吗？"并指明文件

---

## 覆盖说明
- Agent 行为测试无自动化运行器——这些测试需人工审查或通过 `/skill-test` 执行
- 版本感知（用例 3、用例 5）是本 Agent 最高风险的失败模式；引擎版本变更时应定期测试
- 用例 3 和用例 5 仅在 Agent 被引导至 `docs/engine-reference/unreal/` 时通过；`CLAUDE.md` 仅在 `/setup-engine` 选择 Unreal 后才导入 Unreal 的 `VERSION.md`，因此 Agent 自身文件必须携带该引导
- 用例 4 与 lead-programmer 的集成是协调测试，而非技术正确性测试