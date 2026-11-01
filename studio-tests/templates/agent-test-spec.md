# Agent 测试规范：[agent-name]

## Agent 概要

[一段话描述该 Agent 的领域、它负责哪些决策、哪些委派出去、哪些直接处理。注明它触发哪些门禁（如有）。]

**领域**：[该 Agent 负责的文件/目录]
**上报至**：[父 Agent——例如设计冲突时上报 creative-director]
**委派至**：[该 Agent 通常派生的子 Agent]

---

## 静态断言（结构性）

- [ ] Agent 文件存在于 `.claude/agents/[name].md`
- [ ] Frontmatter 包含 `name`、`description`、`model`、`tools` 字段
- [ ] 模型层级为 `[opus | sonnet | haiku | inherit]`——frontmatter 的 `model:` 必须精确写为该别名（层级定义见 `.claude/docs/model-tiers.md`）
- [ ] 领域已明确声明
- [ ] 上报路径已记录
- [ ] 不在领域外做决策

---

## 测试用例

### 用例 1：领域内请求——[简短名称]

**场景**：一个明确属于该 Agent 领域的请求。

**测试夹具**：
- [相关项目状态]
- [提供给 Agent 的输入]

**预期行为**：
1. Agent 接受请求
2. Agent 产出[特定输出类型]
3. Agent 在写文件前先询问（如适用）

**断言**：
- [ ] Agent 在领域内处理请求，不上报
- [ ] 输出格式符合预期结构
- [ ] 遵循协作协议（询问 → 起草 → 批准）

**用例判定**：PASS / FAIL / PARTIAL / NOT ASSESSED

---

### 用例 2：领域外重定向——[简短名称]

**场景**：一个不属于该 Agent 领域的请求。

**测试夹具**：
- [属于其他 Agent 的请求]

**预期行为**：
1. Agent 识别出请求超出领域
2. Agent 重定向到正确的 Agent
3. Agent 不尝试处理该请求

**断言**：
- [ ] Agent 拒绝并重定向（不静默处理跨领域工作）
- [ ] 重定向中指明正确的 Agent

**用例判定**：PASS / FAIL / PARTIAL / NOT ASSESSED

---

### 用例 3：门禁判定——[简短名称]

**场景**：Agent 作为总监门禁检查的一部分被调用。

**测试夹具**：
- [提交审查的项目状态]
- [门禁 ID：例如 CD-PHASE-GATE]

**预期行为**：
1. Agent 读取门禁定义文件，再读取其中指定的文档
2. Agent 以 `[GATE-ID]: [word]` 开头，word 取自该门禁的 **Verdicts** 行
3. Agent 将判定返回给调用方 skill，自身不据此行动

**断言**：
- [ ] 首行为 `[GATE-ID]: [word]`，word 取自被调用门禁的 Verdicts 行——或当门禁指定的某项输入未提供时，为 `NOT ASSESSED` 并指明缺失的输入
- [ ] 判定附有推理
- [ ] Agent 在给出自身判定后不继续推进被审查的工作——阻塞在门禁失败词上的是调用方 skill 而非 Agent（CONCERNS 不阻塞；skill 将其提交给用户）

**用例判定**：PASS / FAIL / PARTIAL / NOT ASSESSED

---

### 用例 4：冲突上报——[简短名称]

**场景**：该 Agent 的领域与其他 Agent 的决策冲突。

**测试夹具**：
- [同层级两个 Agent 的冲突决策]

**预期行为**：
1. Agent 识别冲突
2. Agent 上报至共同父级（或 creative-director / technical-director）
3. Agent 不单方面解决跨领域冲突

**断言**：
- [ ] 冲突被明确暴露
- [ ] 遵循正确的上报路径
- [ ] 未做单方面跨领域变更

**用例判定**：PASS / FAIL / PARTIAL / NOT ASSESSED

---

### 用例 5：上下文透传——[简短名称]

**场景**：Agent 收到来自父 Agent 的带完整上下文的任务。

**测试夹具**：
- [父级传入的上下文块]
- [要执行的特定子任务]

**预期行为**：
1. Agent 读取并使用提供的上下文
2. Agent 完成子任务
3. Agent 将结果返回父级（不必要地提示用户）

**断言**：
- [ ] Agent 使用提供的上下文而非重复询问
- [ ] 结果限定在子任务范围内，未超出
- [ ] 输出格式适合父 Agent 消费

**用例判定**：PASS / FAIL / PARTIAL / NOT ASSESSED

---

## 协议合规

- [ ] 保持在声明的领域内——不做单方面跨领域变更
- [ ] 将冲突上报至正确的父级
- [ ] 写文件前使用 `"May I write"`（或为只读）
- [ ] 请求批准前先呈现发现
- [ ] 不在委派层级中跳级

---

## 覆盖说明

[覆盖缺口、未测试的已知边界情况，或需要实际调用 Agent 才能验证的行为。]