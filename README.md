# agent-skill-库

一处安装、所有 agent 共用的技能库管理工具。技能只存一份真实副本，各 agent 技能目录通过软链指向它，避免多副本内容漂移；同类技能自动生成「技能元」索引，先检索 3 行摘要再加载全文，节省 token。

## 能做什么

- 单一技能库 + 软链分发：技能只存 `~/.agent-skills/` 一份，各 agent 软链指向它，改一处全生效
- 收编散落副本：自动去重、按修改时间合并版本
- 分类技能元：同类技能生成 `xxx-meta` 索引，3 行摘要快速定位，省 token
- 一键体检：断链 / 缺分发 / 漂移副本 / 空目录
- 导出 Obsidian：分类总览 + 每技能一篇，带双链和 tags
- 自动备份：所有变更前备份到 `~/.agent-skills/.backups/`

## 安装

依赖：Python 3.10+、PyYAML。

```bash
git clone git@github.com:kingfreya96-lab/agent-skill-库.git
cd agent-skill-库
pip install pyyaml
ln -s "$(pwd)/bin/askill" ~/.local/bin/askill
```

> `~/.local/bin` 需在 `PATH` 中；否则软链到任意已在 `PATH` 的目录即可。

## 命令速查

| 命令 | 用途 |
|---|---|
| `askill agents` | 查看探测到的 agent 技能目录 |
| `askill add <目录\|git-url>` | 安装技能进库并分发到所有 agent |
| `askill collect --apply` | 收编任意 agent 里新建的技能 |
| `askill migrate --apply` | 收编重复副本（默认只收编 ≥2 份） |
| `askill meta` | 生成 / 刷新分类技能元 |
| `askill doctor` | 体检（断链 / 缺分发 / 漂移 / 空目录） |
| `askill link [技能...]` | 补 / 刷新软链 |
| `askill unlink [技能...]` | 移除软链 |
| `askill clean --apply` | 清理空目录 / 断链 |
| `askill remove <技能>` | 删除技能 |
| `askill export --to <目录>` | 导出 Obsidian 笔记 |

## 目录结构

```
.
├── askill/          # askill 的 skill 描述（SKILL.md）
├── bin/askill       # 命令行入口（Python 脚本）
├── .trae/rules/     # 项目维护规则
└── README.md
```

## 技能库约定

| 路径 | 用途 |
|---|---|
| `~/.agent-skills/` | 技能库（唯一真实副本） |
| `~/.agent-skills/.backups/` | 变更前备份 |
| `~/.agent-skills/config.json` | 分类映射等配置 |