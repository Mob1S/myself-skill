---
name: next
description: Switches to the next test/train character during Mob1S /test or /train sessions. Use when the user types /next. If the user provides new parameters, use those. If input is empty, randomly generate from scenario-pool.md.
---

# /next — 切换下一个角色

`/test` 和 `/train` 共用的子指令。结束当前角色对话，切换到新角色。

## 开发者模式

检查 `.claude/.mode_dev` 文件是否存在：
- 存在 → 基路径 = `dev/.claude/`
- 不存在 → 基路径 = `.claude/`

## 模式检查

| 当前状态 | 行为 |
|---|---|
| 在 `/test` 中 | 结束当前测试者对话，开始下一个测试者 |
| 在 `/train` 中 | 结束当前扮演身份对话，换身份继续训练 |
| 不在任何模式 | 回复"当前不在测试或训练模式中，/next 需要在 /test 或 /train 中使用" |

## 参数输入

### 用户提供了参数
使用用户提供的参数。格式可一次性或逐步：

- 一次性：`/next 游戏搭子 要好的 约游戏 '上号上号'`
- 逐步：`/next` + 用户后续消息逐项提供（模型逐项询问缺失项）

需收集的参数：

| 模式 | 参数 |
|---|---|
| `/test` 中 | 与 Mob1S 的关系、亲密度（自然语言标签）、场景、首条消息 |
| `/train` 中 | 扮演身份、亲密度（自然语言标签）、场景 |

### 用户空输入
用户仅输入 `/next` 无其他内容时：

1. 如果 `{base}.claude/persona/scenario-pool.md` 存在，读取并随机组合
2. **随机抽取角色**：从角色表随机选一个
3. **随机亲密度**：在该角色的亲密度范围内随机选一个标签
4. **随机抽取场景**：从场景表随机选一个
5. **随机消息**（仅 /test 需要）：从该场景的消息模板中随机选一条

## 切换后

### 在 /test 中
向用户展示新测试者信息，然后模型以 Mob1S 人格回复。

### 在 /train 中
向用户展示新扮演身份，然后等待用户回复。
