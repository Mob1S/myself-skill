# /protect 隐私保护模块设计文档

## 概述

新增 `/protect` 指令，交互式向导帮助用户生成 `.gitignore`，保护蒸馏过程中的隐私信息。场景：

- 用户想将蒸馏后的人格 Skill 发给朋友
- 用户想将源码上传到公域
- 不希望聊天原文、真实身份、人物关系等隐私被暴露

**核心原则**：不影响模型功能，仅控制 git 版本管理中的可见性。

## 交互流程

```
用户输入 /protect
  ↓
Step 1：扫描 .claude/ 下所有蒸馏产物
  ├── .claude/persona/ 下的所有文件
  ├── .claude/skills/ 下的所有文件
  └── 项目根目录下可能的原始聊天记录文件（*.txt、*chat* 等）
  ↓
Step 2：逐文件展示敏感度评级 + 理由
  ├── 🔴 高 — 默认排除
  ├── 🟡 中 — 默认排除，建议用户确认
  └── 🟢 低 — 默认公开
  ↓
Step 3：用户逐项确认或修改（排除 / 公开）
  ↓
Step 4：生成 .gitignore 规则
  ├── 已排除文件 → 追加到 .gitignore
  └── 已公开文件 → 不添加
  ↓
Step 5：输出隐私报告
```

## 关键规则

- `/protect` 随时可用，不受 `/test` / `/train` 模式锁定
- 如果 `.gitignore` 已存在，追加而非覆盖，添加 `# /protect generated` 注释分隔符
- 如果不存在，新建
- 蒸馏未完成（`.claude/persona/` 不存在）时提示"还没有蒸馏模型，无需保护"
- 排除规则使用相对路径

## 敏感度判定规则

| 文件 | 敏感度 | 理由 |
|---|---|---|
| `base-identity.md` | 🔴 高 | 含真实姓名、年龄、职业、城市 |
| `linguistic-fingerprint.md` | 🔴 高 | 含大量聊天原文引用 |
| `person-profiles.md` | 🔴 高 | 含真实人物姓名、关系、共享记忆 |
| 根目录原始聊天记录 | 🔴 高 | 原始对话数据 |
| `model.md` | 🟡 中 | 含提炼后的身份信息、姓名、称呼体系 |
| `{name}/SKILL.md` | 🟡 中 | 含姓名和性格描述，但这是共享的目标产物 |
| `CHANGELOG.md` | 🟢 低 | 仅版本记录 |
| `rules.md` | 🟢 低 | 调用规则模板 |
| `scenario-pool.md` | 🟢 低 | 通用场景池 |
| 指令 Skill 文件 | 🟢 低 | 通用指令模板 |

默认行为：高和中敏感度默认排除，低敏感度默认公开。

## 隐私报告格式

```
隐私保护报告

🔒 已排除（N 个文件）：
- base-identity.md        # 含真实姓名、年龄、城市
- linguistic-fingerprint.md  # 含聊天原文引用
- ...

🌐 对外可见（N 个文件）：
- rules.md                # 调用规则
- ...

.gitignore 已生成。以上规则已追加到项目根目录。
```

## 涉及的代码改动

### SKILL.md
1. `/enrich` 规则之后新增 `/protect` 完整执行规则
2. 文件结构总览无需变更

### references/output-templates.md
1. 新增 `protect Skill 模板` 章节
2. rules.md 常驻指令列表新增 `/protect`
3. 状态隔离补充 `/protect`

### 指令清单（更新后）

| 指令 | 说明 |
|---|---|
| `/{name}` | 与数字分身对话 |
| `/test` | 进入测试模式 |
| `/train` | 进入训练模式 |
| `/enrich` | 加权文件丰富模型 |
| `/protect` | **新增** — 隐私保护向导，生成 .gitignore |
| `/next` | 切换角色 |
| `/fine` | 保存退出测试 |
| `/end` | 总结退出训练 |
| `/status` | 查看状态 |
| `/skill` | 生成 Skill |
