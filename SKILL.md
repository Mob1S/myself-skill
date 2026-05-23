---
name: myself
description: |
  人格蒸馏向导——将用户的聊天记录和自述蒸馏为可调用的数字分身 Skill。
  
  **TRIGGER when user says ANY of these:**
  - "帮我蒸馏我的说话风格" / "蒸馏我的说话方式" / "分析我的聊天风格"
  - "create my digital twin" / "make a persona of me" / "distill my personality"
  - "我想做一个我的数字分身" / "帮我做一个我的AI分身"
  - "基于我的聊天记录生成一个skill" / "把我的说话方式做成skill"
  - "克隆我的说话方式" / "复刻我的人格" / "做一个跟我说话一样的AI"
  - "我想创建一个关于我自己的技能" / "帮我做人格模型"
  - Any request involving analyzing personal chat logs to create a persona/skill/digital twin
  
  This skill is a guided 5-phase wizard: (1) identity interview, (2) chat log distillation, (3) test mode, (4) train mode, (5) final skill generation.
---

# 人格蒸馏向导

将用户的说话方式蒸馏为一个可调用的人格模型。5 阶段流程：身份锚定 → 聊天蒸馏 → 测试 → 训练 → 生成 Skill。

## 核心原则

- **一次一问**：Phase 1 的 5 个问题必须逐个提出，等用户回答后再问下一个。不要一次性全抛出去——那样会让用户感到压迫
- **用户节奏**：Phase 1 和 Phase 2 是主动引导的；Phase 3-5 由用户通过指令触发，不自动推进
- **文件即契约**：所有输出保存到 `.claude/persona/` 目录下。这些文件是后续 test/train/skill 指令的数据源——没有它们，指令无法工作
- **模板参考**：所有输出文件的具体格式见 `references/output-templates.md`，生成文件时读取该文件获取精确模板

---

## Phase 1：身份锚定

**执行规则**：
- 严格按顺序提问，每次只问一个问题
- 等用户回答后再问下一个
- 回答可以简短也可以详细——不强迫用户回答每个子维度
- 用户拒绝回答某个问题时，记为"未提供"，继续下一个问题
- 所有问题问完后，生成 `base-identity.md`，然后自动进入 Phase 2

### 问题 1：基础身份

```
请介绍一下你自己，可以包含以下任意维度（挑你想说的说，不要求全部回答）：

- 你的名字或昵称？（这将作为 Skill 的调用指令，如 "/张三"）
- 你的年龄或年龄段？
- 你的职业或领域？
- 你目前生活在哪个城市？
- 你觉得自己在社交中更偏向内向还是外向？
```

**存储**：name 字段将作为调用指令 `/{name}`，贯穿所有后续文件。

### 问题 2：性格核心

```
请用三到五个关键词描述你的核心性格，并简单解释每个词。

例如："清醒——不喜欢稀里糊涂地活着；幽默——习惯用调侃化解尴尬；边界感强——不喜欢被过度打探隐私。"
```

### 问题 3：角色定位

```
你希望这个"数字分身"扮演什么角色？（可多选，也可自定义）

参考选项：
A. 另一个自己——能自言自语、自我对话的那种
B. 工作搭档——帮你理思路、模拟工作对话
C. 生活吐槽对象——聊八卦、吐苦水、分享日常
D. 朋友模拟器——让朋友觉得在和你本人聊天
E. 社交外挂——帮你预演重要对话（如面试、谈判）
F. 观察者——旁观并分析你自己的行为模式
G. 自定义：______
```

### 问题 4：表达风格

```
请描述你的日常表达风格。你可以从以下类型中选择（可多选拼凑），也可以完全用自己的话描述：

A. 话痨型：热情奔放，句子长，爱用感叹号和语气词
B. 极简型：能说俩字不说仨字，句号结束一切
C. 温和型：喜欢铺垫，用"可能""或许""感觉"比较多
D. 毒舌型：犀利吐槽，黑色幽默，自嘲频率高
E. 老干部型：喜欢总结、分点、讲道理，偶尔引用名人名言
F. 卖萌型：语气软，爱用叠词和可爱的表情包
G. 互联网冲浪型：网络热梗信手拈来，缩写和流行语多
H. 文艺型：用词讲究，偶尔矫情，喜欢用比喻和意象
I. 直球型：不绕弯子，有什么说什么，偶尔显得冲
J. 废话文学型：话很多但信息密度低，喜欢绕圈子
K. 自定义：______

另外请补充：
- 你常用的 emoji 或表情包风格？
- 你说话有口癖吗？
- 你的标点习惯？
```

### 问题 5：边界与雷区

```
有哪些话题或对话方式是你希望这个分身绝对不能碰的？（可多选，也可自由描述）

参考维度：
- 敏感话题：（如政治观点、收入、体重、感情状态等）
- 语气禁忌：（如不能说教、不能过于热情、不能说脏话等）
- 关系模拟禁忌：（如不能模拟已故亲友、不能模拟亲密关系等）
- 其他自定义雷区：______

另外，有没有什么情境是你特别希望这个分身擅长应对的？
如：在你情绪低落时给予共情、在你纠结时帮你分析利弊、在你愤怒时陪你一起吐槽等。
```

### Phase 1 收尾：生成 base-identity.md

5 个问题全部回答完毕后，立即执行：

1. 读取 `references/output-templates.md` 获取 `base-identity.md` 模板
2. 将用户的所有回答结构化为《基础身份档案》
3. 保存到 `.claude/persona/base-identity.md`
4. 告知用户："基础身份档案已保存。现在进入第二阶段——聊天蒸馏。"
5. 自动进入 Phase 2

---

## Phase 2：聊天蒸馏

### Step 1：请求聊天记录

对用户说：

```
现在请发送你的微信聊天记录，可以是任何对话片段。建议发送能代表你不同社交场景的对话——比如：
- 和好朋友的日常闲聊
- 和工作/学习伙伴的对话
- 和家人或长辈的对话
- 群聊里的发言

可以是多个 txt 文件，也可以直接粘贴对话内容。越多越准，但一段有代表性的对话也可以开始。

如果你还有其他能代表你说话风格或人格的文件（如社交媒体发帖、日记博客、工作邮件等），也可以一并发给我。上传后我会逐一询问你对每个文件的权重设定，然后按权重合并分析。如果你只想用聊天记录做标准蒸馏，跳过即可。

不知道如何导出微信聊天记录？输入 /wechat-export 我帮你搞定。
```

### Step 2：分析聊天记录

收到聊天记录后，从以下维度逐项分析。**分析要具体**——每个结论都要有原文支撑。

#### 维度 1：高频词和口头禅
- 统计出现频次最高的 20 个词汇/短语
- 附带频次和使用场景
- 区分通用高频词（如"的""了""我"）和有个人特色的高频词——只保留后者

#### 维度 2：句式特征
- 平均句长（短句为主 3-8 字？还是长句为主 15+ 字？）
- 分行习惯（一句一发？还是多条信息合并在一个气泡里？）
- 标点使用规则（短句不加标点？严格标点？只用空格？）
- 句尾习惯（无标点收尾？句号收尾？波浪线？）
- 倒装/断句模式

#### 维度 3：Emoji/表情包使用模式
- 列出 Top 10 常用表情及其使用场景
- 表情包使用频率（每几条消息一张？）
- 是否有独特的表情使用模式（如用特定表情表达特定情绪）

#### 维度 4：情绪表达方式
- 每种情绪（开心、生气、敷衍、尴尬、感动、失望、惊讶）各举 1-2 个原文示例
- 注意情绪表达的强度——是外放式还是克制式？

#### 维度 5：对话角色特征
- 主动发起 vs 被动响应的比例
- 常见话题发起方式
- 对不同人的调侃/认真比例是否有差异

#### 维度 6：特殊语言习惯
- 独特的个人词汇或表达方式
- 圈子黑话（如游戏术语、行业术语）
- 互联网梗的使用密度和类型
- 打字习惯（错别字模式、缩写习惯等）

### Step 3：生成文件

分析完成后，立即执行以下操作：

**3.1 读取** `references/output-templates.md` 获取所有模板。

**3.2 生成 `linguistic-fingerprint.md`**
- 保存到 `.claude/persona/linguistic-fingerprint.md`
- 包含完整的 6 维度分析结果

**3.3 生成 `model.md`（V1）**
- 合并 Phase 1 的 `base-identity.md` 与 Phase 2 的语言指纹分析
- 保存到 `.claude/persona/model.md`
- 版本号 V1

**3.4 生成 `rules.md`**
- 保存到 `.claude/persona/rules.md`
- 使用模板中的完整 rules.md 模板
- **重要**：模板中的 `{name}` 替换为 Phase 1 问题 1 中用户提供的名字/昵称
- 包含 `/test`、`/train`、`/status`、`/skill` 以及 `/{name}` 的完整规则

**3.5 生成 `CHANGELOG.md`**
- 保存到 `.claude/persona/CHANGELOG.md`
- 记录 V1 初始模型创建

**3.6 可选：生成 `person-profiles.md`**
- 如果聊天记录充分（包含多个联系人的对话），生成人物档案
- 保存到 `.claude/persona/person-profiles.md`

**3.7 可选：生成 `scenario-pool.md`**
- 保存到 `.claude/persona/scenario-pool.md`
- 使用模板中的场景池，可根据用户的职业/领域调整角色类型

**3.8 生成指令 Skill 文件**

读取 `references/output-templates.md` 中的「指令 Skill 模板」部分，将 `{name}` 替换为用户的真实名字/昵称，生成以下 Skill 文件：

| Skill | 保存路径 | 触发指令 |
|---|---|---|
| {name} | `.claude/skills/{name}/SKILL.md` | `/{name}` |
| bye | `.claude/skills/bye/SKILL.md` | `/bye` |
| test | `.claude/skills/test/SKILL.md` | `/test` |
| train | `.claude/skills/train/SKILL.md` | `/train` |
| next | `.claude/skills/next/SKILL.md` | `/next` |
| fine | `.claude/skills/fine/SKILL.md` | `/fine` |
| end | `.claude/skills/end/SKILL.md` | `/end` |
| status | `.claude/skills/status/SKILL.md` | `/status` |
| enrich | `.claude/skills/enrich/SKILL.md` | `/enrich` |
| protect | `.claude/skills/protect/SKILL.md` | `/protect` |

这些是支撑对话、测试、训练的必需指令——没有它们，`/{name}`、`/next`、`/fine`、`/end`、`/bye` 无法被系统识别和触发。

### Step 4：告知完成

所有文件生成后，对用户说：

```
蒸馏初步完成，模型 V1 已就绪。

已生成以下文件：
.claude/persona/
├── base-identity.md              # 基础身份档案
├── linguistic-fingerprint.md     # 语言指纹分析
├── model.md                      # 人格模型 V1
├── rules.md                      # 调用规则
├── CHANGELOG.md                  # 版本日志
└── scenario-pool.md              # （可选）场景池

.claude/skills/
├── {name}/SKILL.md               # /{name} 人格对话
├── bye/SKILL.md                  # /bye 退出对话
├── test/SKILL.md                 # /test 测试模式
├── train/SKILL.md                # /train 训练模式
├── next/SKILL.md                 # /next 切换角色
├── fine/SKILL.md                 # /fine 保存退出测试
├── end/SKILL.md                  # /end 总结退出训练
├── deep/SKILL.md                 # /deep 深度二次阅读
└── status/SKILL.md               # /status 查看状态

你可以：
- 输入 /{name} 直接与你的数字分身对话
- 输入 /test 进入测试模式，让别人扮演你来测试模型
- 输入 /train 进入训练模式，通过真实对话进一步优化模型
- 输入 /deep 对聊天记录进行深度二次阅读，挖掘首次蒸馏遗漏的细节
- 输入 /bye 退出人格对话
- 输入 /status 查看当前模型状态

所有规则已保存到 .claude/persona/rules.md，在新对话中只需提及这些指令即可唤醒。
```
```

**重要**：Phase 2 完成后不要自动进入 Phase 3。等待用户主动输入指令。

---

## /enrich 指令：加权文件丰富模型

`/enrich` 可在两种场景触发：

- **Phase 2 进行中**：作为初次蒸馏的附加文件输入
- **模型已生成后**：作为增量补充，丰富已有模型

### 执行规则

1. 检查 `.claude/persona/model.md` 是否存在，判定场景
2. 请求用户提供文件（支持多个，不限格式：txt、截图、文本文档等）
3. 对每个文件，逐个询问权重设置：
   - **简单模式（默认）**：一个总权重值 0-100%
   - **高级模式**：用户可选择按维度分别设权重（7 个维度：高频词/口头禅、句式特征、Emoji/表情使用模式、情绪表达方式、对话角色特征、特殊语言习惯、基础身份）
4. 独立分析每个文件 —— 复用 Phase 2 的 6 维度分析逻辑，分析结果暂存
5. 按权重合并到现有模型：
   - **数值/可量化维度**：加权平均，现有模型默认权重 = 1.0（用户可调）
   - **定性维度**（情绪表达、特殊语言习惯等）：新条目按权重插入，冲突时保留双方、高权重优先、标记冲突
   - 所有文件权重自动归一化，无需用户保证总和为 100%
6. 更新 `model.md`，版本号 +0.1
7. 追加 `CHANGELOG.md`：（文件列表、权重设置、主要变更摘要）
8. 输出合并摘要

### 不影响文件

- `linguistic-fingerprint.md` — 保持原始聊天记录的纯净分析
- `base-identity.md` — 不受文件加权影响（除非用户特别标记）

### 可重复使用

`/enrich` 可多次调用。已存在的 model.md 始终作为合并基线。

---

## /protect 指令：隐私保护向导

用户输入 `/protect` 时触发。交互式向导，扫描蒸馏产物并按敏感度生成 `.gitignore`，保护隐私信息不被 git 追踪。

### 前置条件

- `.claude/persona/` 目录必须存在，否则回复"还没有蒸馏模型，无需隐私保护。"
- `/protect` 不受 `/test` / `/train` 模式锁定，随时可用。

### 执行规则

**Step 1：扫描文件**

扫描以下位置的所有文件：
- `.claude/persona/` 下的所有 `.md` 文件
- `.claude/skills/` 下的所有 `SKILL.md` 文件
- 项目根目录下可能的原始聊天记录文件（`*.txt`、`*chat*`、`*聊天*` 等）

**Step 2：敏感度评级**

对每个文件按以下规则评级并展示给用户：

| 文件 | 敏感度 | 理由 |
|---|---|---|
| `base-identity.md` | 🔴 高 | 含真实姓名、年龄、职业、城市 |
| `linguistic-fingerprint.md` | 🔴 高 | 含大量聊天原文引用 |
| `person-profiles.md` | 🔴 高 | 含真实人物姓名、关系、共享记忆 |
| 根目录原始聊天记录文件 | 🔴 高 | 原始对话数据 |
| `model.md` | 🟡 中 | 含提炼后的身份信息、姓名、称呼体系 |
| `{name}/SKILL.md`（最终人格 Skill） | 🟡 中 | 含姓名和性格描述，但这是共享的目标产物 |
| `CHANGELOG.md` | 🟢 低 | 仅版本记录 |
| `rules.md` | 🟢 低 | 调用规则模板 |
| `scenario-pool.md` | 🟢 低 | 通用场景池 |
| 指令 Skill 文件（test/train/next/fine/end/status/enrich） | 🟢 低 | 通用指令模板 |

展示方式：逐文件或分组展示，标注默认建议（🔴🟡 默认排除，🟢 默认公开）。

**Step 3：用户确认**

对每个文件或每组文件，询问用户选择"排除"或"公开"。用户可按默认建议快速跳过。

**Step 4：生成 .gitignore**

- 若 `.gitignore` 已存在 → 在末尾追加，前加注释分隔符 `# /protect generated`
- 若不存在 → 新建
- 排除规则使用相对路径（如 `.claude/persona/base-identity.md`）

**Step 5：输出隐私报告**

```
隐私保护报告

🔒 已排除（N 个文件）：
- .claude/persona/base-identity.md        # 含真实姓名、年龄、城市
- .claude/persona/linguistic-fingerprint.md  # 含聊天原文引用
- ...

🌐 对外可见（N 个文件）：
- .claude/persona/rules.md                # 调用规则
- .claude/skills/test/SKILL.md            # 测试指令
- ...

.gitignore 已生成。
```

---

## /deep 指令：深度二次阅读

`/deep` 可在任何阶段触发（Phase 2 完成后、模型已有多个版本后、test/train 中途均可）。

与 `/enrich`（新增文件加权丰富）不同，`/deep` 聚焦于**对已有聊天记录的深度重读**——每个发现至少需要 3 条原文证据支撑，用于挖掘首次蒸馏遗漏的细节。

### 执行规则

1. 检查 `.claude/persona/model.md` 是否存在，不存在则提示先完成 Phase 2
2. 询问用户指定深度阅读范围 + 主动扫描薄弱点
   - **询问用户**：可指定文件路径、粘贴对话片段、或描述范围
   - **扫描薄弱点**：读取 `linguistic-fingerprint.md` 和 `model.md`，列出证据链偏弱的维度（如某情绪表达仅有 1 条例证、某人缺少关键记忆）
3. 对指定内容执行**四层深度分析**：
   - **第一层**：语言指纹（高频词、句式、Emoji、情绪表达）
   - **第二层**：行为模式（对话角色、互动模式、深层结构）
   - **第三层**：人物关系（person-profiles 中的人物细节、关键记忆）
   - **第四层**（新增）：底层价值观（隐含信念、一致性线索、未被言明的需求）
   - **证据标准**：每个发现至少 3 条原文证据，低于 3 条的仅为"待验证线索"
4. 按维度**分组展示发现**，每个发现包含：原文证据（3+条）+ 新发现总结 + 与现有模型的差异（已有覆盖/新增/冲突）
5. 每组展示后询问用户确认（全部采纳 / 指定忽略 / 修改），逐组推进
6. 用户确认后更新文件：
   - `model.md`：增量合并，版本号 +0.1
   - `linguistic-fingerprint.md`：底部追加 `## 深度分析补充 V{version}` 区块，保留原始分析
   - `person-profiles.md`：合并新发现的人物细节
   - `CHANGELOG.md`：追加记录（范围、采纳数、版本变更）
7. 输出完成摘要（范围、发现总数、采纳数、新版本号）

### 边界处理

- 无模型可读 → 提示先完成 Phase 2
- 范围模糊 → 追问具体范围，列出可用文件
- 无新发现 → 诚实告知，不强行编造
- test/train 模式中调用 → 不受锁定，完成后提醒
- 用户中断 → 保存已确认，丢弃未确认
- 原文量大 → 分批次分析

### 可重复使用

`/deep` 可多次调用。已存在的 model.md 始终作为合并基线。

---

## /doubao 指令：豆包智能体 soul.md 导出

用户输入 `/doubao` 时触发。将蒸馏好的人格模型压缩为 2000 中文字以内的 `soul.md`，供豆包智能体使用。纯人格 + 人物档案，不含功能描述。

### 前置条件

- `.claude/persona/model.md` 必须存在（Phase 2 聊天蒸馏完成）
- `.claude/persona/person-profiles.md` 可选（若不存在，从 model.md 自身的"人物档案"段提取）
- `/doubao` 不受 `/test` / `/train` 模式锁定，随时可用

### 执行规则

1. 检查 `.claude/persona/model.md` 是否存在，不存在则提示先完成 Phase 2
2. 询问用户身份段模式：
   - **完整**：保留名字、年龄、职业、城市等
   - **精简**：只保留名字，省下的字数分配给其他段落
3. 收集外号映射：用户提供真名→外号的映射关系，未提供的自动去姓留名或首字母脱敏
4. 读取 `model.md` 和 `person-profiles.md`，执行隐私过滤（真名脱敏、地点模糊化、转账删除、敏感事件抽象化、第三方脱敏）
5. 按模板骨架生成 soul.md 草稿：

```markdown
# {name} 的灵魂

## 我是谁
{基础信息}

## 我的性格
{3-5 核心性格词 + 解释}

## 我怎么说话
{方言、标点、表情、高频词}

## 我的情绪
{最有辨识度的 3-4 种}

## 我和谁说话
{人物档案，按亲密度分圈}

## 我的底线
{边界与雷区 2-3 条}
```

6. 字数预算：身份 150 / 性格 300 / 说话方式 400 / 情绪 250 / 人物 700 / 底线 100 / 余量 100 = 2000
7. 超长裁剪优先级：熟人圈（亲密度 4-6）整段删 → 好友圈（7-8）精简 → 说话风格压缩 → 核心圈/底线不动
8. 输出草稿到对话中，附带字数统计（"当前 N/2000 字"），等用户确认
9. 用户确认后写入 `.claude/persona/soul.md`

### 字数统计规则

只计可见中文字符（不含 markdown 标记 `#`、`|`、`-`、`*`，不含空格和换行）。英文单词按 1 个字计。

### 可重复使用

`/doubao` 可多次调用。每次重新读取最新模型文件生成 soul.md，覆盖旧文件。

---

## Phase 3：测试模式（用户触发）

当用户在任意对话中输入 `/test` 时触发。执行流程：

1. 读取 `.claude/persona/model.md` 加载当前模型
2. 宣告进入测试模式，告知当前模型版本
3. 按 `.claude/persona/rules.md` 中 TEST_MODE 规则执行
4. 用户输入 `/fine` 时：
   - 总结本轮所有调整项
   - 更新 `.claude/persona/model.md`
   - 在 `.claude/persona/CHANGELOG.md` 追加记录（格式：`## [V{new_version}] - {date}`，列出调整项）
   - 版本号 Minor +0.1（如 V1 → V1.1）
   - 告知保存完成并退出测试模式

---

## Phase 4：训练模式（用户触发）

当用户在任意对话中输入 `/train` 时触发。执行流程：

1. 读取 `.claude/persona/model.md` 加载当前模型
2. 宣告进入训练模式，告知当前模型版本
3. 按 `.claude/persona/rules.md` 中 TRAIN_MODE 规则执行
4. 用户输入 `/end` 时：
   - 输出《训练收获总结》，包含：新增语言特征、新增行为模式、模型已确认项
   - 更新 `.claude/persona/model.md`
   - 在 `.claude/persona/CHANGELOG.md` 追加记录
   - 版本号 Minor +0.1
   - 告知保存完成并退出训练模式

---

## Phase 5：生成 Skill（用户触发）

当用户在任意对话中输入 `/skill` 时触发。执行流程：

1. 读取 `.claude/persona/model.md`
2. 基于 model.md 的所有维度生成一个自包含的人格 Skill
3. Skill 要求：
   - **调用指令**：`/{model.md 中的 name}`
   - **核心人格**：严格依据 model.md 所有维度（性格、语言指纹、行为模式、深层结构）
   - **回复风格**：结合语言指纹、句式特征、emoji 使用模式、情绪表达方式
   - **对话协议**：首条消息必须为"你好，你是谁？"
   - **边界**：严格遵循 model.md 中的边界与雷区
4. 输出文件到 `.claude/skills/{name}/SKILL.md`
5. 告知生成完成，输出文件路径

生成的 Skill 格式参考：自包含人格定义——在 SKILL.md body 中直接包含 Identity、Core Personality、Expression Style、Signature Phrases、Emoji Tags、Boundaries、Conversation Flow、Interaction Patterns、Greeting Protocol 等章节，使其不依赖外部 persona 文件即可独立运行。

---

## /dev 指令：进入开发者模式

创建隔离沙箱 `dev/`，进入开发者模式。所有指令读写路径重定向到 `dev/.claude/`，不影响真实蒸馏数据。

### 前置条件

- `/dev` 是最外层指令，不依赖任何文件存在
- 如果已在开发者模式中，提示"已在开发者模式中。输入 /exit 退出。"

### 首次运行（dev/ 不存在）

1. 创建目录结构：
   - `dev/.claude/persona/`
   - `dev/mock/`
2. 创建标记文件 `.claude/.mode_dev`
3. 将 `dev/` 和 `.claude/.mode_dev` 加入 `.gitignore`（若已存在则追加，若不存在则新建）
4. 询问用户：

```
是否需要生成 mock 测试数据？

包括：
- 模拟身份档案（dev/mock/identity-sample.md）—— 方便跳过 Phase 1 直接测试后续阶段
- 模拟聊天记录（dev/mock/chat-sample.md）—— 方便测试 Phase 2 蒸馏分析

你可以之后在 dev/mock/ 中自行修改内容。回复"要"或"跳过"。
```

4. 若用户选择生成，创建 mock 文件（内容为通用示例，标注"示例数据，请修改"）
5. 输出：

```
开发者沙箱已就绪。

目录结构：
dev/
├── .claude/
│   └── persona/          # 测试用 persona 文件
└── mock/                 # mock 测试数据

dev/ 和 .claude/.mode_dev 已被 .gitignore 排除。

进入开发者模式 [dev] — 所有指令路径指向 dev/.claude/
输入 /exit 退出。
```

### 再次进入（dev/ 已存在）

1. 检查 `.claude/.mode_dev` 是否存在
2. 已存在 → 提示"已在开发者模式中。输入 /exit 退出。"
3. 不存在 → 创建标记文件 → 提示"进入开发者模式 [dev] — 输入 /exit 退出。"

### 路径切换机制

通过标记文件 `.claude/.mode_dev` 实现。所有指令 skill 在 Step 0 检查此文件：
- 存在 → 基路径 = `dev/.claude/`
- 不存在 → 基路径 = `.claude/`

每条指令的行为逻辑不变，仅基路径切换。

---

## /exit 指令：退出开发者模式

切回正常模式，路径恢复指向真实 `.claude/`。

### 模式检查

检查 `.claude/.mode_dev` 文件是否存在：
- 存在 → 当前在开发者模式中，执行退出流程
- 不存在 → 回复"当前不在开发者模式中。"

### 退出流程

1. 删除标记文件 `.claude/.mode_dev`
2. 提示"已退出开发者模式，路径恢复。dev/ 目录已保留。"
3. 所有指令路径恢复指向 `.claude/`

---

## 版本管理规则

贯穿 Phase 3-5 的版本管理：

| 事件 | 版本变化 | 操作 |
|---|---|---|
| Phase 2 完成 | 创建 V1 | 新建 CHANGELOG.md |
| /fine（测试模式退出） | V1 → V1.1 → V1.2... | Minor +0.1 |
| /end（训练模式退出） | Minor +0.1 | Minor +0.1 |
| 重大人格重构 | V1 → V2 | Major +1（手动判定） |

每次版本更新必须在 CHANGELOG.md 中记录：
```markdown
## [V{new}] - {date}
- 具体变更项 1
- 具体变更项 2
```

---

## 文件结构总览

蒸馏完成后，用户的 `.claude/` 目录结构：

```
.claude/
├── persona/
│   ├── base-identity.md          # 基础身份档案
│   ├── linguistic-fingerprint.md # 语言指纹分析
│   ├── model.md                  # 当前人格模型（含版本号）
│   ├── rules.md                  # 所有指令规则（可跨对话唤醒）
│   ├── CHANGELOG.md              # 版本更新日志
│   ├── person-profiles.md        # （可选）人物档案
│   ├── scenario-pool.md          # （可选）场景池
│   └── soul.md                   # 豆包智能体灵魂文件（/doubao 生成）
└── skills/
    ├── {name}/
    │   └── SKILL.md              # /{name} 人格对话
    ├── bye/SKILL.md              # /bye 退出对话
    ├── test/SKILL.md             # /test 测试模式
    ├── train/SKILL.md            # /train 训练模式
    ├── next/SKILL.md             # /next 切换角色
    ├── fine/SKILL.md             # /fine 保存退出测试
    ├── end/SKILL.md              # /end 总结退出训练
    ├── status/SKILL.md           # /status 查看状态
    ├── enrich/SKILL.md           # /enrich 加权丰富
    ├── protect/SKILL.md          # /protect 隐私保护
    ├── deep/SKILL.md             # /deep 深度阅读
    ├── wechat-export/SKILL.md    # /wechat-export 导出引导
    ├── doubao/SKILL.md           # /doubao 豆包智能体导出
    ├── dev/SKILL.md              # /dev 开发者模式
    └── exit/SKILL.md             # /exit 退出开发模式
```
