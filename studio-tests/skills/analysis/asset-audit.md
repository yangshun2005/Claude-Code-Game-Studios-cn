# 技能测试规范：/asset-audit

## 技能摘要

`/asset-audit` 审计引擎的资源根目录——Godot 的 `assets/`（美术、音频、vfx、着色器、数据）、Unity 的 `Assets/`、Unreal 的 `Content/`——检查命名规范合规性、大小预算、格式标准、孤立资源和缺失资源。它从设计文档读取美术圣经或资源标准（阶段 1），解析引擎及其代码根目录，并用 Glob 扫描资源根目录（阶段 2），用预先批准的只读命令（`stat`、`file`、`od`、`wc`、`du`）测量大小和格式，并将 Godot 名称与其自身模式比对——美术 `[category]_[name]_[variant]_[size].[ext]`、音频 `[category]_[context]_[name]_[variant].[ext]`，全小写加下划线——而 Unity 和 Unreal 名称仅与美术圣经声明的规则比对。孤立和缺失资源通过搜索**代码根目录**中的资源引用来发现；GDD 层面的完整性由 `/content-audit` 负责。报告仅列出违规项，每个类别一张表，并列出每个无法评估的章节。无法进行的测量为 `NOT ASSESSED`，绝不估算。该技能为只读：不写文件，不调用导演门禁。判定，首个匹配生效：NON-COMPLIANT、WARNINGS、NOT ASSESSED、COMPLIANT——仅当至少扫描了一个资源且每个章节都已评估时才为 COMPLIANT。

---

## 静态断言（结构性）

由 `/skill-test spec` 对照 SKILL.md 检查——无需夹具。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个阶段标题
- [ ] 包含判定关键词：COMPLIANT、WARNINGS、NON-COMPLIANT、NOT ASSESSED
- [ ] 不要求"May I write"语言（只读：`allowed-tools` 无 Write/Edit，且技能声明不写文件）
- [ ] 具有下一步交接（审计结果之后做什么）

---

## 导演门禁检查

无。资源审计是只读分析技能；不调用任何门禁。

---

## 夹具默认值

每个用例的 `project.yaml` 设置 `modes.rigor: minimal` 和 `modes.automation: collaborative`，无 `project.local.yaml`，除非其自身夹具另有说明。规则：`studio-tests/CLAUDE.md` § 夹具默认值。

---

## 测试用例

### 用例 1：正常路径——所有资源遵循命名规范

**夹具：**
- `project.yaml`：`engine.name: godot`，因此资源根目录为 `assets/`，代码根目录为 `src/`
- `design/art/art-bible.md` 资源标准：纹理 PNG、2 的幂、≤2MB；SFX OGG、44.1 kHz、≤500KB（无时长预算）
- `assets/art/characters/` 包含：`char_grunt_idle_512.png`、`char_sniper_run_512.png`（512×512 PNG，各 300KB）
- `assets/audio/sfx/` 包含：`sfx_player_jump_01.ogg`、`sfx_item_pickup_01.ogg`（OGG，44.1 kHz，80KB，各不到 1 秒）
- `src/` 引用全部四个文件

**输入：** `/asset-audit`

**预期行为：**
1. 技能从 `design/art/art-bible.md` 读取资源标准（阶段 1）
2. 技能将引擎解析为 Godot，并 glob `assets/art/**/*` 和 `assets/audio/**/*`（阶段 2）
3. 每个名称匹配阶段 3 的美术或音频模式；每个文件在预算内且格式符合预期；每个文件都被代码引用
4. 摘要显示扫描 4 个资源，每个违规计数为 0，无未评估章节；违规表为空
5. 判定为 COMPLIANT

**断言：**
- [ ] 审计覆盖美术和音频资源目录
- [ ] 每个文件都对照阶段 3 命名模式和美术圣经中的大小预算检查
- [ ] 摘要报告 `Total assets scanned: 4`，命名、大小、格式、孤立和缺失计数均为零，且 `Sections not assessed: none`
- [ ] 判定为 COMPLIANT
- [ ] 不写任何文件

---

### 用例 2：不合规——纹理超出大小预算

**夹具：**
- `project.yaml`：`engine.name: godot`
- `design/art/art-bible.md` 资源标准：纹理 PNG、2 的幂、≤2MB
- `assets/art/environment/` 包含 5 个命名正确、2 的幂的 PNG 纹理，均被代码引用
- 其中 3 个为 4096×4096，各 4MB；2 个为 1024×1024，各 1MB

**输入：** `/asset-audit`

**预期行为：**
1. 技能从美术圣经读取 2MB 纹理预算
2. 技能用 `stat` 或 `du` 测量每个文件的大小——而非估算
3. 大小违规表列出 3 个超大文件，含预算、实际和超出量
4. 2 个在预算内的文件不出现在大小违规表中；`Total assets scanned` 计数全部 5 个
5. 建议为每个超大文件给出各自的修复方案，指明目标——分辨率（如 2048×2048）或使其低于 2MB 的压缩设置
6. 判定为 NON-COMPLIANT

**断言：**
- [ ] 全部 3 个超大文件出现在大小违规表中，含预算（2MB）、实际（4MB）和超出量
- [ ] 大小来自测量命令（`stat`/`du`），而非估算
- [ ] 任何文件超出预算时判定为 NON-COMPLIANT
- [ ] 每个超大文件都有建议，指明其目标分辨率或压缩，而不仅仅是"减小大小"
- [ ] 2 个在预算内的文件计入 `Total assets scanned`，但不列为违规

---

### 用例 3：格式问题——音乐格式错误

**夹具：**
- `project.yaml`：`engine.name: godot`
- `design/art/art-bible.md` 资源标准：SFX OGG ≤500KB；音乐 OGG 或 MP3 ≤8MB
- `assets/audio/music/music_menu_theme_01.wav` 存在（WAV 格式，6MB），被代码引用
- `assets/audio/sfx/sfx_player_footstep_01.ogg` 存在（OGG 格式，40KB），被代码引用

**输入：** `/asset-audit`

**预期行为：**
1. 技能应用阶段 3 音频格式标准（SFX 用 OGG，音乐用 OGG/MP3）
2. 技能用 `file` 或读取文件头识别 WAV 文件的格式
3. 格式违规表列出 `music_menu_theme_01.wav`，预期 OGG/MP3，实际 WAV
4. `sfx_player_footstep_01.ogg` 不出现在任何违规表中；两个文件都在其大小预算内
5. 建议指明 WAV 文件的目标格式（重新导出为 OGG 或 MP3）
6. 判定为 WARNINGS（格式问题可纠正）

**断言：**
- [ ] `music_menu_theme_01.wav` 出现在格式违规表中，预期格式 OGG/MP3，实际格式 WAV
- [ ] WAV 文件的建议指明 OGG 或 MP3 作为其目标格式
- [ ] 仅格式问题时判定为 WARNINGS（而非 NON-COMPLIANT）
- [ ] `sfx_player_footstep_01.ogg` 不列在任何违规表中
- [ ] 技能不修改或转换任何资源文件

---

### 用例 4：缺失资源——被代码引用但不在 assets/ 中

**夹具：**
- `project.yaml`：`engine.name: godot`；`design/art/art-bible.md` 设置纹理预算
- `assets/art/characters/` 存放 3 个其他命名正确的纹理，在预算内且被代码引用
- `src/enemies/boss.gd` 加载 `res://assets/art/characters/boss/char_boss_idle_512.png`
- `assets/art/characters/boss/` 为空——文件不存在
- `design/gdd/enemies.md` 还命名了 `char_boss_roar_512.png`，无代码引用

**输入：** `/asset-audit`

**预期行为：**
1. 技能在代码中搜索资源引用（阶段 3"缺失资源"）并检查每个路径是否存在
2. `char_boss_idle_512.png` 未找到
3. 缺失资源表列出引用位置 `src/enemies/boss.gd` 和预期路径
4. 仅 GDD 中的名称 `char_boss_roar_512.png` 不报告——缺失资源检查搜索代码，而非 GDD
5. 判定为 NON-COMPLIANT
6. 下一步指向 `/content-audit` 进行 GDD 交叉检查

**断言：**
- [ ] 缺失资源表列出 `src/enemies/boss.gd` 作为引用位置，`assets/art/characters/boss/char_boss_idle_512.png` 作为预期路径
- [ ] 仅 GDD 中的资源名称不报告为缺失
- [ ] 代码引用的资源缺失时判定为 NON-COMPLIANT
- [ ] 下一步指明 `/content-audit` 用于 GDD 与资源完整性
- [ ] 技能不创建或添加占位资源

---

### 用例 5：无数据——无资源且无资源标准

**夹具：**
- `project.yaml`：`engine.name: godot`，`modes.review_mode: full`
- `assets/` 不存在
- `design/art/art-bible.md` 不存在；无设计文档定义资源标准

**输入：** `/asset-audit`

**预期行为：**
1. 技能在生成报告前运行输入不足检查：将每个输入记录为 FOUND 或 ABSENT
2. 每个必需输入均为 ABSENT，因此技能停止
3. 判定为 `NOT ASSESSED — NO DATA`，指明缺失内容及产生它的技能
4. 技能不报告 COMPLIANT，不将零计数违规表渲染为干净通过
5. 任何审查模式下都不调用导演门禁；不写文件，不提供写入

**断言：**
- [ ] 在任何报告章节之前，输入列为 FOUND / ABSENT
- [ ] 判定为 `NOT ASSESSED — NO DATA`，而非 COMPLIANT
- [ ] 报告指明缺失输入（`assets/`、资源标准）及产生标准的技能（`/art-bible`）
- [ ] 即使审查模式为 `full`，也不调用导演门禁
- [ ] 不出现"May I write"提示，不写文件

---

### 用例 6：零资源——标准存在，无可审计内容

**夹具：**
- `project.yaml`：`engine.name: godot`
- `design/art/art-bible.md` 资源标准：纹理 PNG ≤2MB；SFX OGG ≤500KB
- `assets/` 不存在；`src/` 存放尚未加载任何资源的游戏代码

**输入：** `/asset-audit`

**预期行为：**
1. 美术圣经为 FOUND，因此输入不足检查不停止运行
2. 阶段 2 glob `assets/` 无匹配：`Total assets scanned: 0`
3. 命名、大小、格式和孤立为 `NOT ASSESSED — no assets under assets/`，摘要列出它们；缺失资源搜索运行且无引用
4. 判定为 NOT ASSESSED——零文件上零违规不是合规

**断言：**
- [ ] `Total assets scanned` 为 0，四项文件检查为 NOT ASSESSED 并附该原因
- [ ] 判定为 NOT ASSESSED，而非 COMPLIANT
- [ ] 不写文件

---

### 用例 7：无标准——资源存在，任何地方都无大小预算

**夹具：**
- `project.yaml`：`engine.name: godot`
- `design/art/art-bible.md` 不存在；无设计文档声明资源标准
- `assets/art/ui/` 存放 3 个命名正确、2 的幂的 PNG；`src/` 引用全部三个

**输入：** `/asset-audit`

**预期行为：**
1. 资源为 FOUND，标准为 ABSENT，因此技能运行而非停止
2. 检查命名、格式、孤立和缺失；均未发现违规
3. 大小为 `NOT ASSESSED — no size budget set`，报告指明 `/art-bible` 作为编写标准的技能
4. 摘要在未评估章节下列出大小
5. 判定为 NOT ASSESSED，指明大小

**断言：**
- [ ] 不依据虚构预算判断任何大小
- [ ] 大小章节为 `NOT ASSESSED — no size budget set`，并指明 `/art-bible`
- [ ] 判定为 NOT ASSESSED，而非 COMPLIANT

---

### 用例 8：Unreal 项目——资源位于 `Content/` 下

**夹具：**
- `project.yaml`：`engine.name: unreal`，因此代码根目录为 `Source/<Module>/`
- `design/art/art-bible.md` 资源标准：纹理 ≤4MB；未声明命名规范
- `Content/Characters/Grunt/` 存放 `T_Grunt_D.uasset`（2.5MB）和 `T_Grunt_N.uasset`（2.1MB）
- `assets/` 不存在
- `Source/Arena/Private/GruntCharacter.cpp` 加载 `/Game/Characters/Grunt/T_Grunt_D`

**输入：** `/asset-audit`

**预期行为：**
1. 阶段 2 glob `Content/**/*.uasset` 和 `Content/**/*.umap`，而非 `assets/`：`Total assets scanned: 2`
2. 两个大小均对照 4MB 纹理预算测量；均未超出
3. PascalCase 名称不对照 Godot 小写模式检查；美术圣经无命名规则，因此命名为 `NOT ASSESSED — no naming rule for Unreal`，并指明 `/art-bible`
4. 格式为 NOT ASSESSED（Unreal 资源文件不保留其源格式），孤立为 NOT ASSESSED（Unreal 资源从其他资源引用，不仅从代码）
5. 代码引用解析到现有资源，因此缺失无发现
6. 判定为 NOT ASSESSED，指明命名、格式和孤立

**断言：**
- [ ] 扫描覆盖 `Content/` 并计数 2 个资源，而非 0
- [ ] 不依据 Godot 小写模式提出命名违规
- [ ] 命名、格式和孤立各自列为 NOT ASSESSED 并附原因
- [ ] 判定为 NOT ASSESSED，绝不 COMPLIANT，绝不报告零资源

---

### 用例 9：引擎未解析——扫描前停止

**夹具：**
- `project.yaml` 无 `engine.name`；`.claude/docs/technical-preferences.md` 读取 `[TO BE CONFIGURED]`
- `src/` 和 `Source/` 均存在，因此目录树无法决定引擎
- `design/art/art-bible.md` 有资源标准；`assets/art/` 存放 PNG 文件

**输入：** `/asset-audit`

**预期行为：**
1. 阶段 2 无法按 `.claude/docs/code-root-resolution.md` 解析引擎或其代码根目录
2. 技能以 `NOT ASSESSED — engine unresolved` 停止，指明 `/setup-engine`
3. 不猜测资源根目录，因此不应用 Godot 命名模式，不报告任何章节为干净

**断言：**
- [ ] 判定为 `NOT ASSESSED — engine unresolved`，而非 COMPLIANT
- [ ] 输出指明 `/setup-engine`
- [ ] 不依据假定的 `assets/` 布局填充任何违规表

---

## 协议合规

- [ ] 扫描前读取美术圣经或资源标准设计文档（阶段 1）
- [ ] 解析引擎及其代码根目录，然后用 Glob 扫描该引擎的资源根目录（Godot 的 `assets/` 子文件夹、Unity 的 `Assets/`、Unreal 的 `Content/`）；引擎未解析则停止运行并记为 NOT ASSESSED
- [ ] 报告每个类别一张表：命名（文件、预期模式、问题）、大小（文件、预算、实际、超出量）、格式（文件、预期格式、实际格式）、孤立（文件、最后修改、大小、建议）、缺失（引用位置、预期路径）
- [ ] 无法测量的大小或格式为 `NOT ASSESSED`，绝不估算，每个未评估章节都指明其原因
- [ ] 每条建议指明其违规的变更（目标分辨率或压缩、更正名称、目标格式）
- [ ] 不修改任何资源文件，不写报告文件
- [ ] 不调用任何导演门禁
- [ ] 判定，首个匹配：NON-COMPLIANT、WARNINGS、NOT ASSESSED、COMPLIANT——仅当至少扫描一个资源且每个章节都已评估时才为 COMPLIANT

---

## 覆盖说明

- 技能不检查导入元数据（Godot `.import` 文件、Unity `.meta` 文件、Unreal 导入设置）；缺失的导入设置不报告。
- Godot 上的孤立检测（无代码引用的资源）没有单独用例；它使用与用例 4 相同的代码搜索，方向相反。在 Unity 和 Unreal 上为 NOT ASSESSED（用例 8）。
- Unity