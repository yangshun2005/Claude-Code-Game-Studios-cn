# Agent 测试规范：godot-shader-specialist

## Agent 摘要
领域：Godot 着色语言（GLSL 衍生）、可视化着色器（VisualShader 图）、材质设置、粒子着色器和后处理效果。
不负责：游戏玩法代码、美术风格方向。
未分配门禁 ID。

---

## 静态断言（结构性）

- [ ] `description:` 字段存在且领域明确（引用 Godot 着色语言 / 材质 / 后处理）
- [ ] `tools:` 列表包含 Read、Write、Edit、Glob、Grep
- [ ] 模型层级为 `sonnet` — frontmatter 中 `model:` 精确为 `sonnet`（层级说明：`.claude/docs/model-tiers.md`）
- [ ] Agent 定义引用 `docs/engine-reference/godot/VERSION.md` 作为 Godot 着色器 API 变更的权威来源

---

## 测试用例

### 用例 1：领域内请求 — 输出恰当
**输入：** "为 Godot 中的敌人死亡编写一个溶解效果着色器。"
**预期行为：**
- 生成有效的 Godot 着色语言代码（不是 HLSL，也不是直接写 GLSL）
- 按需使用 `shader_type spatial;` 或 `canvas_item`
- 定义 `uniform float dissolve_amount : hint_range(0.0, 1.0);`
- 采样噪声纹理以确定逐像素溶解阈值
- 对低于阈值的像素使用 `discard;`
- 可选：在溶解边界附近用自发光添加边缘辉光
- 代码在 Godot 着色语言中语法正确

### 用例 2：HLSL 重定向
**输入：** "为这个溶解效果编写一个 HLSL 计算着色器。"
**预期行为：**
- 不生成 HLSL 代码
- 说明 Godot 材质使用 Godot 自有着色语言（`.gdshader`，GLSL 衍生），而非 HLSL
- 将 HLSL 意图转换为等效的 `.gdshader` 方案（噪声阈值、`discard`、溶解边缘自发光）
- 若确实需要计算着色器，指出计算是独立的底层路径（RenderingDevice；Compatibility 渲染器不可用），并引入 godot-gdextension-specialist 处理计算着色器卸载

### 用例 3：截止后 API 变更 — 着色器纹理类型（Godot 4.4）
**输入：** "在着色器中使用 `texture()` 配合 sampler2D 采样噪声纹理。"
**预期行为：**
- 编写着色器前先查版本参考（`VERSION.md`，然后 `breaking-changes.md` 和 `modules/rendering.md`）
- 识别 4.4 变更的范围：引擎侧着色器纹理参数/返回类型从 `Texture2D` 改为 `Texture` — 涉及处理着色器纹理的脚本和引擎 API 代码，而非着色语言语法
- 使用 `uniform sampler2D noise_texture;` 配合 `texture(noise_texture, UV)` — 即其自身模式所用的形式；参考文档未记录着色语言采样语法的变更
- 不因 4.4 条目而臆造 4.4–4.6 的采样语法变更，也不重写 `sampler2D` / `texture()`

### 用例 4：片元着色器 LOD 策略
**输入：** "水面片元着色器有 8 次纹理采样，在中端硬件上造成 GPU 瓶颈。"
**预期行为：**
- 识别逐片元纹理采样次数为主要开销来源
- 提出 LOD 策略：
  - 远处降低采样次数，使用简化 LOD 材质处理远处水面，而非在片元着色器中做逐像素距离分支
  - 将无需逐像素计算的 UV/滚动运算移入顶点着色器，通过 `varying` 传递
  - 确保水面纹理已生成 mipmap（如 `filter_linear_mipmap`），使远处采样读取更小的 mip 层级
- 提供实现 LOD 方案的着色器代码修改
- 不改变水系统的游戏玩法行为

### 用例 5：上下文传递 — Godot 4.6 辉光重构
**输入：** 引擎版本上下文：Godot 4.6。请求："为场景添加泛光/辉光后处理效果。"
**预期行为：**
- 引用 VERSION.md 说明：Godot 4.6 包含辉光重构
- 由于 `VERSION.md` 记录 `Installed at pin time` 为 NOT DETERMINED，在针对 4.6 辉光行为调参前先询问安装的编辑器版本
- 在 `WorldEnvironment` 节点的 `Environment` 资源上给出辉光配置指导
- 说明文档记录的 4.6 变更：辉光现在在色调映射之前处理（此前为之后），采用屏幕混合 — 因此在早期版本上调好的辉光强度/混合会看起来不同，可能需要重新调参
- 不臆造 `breaking-changes.md` 和 `modules/rendering.md` 未记录的辉光属性重命名或移除
- 标记任何因截止后时序导致 LLM 训练数据可能信息不正确的属性

---

## 协议合规

- [ ] 保持在声明领域内（Godot 着色语言、材质、VFX 着色器、后处理）
- [ ] 将游戏玩法代码请求重定向至 gameplay-programmer
- [ ] 将材质和后处理着色器写为 `.gdshader`（Godot 着色语言）— 绝不使用 HLSL
- [ ] 检查引擎版本参考以应对截止后着色器 API 变更（4.4 纹理类型、4.6 辉光重构）
- [ ] 返回结构化输出（着色器代码附带 uniform 文档、LOD 策略附带性能依据）
- [ ] 标记任何截止后 API 用法为需要验证
- [ ] 写入前询问"我可以将其写入 [filepath] 吗？"并指明文件

---

## 覆盖说明
- 溶解着色器（用例 1）应搭配 `production/qa/evidence/` 中的视觉测试截图
- 纹理类型用例（用例 3）确认 Agent 在应用截止后变更前先阅读该变更实际覆盖的内容，而非重写变更未触及的语法
- 辉光重构（用例 5）是 Godot 4.6 专属测试 — 验证 Agent 应用截止后迁移说明