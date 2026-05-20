---
name: status
description: Shows the current Mob1S persona model status. Use when the user types /status to view version info, core personality keywords, and last update time.
---

# Mob1S Status

Display the current model status.

## 开发者模式

检查 `.claude/.mode_dev` 文件是否存在：
- 存在 → 基路径 = `dev/.claude/`
- 不存在 → 基路径 = `.claude/`

## Procedure

1. If `{base}.claude/persona/model.md` and `{base}.claude/persona/CHANGELOG.md` exist, read them for full status
2. Output:
   - Version number
   - Core personality keywords
   - Last update time
   - Total test/training rounds (if CHANGELOG available)
