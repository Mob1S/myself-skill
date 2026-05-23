# CHANGELOG

## [v1.7.0] - 2026-05-23
- **新增** `/doubao` 指令模块：导出豆包智能体 soul.md（2000字人格压缩）
- 模板骨架：身份 / 性格 / 说话方式 / 情绪 / 人物档案 / 底线
- 隐私过滤：真名脱敏（支持用户自定义外号）、地点模糊化、转账删除、敏感事件抽象化
- 裁剪优先级：熟人圈先删 → 好友圈精简 → 说话风格压缩 → 核心圈/底线不动
- 身份段可选精简模式（只留名字），省下字数分配给其他段落
- 输出草稿供用户确认后写入 `.claude/persona/soul.md`
- SKILL.md 新增 `/doubao` 完整执行规则
- 文件结构新增 `doubao/SKILL.md`、`soul.md`

## [v1.6.0] - 2026-05-21
- **新增** `/wechat-export` 指令模块：微信聊天记录导出引导
- 推荐 WeFlow（主力）+ WechatDump202601（密钥兜底）工具链
- 协助下载安装：检测平台 → 下载 → 用户授权后自动安装
- 逐步引导使用：打开 WeFlow → 获取密钥 → 导出 TXT
- 密钥获取失败时推荐 WechatDump202601 手动提取，不阻塞流程
- 导出后无缝衔接到 Phase 2 蒸馏分析
- 首次推荐附隐私说明和手动替代方案（复制粘贴/截图）
- Phase 2 Step 1 末尾联动提示 `/wechat-export` 可用
- SKILL.md 新增 `/wechat-export` 完整执行规则
- output-templates.md 新增 `/wechat-export` Skill 模板、rules.md 模板新增规则
- README.md 核心指令表新增 `/wechat-export`

## [v1.5.0] - 2026-05-21
- **新增** `/deep` 指令模块：深度二次阅读聊天记录，挖掘首次蒸馏遗漏的细节
- 四层深度分析框架：语言指纹 → 行为模式 → 人物关系 → 底层价值观（新增维度）
- 每个发现至少 3 条原文证据支撑，不足 3 条的仅为"待验证线索"
- 按维度分组展示发现，用户逐组确认后写入模型
- 更新 model.md（增量）、linguistic-fingerprint.md（底部追加）、person-profiles.md（合并）
- Phase 2 Step 4 末尾联动提示 `/deep` 可用
- SKILL.md 新增 `/deep` 完整执行规则
- output-templates.md 新增 `/deep` Skill 模板、rules.md 模板新增 `/deep` 模式规则
- 文件结构和指令列表新增 `deep/SKILL.md`

## [v1.4.0] - 2026-05-21
- **新增** `/{name}` 人格调用 Skill：用户输入 `/{name}` 与蒸馏后的人格模型对话
- 对话启动协议：首条消息"你好，你是谁？"→ 身份匹配 → 按档案规则对话
- 采用引用式设计，运行时读取 model.md，模型更新后自动反映最新版本
- **新增** `/bye` 指令：退出 `/{name}` 对话模式，退出前以人格语气输出结束语
- **更新** 模式互斥：`/{name}`、`/test`、`/train` 三者互斥，含完整互斥矩阵
- Phase 2 Step 3.8 生成列表新增 `{name}` 和 `bye`
- output-templates.md 新增 `/{name}` 和 `/bye` Skill 模板、rules.md 模板更新互斥表
- 主 SKILL.md 更新 Phase 2 完成告知和文件结构总览
- README.md 更新指令列表和输出结构

## [v1.3.1] - 2026-05-20
- **修复** `/dev` 开发者模式指令 skill 无法触发的 bug
- 引入 `.claude/.mode_dev` 标记文件作为模式检测机制
- 所有指令 skill 的 Step 0 统一检查标记文件判断基路径
- 删除不可达的 `dev/.claude/skills/` 目录
- 更新主 SKILL.md、所有指令 skill、output-templates.md 模板
- `.gitignore` 追加 `.claude/.mode_dev`

## [v1.3.0] - 2026-05-20
- **新增** `/dev` 指令模块：进入开发者模式，创建隔离沙箱 `dev/`
- **新增** `/exit` 指令模块：退出开发者模式
- 沙箱内所有指令路径自动重定向到 `dev/.claude/`，与真实数据完全隔离
- 可选 mock 测试数据生成（身份档案 + 聊天记录）
- 项目根目录新增 `.gitignore` 预置 `dev/` 规则
- SKILL.md 新增 `/dev` 和 `/exit` 完整执行规则
- output-templates.md 新增 dev 和 exit Skill 模板

## [v1.2.0] - 2026-05-20
- **新增** `/protect` 指令模块：隐私保护向导
- 扫描蒸馏产物，按敏感度三级（高/中/低）自动评级
- 交互式确认：用户可逐文件选择排除或公开
- 生成 `.gitignore` 保护隐私文件不被 git 追踪
- 输出隐私报告（已排除 vs 对外可见）
- SKILL.md 新增 `/protect` 完整执行规则
- output-templates.md 新增 protect Skill 模板、rules.md 模板新增 `/protect` 模式规则

## [v1.1.0] - 2026-05-20
- **新增** `/enrich` 指令模块：加权文件丰富人格模型
- Phase 2 Step 1 追加多文件引导语，提示用户可附带其他文本文件
- SKILL.md 新增 `/enrich` 完整执行规则（7 Step 流程）
- output-templates.md 新增 `/enrich` Skill 模板、rules.md 模板新增 `/enrich` 模式规则
- 文件结构新增 `enrich/SKILL.md`

## [v1.0.0] - 2026-05-20
- 初始版本：5 阶段人格蒸馏向导
- Phase 1-5 完整流程定义
- 语言指纹 6 维度分析
- 输出模板（base-identity, linguistic-fingerprint, model, rules, CHANGELOG, person-profiles, scenario-pool）
- 6 个指令 Skill 模板（test, train, next, fine, end, status）
- 中文 README 文档
