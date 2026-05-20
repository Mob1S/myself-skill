---
name: protect
description: Privacy protection wizard for the Mob1S persona. Use when the user types /protect. Scans distilled persona files, rates sensitivity, and generates a .gitignore to exclude private data from version control before sharing or publishing.
---

# /protect — Mob1S 隐私保护向导

交互式向导，扫描蒸馏产物，按敏感度生成 `.gitignore`，在分享或公开发布前保护隐私。

## 前置条件

- 检查 `.claude/.mode_dev` 文件是否存在：存在 → 基路径 = `dev/.claude/`；不存在 → 基路径 = `.claude/`
- `{base}.claude/persona/` 目录必须存在，否则回复"还没有蒸馏模型，无需隐私保护。"

## 进入流程

### Step 1：扫描文件

扫描以下位置：
- `{base}.claude/persona/` 下的所有 `.md` 文件
- `{base}.claude/skills/` 下的所有 `SKILL.md` 文件
- 项目根目录下可能的原始聊天记录文件（`*.txt`、`*chat*`、`*聊天*` 等）

### Step 2：敏感度评级

对每个文件评级并展示：

| 敏感度 | 标记 | 默认行为 |
|---|---|---|
| 高 | 🔴 | 默认排除 |
| 中 | 🟡 | 默认排除 |
| 低 | 🟢 | 默认公开 |

评级规则：

| 文件 | 敏感度 | 理由 |
|---|---|---|
| `base-identity.md` | 🔴 高 | 含真实姓名、年龄、职业、城市 |
| `linguistic-fingerprint.md` | 🔴 高 | 含大量聊天原文引用 |
| `person-profiles.md` | 🔴 高 | 含真实人物姓名、关系、共享记忆 |
| 根目录原始聊天记录文件 | 🔴 高 | 原始对话数据 |
| `model.md` | 🟡 中 | 含提炼后的身份信息、姓名、称呼体系 |
| `Mob1S/SKILL.md` | 🟡 中 | 含姓名和性格描述，但这是共享的目标产物 |
| `CHANGELOG.md` | 🟢 低 | 仅版本记录 |
| `rules.md` | 🟢 低 | 调用规则模板 |
| `scenario-pool.md` | 🟢 低 | 通用场景池 |
| 指令 Skill 文件 | 🟢 低 | 通用指令模板 |

### Step 3：用户确认

逐文件或分组询问。用户可：
- 对单个文件修改："model.md 公开"
- 按组操作："所有中敏感都公开"
- 一键确认："全按默认"

### Step 4：生成 .gitignore

- 若 `.gitignore` 已存在 → 在末尾追加，前加注释 `# /protect generated`
- 若不存在 → 新建
- 排除规则使用相对路径

### Step 5：输出隐私报告

## 模式互斥

`/protect` 不受 `/test` / `/train` 模式锁定，随时可用。
