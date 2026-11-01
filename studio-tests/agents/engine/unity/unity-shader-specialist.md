# Agent 测试规范：unity-shader-specialist

## Agent 摘要
领域：Unity Shader Graph、自定义 HLSL、VFX Graph、URP/HDRP 管线定制、后处理效果。
不负责：玩法代码、美术风格方向。
未分配门禁 ID。

---

## 静态断言（结构性）

- [ ] `description:` 字段存在且领域明确（引用 Shader Graph / HLSL / VFX Graph / URP / HDRP）
- [ ] `tools:` 列表包含 Read、Write、Edit、Glob、Grep
- [ ] 模型层级为 `sonnet` —— frontmatter 中 `model:` 精确为 `sonnet`（层级说明：`.claude/docs/model-tiers.md`）
- [ ] Agent 定义未声称对玩法代码或美术方向拥有权限

---

## 测试用例

### 用例 1：领域内请求 —— 输出恰当
**输入：**“在 URP 中用 Shader Graph 为角色创建描边效果。”
**预期行为：**
- 产出 Shader Graph 节点搭建说明（顶点偏移的反向外壳，或屏幕空间深度/法线边缘检测），按命名约定命名（如 `SG_Char_Outline`），可复用逻辑放入 Sub Graph
- 优先使用 Shader Graph，仅在 Shader Graph 无法实现时才降级到自定义 HLSL；任何 HLSL 都保持 SRP Batcher 兼容（`UnityPerMaterial` CBUFFER）
- 屏幕空间方案使用 URP `ScriptableRendererFeature` / `ScriptableRenderPass`，通过 RenderGraph API（`RecordRenderGraph`）录制，而非已弃用的 `Execute` / CommandBuffer 路径 —— 依据 `docs/engine-reference/unity/modules/rendering.md`
- 未确认渲染管线前，不产出 HDRP 专用节点

### 用例 2：领域外重定向
**输入：**“用代码实现角色血条 UI。”
**预期行为：**
- 不产出 UI 实现代码
- 明确说明 UI 实现属于 `ui-programmer`（或 `unity-ui-specialist`）
- 恰当重定向请求
- 保留的部分仅为着色器驱动的效果（如血条上的溶解或填充渐变）—— 不编写任何血条 UI 代码

### 用例 3：HDRP 自定义 Pass 描边
**输入：**“我们用的是 HDRP，想把描边做成后处理效果。”
**预期行为：**
- 产出其自身规范命名的 HDRP 自定义 Pass 模式：继承 `CustomPass` 的 C# 类，由 Custom Pass Volume 运行，绘制全屏边缘检测着色器，采样深度/法线缓冲
- 使用 `docs/engine-reference/unity/current-best-practices.md` 记录的 HDRP 重写方法 —— `Setup(ScriptableRenderContext, CommandBuffer)`、`Execute(CustomPassContext ctx)`、`Cleanup()` —— 绝不使用 URP 的 `RecordRenderGraph`；用到的其他 HDRP 成员（渲染目标辅助方法、`CustomPassContext` 字段）标记为未验证 —— 除这三个方法外不从记忆中断言任何内容
- 说明 CustomPass 需要 HDRP 包，在 URP 中不可用
- 提供 HDRP 专用代码前先确认项目使用 HDRP

### 用例 4：VFX Graph 性能 —— 粒子预算
**输入：**“爆炸 VFX Graph 每个事件有 10,000 个粒子，同时生成 20 个爆炸导致 GPU 帧率尖峰。”
**预期行为：**
- 识别 GPU 粒子生成是开销来源（同时 200,000 个粒子）
- 为每个效果设置粒子容量上限，并按 VFX GPU 总预算 < 2ms 来确定容量
- 池化爆炸 VFX 实例并通过事件触发，而非每次爆炸都创建图
- 降低远处或屏幕外爆炸的开销（粒子 LOD、基于包围盒的剔除），并按质量档位缩放数量
- 避免任何将 GPU 粒子数据回读到 CPU 的修复方案
- 不修改玩法事件系统 —— 提出 VFX 侧的预算方案

### 用例 5：上下文传递 —— 渲染管线（URP 或 HDRP）
**输入：**项目上下文：URP 渲染管线，目标平台 PC 和 Nintendo Switch。请求：“添加景深后处理。”
**预期行为：**
- 使用 URP Volume 框架：Volume Profile 中的 `DepthOfField` 重写，脚本通过 `profile.TryGet<>()` 访问，如 `docs/engine-reference/unity/modules/rendering.md` 所示
- 不使用 HDRP Volume 组件（如 HDRP 参数名不同的 `DepthOfField`）
- 放在 Global Volume 上作为基准外观，或放在局部 Volume 上针对特定区域，并设置优先级和混合距离
- 按平台和质量档位控制效果 —— 在 Switch 上降低或禁用 —— 并将后处理控制在 1-2ms 帧预算内

---

## 协议合规

- [ ] 保持在声明的领域内（Shader Graph、HLSL、VFX Graph、URP/HDRP 定制）
- [ ] 将玩法和 UI 代码重定向到合适的 Agent
- [ ] 返回结构化输出（节点图说明、HLSL 代码、CustomPass 模式）
- [ ] 区分 URP 和 HDRP 方案 —— 绝不混用管线专用 API
- [ ] 自定义 URP Pass 使用 RenderGraph API，而非已弃用的 `Execute` / CommandBuffer 路径
- [ ] 产出的 VFX 优化不改变玩法行为
- [ ] 写入前询问“May I write this to [filepath]?”并指明文件

---

## 覆盖说明
- 描边效果（用例 1）应在 `production/qa/evidence/` 中配套视觉截图测试
- HDRP CustomPass（用例 3）确认 Agent 产出正确的 Unity 模式，而非通用后处理方案
- 管线区分（用例 5）验证 Agent 在无上下文时绝不假设渲染管线