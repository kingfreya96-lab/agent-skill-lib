# 项目规则

## 1. 工作日报

当天完成的工作内容，整理成工作日报，记录：
- 完成了什么
- 遇到的问题与解决方式
- 下一步计划

## 2. Obsidian 同步

产生新内容（新技能、文档、笔记）时，同步到对应的 Obsidian 文档中：
- 技能库内容 → `OPC/AI Agent/skills/`（用 `askill export --to "/Users/freya/Downloads/Documents/Obsidian Vault/OPC/AI Agent/skills"`）
- 其他内容 → Obsidian Vault 下对应主题目录

## 3. 技能同步到共享库

项目中创建的 skill **需要**同步到共享技能库 `~/.agent-skills/`，保证所有 agent 可用、内容单一来源不漂移。

判定标准：
- **要同步**：通用可复用的 skill（跨 agent 都能用、无厂商专属内容、无敏感信息）
- **不要同步**：厂商专属 skill、含密钥 / 账号 / 代理的 skill、一次性临时脚本

流程：
1. 新建技能后运行 `askill collect --apply` 收编进库
2. 运行 `askill meta` 刷新分类技能元
3. 运行 `askill export` 同步到 Obsidian
4. 运行 `askill doctor` 确认分发无缺口