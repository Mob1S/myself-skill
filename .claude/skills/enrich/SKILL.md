---
name: enrich
description: Enriches the Mob1S persona model with additional weighted files. Use when the user types /enrich to merge new text sources (chat logs, social media posts, diaries, etc.) into the existing model. Each file gets user-defined weights for weighted merging.
---

# /enrich — Mob1S 加权文件丰富模型

用额外文件丰富人格模型。用户设定每个文件的权重，系统分析后加权合并到现有模型。

## 进入流程

### Step 0：判定工作目录

检查 `.claude/.mode_dev` 文件是否存在：
- 存在 → 基路径 = `dev/.claude/`
- 不存在 → 基路径 = `.claude/`

### Step 1：检查场景
- `{base}.claude/persona/model.md` 存在 → 增量补充模式
- `{base}.claude/persona/model.md` 不存在 → 初次蒸馏附加文件模式

### Step 2：收集文件
向用户请求文件：

```
请发送你想要用来丰富模型的文件（支持多个）：
- 可以是聊天记录的 txt 导出
- 可以是社交媒体发帖、日记博客、工作邮件等
- 可以是截图（我会提取其中的文字）
- 任何能代表你说话风格或人格的文本都可以

发完后告诉我"好了"。
```

### Step 3：逐个设置权重

对每个文件，询问权重：

```
文件 N：{文件名}

请设置这个文件的权重（0-100%）：
- 简单模式：输入一个数字，代表该文件对模型的整体影响力
- 高级模式：输入 "高级" 可对以下维度分别设权重：
  · 高频词/口头禅
  · 句式特征
  · Emoji/表情使用模式
  · 情绪表达方式
  · 对话角色特征
  · 特殊语言习惯
  · 基础身份
```

**简单模式**：数字即权重，如 `80`
**高级模式**：逐维度输入，如 `高频词:80, 句式:50, Emoji:30, 情绪:70, 角色:60, 语言习惯:90, 基础身份:20`

所有文件权重自动归一化，无需用户保证总和为 100%。

### Step 4：分析文件
对每个文件独立执行 Phase 2 的 6 维度分析。分析结果暂存供合并使用。

### Step 5：加权合并

**数值/可量化维度**（句式特征、Emoji 频率等）：
```
合并值 = (现有模型 × 1.0 + Σ(文件_i × 归一化权重_i)) / (1.0 + Σ归一化权重_i)
```

**定性维度**（情绪表达、特殊语言习惯等）：
- 新条目按权重决定插入位置（高权重排在前面）
- 与现有条目冲突时保留双方，高权重的优先显示
- 冲突点标记，供用户审查

### Step 6：保存输出

- 更新 `{base}.claude/persona/model.md`（版本号 +0.1）
- 追加 `{base}.claude/persona/CHANGELOG.md`
- **不更新** `{base}.claude/persona/linguistic-fingerprint.md`（保持原始纯净）
- **不更新** `{base}.claude/persona/base-identity.md`（除非用户特别标记）

### Step 7：输出摘要

```
/enrich 完成。

本次合并：
- 文件数：N
- 权重设置：[简要列出]
- 主要变更：[列出受影响最大的维度]

模型已更新至 V{new_version}。
```

## 可重复使用

`/enrich` 可多次调用。已存在的 model.md 始终作为合并基线。

## 模式互斥

`/enrich` 不受 `/test` / `/train` 模式锁定，随时可用。
