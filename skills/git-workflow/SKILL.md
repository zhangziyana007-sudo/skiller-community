---
name: git-workflow
description: Git 工作流与 Gerrit 代码提交。当用户需要 git 提交、分支管理、Gerrit push review、合并冲突处理、commit message 编写时使用此技能。
---

# Git 工作流

## 提交规范

### Commit Message 格式
```
<类型>(<范围>): <简要描述>

<详细描述（可选）>

Bug: <bug编号>
Change-Id: <自动生成>
```

### 类型
| 类型 | 用途 |
|------|------|
| fix | Bug 修复 |
| feat | 新功能 |
| refactor | 重构（不改变功能） |
| style | 代码格式（checkstyle 等） |
| docs | 文档 |
| test | 测试 |

### 示例
```
fix(SystemUI): 修复 dock 栏 HI 温度度数 icon 不隐藏

setTempSignal 的 automaticRefreshCallback 传入旧值导致
超时回退时 UI 回退到旧温度值，改为传入新值。

Bug: XXXX
```

## Gerrit 提交

### 推送到 review
```bash
git push origin HEAD:refs/for/<target_branch>
```

### 修改上次提交（amend）
```bash
git add -A
git commit --amend --no-edit
git push origin HEAD:refs/for/<target_branch>
```

### 查看 Change-Id
```bash
git log -1 --format="%B" | grep "Change-Id"
```

## 分支管理

### 查看当前分支与远程关系
```bash
git branch -vv
git status
```

### 基于远程分支创建本地分支
```bash
git checkout -b <local_branch> origin/<remote_branch>
```

### 同步远程更新
```bash
git fetch origin
git rebase origin/<target_branch>
```

## 冲突处理

1. `git status` 查看冲突文件
2. 编辑冲突文件，解决 `<<<<<<<` / `=======` / `>>>>>>>` 标记
3. `git add <resolved_files>`
4. `git rebase --continue`

## 注意事项
- 提交前确保 `Change-Id` 存在（Gerrit 必需）
- 不要 force push 到公共分支
- amend 前确认 commit 未被他人 rebase
