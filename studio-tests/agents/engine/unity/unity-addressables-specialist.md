# Agent 测试规范：unity-addressables-specialist

## Agent 摘要
领域：可寻址资源系统（Addressable Asset System）——分组、异步加载/卸载、句柄生命周期管理、内存预算、内容目录和远程内容分发。
不负责：渲染系统（engine-programmer）、使用已加载资源的游戏逻辑（gameplay-programmer）。
未分配门禁 ID。

---

## 静态断言（结构性）

- [ ] `description:` 字段存在且与领域相关（引用 Addressables / 资源加载 / 内容目录 / 远程分发）
- [ ] `tools:` 列表包含 Read、Write、Edit、Bash、Glob、Grep
- [ ] 模型层级为 `sonnet` —— frontmatter 中 `model:` 精确为 `sonnet`（层级说明：`.claude/docs/model-tiers.md`）
- [ ] Agent 定义未声称对渲染系统或使用已加载资源的游戏逻辑拥有权限

---

## 测试用例

### 用例 1：领域内请求——输出恰当
**输入：**“异步加载一个角色纹理，并在角色销毁时释放它。”
**预期行为：**
- 产出 `Addressables.LoadAssetAsync<Texture2D>()` 调用模式
- 将返回的 `AsyncOperationHandle<Texture2D>` 存储在请求对象中
- 在角色销毁时（`OnDestroy()`），用存储的句柄调用 `Addressables.Release(handle)`
- 不使用 `Resources.Load()` 作为加载机制
- 在使用 `Result` 前检查加载是否成功（`handle.Status`），且绝不释放自己不持有的句柄（例如在加载开始前就被销毁的角色）
- 对已加载资源使用 `Addressables.Release(handle)`，对 `InstantiateAsync` 的结果保留使用 `Addressables.ReleaseInstance()` —— 不混用两者

### 用例 2：领域外重定向
**输入：**“实现将已加载纹理应用到角色网格的渲染系统。”
**预期行为：**
- 不产出渲染或网格材质赋值代码
- 明确说明渲染系统实现属于 `engine-programmer`
- 将请求重定向到 `engine-programmer`
- 转而交接资源契约：加载成功后，`Texture2D` 来自句柄的 `Result`，且仅在所属句柄释放前有效，因此使用方不得在该时间点之后继续持有它

### 用例 3：内存泄漏——未释放的句柄
**输入：**“每次关卡加载后内存占用持续攀升。我们使用 Addressables 加载关卡资源。”
**预期行为：**
- 诊断可能原因：`AsyncOperationHandle` 对象在使用后未被释放
- 识别句柄泄漏模式：将资源加载到局部变量、丢失引用、从未调用 `Addressables.Release()`
- 产出审计方法：每个 `LoadAssetAsync` 配对 `Release()`，每个 `InstantiateAsync` 配对 `ReleaseInstance()`，每个 `LoadSceneAsync` 配对 `UnloadSceneAsync()`
- 提供修正模式：每个活跃句柄保存在受跟踪的集合中，并在关卡卸载时释放，确保跨关卡切换时不累积
- 在得出结论前，用 Addressables Event Viewer（逐资源引用计数）或 Memory Profiler 确认诊断 —— 无证据时不假设泄漏在别处

### 用例 4：远程内容分发——目录版本管理
**输入：**“我们需要支持可下载内容更新，而无需用户重新安装整个应用。”
**预期行为：**
- 产出 `docs/engine-reference/unity/plugins/addressables.md` 中记录的远程目录更新模式：
  - 启动时调用 `Addressables.CheckForCatalogUpdates()`
  - 对检测到的更新调用 `Addressables.UpdateCatalogs()`
  - 下载前调用 `Addressables.GetDownloadSizeAsync()`，然后调用 `Addressables.DownloadDependenciesAsync()` 预下载更新内容
- 仅重新下载变更的 bundle；目录带版本号，客户端可回退到缓存内容；下载失败时以指数退避重试并显示进度
- 提出会话中途的情况（会话运行期间收到目录更新），询问用户希望哪种行为，而非静默选择一种
- 规划更新路径测试：全新安装、V1 到 V2、V1 到 V3 跳过 V2
- 不设计服务端 CDN 基础设施（交由 devops-engineer）

### 用例 5：上下文传递——平台内存约束
**输入：**平台上下文：目标平台 Nintendo Switch，4GB RAM，实际资源内存上限 512MB。请求：“为一个大型开放世界关卡设计 Addressables 加载策略。”
**预期行为：**
- 引用所提供上下文中的 512MB 内存上限，并用它替代自身默认的主机预算
- 设计流式加载策略：
  - 通过 `Addressables.LoadSceneAsync()` / `Addressables.UnloadSceneAsync()` 进行附加场景流式加载，按加载上下文将分组按区域组织
  - 每个活跃区域的内存预算，其总和保持在 512MB 上限之下，区域在切换时卸载而非累积
  - 关键区域内容优先加载，可选内容随后流式加载
- 将区域边界划分交由 `level-designer`
- 将目标硬件的存储 I/O 速度标记为需在设备上验证的加载时间风险，不凭记忆引用平台加载速度
- 不产出会超过所述 512MB 上限而未标记的加载策略

---

## 协议合规性

- [ ] 保持在声明的领域内（Addressables 加载、句柄生命周期、内存、目录、远程分发）
- [ ] 将渲染和游戏逻辑的资源使用代码重定向到 engine-programmer 和 gameplay-programmer
- [ ] 返回结构化输出（加载模式、句柄生命周期代码、流式区域设计）
- [ ] 始终将 `LoadAssetAsync` 与对应的 `Release()` 配对 —— 将句柄泄漏标记为内存缺陷
- [ ] 依据提供的内存上限设计加载策略，该上限覆盖其默认的按平台预算
- [ ] 仅规定客户端契约（远程加载路径 / URL 结构、目录版本管理、缓存头要求），将 CDN 托管和分发流水线交由 devops-engineer
- [ ] 在写入前询问“我可以将此写入 [filepath] 吗？”并指明文件

---

## 覆盖说明
- 句柄生命周期（用例 1）必须包含一个验证释放后内存被回收的测试
- 句柄泄漏诊断（用例 3）应产出适合作为缺陷工单的发现报告
- 平台内存用例（用例 5）验证 Agent 应用上下文中的硬约束，而非默认假设