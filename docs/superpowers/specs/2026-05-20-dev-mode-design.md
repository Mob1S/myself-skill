# /dev 开发者模式设计文档

## 概述

新增 `/dev` 指令进入持续开发者模式，在 `dev/` 隔离沙箱中测试 Skill 定义和蒸馏流程。新增 `/exit` 指令退出开发者模式。`dev/` 整体被 gitignore，测试产物不会被误提交。

## 交互流程

```
用户输入 /dev
  ↓
检查 dev/ 是否存在
  ├── 不存在 → 初始化沙箱
  │     1. 创建 dev/.claude/persona/
  │     2. 创建 dev/.claude/skills/
  │     3. 可选：生成 mock 测试数据（需用户确认）
  │     4. dev/ 加入 .gitignore
  │     5. 输出"沙箱已就绪，进入开发者模式 [dev]"
  └── 已存在 → 输出"进入开发者模式 [dev]"
  ↓
开发者模式中：
  - 所有指令路径重定向到 dev/.claude/
  - 提示标注 [dev]
  ↓
用户输入 /exit
  - 切回正常模式，路径恢复指向真实 .claude/
  - dev/ 目录保留
```

## 沙箱目录结构

```
dev/
├── .gitkeep
├── .claude/
│   ├── persona/
│   │   ├── base-identity.md
│   │   ├── linguistic-fingerprint.md
│   │   ├── model.md
│   │   ├── rules.md
│   │   ├── CHANGELOG.md
│   │   ├── person-profiles.md
│   │   └── scenario-pool.md
│   └── skills/
│       ├── {name}/
│       ├── test/
│       ├── train/
│       ├── next/
│       ├── fine/
│       ├── end/
│       ├── status/
│       ├── enrich/
│       └── protect/
└── mock/
    ├── chat-sample.md
    └── identity-sample.md
```

## 路径映射

开发者模式中，所有文件读写基路径切换：

| 正常模式 | 开发者模式 |
|---|---|
| `.claude/persona/` | `dev/.claude/persona/` |
| `.claude/skills/` | `dev/.claude/skills/` |

每条指令的行为逻辑不变，仅基路径切换。

## Mock 数据

首次初始化时询问用户是否生成 mock 数据：
- 模拟身份档案 — 方便跳过 Phase 1 直接测试后续阶段
- 模拟聊天记录 — 方便测试 Phase 2 蒸馏分析

生成到 `dev/mock/`，用户可自行修改内容。

## 关键规则

- `/dev` 和 `/exit` 是最外层指令，不依赖 `.claude/persona/` 是否存在
- 开发者模式中所有现有指令功能不变，仅读写路径切换
- `/status` 在 dev 模式中额外标注当前处于开发者模式
- 模式间互斥：不能在 dev 模式中输入 `/dev`（提示"已在开发者模式中"），不能在正常模式中输入 `/exit`（提示"当前不在开发者模式中"）
- `dev/` 通过 .gitignore 排除，无需 `/protect` 参与

## 涉及的代码改动

### SKILL.md
1. `/protect` 规则之后新增 `/dev` 和 `/exit` 完整执行规则

### references/output-templates.md
1. 新增 `dev Skill 模板` 章节
2. 新增 `exit Skill 模板` 章节

### .gitignore
- 项目根目录新建 `.gitignore`，预置 `dev/` 规则

### 指令清单（更新后）

| 指令 | 版本 | 说明 |
|---|---|---|
| `/{name}` | v1.0.0 | 与数字分身对话 |
| `/test` | v1.0.0 | 测试模式 |
| `/train` | v1.0.0 | 训练模式 |
| `/enrich` | v1.1.0 | 加权文件丰富模型 |
| `/protect` | v1.2.0 | 隐私保护向导 |
| `/dev` | v1.3.0 | **新增** — 进入开发者模式 |
| `/exit` | v1.3.0 | **新增** — 退出开发者模式 |
