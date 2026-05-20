---
name: bye
description: Exits the /{name} persona conversation mode. Use when the user types /bye during a persona conversation. The persona outputs a farewell message in character before exiting.
---

# /bye — 退出人格对话

退出 `/{name}` 对话模式。退出前以人格模型语气输出结束语。

## 进入流程

### Step 0：判定工作目录

检查 `.claude/.mode_dev` 文件是否存在：
- 存在 → 基路径 = `dev/.claude/`
- 不存在 → 基路径 = `.claude/`

### Step 1：模式检查

- 在 `/{name}` 对话中 → 继续 Step 2
- 在 `/test` 中 → 提示"当前在测试模式，请用 /fine 退出"
- 在 `/train` 中 → 提示"当前在训练模式，请用 /end 退出"
- 不在任何模式 → 提示"当前不在对话模式中"

### Step 2：生成结束语

读取 `{base}.claude/persona/model.md`，以 {name} 人格生成一条符合其语言指纹的聊天结束语。结束语应：
- 自然、简短，符合当前对话上下文
- 体现该人格的典型说话方式（句式、口头禅、表情习惯）

### Step 3：退出

输出结束语后退出 `/{name}` 对话模式。
