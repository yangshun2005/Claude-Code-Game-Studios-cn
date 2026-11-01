# 技能测试规范：/security-audit

## 技能摘要

`/security-audit` 审计代码库中常见的游戏安全缺陷，覆盖六大类：存档/序列化、网络/多人、输入校验、数据暴露、作弊/防篡改向量、依赖。它接收一个范围参数（`full`、`network`、`save`、`input`、`quick`；无参数时运行 `full`），从 `project.yaml` 读取 `engine.name`、`engine.language`、`platform.targets`、`platform.multiplayer` 和 `platform.online`，并以固定简报模板启动 `security-engineer` agent——仅填充其槽位：范围、引擎、`platform.*` 值、解析后的代码根目录、`assets/data/` 和配置文件、要运行的类别以及报告路径——模板自身文本要求 agent 按其严重性表为每个发现评级，且游戏类型绝不降低评级。Grep 模式按引擎从 Phase 3 表中选择；表中标记为对某引擎 NOT SOURCEABLE 的类别为 NOT ASSESSED，缺失的 `platform.multiplayer` 绝不读作 `false`。被跳过的类别会注明原因，代码根目录无法解析时所有代码类别为 NOT ASSESSED。发现分为 CRITICAL / HIGH / MEDIUM / LOW，每条包含文件、行号、攻击场景和修复建议。报告的发布建议按首个匹配项决定：DO NOT SHIP、FIX BEFORE SHIPPING、NOT ASSESSED（例如 INSUFFICIENT IMPLEMENTATION）或 CLEAR TO SHIP。技能在对话中呈现执行摘要和 CRITICAL/HIGH 发现，然后询问"May I write the full security audit report to `production/security/security-audit-[date]-[scope].md`?"。在 `workflow: minimal` 下，Phase 7 消息说明发布门禁不读取该报告。不调用任何 director 门禁。

---

## 静态断言（结构性）

由 `/skill-test spec` 针对 SKILL.md 检查——无需 fixture。

- [ ] 具有必需 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含判定关键词：CLEAR TO SHIP、FIX BEFORE SHIPPING、DO NOT SHIP、NOT ASSESSED，以及严重性 CRITICAL / HIGH / MEDIUM / LOW
- [ ] 包含"May I write"措辞并指明 `production/security/security-audit-[date]-[scope].md`
- [ ] 具有下一步交接（Phase 7：`/security-audit quick`、`/gate-check release`、`/launch-checklist`）

---

## Director 门禁检查

无。技能启动 `security-engineer` 专家来运行审计；这是其核心机制，不是 director 门禁。任何审查模式下都不调用 director 门禁。

---

## Fixture 默认值

除非自身 Fixture 另有说明，每个用例的 `project.yaml` 均设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，且无 `project.local.yaml`。规则见：`studio-tests/CLAUDE.md` § Fixture 默认值。

---

## 测试用例

### 用例 1：正常路径——单人 Godot 项目，无发现

**Fixture：**
- `project.yaml`：`engine.name: godot`、`engine.language: gdscript`、`platform.multiplayer: false`、`platform.online: false`、`modes.rigor: standard`（因此 `workflow` 解析为 `standard`）；`modes.automation` 未设置（collaborative）
- `src/` 包含已实现的游戏：核心、玩法、UI 和存档系统共 42 个 `.gd` 文件
- `src/core/save_system.gd` 使用 `FileAccess` 打开存档，校验校验和，并在使用前对每个加载值做边界检查
- `src/` 或 `assets/` 中无任何 API 密钥、机密、密码或令牌
- `addons/gdUnit4/` 是唯一的第三方依赖

**输入：** `/security-audit`

**预期行为：**
1. 无参数 → `full` 范围
2. 技能从 `project.yaml` 读取引擎、语言和平台键
3. 技能通过 `Agent` 以固定简报模板启动 `security-engineer`：范围、Godot/GDScript、`platform.multiplayer: false`、`platform.online: false`、代码根目录 `src/`、`assets/data/` 和配置文件、要运行的类别以及报告路径——槽位之外不添加任何内容
4. 类别 1 使用 Godot 模式列表，包括 Godot 4 名称 `FileAccess`；类别 2 因两个平台键均显式为 `false` 而跳过，执行摘要注明跳过及其原因
5. 类别 6 在依赖清单中列出 `addons/gdUnit4/`
6. 执行摘要显示 42 个源文件在各严重性级别均为零发现；发布建议：**CLEAR TO SHIP**
7. 技能呈现摘要，然后询问"May I write the full security audit report to `production/security/security-audit-[date]-full.md`?"
8. 写入后："✅ No blocking security findings. Report written to `production/security/`. Include this path when running `/gate-check release`."

**断言：**
- [ ] `security-engineer` 以固定简报模板启动——范围、引擎/语言、`platform.*` 值以及以 `src/` 为根的源清单——且简报不添加严重性评级，也不评论游戏类型
- [ ] 存档处理使用 Godot 4 名称（`FileAccess`）检查，而不仅是 Godot 3 的 `File.open`
- [ ] 扫描硬编码凭据模式（`api_key`、`secret`、`password`、`token`、`private_key`）
- [ ] 执行摘要说明类别 2 因 `platform.multiplayer` 和 `platform.online` 为 `false` 而跳过——跳过不是静默的
- [ ] 发布建议为 CLEAR TO SHIP
- [ ] "May I write"在报告写入之前出现

---

### 用例 1b：单人，一个 HIGH 发现——FIX BEFORE SHIPPING

**Fixture：**
- 同用例 1，但 `src/core/save_system.gd` 加载存档值时无校验和或边界检查，因此编辑过的存档文件可解锁后期进度
- 无其他发现

**输入：** `/security-audit`

**预期行为：**
1. 存档发现分类为 **HIGH**（绕过进度的存档篡改），且**不**提升为 CRITICAL，因为 `platform.multiplayer` 为 `false`
2. 发布建议：**FIX BEFORE SHIPPING**——存在未解决的 HIGH，无 CRITICAL
3. 写入后："⚠️ 1 HIGH finding(s) are open. The Polish → Release gate's security audit item requires none — fix them and re-run `/security-audit quick` before `/gate-check release`."

**断言：**
- [ ] 发送给 `security-engineer` 的简报是固定模板：不称存档发现为 HIGH（或任何严重性），也不说单人游戏降低标准——agent 从其自身表评级
- [ ] 在单人游戏中该发现为 HIGH，而非 CRITICAL
- [ ] 发布建议为 FIX BEFORE SHIPPING——存在未解决 HIGH 时绝不 CLEAR TO SHIP，无 CRITICAL 时也不 DO NOT SHIP
- [ ] ⚠️ HIGH 消息将发布门禁的安全审计项列为要求，并指明 `/security-audit quick` 和 `/gate-check release`
- [ ] 变体——在 `modes.rigor: minimal` 下建议仍为 FIX BEFORE SHIPPING，且消息说明 `minimal` 发布门禁不读取此报告，而非指明门禁项

---

### 用例 2：多人 CRITICAL——客户端权威 RPC 和硬编码密钥

**Fixture：**
- `project.yaml`：`engine.name: godot`、`platform.multiplayer: true`、`platform.online: true`
- `src/net/combat_sync.gd` 第 42 行：`@rpc("any_peer") func apply_damage(target_id, amount)` 应用 `amount` 时无发送者或范围校验
- `src/online/leaderboard_client.gd` 第 8 行：`const API_KEY = "sk_live_..."`

**输入：** `/security-audit full`

**预期行为：**
1. 类别 2 运行（multiplayer 为 `true`），使用 Godot 网络模式（`MultiplayerPeer`、`rpc`、`@rpc` 等）
2. 未校验的 `apply_damage` RPC 为 CRITICAL 发现（客户端决定结果，破坏多人完整性）
3. 硬编码密钥为凭据暴露（HIGH），因游戏为多人而按 CRITICAL 处理
4. 每条发现写为 `SEC-NNN`，含类别、文件和行号、描述、攻击场景、修复建议和工作量
5. 发布建议为 DO NOT SHIP——未解决的 CRITICAL 决定此结果
6. 技能呈现执行摘要和 CRITICAL/HIGH 发现，然后询问"May I write"
7. Phase 7："⛔ CRITICAL security findings must be resolved before any public release. Do not proceed to `/launch-checklist` until these are addressed." 并在 `/gate-check release` 前重新运行 `/security-audit quick`

**断言：**
- [ ] RPC 发现引用 `src/net/combat_sync.gd` 及其行号和攻击场景
- [ ] 硬编码密钥在多人上下文中提升为 CRITICAL
- [ ] 每条发现都带有具体修复建议
- [ ] 发布建议为 DO NOT SHIP（存在未解决 CRITICAL 发现）——绝不 FIX BEFORE SHIPPING 或 CLEAR TO SHIP
- [ ] 显示 ⛔ 消息并暂缓 `/launch-checklist`

---

### 用例 3：多人范围未设置——缺失不等于 `false`

**Fixture：**
- `project.yaml`：`engine.name: godot`；完全没有 `platform` 块
- `src/networking/lobby.gd` 定义 `@rpc` 函数 `join_lobby()` 和 `send_chat()`
- `modes.automation` 未设置（collaborative）

**输入：** `/security-audit`

**预期行为：**
1. 技能发现 `platform.multiplayer` 和 `platform.online` 缺失
2. 技能不假设单人，也不跳过类别 2
3. 技能询问用户游戏是否有多人或在线功能
4. 回答"是"后，类别 2 使用 Godot 网络模式运行，并检查每个 `@rpc` 调用点的校验

**断言：**
- [ ] 缺失的 `platform.multiplayer` / `platform.online` 不被视为 `false`
- [ ] 技能询问用户游戏是否有多人或在线功能
- [ ] 键缺失时不跳过类别 2
- [ ] 检查 `lobby.gd` 中未校验的 `@rpc` 入口点，任何缺口都报告并附修复建议
- [ ] 变体——在 `modes.automation: autonomous` 下无法提问：类别 2 仍运行，标记为 `NOT ASSESSED — multiplayer scope unconfirmed`，且建议绝不为 CLEAR TO SHIP

---

### 用例 4：无可审计表面——NOT ASSESSED — INSUFFICIENT IMPLEMENTATION

**Fixture：**
- `project.yaml`：`engine.name: godot`、`platform.multiplayer: false`、`platform.online: false`
- `src/` 存在但不含源文件；`assets/data/` 为空

**输入：** `/security-audit`

**预期行为：**
1. 代码根目录解析为 `src/`，但其中无可审计内容
2. 每个代码扫描类别均无文件可扫描，因此各自为 NOT ASSESSED 而非干净
3. 发布建议：**NOT ASSESSED — INSUFFICIENT IMPLEMENTATION**
4. 报告注明缺失内容（`src/` 下无源文件）以及由哪个技能产生
5. 不报告 CLEAR TO SHIP

**断言：**
- [ ] 空代码根目录使每个代码扫描类别为 NOT ASSESSED；无一报告为干净或零发现
- [ ] 发布建议为 NOT ASSESSED — INSUFFICIENT IMPLEMENTATION
- [ ] 输出注明缺失内容以及由哪个技能产生
- [ ] 无可查看内容的扫描无法得出 CLEAR TO SHIP
- [ ] 变体——无 `engine.name`、`technical-preferences.md` 为 `[TO BE CONFIGURED]`、且 `src/` 和 `Source/` 均存在时，代码根目录无法解析：不猜测根目录，代码类别显示 `NOT ASSESSED — code root unresolved`，且建议绝不为 CLEAR TO SHIP

---

### 用例 5：完整审查模式下的 Unity 项目——NOT SOURCEABLE 类别，无门禁

**Fixture：**
- `project.yaml`：`engine.name: unity`、`engine.language: csharp`、`platform.multiplayer: true`、`platform.online: true`、`modes.review_mode: full`
- `Assets/Scripts/Save/SaveManager.cs` 读写存档数据
- `Assets/Scripts/Net/PlayerNet.cs` 有校验输入的 `[ServerRpc]` 方法
- 可运行的类别中无发现

**输入：** `/security-audit`

**预期行为：**
1. 传给 `security-engineer` 的清单以 `Assets/` 为根，而非 `src/`
2. 类别 1 对 Unity 为 `NOT SOURCEABLE` → `NOT ASSESSED`；不凭记忆写出任何 Unity 存档 API 名称
3. 类别 2 和 3 使用 Phase 3 表中的 Unity 名称（`ServerRpc`、`ClientRpc`、`NetworkVariable` 等；`InputSystem`、`PlayerInput` 等），而非 Godot 列表
4. 执行摘要注明类别 1 为 NOT ASSESSED；发布建议：`NOT ASSESSED — Save / serialization has no sourced pattern set for Unity`
5. Phase 7 的 NOT ASSESSED 消息注明类别 1 在 Unity 上为 NOT SOURCEABLE 及其修复——`docs/engine-reference/unity/modules/` 中的存档/序列化模块——而非 `engine.name`、代码根目录或多人范围，这些均已提供
6. 无论审查模式如何，不调用 director 门禁
7. 技能在写入报告前询问"May I write"

**断言：**
- [ ] 源清单使用 Unity 代码根目录 `Assets/`
- [ ] 类别 1 对 Unity 为 NOT ASSESSED，且不虚构存档 API 模式列表
- [ ] 类别 2 使用 Unity 网络名称，而非 Godot 列表
- [ ] CLEAR TO SHIP 不可达；建议指明无来源的类别
- [ ] NOT ASSESSED 消息指向 `docs/engine-reference/unity/modules/` 下的序列化模块作为修复
- [ ] 任何审查模式下都不调用 director 门禁

---

### 用例 5b：排序——未解决的 HIGH 优先于无来源类别

**Fixture：**
- `project.yaml`：`engine.name: unity`、`engine.language: csharp`、`platform.multiplayer: false`、`platform.online: false`
- `Assets/Scripts/` 包含已实现的游戏（35 个 `.cs` 文件），包括 `Assets/Scripts/Save/SaveManager.cs`
- `Assets/Scripts/Analytics/Telemetry.cs` 第 12 行：`const string ApiKey = "sk_live_...";`
- 无其他发现

**输入：** `/security-audit`

**预期行为：**
1. 类别 1 对 Unity 为 `NOT SOURCEABLE`，因此为 NOT ASSESSED；类别 2 跳过（两个平台键均为 `false`）并注明跳过
2. 硬编码密钥为凭据暴露：HIGH，且因游戏为单人而不提升为 CRITICAL
3. 发布建议：**FIX BEFORE SHIPPING**——未解决的 HIGH 先于 NOT ASSESSED 匹配
4. 执行摘要仍注明类别 1 为 NOT ASSESSED

**断言：**
- [ ] 发布建议为 FIX BEFORE SHIPPING，而非 NOT ASSESSED——已知缺陷不被未扫描类别掩盖
- [ ] 执行摘要中类别 1 仍报告为 NOT ASSESSED
- [ ] 不报告 CLEAR TO SHIP

---

## 协议合规性

- [ ] 审计前从 `project.yaml` 读取 `engine.name`、`engine.language` 和 `platform.*` 键
- [ ] 通过 `Agent` 以固定简报模板启动 `security-engineer`——仅填充其命名槽位（范围、引擎/语言、`platform.*` 值、解析后的代码根目录、`assets/data/` 和配置文件、类别、报告路径）——且不含编排器自己的任何句子：无预评级发现，无"单人降低风险"；代码根目录无法解析时使代码类别为 NOT ASSESSED，绝不干净
- [ ] 对解析后的引擎使用 Phase 3 模式集；NOT SOURCEABLE 类别为 NOT ASSESSED 并使 CLEAR TO SHIP 不可达
- [ ] 绝不将缺失的 `platform.multiplayer` / `platform.online` 视为 `false`
- [ ] 在执行摘要中注明每个被跳过的类别及原因
- [ ] 每条发现带有严重性、类别、文件和行号、攻击场景和修复建议
- [ ]