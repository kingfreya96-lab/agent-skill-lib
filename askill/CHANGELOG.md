# askill 更新日志

> 每次改动 askill（命令 / 分类规则 / 导出 / 体检逻辑）在此记一条。日志跟着技能走，不写进项目级工作日报。

## 2026-11-26

### 两级分类（分类 / 场景）
- 新增 `cat_root()` / `cat_sub()`，兼容 `分类/场景` 两级路径，原有单级分类无需重写
- `meta.py` 新增 `--groups` / `--group <分组>`，技能元按场景分节，检索可只进一组
- `index.json` 新增 `groups` 字段，统计每个场景的技能数
- `cmd_export` 总览改为「分类 → 场景」两级排列，笔记 tags 同时打根分类与场景标签
- 效果：172 个技能从 17 个根分类细化为 45 个场景分组

### 新增 `askill dupes` 重复检测
- 基于 difflib 比对技能名（去横杠）与归一化摘要，取 `max(名字相似度, 描述相似度)` 排序
- 阈值 0.62 → 70 对；阈值 0.75 → 19 对
- 设计原则：只排序提示、不自动合并 —— 误删有效技能的代价高于保留冗余
- 配合对 22 个技能做描述去重 + 头部 `> **区分**` 段（不动正文、不合并），阈值 0.75 下 19 对降到 8 对

### 修复
- `~/.local/bin/askill` 软链原指向不存在的 `bin/askill`，改指 `askill/bin/askill`，命令恢复
- 修复 `pm-meta/SKILL.md` 里 6 处失效的 `~/.codex/skills/` 旧路径

### 仓库结构
- 脚本与 README 收拢至 `askill/` 子目录，仓库根不再混放项目级文件（规则、工作日报不进仓库）
- 仓库边界：一个仓库只装 askill 本体，根目录 README 由用户在 GitHub 首页手动维护

### 体检
- `askill meta` → 16 个技能元刷新（手写 pm-meta 保护性跳过）
- `askill export` → 189 笔记 + 1 总览（分类 / 场景两层布局）
- `askill doctor` → 通过；claude / codex / trae / agents / workbuddy / loomy 软链 189 / 断链 0 / 缺分发 0

## 2026-11-26（初始）

- 首版：单一技能库 `~/.agent-skills/` + 软链分发 + 分类技能元 + 导出 Obsidian + 自动备份
- 命令：`agents` / `add` / `collect` / `migrate` / `meta` / `doctor` / `link` / `unlink` / `clean` / `remove` / `export`