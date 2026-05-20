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

读取 `references/output-templates.md` 中的「指令 Skill 模板」部分，将 `{name}` 替换为用户的真实名字/昵称，生成以下 6 个 Skill 文件：

| Skill | 保存路径 | 触发指令 |
|---|---|---|
| test | `.claude/skills/test/SKILL.md` | `/test` |
| train | `.claude/skills/train/SKILL.md` | `/train` |
| next | `.claude/skills/next/SKILL.md` | `/next` |
| fine | `.claude/skills/fine/SKILL.md` | `/fine` |
| end | `.claude/skills/end/SKILL.md` | `/end` |
| status | `.claude/skills/status/SKILL.md` | `/status` |
| enrich | `.claude/skills/enrich/SKILL.md` | `/enrich` |

这些是支撑 test/train/enrich 多轮对话循环的必需指令——没有它们，`/next`、`/fine`、`/end` 无法被系统识别和触发。

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
├── test/SKILL.md                 # /test 测试模式
├── train/SKILL.md                # /train 训练模式
├── next/SKILL.md                 # /next 切换角色
├── fine/SKILL.md                 # /fine 保存退出测试
├── end/SKILL.md                  # /end 总结退出训练
└── status/SKILL.md               # /status 查看状态

你可以：
- 输入 /{name} 直接与你的数字分身对话
- 输入 /test 进入测试模式，让别人扮演你来测试模型
- 输入 /train 进入训练模式，通过真实对话进一步优化模型
- 输入 /status 查看当前模型状态

所有规则已保存，在新对话中只需提及这些指令即可唤醒。
```
- 输入 /status 查看当前模型状态

所有规则已保存到 .claude/persona/rules.md，在新对话中只需提及这些指令即可唤醒。
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
│   └── scenario-pool.md          # （可选）场景池
└── skills/
    ├── {name}/
    │   └── SKILL.md              # Phase 5 生成的最终 Persona Skill
    ├── test/SKILL.md             # /test 指令
    ├── train/SKILL.md            # /train 指令
    ├── next/SKILL.md             # /next 指令
    ├── fine/SKILL.md             # /fine 指令
    ├── end/SKILL.md              # /end 指令
    ├── end/SKILL.md              # /end 指令
    ├── status/SKILL.md           # /status 指令
    └── enrich/SKILL.md           # /enrich 指令
```
