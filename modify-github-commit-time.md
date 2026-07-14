# 修改 GitHub 上传（提交）时间
作者：pangxiegong  
日期：2026-07-15

## 简介
目标：记录生活 —— 将本地 Git 仓库中最近一次提交的 AuthorDate 与 CommitDate 修改为指定时间，并推送到 GitHub。

> 注意：修改已推送的提交会改写历史，需谨慎操作并告知协作者。建议先创建远端备份分支以便回滚。

---

## 1. 环境准备（如已安装可跳过）
1. 下载并安装 Git（Windows）：https://git-scm.com/download/win  
   - 建议安装时选择 “Use Git from the Windows Command Prompt”（把 Git 加入 PATH），以便在 PowerShell 中使用。  
2. 安装完成后打开 PowerShell，验证：
```powershell
git --version
```
你应看到类似 `git version 2.xx.x` 的输出。

---

## 2. 克隆仓库（如已克隆可跳过）
在工作目录运行：
```powershell
git clone https://github.com/你的/仓库.git
```
验证当前目录下的子目录：
```powershell
Get-ChildItem -Directory
```

---

## 3. 修改上传（提交）时间 — 步骤（PowerShell）
在开始前强烈建议先做备份分支：

1) 进入仓库根目录（若目录名以 `-` 开头，使用前缀 `\.\`）：
```powershell
cd .\-你的库名\
# 或
Set-Location -LiteralPath ".\-你的库名"
```

2) 确认当前路径是否在 Git 工作树内：
```powershell
git rev-parse --is-inside-work-tree
# 输出 "true" 表示在工作树中
```

3) 查看当前分支（确认要操作的分支）：
```powershell
git branch --show-current
# 输出示例： main
```

4) （可选但推荐）先备份当前远端分支：
```powershell
git fetch origin
git checkout main                 # 若分支不是 main，请替换为实际分支名
git pull origin main
git branch backup-main
git push origin backup-main
```

5) 配置当前仓库的提交者信息（如果尚未设置）：
```powershell
git config user.name "pangxiegong"
git config user.email "2871300742@qq.com"
```

6) 修改最近一次提交的时间（逐行执行，不要合并到一行）：
```powershell
# 在当前 PowerShell 会话中设置临时 committer 时间
$env:GIT_COMMITTER_DATE="2025-07-13T12:00:00+08:00"

# amend 最近一次提交（只改时间，不改提交信息）
git commit --amend --no-edit --date="2025-07-13T12:00:00+08:00"

# 删除临时环境变量
Remove-Item Env:GIT_COMMITTER_DATE
```

7) 强制推送改写后的分支到远端（更安全的强推）：
```powershell
git push --force-with-lease origin main
```

8) 验证最近一次提交的时间：
```powershell
git show --pretty=fuller -s HEAD
```
在输出中应能看到 `AuthorDate` 与 `CommitDate` 已更新为你指定的时间。

---

## 4. 关键输出示例（推送成功）
- 认证提示（若使用 Git Credential Manager）：
```
info: please complete authentication in your browser...
```
- 强制更新远端分支示例：
```
+ f45d4cb...54e667e main -> main (forced update)
```
- `git show --pretty=fuller -s HEAD` 中应出现：
```
AuthorDate: Sun Jul 13 12:00:00 2025 +0800
CommitDate: Sun Jul 13 12:00:00 2025 +0800
```

---

## 5. 常见问题与解决
- 错误：把多条命令放在同一行（例如把 PowerShell 命令与 git 命令合并）  
  - 解决：每条命令单独执行，或用 `;` 分隔。  
- 错误：`error: pathspec 'Remove-Item' did not match any file(s) known to git`  
  - 原因：PowerShell 命令被当作 git 参数。不要合并 `$env:...`、`git commit --amend`、`Remove-Item` 为一行。  
- 错误：`Please tell me who you are`（git 要求设置用户名/邮箱）  
  - 解决：在仓库中运行 `git config user.name` 和 `git config user.email` 设置本地值。  
- 认证/网络问题：若出现 `Could not connect to server` 或浏览器认证失败，先运行：
```powershell
ping github.com
Test-NetConnection github.com -Port 443
git remote -v
```
并根据网络环境选择：在浏览器完成认证、使用 PAT（个人访问令牌），或配置 SSH。

---

## 6. 可选：修改特定历史提交（交互式 rebase）
若需修改历史中更早的某个提交，请使用交互式 rebase（谨慎）：
```bash
git log --oneline -n 50
git rebase -i <TARGET_SHA>^
# 将要修改的行从 pick 改为 edit，保存退出
# 然后在暂停时：
GIT_COMMITTER_DATE="2025-07-13T12:00:00+08:00" git commit --amend --no-edit --date="2025-07-13T12:00:00+08:00"
git rebase --continue
# 完成后强推
git push --force-with-lease origin main
```
若遇冲突，按提示解决或使用 `git rebase --abort` 中止。

---

## 7. 恢复与清理
- 若需回滚到备份分支：
```bash
git push --force origin backup-main:main
```
- 删除远端备份分支（确认不再需要时）：
```bash
git push origin --delete backup-main
```

---

## 8. 建议
- 修改已推送提交会改写远端历史，请务必通知其他协作者。  
- 使用 `--force-with-lease` 比 `--force` 更安全（尽量避免覆盖别人新推的提交）。  
- 长期建议使用 SSH key 或 PAT 以减少每次推送时的交互认证操作。

---

## 9. 附：常用命令速查
```powershell
# 检查是否在工作树中
git rev-parse --is-inside-work-tree

# 显示当前分支
git branch --show-current

# 修改最近一次提交时间（PowerShell）
$env:GIT_COMMITTER_DATE="2025-07-13T12:00:00+08:00"
git commit --amend --no-edit --date="2025-07-13T12:00:00+08:00"
Remove-Item Env:GIT_COMMITTER_DATE
git push --force-with-lease origin main
git show --pretty=fuller -s HEAD
```
