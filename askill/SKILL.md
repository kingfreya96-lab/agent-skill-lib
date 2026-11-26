---
name: askill
description: 一处安装、所有 agent 共用的技能库管理工具（软链分发 + 收编副本 + 分类技能元 + 导出 Obsidian）。当需要安装技能、收编 agent 中新建的 skill、刷新技能元、体检分发状态、或同步技能到 Obsidian 时调用。
---

# askill — 一处安装，所有 agent 共用技能

解决多 agent 环境下技能重复安装、内容漂移、定位困难、token 消耗高的问题。

## 核心机制

1. **单一技能库** `~/.agent-skills/`：技能只存一份真实副本。
2. **软链分发**：各 agent 技能目录通过软链指向技能库，改一处全部生效。
3. **技能元**：同类技能自动生成 `分类-meta` 索引，只返回 3 行摘要，先定位再加载全文，节省 token。
4. **自动备份**：所有变更前备份到 `~/.agent-skills/.backups/<时间戳>/`。

## 安装

脚本位于本仓库 `bin`，软链到 PATH 即可：

```bash
ln -s "$(pwd)/bin/askill" ~/.local/bin/askill
```

## 命令

### 查看
| 命令 | 用途 |
|---|---|
| `askill agents` | 查看探测到的 agent 技能目录 |
| `askill list` | 技能库与分发矩阵（谁有谁没有） |
| `askill doctor` | 体检（断链 / 缺分发 / 漂移副本 / 空目录） |
| `askill dupes [--threshold N]` | 找疑似重复/重叠的技能对（人工判断后再合并） |

### 安装与分发
| 命令 | 用途 |
|---|---|
| `askill add <目录\|git-url> [--name]` | 安装技能进库并分发到所有 agent |
| `askill link [技能...]` | 补 / 刷新软链 |
| `askill unlink [技能...]` | 移除软链 |

### 收编（把散落副本并进技能库）
| 命令 | 用途 |
|---|---|
| `askill collect --apply [--only 名] [--from-agent x]` | 收编任意 agent 里新建的技能（单个也算），收编后自动刷新技能元 |
| `askill migrate --apply [--all]` | 收编已有副本（默认只收编 ≥2 份重复出现的，避免污染厂商专属技能） |

### 技能元
| 命令 | 用途 |
|---|---|
| `askill meta [--min N] [--group-by auto\|prefix\|flat]` | 生成 / 刷新分类技能元 |

检索示例（路径全机唯一，任何 agent 里都是同一条命令）：

```bash
python3 ~/.agent-skills/office-meta/meta.py --list
python3 ~/.agent-skills/office-meta/meta.py --keyword "表格"
python3 ~/.agent-skills/office-meta/meta.py --name docx
```

### 维护
| 命令 | 用途 |
|---|---|
| `askill clean --apply` | 清理空目录 / 断链 |
| `askill remove <技能> [--purge]` | 删除技能（--purge 连同备份一起删） |

### 导出
| 命令 | 用途 |
|---|---|
| `askill export --to <obsidian目录> [--layout flat\|subdir\|category]` | 把技能库导出为 Obsidian 笔记（分类总览 + 每技能一篇，带 tags 和双链） |

## 典型工作流

```bash
askill doctor                    # 先体检，看当前状态
askill add <新技能>              # 装外部技能
askill collect --apply           # 收编某 agent 里新建的技能
askill meta                      # 刷新技能元（collect/add 已自动做）
askill export --to "~/Downloads/Documents/Obsidian Vault/OPC/AI Agent/skills"
```

## 目录约定

| 路径 | 用途 |
|---|---|
| `~/.agent-skills/` | 技能库（唯一真实副本） |
| `~/.agent-skills/.backups/` | 变更前备份 |
| `~/.agent-skills/config.json` | 分类映射等配置 |

## 分类规则

类别判定优先级：`config.json` 的 `categories`（显式指定，最高）→ frontmatter 的 `category` → `config.json` 的 `category_keywords` / 内置关键词表 → 名称前缀（≥2 个同名前缀才成类）→ `other`。同类不足 `--min`（默认 2）个不生成技能元。

**两级分类**：`categories` 的值支持 `分类/场景`，即「内容域 → 场景」二次分级。技能元按场景分节，`meta.py --groups` 看分组、`--group <场景>` 只进一组；导出 `--layout category` 也按两层建目录。想让分组更细，直接在 `config.json` 里逐个技能改值再跑 `askill meta`，无需改代码。


## 安全设计

- 只覆盖带 `.askill-generated` 标记的技能元，手写的技能元（如 `pm-meta`）永不覆盖。
- 某类技能减少到不足阈值时，旧技能元自动下线并备份。
- 厂商专属技能默认不收编，避免污染其他 agent 的调用列表；确需共享时用 `askill collect --only <名字>` 单独收编。