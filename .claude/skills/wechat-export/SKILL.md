---
name: wechat-export
description: Guides the user through exporting WeChat chat records for persona distillation. Use when the user types /wechat-export or asks how to export WeChat chat logs. Recommends WeFlow as the primary tool with WechatDump202601 as key-extraction fallback. Assists with download, installation, step-by-step usage, and seamless handoff to Phase 2 distillation.
---

# /wechat-export — 微信聊天记录导出引导

引导用户导出微信聊天记录，用于人格蒸馏。推荐工具链（WeFlow 主力 + WechatDump202601 兜底），协助下载安装，逐步引导使用，导出后无缝衔接到蒸馏流程。

## 进入流程

### Step 0：判定工作目录

检查 `.claude/.mode_dev` 文件是否存在：
- 存在 → 基路径 = `dev/.claude/`
- 不存在 → 基路径 = `.claude/`

### Step 1：推荐工具链

首次使用时输出：

```
要导出微信聊天记录，推荐使用 WeFlow（https://github.com/hicccc77/WeFlow）——开源的微信聊天记录分析和导出工具，完全本地运行，支持导出为 TXT、HTML、JSON 等格式。

WeFlow 是开源工具（GitHub 7k+ stars），完全本地运行，不会上传你的聊天记录。但如果你有顾虑，也可以手动复制粘贴——在微信中选中对话，复制后粘贴到 txt 文件里发给我。如果你使用的模型支持多模态，直接截图粘贴给我也行。

需要我帮你下载安装吗？
```

### Step 2：安装引导

#### 2.1 检测平台

识别当前操作系统（Windows / macOS / Linux）。若非这三者，回复："当前平台暂不支持自动安装。建议你手动访问 https://github.com/hicccc77/WeFlow/releases 下载对应版本，或者使用手动复制粘贴的方式。"

#### 2.2 下载

从 GitHub Releases 下载最新版本到用户下载目录。

**下载前告知**：
```
即将从 GitHub Releases 下载 WeFlow 最新版本到你的下载目录。确认下载？
```

用户确认后执行下载。

#### 2.3 安装

下载完成后询问：

```
下载完成。是否授权我自动安装？
```

- **授权** → 执行安装：
  - Windows: 运行 `.exe` 安装包
  - macOS: 挂载 `.dmg` 并拷贝到 `/Applications/`
  - Linux: `chmod +x` AppImage 或解压 tar.gz
- **不授权** → 告知文件位置，等用户手动操作后说"好了"
- **安装失败** → 告知错误信息，回退："安装似乎遇到了问题。你可以手动双击安装包完成安装，完成后告诉我'好了'。"

### Step 3：逐步使用引导

每一步等用户确认后再进入下一步。

#### 3.1 打开 WeFlow

```
打开 WeFlow，它会自动检测你电脑上的微信。确认微信已登录，然后告诉我"好了"。
```

如果用户反馈 WeFlow 提示"未检测到微信"：
- 确认微信版本 ≥ 4.0
- 确认微信已登录
- 建议重启 WeFlow 重试

#### 3.2 获取密钥

WeFlow 自动尝试获取数据库密钥。

**成功路径**：
```
密钥获取成功。接下来在 WeFlow 中选择你要导出的联系人或群聊，然后告诉我"好了"。
```

**失败路径**：
```
自动获取密钥失败了。可以试试 WechatDump202601（https://github.com/Zst0NE/WechatDump202601），这是一个专门的微信数据库密钥提取工具。下载运行后，将获取到的密钥填入 WeFlow 的"手动输入密钥"即可。

需要我帮你下载 WechatDump202601 吗？
```

如果用户需要下载 WechatDump202601：告知仓库地址，由用户自行下载。该工具信息未完全确认，不主动代为下载。

用户搞不定密钥 → 不阻塞：
```
没关系。你也可以直接在微信中选中聊天记录，复制后粘贴到 txt 文件里发给我。如果你用的模型支持多模态，截图发我也行。
```

#### 3.3 导出聊天记录

```
在 WeFlow 中选择导出格式为 TXT，选择导出范围（建议选"全部"），点击导出。完成后把文件路径发给我。
```

### Step 4：衔接到蒸馏流程

导出完成后：

```
聊天记录已导出成功。要不要我直接加载这个文件开始蒸馏分析？
```

- **用户同意** → 读取 TXT 文件内容，进入 Phase 2 的 6 维度分析流程（参考主 SKILL.md Phase 2 Step 2-3）。如果 `{base}.claude/persona/model.md` 已存在，按增量更新处理，版本号 +0.1；如果不存在，按初次蒸馏流程生成所有文件。
- **用户拒绝** → "好的，文件在 `{path}`。你随时可以把文件发给我，或输入 /enrich 来蒸馏。"

## 边界处理

| 场景 | 处理 |
|---|---|
| 不在 Windows/macOS/Linux | 告知不支持自动安装，建议手动下载或复制粘贴 |
| 下载失败（网络问题） | 告知下载链接，建议用户手动下载 |
| 安装失败 | 告知错误信息，回退到手动安装模式 |
| 密钥获取两次都失败 | 不阻塞，建议回退到手动复制粘贴或截图 |
| WechatDump202601 | 仅告知仓库地址和用途，不主动下载 |
| 用户在 /test 或 /train 中 | 不受锁定，正常执行 |
| 用户已安装 WeFlow | 跳过安装步骤，直接进入使用引导 |

## 模式互斥

`/wechat-export` 不受 `/test` / `/train` 模式锁定，随时可用。
