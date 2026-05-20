# CHANGELOG

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
