# CHANGELOG

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
