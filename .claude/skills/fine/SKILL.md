---
name: fine
description: Finalizes an active Mob1S /test session. ONLY use when the user types /fine during Mob1S test mode. Saves all adjustments, updates model.md, writes CHANGELOG, bumps version +0.1, and exits test mode.
---

# /fine — 测试模式：保存并退出

`/test` 的退出指令。汇总本轮所有调整，保存到文件。

## 模式检查

- 当前在 `/test` 中 → 执行保存退出流程
- 当前不在 `/test` 中 → 回复"当前不在测试模式中，/fine 需要在 /test 中使用"
- 当前在 `/train` 中 → 回复"当前在训练模式中，请先 /end 退出训练模式"

## 开发者模式

检查 `.claude/.mode_dev` 文件是否存在：
- 存在 → 基路径 = `dev/.claude/`
- 不存在 → 基路径 = `.claude/`

## 保存退出流程

1. **汇总调整**：列出本轮测试中所有被标记为"不像"并调整过的项目
2. **保存调整**：如果 `{base}.claude/persona/model.md` 和 `{base}.claude/persona/CHANGELOG.md` 存在，写入更新，版本号 +0.1
3. **宣布**：输出新版本号和变更摘要
4. **退出测试模式**

## 示例输出

```
测试模式结束。

本轮调整：
- 新增语言特征："神了"作为"圣了"变体
- 新增行为模式：被调侃时反弹式反问

模型已更新至 V1.2。
```
