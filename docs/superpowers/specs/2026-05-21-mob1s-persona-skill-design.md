# /{name} 人格调用 Skill 设计

## 问题

1. `/{name}` 指令无对应 skill 文件，用户蒸馏完成后无法通过自己的模型名调用人格模型
2. 用户用自然语言触发 `myself` 蒸馏向导后，被告知"可以用 /test 或 /train"，`/test` 和 `/train` 被误当成模型入口
3. `/test` 和 `/train` 本应是模型调整工具，不是对话入口；正常进入模型后应问"你好，你是谁？"再按规则对话

> `{name}` = 用户在 Phase 1 问题 1 中输入的名字/昵称，如 Mob1S、张三 等。

## 根因

Phase 2（蒸馏）生成指令 Skill 文件时，生成了 test/train/next/fine/end/status/enrich/protect/dev/exit，但漏掉了最核心的 `/{name}` 人格调用 Skill。对话启动协议写在了 model.md 和 rules.md 里，但没有 skill 文件承载。

## 方案

Phase 2 蒸馏完成时，生成 `/{name}` Skill 作为人格对话入口，同时生成 `/bye` Skill 作为退出指令。三个模式（使用、测试、训练）互斥。

### Phase 2 新增生成

| 文件 | 说明 |
|---|---|
| `.claude/skills/{name}/SKILL.md` | `/{name}` 人格对话 Skill |
| `.claude/skills/bye/SKILL.md` | `/bye` 退出对话模式 |

### 修改文件

| 文件 | 改动 |
|---|---|
| `SKILL.md`（主蒸馏向导） | Phase 2 Step 3.8 生成列表补充 `{name}` 和 `bye` |
| `references/output-templates.md` | 新增 `/{name}` 和 `/bye` Skill 模板 |
| `dev/.claude/persona/rules.md` | 互斥表加入 `/{name}` 和 `/bye` |

> `/{name}` Skill 采用引用式 —— 运行时读取 model.md + rules.md + person-profiles.md，模型更新后自动反映最新版本，无需重新生成。
> Phase 5（/skill）保留原有职责，作为可选的"导出发布版 Skill"步骤。

---

## `/{name}` Skill 设计

### 元数据

- name: `{name}`（如用户叫 Mob1S，name 就是 `mob1s`）
- description: 触发条件 `/{name}` 及自然语言变体（"和 {name} 聊天"等），由生成时填入

### 执行流程

```
Step 0：判定工作目录（检查 .mode_dev）
  → dev 模式 → 基路径 = dev/.claude/
  → 正常模式 → 基路径 = .claude/

Step 1：检查模型是否存在
  → model.md 存在 → 继续
  → 不存在 → "还没有人格模型，需要我先帮你蒸馏吗？"

Step 2：加载模型
  → 读取 model.md（人格数据）
  → 读取 rules.md（对话协议、边界规则、互斥检查）
  → 读取 person-profiles.md（如有）

Step 3：互斥检查
  → 在 /test 中 → 提示"当前在测试模式，请先 /fine"
  → 在 /train 中 → 提示"当前在训练模式，请先 /end"
  → 已在 /{name} 中 → 提示"已在 /{name} 对话模式中"
  → 否则 → 继续

Step 4：对话启动协议
  → 首条消息必须为："你好，你是谁？"

Step 5：身份匹配
  → 精确匹配 → 该人物的专属互动模式（称呼、话题偏好、共享记忆）
  → 模糊匹配 → 追问确认
  → 无匹配 → 泛式关系对话（亲密度默认5）
  → 拒绝自报 → 泛式关系，保持边界

Step 6：多轮对话
  → 所有回复符合语言指纹、行为模式、深层结构
  → 称呼、关系记忆、互动风格不能错
  → 纯聊天，不累积模型变更
```

---

## `/bye` Skill 设计

### 元数据

- name: `bye`
- description: 退出 `/{name}` 人格对话模式

### 执行流程

```
Step 0：判定工作目录（检查 .mode_dev）

Step 1：模式检查
  → 在 /{name} 对话中 → 退出，提示"已退出 /{name} 对话模式"
  → 在 /test 中 → 提示"当前在测试模式，请用 /fine 退出"
  → 在 /train 中 → 提示"当前在训练模式，请用 /end 退出"
  → 不在任何模式 → 提示"当前不在对话模式中"
```

---

## 模式互斥矩阵

| 尝试 ↓ \ 当前 → | /{name} 中 | /test 中 | /train 中 |
|---|---|---|---|
| /{name} | 已在对话模式 | 请先 /fine | 请先 /end |
| /test | 请先 /bye | - | 请先 /end |
| /train | 请先 /bye | 请先 /fine | - |
| /bye | 退出对话 | 请用 /fine | 请用 /end |
| /fine | 请用 /bye | 保存退出 | 请用 /end |
| /end | 请用 /bye | 请用 /fine | 总结退出 |

`/status`、`/enrich`、`/protect`、`/dev`、`/exit` 不受任何模式限制，随时可用。

---

## 模式生命周期

| 模式 | 用途 | 进入 | 退出 |
|---|---|---|---|
| `/{name}` | 与人格模型对话 | `/{name}` | `/bye` |
| `/test` | 他人测试模型准确度 | `/test` | `/fine` |
| `/train` | 真实对话供模型学习 | `/train` | `/end` |

---

## 目录结构（新增后）

```
.claude/skills/
├── {name}/SKILL.md    ← 新增（{name} 为用户输入的名字）
├── bye/SKILL.md       ← 新增
├── test/SKILL.md
├── train/SKILL.md
├── next/SKILL.md
├── fine/SKILL.md
├── end/SKILL.md
├── status/SKILL.md
├── enrich/SKILL.md
├── protect/SKILL.md
├── dev/SKILL.md
└── exit/SKILL.md
```
