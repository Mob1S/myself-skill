---
name: end
description: Finalizes an active Mob1S /train session. ONLY use when the user types /end during Mob1S training mode. Outputs a Training Harvest Summary, updates model.md, writes CHANGELOG, bumps version +0.1, and exits training mode.
---

# /end — 训练模式：总结并退出

`/train` 的退出指令。输出《训练收获总结》，保存到文件。

## 模式检查

- 当前在 `/train` 中 → 执行总结退出流程
- 当前不在 `/train` 中 → 回复"当前不在训练模式中，/end 需要在 /train 中使用"
- 当前在 `/test` 中 → 回复"当前在测试模式中，请先 /fine 退出测试模式"

## 开发者模式

检查 `.claude/.mode_dev` 文件是否存在：
- 存在 → 基路径 = `dev/.claude/`
- 不存在 → 基路径 = `.claude/`

## 总结退出流程

1. **输出《训练收获总结》**，结构如下：

```
## 训练收获总结

### 新增语言特征
| 特征 | 说明 |
|---|---|
| ... | ... |

### 新增行为模式
| 模式 | 说明 |
|---|---|
| ... | ... |

### 模型已确认项
- ... ✓
- ... ✓
```

2. **保存收获**：如果 `{base}.claude/persona/model.md` 和 `{base}.claude/persona/CHANGELOG.md` 存在，写入更新，版本号 +0.1
3. **宣布**：输出新版本号
4. **退出训练模式**
