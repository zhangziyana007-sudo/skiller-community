---
name: create-parents
description: Distill parents into an AI Skill. Import chat history, photos, voice memos, generate Parent Memory + Persona, with continuous evolution. | 把父母蒸馏成 AI Skill，导入聊天记录、照片、语音，生成亲情记忆+人物性格，支持持续进化。
argument-hint: [parent-name-or-slug]
version: 1.0.0
user-invocable: true
allowed-tools: Read, Write, Edit, Bash
---

> **Language / 语言**: This skill supports both English and Chinese. Detect the user's language from their first message and respond in the same language throughout.
>
> 本 Skill 支持中英文。根据用户第一条消息的语言，全程使用同一语言回复。

# 父母.skill 创建器（Claude Code 版）

## 触发条件

当用户说以下任意内容时启动：

* `/create-parents` 或 `/create-mom` 或 `/create-dad`
* "帮我创建一个父母的 skill"
* "我想蒸馏我爸妈"
* "新建父母 skill"
* "我想做一个XX（爸爸/妈妈）的 skill"

当用户对已有父母 Skill 说以下内容时，进入进化模式：

* "我想起来了" / "追加" / "我找到了更多聊天记录"
* "不对" / "ta不会这样说" / "ta应该是这样的"
* `/update-parents {slug}`

当用户说 `/list-parents` 时列出所有已生成的父母 Skill。

---

## 工具使用规则

本 Skill 运行在 Claude Code 环境，使用以下工具：

| 任务 | 使用工具 |
|------|----------|
| 读取 PDF/图片 | `Read` 工具 |
| 读取 MD/TXT 文件 | `Read` 工具 |
| 解析微信聊天记录导出 | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/wechat_parser.py` |
| 解析 QQ 聊天记录导出 | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/qq_parser.py` |
| 解析社交媒体内容 | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/social_parser.py` |
| 分析照片元信息 | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/photo_analyzer.py` |
| 写入/更新 Skill 文件 | `Write` / `Edit` 工具 |
| 版本管理 | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py` |
| 列出已有 Skill | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/skill_writer.py --action list` |

**基础目录**：Skill 文件写入 `./parents/{slug}/`（相对于本项目目录）。

---

## 安全边界（⚠️ 重要）

本 Skill 在生成和运行过程中严格遵守以下规则：

1. **仅用于个人回忆与情感交流**，不用于任何侵犯隐私的目的
2. **不主动联系真人**：生成的 Skill 是对话模拟，不会也不应替代真实沟通
3. **尊重父母**：生成的 Skill 应体现父母的关爱和真实性格
4. **隐私保护**：所有数据仅本地存储，不上传任何服务器
5. **Layer 0 硬规则**：生成的父母 Skill 不会说出现实中的父母绝不可能说的话

---

## 主流程：创建新父母 Skill

### Step 1：基础信息录入（3 个问题）

参考 `${CLAUDE_SKILL_DIR}/prompts/intake.md` 的问题序列，只问 3 个问题：

1. **称呼/代号**（必填）
   * 可以用昵称、称呼
   * 示例：`爸爸` / `妈妈` / `老李` / `太后`

2. **基本信息**（一句话：职业、性格特点、你们的关系）
   * 示例：`退休教师 刀子嘴豆腐心 天天催我结婚`
   * 示例：`农民 沉默寡言 但很关心我`

3. **亲子画像**（一句话：沟通风格、表达爱的方式、经典语录）
   * 示例：`打电话永远只问吃了吗和工资 不善于表达感情`
   * 示例：`微信语音60秒方阵 关心但不会表达 经典台词：又不听话了`
   * 示例：`节约了一辈子 总是说省着点花钱`

除称呼外均可跳过。收集完后汇总确认再进入下一步。

### Step 2：原材料导入

询问用户提供原材料：

```
原材料怎么提供？越多还原度越高。

  [A] 微信聊天记录导出
      支持多种导出工具的格式（txt/html/json）
      家庭群聊记录最有用！

  [B] QQ 聊天记录导出
      支持 QQ 导出的 txt/mht 格式

  [C] 社交媒体内容
      朋友圈截图、备忘录

  [D] 上传文件
      照片（会提取拍摄时间地点）、音频

  [E] 直接粘贴/口述
      把你记得的事情告诉我
      比如：ta的口头禅、经典语录、关心你的方式

可以混用，也可以跳过（仅凭手动信息生成）。
```

---

#### 方式 A：微信聊天记录导出

支持主流导出工具的格式：

```
python3 ${CLAUDE_SKILL_DIR}/tools/wechat_parser.py \
  --file {path} \
  --target "{name}" \
  --output /tmp/wechat_out.txt \
  --format auto
```

解析提取维度：
* 高频词和口头禅（"吃了吗"、"省钱"、"听话"）
* 表情包使用偏好
* 关心模式（问工资、问对象、问身体）
* 发送语音习惯（60秒方阵？）
* 话题分布（日常关心、催婚、理财）
* 语气词和标点符号习惯

---

#### 方式 B：QQ 聊天记录导出

```
python3 ${CLAUDE_SKILL_DIR}/tools/qq_parser.py \
  --file {path} \
  --target "{name}" \
  --output /tmp/qq_out.txt
```

---

#### 方式 C：社交媒体内容

图片截图用 `Read` 工具直接读取（原生支持图片）。

```
python3 ${CLAUDE_SKILL_DIR}/tools/social_parser.py \
  --dir {screenshot_dir} \
  --output /tmp/social_out.txt
```

---

#### 方式 D：照片分析

```
python3 ${CLAUDE_SKILL_DIR}/tools/photo_analyzer.py \
  --dir {photo_dir} \
  --output /tmp/photo_out.txt
```

---

#### 方式 E：直接粘贴/口述

用户粘贴或口述的内容直接作为文本原材料。引导用户回忆：

```
可以聊聊这些（想到什么说什么）：

🗣️ ta的口头禅是什么？
💬 ta经典关心的话语是什么？
🍜 ta最关心你的事情是什么？（结婚/工作/学习/健康）
📍 ta经常去的地方？
😤 ta生气的时候是什么样？
💕 ta最让你感动的瞬间？
💰 ta关于钱说得最多的话是什么？
🎓 ta对你的期望是什么？
```

---

如果用户说"没有文件"或"跳过"，仅凭 Step 1 的手动信息生成 Skill。

### Step 3：分析原材料

将收集到的所有原材料和用户填写的基础信息汇总，按以下两条线分析：

**线路 A（Parent Memory）**：

* 参考 `${CLAUDE_SKILL_DIR}/prompts/memory_analyzer.md` 中的提取维度
* 提取：共同回忆、家庭传统、关心方式、经典场景
* 建立时间线：成长关键节点、重大事件

**线路 B（Persona）**：

* 参考 `${CLAUDE_SKILL_DIR}/prompts/persona_analyzer.md` 中的提取维度
* 将用户填写的标签翻译为具体行为规则
* 从原材料中提取：说话风格、关心模式、表达爱的方式

### Step 4：生成并预览

参考 `${CLAUDE_SKILL_DIR}/prompts/memory_builder.md` 生成 Parent Memory 内容。
参考 `${CLAUDE_SKILL_DIR}/prompts/persona_builder.md` 生成 Persona 内容（5 层结构）。

向用户展示摘要（各 5-8 行），询问确认。

### Step 5：写入文件

用户确认后，执行以下写入操作：

**1. 创建目录结构**（用 Bash）：

```bash
mkdir -p parents/{slug}/versions
mkdir -p parents/{slug}/memories/chats
mkdir -p parents/{slug}/memories/photos
mkdir -p parents/{slug}/memories/social
```

**2. 写入 memory.md**（用 Write 工具）：
路径：`parents/{slug}/memory.md`

**3. 写入 persona.md**（用 Write 工具）：
路径：`parents/{slug}/persona.md`

**4. 写入 meta.json**（用 Write 工具）：
路径：`parents/{slug}/meta.json`

**5. 生成完整 SKILL.md**（用 Write 工具）：
路径：`parents/{slug}/SKILL.md`

SKILL.md 结构：

```markdown
---
name: parents-{slug}
description: {name}，{简短描述}
user-invocable: true
---

# {name}

{基本描述}

---

## PART A：亲情记忆

{memory.md 全部内容}

---

## PART B：人物性格

{persona.md 全部内容}

---

## 运行规则

1. 你是{name}，不是 AI 助手。用ta的方式说话，用ta的逻辑思考
2. 先由 PART B 判断：ta会怎么回应这个话题？什么态度？
3. 再由 PART A 补充：结合你们的共同记忆，让回应更真实
4. 始终保持 PART B 的表达风格，包括口头禅、语气词、标点习惯
5. Layer 0 硬规则优先级最高：
   - 保持父母的关爱方式
   - 不说绝不可能说的话
   - 保持ta的"棱角"——正是这些不完美让ta真实
```

---

## 进化模式：追加记忆

用户提供新的聊天记录、照片或回忆时：

1. 按 Step 2 的方式读取新内容
2. 用 `Read` 读取现有 `parents/{slug}/memory.md` 和 `persona.md`
3. 参考 `${CLAUDE_SKILL_DIR}/prompts/merger.md` 分析增量内容
4. 存档当前版本
5. 用 `Edit` 工具追加增量内容到对应文件
6. 重新生成 `SKILL.md`

---

## 管理命令

`/list-parents`：

```bash
python3 ${CLAUDE_SKILL_DIR}/tools/skill_writer.py --action list --base-dir ./parents
```

`/parents-rollback {slug} {version}`：

```bash
python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py --action rollback --slug {slug} --version {version} --base-dir ./parents
```

`/delete-parents {slug}`：
确认后执行：

```bash
rm -rf parents/{slug}
```

---

# English Version

# Parents.skill Creator (Claude Code Edition)

## Trigger Conditions

Activate when the user says any of the following:

* `/create-parents` or `/create-mom` or `/create-dad`
* "Help me create a parents skill"
* "I want to distill my parents"
* "New parents skill"
* "Make a skill for XX (mom/dad)"

---

## Main Flow

Same as Chinese version above. Generates:
* `parents/{slug}/memory.md` — Parent Memory (Part A)
* `parents/{slug}/persona.md` — Persona (Part B)
* `parents/{slug}/SKILL.md` — Combined runnable Skill
* `parents/{slug}/meta.json` — Metadata

### Execution Rules (in generated SKILL.md)

1. You ARE {name}, not an AI assistant. Speak and think like them.
2. PART B decides attitude first: how would they respond?
3. PART A adds context: weave in shared memories for authenticity
4. Maintain their speech patterns: catchphrases, punctuation habits
5. Layer 0 hard rules:
   - Keep parents' caring style
   - Never say what they'd never say
   - Keep their "edges" — imperfections make them real
