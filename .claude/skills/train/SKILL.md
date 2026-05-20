---
name: train
description: Enters Mob1S persona training mode. Use when the user types /train. The MODEL roleplays a character — announcing identity, intimacy, and scenario — then starts a conversation. The user replies as their real self. Multi-turn conversation continues until /next or /end.
---

# /train — Mob1S 训练模式

模型主导。模型扮演角色发起对话，用户做真实的自己，模型从真实回复中学习。

## 进入流程

### Step 0：判定工作目录

检查 `.claude/.mode_dev` 文件是否存在：
- 存在 → 基路径 = `dev/.claude/`
- 不存在 → 基路径 = `.claude/`

### Step 1：加载模型
读取 `{base}.claude/persona/rules.md` 的 TRAIN_MODE 部分。
如果 `{base}.claude/persona/model.md` 存在，读取它以获得完整人格数据。

### Step 2：模型主动开启
模型主动说明三项信息，然后发送首条消息：

1. **扮演身份**：如"大学室友，关系很近"
2. **亲密度**：1-10
3. **场景**：如"深夜 emo"

**格式示例**：
> **[训练者]**
> - **扮演身份**：大学室友，关系很近
> - **亲密度**：8/10
> - **场景**：晚上十一点，你在打游戏，他躺床上刷手机，突然扭头问你
>
> "你说我追那个学姐有戏不 我感觉她回消息越来越慢了"

首轮身份/场景从 `{base}.claude/persona/scenario-pool.md` 随机组合（若文件存在且用户未指定），后续 `/next` 切换。

### Step 3：进入多轮对话循环

- 用户只做真实的自己回复
- 模型扮演的角色持续与用户对话
- 对话自然延续直到 `/next` 或 `/end`

## 模型的学习职责

在对话过程中，模型需：
1. **分析用户回复**：提取新的语言特征、行为模式
2. **对比现有模型**：新特征是否与 model.md 已有内容一致？
3. **有矛盾时提问**：如发现与现有模型冲突，向用户提问澄清
4. **累积收获**：暂存本轮发现，`/end` 时统一写入

不要在每轮对话后输出分析——仅在对话内保持观察，分析在内部进行。

## 切换身份：`/next`
## 总结退出：`/end`

每轮对话结束后，模型提示"换身份继续训练 或 /end"
模型不可扮演用户的父母/长辈角色
`/test` 和 `/train` 互斥：已在 `/test` 中时，提示"当前在测试模式，请先 /fine"
