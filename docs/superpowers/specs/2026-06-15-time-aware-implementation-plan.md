# 时间感知蒸馏——实现计划

> 基于设计文档：`2026-06-15-time-aware-distillation-design.md`

## 改动总览

| # | 文件 | 改动 |
|---|---|---|
| 1 | `SKILL.md` | Phase 2 流程改造（新增 Step 2/4，改造 Step 3/5） |
| 2 | `SKILL.md` | `/deep` 联动时段分析 |
| 3 | `SKILL.md` | `/enrich` 要求时间信息 |
| 4 | `SKILL.md` | `/doubao` 锚点感知 |
| 5 | `references/output-templates.md` | `model.md` 模板新增字段 |
| 6 | `SKILL.md` | `/status` 展示锚点信息 |
| 7 | `CHANGELOG.md` | 记录 v2.0.0 变更 |

## Step 1：改造 SKILL.md Phase 2

**文件**：`SKILL.md`

**当前 Phase 2 结构**：
```
Step 1：请求聊天记录
Step 2：分析聊天记录（六维度）
Step 3：生成文件
Step 4：告知完成
```

**新 Phase 2 结构**：
```
Step 1：请求聊天记录（不变）
Step 2：时间预处理（新增）
Step 3：分段六维度分析（改造原 Step 2）
Step 4：变化轨迹展示 + 用户确认（新增）
Step 5：加权合并生成模型（改造原 Step 3）
Step 6：告知完成（原 Step 4）
```

### Step 2 新增内容：时间预处理

在 SKILL.md 的 `### Step 2：分析聊天记录` 之前插入新的 Step 2，原 Step 2-4 顺延为 Step 3-6。

新增内容要点：
- 时间提取逻辑：识别常见时间戳格式，每条消息关联时间点
- 跨度判定表：< 3 个月不分段，3 月~1 年按月，1~3 年按季度，3 年以上按半年/年
- 合并兜底：消息量过少的时段与相邻时段合并
- 断层检测：标记空白期，不跨断层推断，用户可手动调整

### Step 3 改造：分段六维度分析

在原 Step 2 的六维度分析基础上：
- 每个时段独立执行六维度分析
- 标注时段范围
- 分析完成后自动生成跨时段对比（仅展示有显著变化的维度）

### Step 4 新增内容：变化轨迹展示

- 以可视化格式展示变化轨迹（高频词变迁、句式演变、Emoji 变迁、情绪表达变化、对话角色变化、特殊语言习惯变化）
- 展示后询问用户确认
- 用户可：确认 / 修正（排除时段）/ 断层调整
- 确认后才进入 Step 5

### Step 5 改造：加权合并生成模型

- 默认衰减策略：越近权重越高，自适应曲线
- 用户排除的时段权重归零
- 默认锚定最新时段
- model.md 新增 `anchor_period` 字段和 `## 时间轴` 段落

### 短跨度兼容

跨度 < 3 个月时，跳过 Step 2 和 Step 4，走原始流程。

## Step 2：SKILL.md /deep 联动

**位置**：SKILL.md 中 `/deep 指令` 部分

改动：
- Step 3 深度分析增加说明：按时段分析，补充到对应时段的分析结果中
- Step 5 更新文件时，更新对应时段的分析数据

## Step 3：SKILL.md /enrich 联动

**位置**：SKILL.md 中 `/enrich 指令` 部分

改动：
- Step 3 权重设置新增时间信息要求：用户提供文件对应的时间信息
- Step 5 合并时纳入时段权重

## Step 4：SKILL.md /doubao 联动

**位置**：SKILL.md 中 `/doubao 指令` 部分

改动：
- Step 4 读取源文件时，使用当前锚点时段的模型

## Step 5：output-templates.md model.md 模板改动

**文件**：`references/output-templates.md`

在 model.md 模板头部新增：
```markdown
| 锚点时段 | {anchor_period} |
```

在 model.md 模板中新增 `## 时间轴` 段落（在 Correction 记录之前）：
```markdown
## 时间轴

> 记录各时段的权重分配和锚点信息。

| 时段 | 权重 | 说明 |
|---|---|---|
| {period_1} | {weight_1} | {note_1} |
| {period_2} | {weight_2} | {note_2} |
| ... | ... | ... |

当前锚点：{anchor_period}
```

## Step 6：SKILL.md /status 展示锚点

**位置**：SKILL.md 中 `/status 指令` 部分，以及 output-templates.md 中 /status Skill 模板

改动：输出新增锚点时段信息

## Step 7：CHANGELOG.md

记录 v2.0.0 变更：
- Phase 2 新增时间预处理和变化轨迹展示
- 分段六维度分析
- 加权合并与锚点机制
- /deep、/enrich、/doubao 联动
- model.md 模板新增字段

## 执行顺序

1. 改 `references/output-templates.md`（model.md 模板）
2. 改 `SKILL.md` Phase 2（核心改动）
3. 改 `SKILL.md` /deep 联动
4. 改 `SKILL.md` /enrich 联动
5. 改 `SKILL.md` /doubao 联动
6. 改 `SKILL.md` /status + output-templates.md /status 模板
7. 更新 `CHANGELOG.md`
8. 提交 + 打 tag
