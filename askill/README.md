# agent-skill-lib

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
git clone git@github.com:kingfreya96-lab/agent-skill-lib.git
cd agent-skill-lib
pip install pyyaml
ln -s "$(pwd)/askill/bin/askill" ~/.local/bin/askill
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
| `askill dupes [--threshold 0.6]` | 找疑似重复/重叠的技能对（人工判断后再合并） |
| `askill link [技能...]` | 补 / 刷新软链 |
| `askill unlink [技能...]` | 移除软链 |
| `askill clean --apply` | 清理空目录 / 断链 |
| `askill remove <技能>` | 删除技能 |
| `askill export --to <目录>` | 导出 Obsidian 笔记（`--layout flat\|subdir\|category`） |

## 目录结构

```
.
├── askill/
│   ├── SKILL.md        # askill 的 skill 描述
│   ├── bin/askill      # 命令行入口（Python 脚本）
│   ├── README.md       # 本说明
│   └── CHANGELOG.md    # 更新日志
└── .gitignore
```

> 本仓库只装 `askill` 本体；项目级文件（`.trae/`、`docs/`、其他 skill 目录）不进本仓库。

## 分类配置

`~/.agent-skills/config.json` 决定技能归到哪个分类（影响 `askill meta` 生成的技能元与 `askill export` 的分组）：

```json
{
  "include_managed": false,
  "categories": {
    "prd-development": "pm/交付与落地",
    "user-story-splitting": "pm/交付与落地",
    "design-system": "life/游戏"
  },
  "category_keywords": {
    "pm": ["prd", "roadmap", "user-story"],
    "design": ["design", "ui", "ux"]
  }
}
```

| 字段 | 作用 |
|---|---|
| `categories` | 技能名 → 分类路径，**优先级最高**。格式 `分类` 或 `分类/场景`，后者是二次分级 |
| `category_keywords` | 分类 → 关键词，覆盖内置默认词表；用于未显式指定的新技能自动归类 |
| `include_managed` | 是否把 agent 自带（托管）技能也纳入库 |

归类优先级：`categories` 显式映射 → frontmatter `category` → `category_keywords` 关键词 → 技能名前缀（同前缀 ≥2 才成类）→ `other`。

### 两级分类：分类（内容域）→ 场景（二次分级）

`分类` 回答"这是什么领域的内容"，`场景` 回答"在这类里我遇到的是哪种情况"。技能元按场景分节呈现，检索时可只进一组：

```bash
python3 ~/.agent-skills/pm-meta/meta.py --groups          # 看 pm 下的场景分组
python3 ~/.agent-skills/pm-meta/meta.py --group "发现与验证"
```

导出到 Obsidian 时 `--layout category` 也会按 `分类/场景` 建两层目录。

## 技能库约定

| 路径 | 用途 |
|---|---|
| `~/.agent-skills/` | 技能库（唯一真实副本） |
| `~/.agent-skills/.backups/` | 变更前备份 |
| `~/.agent-skills/config.json` | 分类映射等配置 |