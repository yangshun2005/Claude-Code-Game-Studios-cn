# Agent 测试规范：ue-gas-specialist

## Agent 摘要
- **领域**：Gameplay Ability System（GAS）——技能（UGameplayAbility）、GameplayEffect、属性集（UAttributeSet）、GameplayTag、技能任务（UAbilityTask）、技能规格（FGameplayAbilitySpec）、GAS 预测与延迟补偿
- **不负责**：技能状态的 UI 显示（ue-umg-specialist）、超出 GAS 内置预测范围的 GAS 数据网络复制（ue-replication-specialist）、技能反馈的美术或 VFX（technical-artist）
- **Gate ID**：无；跨领域调用交由对应专家处理

---

## 静态断言（结构性）

- [ ] `description:` 字段存在且领域明确（引用 GAS、技能、GameplayEffect、AttributeSet）
- [ ] `tools:` 为 Read、Glob、Grep、Write、Edit、Bash——不含 `Agent` 工具
- [ ] 模型层级为 `sonnet`——frontmatter 的 `model:` 精确为 `sonnet`（层级说明见 `.claude/docs/model-tiers.md`）
- [ ] Agent 定义未声称对 UI 实现或底层网络序列化拥有权限

---

## 测试用例

### 用例 1：领域内请求——带冷却的冲刺技能
**输入**："实现一个冲刺技能，让玩家向前移动 500 单位，冷却 1.5 秒。"
**预期行为**：
- 产出 GAS AbilitySpec 结构或大纲：技能继承项目的基础技能类（而非原生 UGameplayAbility），包含 ActivateAbility 逻辑、用于移动的 AbilityTask（如 AbilityTask_ApplyRootMotionMoveToForce 或自定义 root motion），以及用于冷却的 UGameplayEffect
- 冷却 GameplayEffect 使用 Duration 策略，时长 1.5 秒，并用 GameplayTag 阻止重复激活
- 标签按层级约定清晰命名（如 Ability.Dash、Cooldown.Ability.Dash）
- 输出同时包含技能类大纲和 GameplayEffect 定义

### 用例 2：混合领域请求——冷却复制供 UI 使用
**输入**："如何把玩家的技能冷却状态复制到所有客户端，让 UI 正确更新？"
**预期行为**：
- 复制部分基于 GAS 内置机制作答：冷却是授予冷却标签的 Duration GameplayEffect，通过 AbilitySystemComponent 复制
- 说明 ASC 的三种复制模式（Full、Mixed、Minimal）以及各自向哪些客户端传递冷却状态
- 不为 GAS 已复制的状态编写自定义复制（RepNotify 属性或 RPC）——效果驱动的属性变更会自动复制，不得重复复制
- 将冷却指示器 UI 交给 ue-umg-specialist，将超出 ASC 内置处理的多人预测问题交给 ue-replication-specialist

### 用例 3：领域边界——GameplayTag 层级错误
**输入**："我们有个技能施加了名为 'Stunned' 的标签，另一个技能检查 'Status.Stunned'。两者匹配不上。"
**预期行为**：
- 定位根因：标签名必须完全一致，或通过 TagContainer 查询做层级匹配
- 指出命名不一致：'Stunned' 是根级标签；'Status.Stunned' 是 'Status' 下的子标签——两者是不同的标签
- 建议项目标签命名约定：所有状态效果放在 Status.* 下，所有技能放在 Ability.* 下
- 给出修复方案：将施加的标签改名为 'Status.Stunned'，或更新查询以匹配 'Stunned'
- 说明标签定义应放在哪里（DefaultGameplayTags.ini 或 DataTable）

### 用例 4：冲突——两个技能之间的属性集冲突
**输入**："我们的 Shield 技能和 Armor 技能都会修改 'DefenseValue' 属性。它们以非预期的方式叠加——两者都激活后，防御远超上限。"
**预期行为**：
- 定位原因：两个不同的 GameplayEffect 都修改 DefenseValue，且没有任何机制约束它们的合并结果
- 用自身的 Attribute Set 规则解决：为 DefenseValue 定义 min/max，由 `PreAttributeChange()` 将当前值钳制到该范围——对于即时效果对基础值的修改，按所引用的 GAS 参考文档"Clamping Attributes"所述，使用 `PreAttributeBaseChange()` 或 `PostGameplayEffectExecute()`——使任何效果组合都无法将其推过上限
- 它提出的其他方案（Execution Calculation、改变两个效果的合并方式）都建立在钳制之上，而非替代钳制；它不以叠加策略作为修复方案——叠加策略管理的是同一效果的重复施加，而非两个不同效果——也不把 Override 修饰符当作上限
- 不提议移除其中一个技能作为解决方案

### 用例 5：上下文通过——基于现有属性集设计
**输入上下文**：项目已有 AttributeSet，属性为：Health、MaxHealth、Stamina、MaxStamina、Defense、AttackPower。
**输入**："设计一个 Berserker 技能，当 Health 低于 30% 时 AttackPower 提升 50%。"
**预期行为**：
- 使用现有的 Health、MaxHealth、AttackPower 属性——不新增属性，除非先标记为对现有 AttributeSet 的变更并征询确认
- 将 +50% 实现为对 AttackPower 施加 Modifier 的 Infinite GameplayEffect——绝不直接写入 AttackPower
- 响应 Health 变化（如在 `PostGameplayEffectExecute()` 中，或通过等待事件的 Ability Task），在低于 MaxHealth 的 30% 时施加该效果，高于时移除
- 按规范记录该效果：修改什么、叠加行为、持续时间、移除条件
- 用层级 GameplayTag（如 `Effect.Buff.Berserk`）跟踪激活状态，而非布尔值
- 引用所提供 AttributeSet 中的实际属性名（AttackPower，而非 "Damage" 或 "Strength"）

---

## 协议合规

- [ ] 保持在声明的领域内（GAS：技能、效果、属性、标签、技能任务）
- [ ] 将 GAS 状态复制保留在 ASC 内（不重复复制）；技能 UI 交给 ue-umg-specialist，多人预测问题交给 ue-replication-specialist
- [ ] 返回结构化结果（技能大纲 + GameplayEffect 定义），而非模糊描述
- [ ] 主动执行标签层级命名约定
- [ ] 仅使用所提供上下文中存在的属性和标签；新增前先标记为对现有集合的变更
- [ ] 写入前询问"May I write this to [filepath]?"并指明文件

---

## 覆盖说明
- 用例 3（标签层级）是隐蔽 bug 的常见来源；标签命名约定变更时务必测试
- 用例 4 测试 Agent 自身的钳制规则（`PreAttributeChange()` 对照已定义的 min/max，以及即时效果的基础值钩子）；叠加策略和 Override 修饰符无法约束两个不同效果，因此仅作为补充方案接受
- 用例 5 是最重要的上下文感知测试；失败意味着 Agent 忽略了项目状态
- 无自动运行器；手动审查或通过 `/skill-test` 审查