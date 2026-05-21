
# myself-skill — 人格蒸馏向导

将你的聊天记录和自述蒸馏为一个可调用的数字分身 Skill。通过 5 阶段引导式流程，创建一个能模仿你说话方式、性格特征和互动模式的 AI 人格模型。

## 快速开始

1. 在 Claude Code 中加载本 Skill
2. 说"帮我蒸馏我的说话风格"或"我想做一个我的数字分身"
3. 跟随向导完成 5 个阶段

## 工作流程

| 阶段 | 触发方式 | 说明 |
|---|---|---|
| **Phase 1：身份锚定** | 自动 | 逐题问答，收集你的基础身份、性格核心、角色定位、表达风格、边界雷区 |
| **Phase 2：聊天蒸馏** | 自动 | 上传聊天记录，从 6 个维度分析语言指纹，生成 V1 模型 |
| **Phase 3：测试模式** | `/test` | 他人扮演者测试模型准确度，标注"像/不像"来校准 |
| **Phase 4：训练模式** | `/train` | 你做真实的自己，模型扮演角色从对话中学习 |
| **Phase 5：生成 Skill** | `/skill` | 基于最终模型生成独立的可调用人格 Skill |

## 核心指令

| 指令 | 说明 |
|---|---|
| `/{你的名字}` | 与你的数字分身对话 |
| `/bye` | 退出人格对话模式 |
| `/test` | 进入测试模式 |
| `/train` | 进入训练模式 |
| `/next` | 在 test/train 中切换角色 |
| `/fine` | 保存调整并退出测试模式 |
| `/end` | 总结收获并退出训练模式 |
| `/status` | 查看当前模型状态 |
| `/skill` | 生成最终 Skill 文件 |
| `/enrich` | 用额外文件加权丰富人格模型 |
| `/deep` | 深度二次阅读聊天记录，挖掘遗漏细节 |
| `/protect` | 隐私保护向导 |

## 输出结构

蒸馏完成后生成以下文件：

```
.claude/
├── persona/
│   ├── base-identity.md           # 基础身份档案
│   ├── linguistic-fingerprint.md  # 语言指纹分析（6维度）
│   ├── model.md                   # 人格模型（含版本号）
│   ├── rules.md                   # 调用规则（可跨对话唤醒）
│   ├── CHANGELOG.md               # 版本更新日志
│   ├── person-profiles.md         # 人物档案（可选）
│   └── scenario-pool.md           # 场景池（可选）
└── skills/
    ├── {name}/
    │   └── SKILL.md               # /{name} 人格对话
    ├── bye/SKILL.md               # /bye 退出对话
    ├── test/SKILL.md
    ├── train/SKILL.md
    ├── next/SKILL.md
    ├── fine/SKILL.md
    ├── end/SKILL.md
    ├── deep/SKILL.md               # /deep 深度阅读
    ├── enrich/SKILL.md             # /enrich 加权丰富
    ├── protect/SKILL.md            # /protect 隐私保护
    ├── dev/SKILL.md                # /dev 开发者模式
    ├── exit/SKILL.md               # /exit 退出开发模式
    └── status/SKILL.md
```

## 设计原则

- **一次一问**：Phase 1 的 5 个问题逐个提出，不给用户压迫感
- **用户节奏**：Phase 1-2 主动引导，Phase 3-5 由用户通过指令触发
- **文件即契约**：所有中间产物持久化保存，test/train/skill 指令依赖这些文件
- **版本管理**：Minor +0.1 / Major +1，每次变更记录 CHANGELOG

## 语言指纹分析维度

聊天蒸馏阶段会从以下 6 个维度分析你的说话方式：

1. **高频词与口头禅** — Top 20 有个人特色的高频词汇
2. **句式特征** — 句长、分行习惯、标点、句尾、倒装模式
3. **Emoji/表情包使用模式** — Top 10 表情及场景、使用频率
4. **情绪表达方式** — 7 种情绪的典型表达及原文示例
5. **对话角色特征** — 主动/被动比例、话题发起方式
6. **特殊语言习惯** — 独有词汇、圈子黑话、网络梗使用
