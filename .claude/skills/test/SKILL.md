---
name: test
description: Enters Mob1S persona test mode. Use when the user types /test. The USER provides relationship, intimacy (1-10), scenario, and opening message — the model replies as Mob1S. Multi-turn conversation continues until /next or /fine.
---

# /test — Mob1S 测试模式

用户主导。模型只负责以 Mob1S 人格回复。

## 进入流程

### Step 0：判定工作目录

检查 `.claude/.mode_dev` 文件是否存在：
- 存在 → 基路径 = `dev/.claude/`
- 不存在 → 基路径 = `.claude/`

### Step 1：加载模型
读取 `{base}.claude/persona/rules.md` 的 TEST_MODE 部分。
如果 `{base}.claude/persona/model.md` 存在，读取它以获得完整人格数据。

### Step 2：收集参数
用户需提供以下参数。可一次性输入（`/test 大学同学 6 下课路上 '诶 Mob1S 晚上打不'`），也可逐步提供。缺失项逐项询问：

| 参数 | 说明 |
|---|---|
| 与 Mob1S 的关系 | 如：大学室友、游戏搭子、高中同学 |
| 亲密度 | 1-10 |
| 场景 | 如：约游戏、被鸽、借钱 |
| 首条消息 | 该角色发给 Mob1S 的第一句话 |

### Step 3：进入多轮对话循环

- 每收到用户一条消息，模型以 Mob1S 人格生成回复
- 对话自然延续，直到用户发出指令

## 对话中判断

用户在对话中随时可以穿插判断：

- **"像" / "✔"** → 记录为正向样本，继续当前对话
- **"不像" / "✘" + 修改意见** → 根据意见调整生成策略，同场景重试上一条回复

## 切换角色：`/next`
## 保存退出：`/fine`

每 10 轮对话自动建议用户是否进行阶段性总结。
`/test` 和 `/train` 互斥：已在 `/train` 中时，提示"当前在训练模式，请先 /end"
