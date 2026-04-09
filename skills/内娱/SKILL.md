---
name: create-star-skill
description: 创建明星/偶像/公众人物数字人格 Skill 的工坊框架。将歌手/偶像/明星转化为可对话的 AI 数字分身。
version: "1.0"
read:
  - README.md
  - prompts/intake.md
user-invocable: true
triggers:
  - /create-star
  - /star-wizard
---

<div align="center">

# ⭐ 内娱.skill 工坊

> 这鱼，这鱼可以，那鱼不行。那鱼为什么不行？那鱼完了。

**将喜欢的歌手/明星转化为可对话的 AI 数字分身**

[详细说明](README.md) · [创建向导](prompts/intake.md)

</div>

> 这鱼，这鱼可以，那鱼不行。那鱼为什么不行？那鱼完了。

---

## 是什么

内娱.skill 是一个**数字人格创建工坊**，将喜欢的歌手/明星转化为可对话的 AI Skill。

参照 [colleague-skill](https://github.com/titanwings/colleague-skill)（by titanwings）架构，专注文娱场景：

| 跑路了 | 用哪个 |
|--------|--------|
| 同事跑了 | [同事.skill](https://github.com/titanwings/colleague-skill) |
| 前任跑了 | [前任.skill](https://github.com/titanwings/ex-skill) |
| **喜欢的歌手/明星** | **⭐ 内娱.skill** |

---

## 快速开始

```bash
# 安装
git clone https://github.com/yanghaoraneve/star-skill ~/.openclaw/workspace/skills/create-star

# 输入 /create-star 开始创建向导
```

---

## 目录结构

```
star-skill/
├── SKILL.md              # 工坊入口
├── README.md             # 使用文档
│
├── prompts/              # Prompt 模板
│   ├── intake.md             # 信息录入向导
│   ├── persona_builder.md     # Persona 5层生成
│   ├── meta_builder.md        # meta.json 生成
│   ├── knowledge_router.md    # 知识库路由配置
│   └── correction_handler.md  # 纠正处理
│
├── tools/                # Python 工具
│   ├── lyrics_fetcher.py      # 歌词采集（网易云）
│   ├── bilibili_fetcher.py    # B站数据采集
│   ├── weibo_fetcher.py       # 微博采集
│   ├── role_script_builder.py # 角色台词本构建
│   ├── knowledge_builder.py    # 知识库构建
│   ├── skill_generator.py     # Skill 文件生成
│   └── version_manager.py      # 版本管理
│
└── docs/
    └── PRC.md           # 项目需求文档
```

---

## 命令参考

| 命令 | 说明 |
|------|------|
| `/create-star` | 启动创建向导 |
| `/list-stars` | 列出所有偶像 Skill |
| `/{slug}` | 调用完整人格 |
| `/star-rollback {slug} {version}` | 回滚版本 |

详见 [README.md](README.md)
