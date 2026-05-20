# /dev 开发者模式修复设计

## 问题

当前 `/dev` 开发者模式存在三个硬伤：

1. **dev 沙箱的指令 skill 不可达**：Claude Code 只在 `.claude/skills/` 发现 skill，`dev/.claude/skills/` 里的文件永远不会被触发
2. **dev 模式检测从未实现**：根目录指令 skill 声明了 Step 0（判定工作目录），但没有实际的状态检测机制
3. **两套指令 skill 不同步**：根目录版本和 dev 版本内容不一致，维护负担重

后果：用户在 dev 沙箱中蒸馏完成后，无法自然调用 `/test`、`/train`、`/{name}` 等指令，必须用自然语言描述才能触发对应模块。

## 方案

方案 A（选用）：**模式标记文件 + 单套指令 skill**

- `/dev` 进入时创建标记文件 `.claude/.mode_dev`
- `/exit` 退出时删除标记文件
- `.claude/skills/` 下只保留一套指令 skill，通过检测标记文件决定读哪个路径
- 删除 `dev/.claude/skills/` 目录

备选方案 B（放弃）：进出 dev 时备份/重建指令 skill —— 备份恢复脆弱，中途崩溃风险高。

备选方案 C（放弃）：中央调度 —— skill 间调用增加延迟和依赖。

## 核心机制

### 模式标记文件

- 路径：`.claude/.mode_dev`
- 内容：空文件（或写入时间戳用于调试）
- 存在 = 开发者模式激活
- 不存在 = 正常模式
- `.gitignore` 排除此文件

### 指令 skill 统一 Step 0

所有指令 skill（test、train、next、fine、end、status、enrich、protect、`/{name}`）在第一步执行：

```
Step 0：判定工作目录
- 检查 .claude/.mode_dev 是否存在
- 存在 → 基路径 = dev/.claude/
- 不存在 → 基路径 = .claude/
```

后续所有读写操作使用基路径。每个指令 skill 的自身逻辑不变。

## `/dev` 进入流程

### 首次进入（dev/ 不存在）

1. 创建目录：`dev/.claude/persona/`、`dev/mock/`
2. 创建标记文件 `.claude/.mode_dev`
3. `.gitignore` 追加 `dev/` 和 `.claude/.mode_dev`（若已存在则跳过）
4. 询问用户是否需要生成 mock 测试数据
5. 提示"开发者沙箱已就绪。进入开发者模式 [dev]"

### 再次进入（dev/ 已存在）

1. 检查 `.claude/.mode_dev` 是否存在
2. 已存在 → 提示"已在开发者模式中。输入 /exit 退出。"
3. 不存在 → 创建标记 → 提示"进入开发者模式 [dev]"

### 不再执行的操作

- 不再创建 `dev/.claude/skills/` 目录

## `/exit` 退出流程

1. 检查 `.claude/.mode_dev` 是否存在
2. 不存在 → 回复"当前不在开发者模式中。"
3. 存在 → 删除标记文件 → 提示"已退出开发者模式，路径恢复。dev/ 目录已保留。"

`dev/` 目录及其中所有内容完整保留。下次 `/dev` 回到同一沙箱。

## 指令 skill 生成策略

### 原则

指令 skill 只在 `.claude/skills/` 存在一套，没有第二套。

### 三种状态

| 状态 | 说明 |
|---|---|
| 预置模板 | myself skill 安装时自带，含 "Mob1S" 占位名和通用 Step 0 |
| 蒸馏后生成 | Phase 2 完成后，用用户真实名字替换占位，重新生成 |
| dev 模式 | 同上，但 Step 0 检测到标记文件后自动指向 `dev/.claude/` |

### 关键点

- 指令 skill 不再区分 dev 版和正常版——同一套文件，路径由标记决定
- Phase 2 生成指令 skill 时无需区分模式，写回 `.claude/skills/` 即可
- `/dev` 进入时不创建 `dev/.claude/skills/`

## 完整体验路径

```
用户输入 /dev
  → 创建沙箱 + .claude/.mode_dev
  → [dev] 开发者模式

走 Phase 1-2 蒸馏
  → persona 文件写入 dev/.claude/persona/
  → 更新 .claude/skills/ 下的指令 skill（占位名 → 真实名字）

调用 /{name}
  → Step 0 检测标记存在 → 读 dev/.claude/persona/model.md
  → "你好，你是谁？"

调用 /test
  → Step 0 检测标记存在 → 读 dev/.claude/persona/
  → 正常进入测试模式

调用 /train
  → 同上

调用 /exit
  → 删除 .claude/.mode_dev
  → dev/ 保留
  → 回到正常模式
```

## 影响范围

### 需修改的文件

| 文件 | 变更 |
|---|---|
| `SKILL.md` | 更新 `/dev` 和 `/exit` 执行规则（创建/删除标记文件，不再创建 `dev/.claude/skills/`） |
| `.claude/skills/test/SKILL.md` | Step 0 改为检查 `.claude/.mode_dev` |
| `.claude/skills/train/SKILL.md` | 同上 |
| `.claude/skills/next/SKILL.md` | 同上 |
| `.claude/skills/fine/SKILL.md` | 同上 |
| `.claude/skills/end/SKILL.md` | 同上 |
| `.claude/skills/status/SKILL.md` | 同上 |
| `.claude/skills/enrich/SKILL.md` | 同上 |
| `.claude/skills/protect/SKILL.md` | 同上 |
| `.claude/skills/dev/SKILL.md` | 同上 |
| `.claude/skills/exit/SKILL.md` | 同上 |
| `references/output-templates.md` | 更新指令 Skill 模板，Step 0 使用标记文件检测；dev 和 exit Skill 模板更新 |
| `.gitignore` | 追加 `.claude/.mode_dev` |

### 需删除的文件

| 路径 | 原因 |
|---|---|
| `dev/.claude/skills/` 整个目录 | 不可达，已被标记文件机制替代 |

### 不影响

- `dev/.claude/persona/` 及所有 persona 文件
- `dev/mock/` mock 数据
- Phase 1-5 蒸馏流程的核心逻辑
- `/enrich`、`/protect` 功能逻辑
- 版本管理规范

## 测试验证

1. `/dev` 首次进入 → 确认沙箱创建、标记文件存在
2. 在 dev 模式中调用 `/test` → 确认读取 `dev/.claude/persona/`
3. 在 dev 模式中调用 `/train` → 同上
4. 在 dev 模式中调用 `/{name}` → 同上
5. `/exit` → 确认标记文件已删除、路径恢复
6. 退出后调用 `/test` → 确认读取 `.claude/persona/`
7. `/exit` 后再次 `/dev` → 确认沙箱数据完整保留
