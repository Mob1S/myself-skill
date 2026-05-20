---
name: exit
description: Exits Mob1S developer mode. Use when the user types /exit during dev mode. Switches all instruction paths back to the real .claude/ directory. The dev/ sandbox directory is preserved on disk.
---

# /exit — Mob1S 退出开发者模式

切回正常模式，路径恢复指向真实 `.claude/`。

## 模式检查

检查 `.claude/.mode_dev` 文件是否存在：
- 存在 → 当前在开发者模式中，执行退出流程
- 不存在 → 回复"当前不在开发者模式中。"

## 退出流程

1. 删除标记文件 `.claude/.mode_dev`
2. 路径恢复：所有指令读写基路径从 `dev/.claude/` 切回 `.claude/`
3. 输出：

```
已退出开发者模式，路径恢复。dev/ 目录已保留，下次输入 /dev 继续使用。
```

## 退出后

- `dev/` 目录保留在磁盘上
- 所有指令路径恢复指向真实 `.claude/`
