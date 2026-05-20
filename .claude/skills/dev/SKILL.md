---
name: dev
description: Enters developer mode for Mob1S skill testing. Use when the user types /dev. Creates an isolated sandbox at dev/ with its own .claude/ mirror. All instruction paths redirect to dev/.claude/ so testing won't affect real persona data.
---

# /dev — Mob1S 开发者模式

创建隔离沙箱 `dev/`，进入开发者模式测试 Skill 和蒸馏流程。

## 进入流程

### 首次运行（dev/ 不存在）

1. 创建目录结构：
   - `dev/.claude/persona/`
   - `dev/mock/`
2. 创建标记文件 `.claude/.mode_dev`
3. 将 `dev/` 和 `.claude/.mode_dev` 加入 `.gitignore`（若已存在则追加，若不存在则新建）
3. 询问用户：

```
是否需要生成 mock 测试数据？

包括：
- 模拟身份档案（dev/mock/identity-sample.md）
- 模拟聊天记录（dev/mock/chat-sample.md）

你可以之后在 dev/mock/ 中自行修改内容。回复"要"或"跳过"。
```

4. 若用户选择生成，创建 mock 文件（内容为通用示例，标注"示例数据，请修改"）
5. 输出沙箱就绪信息 + 目录结构
6. 进入开发者模式

### 再次进入（dev/ 已存在）

1. 检查 `.claude/.mode_dev` 是否存在
2. 已存在 → 提示"已在开发者模式中。输入 /exit 退出。"
3. 不存在 → 创建标记文件 → 提示"进入开发者模式 [dev] — 输入 /exit 退出。"

## 开发者模式中

- 所有指令路径重定向到 `dev/.claude/`，与真实 `.claude/` 完全隔离
- 所有指令行为逻辑不变，仅基路径切换
- `/status` 额外标注当前处于开发者模式

## 路径切换机制

通过标记文件 `.claude/.mode_dev` 实现。所有指令 skill 在 Step 0 检查此文件：
- 存在 → 基路径 = `dev/.claude/`
- 不存在 → 基路径 = `.claude/`

## 模式互斥

- 已在开发者模式中输入 `/dev` → 提示"已在开发者模式中。输入 /exit 退出。"
- `/exit` 退出开发者模式，删除 `.claude/.mode_dev`
