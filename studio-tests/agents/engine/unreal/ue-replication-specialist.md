# Agent 测试规范：ue-replication-specialist

## Agent 摘要
- **领域**：属性复制（UPROPERTY Replicated/ReplicatedUsing）、RPC（Server/Client/NetMulticast）、客户端预测与校正、网络相关性与始终相关设置、网络序列化（FArchive/NetSerialize）、带宽优化与复制频率调优
- **不负责**：被复制的游戏逻辑（gameplay-programmer）、传输层网络与匹配（network-programmer，其与 devops-engineer 协调服务器托管）、GAS 专属预测（ue-gas-specialist 处理 GAS 网络预测）
- **Gate ID**：无；将安全相关的复制问题提交给 security-engineer

---

## 静态断言（结构性）

- [ ] `description:` 字段存在且领域明确（提及复制、RPC、客户端预测、带宽）
- [ ] `tools:` 读取 Read、Glob、Grep、Write、Edit、Bash —— 无 `Agent` 工具
- [ ] 模型层级为 `sonnet` —— frontmatter `model:` 精确读取为 `sonnet`（层级：`.claude/docs/model-tiers.md`）
- [ ] Agent 定义不声称对服务器基础设施、游戏服务器架构或游戏逻辑正确性拥有权限

---

## 测试用例

### 用例 1：领域内请求 —— 带客户端预测的复制玩家生命值
**输入**："设置复制的玩家生命值，客户端可在本地预测（例如受到自身造成的伤害时），并由服务器校正。"
**预期行为**：
- 在合适的 Character 或 AttributeSet 类中生成 UPROPERTY(ReplicatedUsing=OnRep_Health) 声明
- 描述 OnRep_Health 函数：应用视觉/音频反馈，将预测值与服务器权威值校正
- 解释客户端预测模式：本地客户端立即施加暂定伤害，服务器权威值通过 OnRep 到达并校正任何偏差
- 指出若使用 GAS，内置 GAS 预测会处理此问题 —— 建议与 ue-gas-specialist 协调
- 输出为具体代码结构（属性声明 + OnRep 概要），而非仅概念描述

### 用例 2：领域外请求 —— 游戏服务器架构
**输入**："设计我们的游戏服务器基础设施 —— 需要多少专用服务器、区域部署以及匹配架构。"
**预期行为**：
- 不产出服务器基础设施架构、托管建议或匹配设计
- 声明其范围是运行中游戏会话内的 Unreal 复制层，并指明 network-programmer（其在该层之下网络方面的协调伙伴）为匹配与服务器架构的路由
- 不将游戏内复制与服务器托管问题混为一谈

### 用例 3：领域边界 —— 无服务器权威校验的 RPC
**输入**："我们有一个名为 ServerSpendCurrency 的 Server RPC，用于扣除游戏内货币。客户端调用它，服务器不做任何检查就直接扣除。"
**预期行为**：
- 将其标记为严重安全漏洞：未校验的 Server RPC 可被作弊者发送任意 RPC 调用利用
- 提供所需修复：扣除前进行服务器端校验 —— 检查玩家确实拥有该货币、验证交易有效，否则拒绝并记录日志
- 校验覆盖每个客户端 RPC 所需的三项检查：该玩家现在能否执行此操作、参数是否在有效范围内、请求速率是否在限制内（对 RPC 限流）
- 指出鉴于经济影响，应由 security-engineer（其网络安全校验的指定伙伴）审查
- 不在未解释原始代码危险性的情况下产出"修复后"代码

### 用例 4：带宽优化 —— 高频移动复制
**输入**："我们的玩家移动每 tick 使用 Vector3 位置复制。32 名玩家时，我们超出了带宽预算。"
**预期行为**：
- 识别出全精度 Vector3 的 tick 频率复制带宽开销大
- 提出量化复制：使用 FVector_NetQuantize 或 FVector_NetQuantize100 替代原始 FVector，以减少每次更新的字节数
- 建议降低移动 Actor 的复制频率 —— 使用每 Actor 网络更新频率（其标准中的 `NetUpdateFrequency`），而非每客户端设置 —— 指明所用 API，并在 `docs/engine-reference/unreal/` 未记录时标记为未验证
- 指出 Unreal 内置 Character Movement Component 已有优化的移动复制 —— 建议使用或扩展它，而非自建系统
- 对照其每客户端目标（动作游戏低于 10 KB/s）衡量结果，并指明确认节省的工具（`stat net`、Network Profiler）

### 用例 5：上下文传递 —— 在网络预算内设计
**输入上下文**：项目网络预算为每玩家 64 KB/s，32 名玩家 = 服务器总出站 2 MB/s。当前移动复制已使用每玩家 40 KB/s。
**输入**："我们想添加实时库存复制，让所有客户端能立即看到其他玩家的装备变化。"
**预期行为**：
- 确认现有 40 KB/s 移动开销仅给每玩家其他所有内容留下 24 KB/s
- 指出这些项目数字远高于其自身默认目标（动作游戏每客户端低于 10 KB/s），在项目声明的预算内工作同时标记差距
- 不设计朴素的全库存复制方案（会超出预算）
- 区分其他玩家所见（已装备物品，复制给所有人）与完整库存（`COND_OwnerOnly`）
- 建议仅增量或事件驱动方案：仅复制变化的槽位而非完整库存数组，用 `ReplicatedUsing` 触发定向更新
- 明确说明所提方案相对剩余 24 KB/s 预算的带宽估算

---

## 协议合规

- [ ] 保持在声明领域内（属性复制、RPC、客户端预测、带宽）
- [ ] 将服务器基础设施与匹配请求路由至 network-programmer，不产出基础设施设计
- [ ] 将未校验的 Server RPC 标记为安全问题并建议 security-engineer 审查
- [ ] 返回结构化发现（属性声明、带宽估算、优化选项）而非自由形式建议
- [ ] 评估复制设计选择时使用项目提供的带宽预算数字
- [ ] 写入前询问"May I write this to [filepath]?"并指明文件

---

## 覆盖说明
- 用例 3（RPC 安全）是发布关键测试 —— 未校验的 RPC 是多人游戏十大漏洞利用途径之一
- 用例 5 是最重要的上下文感知测试；Agent 必须使用实际预算数字，而非通用建议
- 用例 1 GAS 分支：若配置了 GAS，Agent 应检测到并交由 ue-gas-specialist 处理 GAS 管理的属性
- 无自动化运行器；手动审查或通过 `/skill-test`