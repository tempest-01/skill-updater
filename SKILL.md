---
name: skill-updater
description: Local-modification-preserving clawhub skill updater. Saves modifications as a diff patch, fetches clawhub updates, applies patch to new version. Conflicts are reported with diff for manual decision.
metadata:
  openclaw:
    requires:
      bins: [python3, clawhub]
      env: [OPENCLAW_SKILLS_DIR]
    triggers: [skill更新, auto update, git sync]
---

# Skill-Updater

> 本地修改保护式 clawhub skill 更新器。运行前请先阅读 Before Installing。

## 核心功能

**用户修改了某个 clawhub skill 的文件** → clawhub 发布了新版 → Skill-Updater 在保留用户修改的前提下更新到新版。

## 工作原理

```
Step 1: 保存修改
  → 自动扫描 skill 目录，生成 unified diff patch
  → 备份原始文件到 .skill-updater/originals/

Step 2: clawhub 更新
  → clawhub update 下载新版文件

Step 3: 尝试合并
  → patch --dry-run 试探将修改应用到新版
  → 成功：直接写入新版 ✅
  → 失败：展示 diff 冲突，用户手动决策
```

## 文件结构

```
skill-dir/
├── .skill-updater/
│   ├── mod.patch      # 本地修改的 unified diff
│   └── originals/      # 安装时的原始文件快照
└── [skill 文件]
```

## 用法

```bash
# 预览：显示哪些 skill 有新版
python3 git_update.py

# 实际更新
python3 git_update.py --apply

# 仅更新指定 skill
python3 git_update.py --apply --skill <slug>

# 查看已保存的本地修改
python3 git_update.py --show-patch

# 丢弃修改（接受新版，放弃本地修改）
python3 git_update.py --discard --skill <slug>
```

## 冲突处理

| 情况 | 结果 |
|------|------|
| patch 成功应用 | 新版 + 修改均保留 ✅ |
| patch 失败（同行冲突） | 显示 diff，用户手动处理 |
| 无本地修改 | 直接 clawhub update ✅ |

## Requirements

- Python 3.8+
- `clawhub` CLI

## Before Installing

1. **Dry-run first** — 始终先用 `python3 git_update.py` 预览，不带 `--apply`
2. **Backup** — 对重要 skill 的修改，建议在 --discard 前手动备份
